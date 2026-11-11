# Phase 13: GPU Runtime Systems — Memory, Scheduling, Synchronization, and Execution

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 13 of 16  
**Estimated duration:** 18–24 weeks  
**Study commitment:** 10–12 hours per week  
**Primary language:** Rust  
**Supporting languages:** WGSL, C++20, SPIR-V  
**Primary platform:** Fedora Linux  
**Primary GPU:** AMD Radeon RX 9070, 16 GB  
**Primary API:** Vulkan through Rust  
**Secondary API:** wgpu  
**Prerequisites:** Phases 0–12

---

# 1. Mission: From GPU Programs to GPU Systems

In previous phases, you implemented GPU algorithms, studied GPU architecture, optimized kernels, and learned how to submit native Vulkan commands.

Now consider an application that performs hundreds of GPU operations.

It may need to:

- Allocate temporary buffers.
- Upload input data.
- Execute several dependent kernels.
- Reuse intermediate storage.
- Overlap independent operations.
- Download selected results.
- Track GPU completion.
- Recover from allocation failures.
- Prevent resources from being destroyed too early.
- Collect performance measurements.

Writing all of this manually for every application is expensive and error-prone.

A **GPU runtime** provides reusable infrastructure for these responsibilities.

A runtime is not simply a wrapper around Vulkan functions. It makes decisions about resource ownership, execution order, scheduling, memory management, synchronization, and performance.

## 1.1 The central design problem

Suppose an application performs:

\[
A \rightarrow B \rightarrow C
\]

where each letter represents a GPU operation.

A runtime must determine:

1. Which operations depend on others?
2. Which operations may overlap?
3. Which buffers must remain alive?
4. Which memory dependencies are necessary?
5. When may temporary memory be reused?
6. How does the CPU learn that execution has finished?
7. What happens if an operation fails?

A correct runtime answers these questions systematically.

## 1.2 Learning outcomes

By the end of this phase, you should be able to:

1. Explain the architecture of a GPU runtime.
2. Design safe Rust interfaces for GPU execution.
3. Build a GPU memory allocator.
4. Implement buffer pooling and suballocation.
5. Track resource ownership and lifetimes.
6. Design asynchronous submission APIs.
7. Model execution dependencies using directed acyclic graphs.
8. Implement topological scheduling.
9. Translate resource dependencies into Vulkan synchronization.
10. Manage fences and timeline semaphores.
11. Distinguish execution completion from memory visibility.
12. Implement deferred resource destruction.
13. Design command-buffer and descriptor reuse strategies.
14. Analyze CPU submission overhead.
15. Implement GPU task-graph execution.
16. Handle errors and device loss.
17. Understand multi-queue and multi-GPU execution.
18. Build profiling and observability infrastructure.
19. Test asynchronous systems systematically.
20. Defend runtime correctness and performance.

## 1.3 What you will build

Your final project is a Rust GPU runtime with this conceptual architecture:

```text
Application
    |
    v
Compute Graph API
    |
    v
Dependency Analyzer
    |
    v
Execution Scheduler
    |
    +-------------------+
    |                   |
    v                   v
Memory Manager      Pipeline Manager
    |                   |
    +---------+---------+
              |
              v
       Command Recorder
              |
              v
       Submission Manager
              |
              v
       Vulkan / GPU Driver
              |
              v
            GPU
```

Supporting components include:

- Completion tracking.
- Deferred destruction.
- Error reporting.
- Validation.
- Profiling.
- Benchmarking.

The final design should be reusable across multiple GPU algorithms rather than hard-coded for one workload.

---

# 2. Study Schedule

| Module | Topic | Duration |
|---|---|---|
| 1 | Runtime architecture and execution contracts | 2 weeks |
| 2 | GPU memory allocation and pooling | 2–3 weeks |
| 3 | Asynchronous resource lifetimes | 2 weeks |
| 4 | Task graphs and dependency analysis | 2–3 weeks |
| 5 | Scheduling and synchronization | 2–3 weeks |
| 6 | Command, pipeline, and descriptor management | 2 weeks |
| 7 | Reliability, multi-queue execution, and observability | 2–3 weeks |
| 8 | Capstone integration and technical defense | 3–4 weeks |

## Weekly study structure

Allocate approximately:

- 3 hours: theory and design.
- 4–5 hours: Rust implementation.
- 2 hours: testing and validation.
- 2 hours: benchmarking and documentation.

For every subsystem, produce:

1. An API contract.
2. An ownership diagram.
3. A list of invariants.
4. A test plan.
5. A benchmark.
6. A short engineering report.

---

# 3. Module One: GPU Runtime Architecture

## 3.1 What is a runtime?

A runtime is software that manages execution resources and coordinates operations on behalf of an application.

Examples outside GPU programming include:

- Language runtimes.
- Thread-pool runtimes.
- Asynchronous I/O runtimes.
- Distributed task schedulers.
- Database execution engines.

GPU runtimes share many of the same concerns.

They must manage resources whose execution and completion are not directly controlled by ordinary sequential program flow.

## 3.2 Runtime responsibilities

A GPU runtime commonly manages several distinct concerns.

### Device management

Discovering and configuring GPU devices.

### Memory management

Allocating, tracking, and recycling GPU memory.

### Execution management

Recording, submitting, and coordinating commands.

### Synchronization

Ensuring operations execute in a valid order and observe required memory accesses.

### Resource lifetime management

Preventing premature destruction or reuse.

### Pipeline management

Managing compiled compute pipelines and resource bindings.

### Error management

Reporting failures and maintaining valid runtime state.

### Performance management

Reducing unnecessary allocations, submissions, synchronization, and data transfers.

## 3.3 Runtime versus driver

A GPU driver implements the hardware-facing execution environment.

An application runtime operates above the driver and decides how application work is organized.

For example, Vulkan may provide memory allocation and command submission primitives.

Your runtime decides:

- When to allocate.
- How to pool allocations.
- Which commands belong together.
- Which buffers can be reused.
- When completed resources can be reclaimed.

## 3.4 Mechanism versus policy

A mechanism provides a capability.

A policy determines how to use it.

Examples:

| Mechanism | Policy |
|---|---|
| Vulkan memory allocation | Which allocation size to request |
| Queue submission | When to submit queued commands |
| Timeline semaphore | Which completion values to assign |
| Buffer reuse | Which free buffer to select |
| Pipeline cache | Which pipelines to retain |

Separating mechanisms from policies makes experimentation easier.

## 3.5 Runtime API design

Consider a conceptual application interface:

```rust
let input = runtime.upload(&data)?;
let output = runtime.create_buffer(output_size)?;

let task = runtime.dispatch(
    pipeline,
    &[input, output],
    workgroups,
)?;

let completion = runtime.submit(task)?;

completion.wait()?;
```

This is illustrative API design, not a complete implementation.

The important question is what each operation guarantees.

For example:

- Does `upload` finish before returning?
- Does `dispatch` execute immediately or merely record work?
- Does `submit` transfer ownership of resources?
- Can the caller drop `input` before completion?
- Does `wait` establish host-visible memory results?
- What happens if the device is lost?

An API without clear answers to these questions is difficult to use correctly.

## 3.6 Synchronous versus asynchronous APIs

A synchronous API might return only after GPU execution completes.

An asynchronous API returns a handle representing pending work.

Synchronous APIs are easier to reason about but may prevent useful overlap.

Asynchronous APIs expose concurrency but require more careful lifetime and error handling.

## 3.7 Explicit versus implicit synchronization

An explicit API requires callers to describe dependencies.

An implicit API infers dependencies from resource usage.

For example, if one task writes buffer A and another reads buffer A, an implicit runtime may infer that the second task depends on the first.

However, dependency inference requires accurate resource-access metadata.

Incorrect metadata can cause missing synchronization or unnecessary serialization.

## Exercises

1. Define a GPU runtime.
2. Distinguish a runtime from a driver.
3. Explain mechanism versus policy.
4. Why is asynchronous API design difficult?
5. What should a completion handle guarantee?
6. Why is resource metadata important?
7. Compare explicit and implicit synchronization.
8. Explain how runtime API design affects performance.
9. Why should runtime policies be configurable?
10. What makes a GPU runtime reusable?

## Lab 1: Runtime Architecture Specification

Design a GPU runtime before implementing it.

Deliver:

- Architecture diagram.
- Module boundaries.
- Public API sketches.
- Resource ownership model.
- Synchronization model.
- Error model.
- Testing strategy.

Identify every operation that may be asynchronous.

---

# 4. Module Two: GPU Memory Allocation

GPU memory allocation is a major runtime responsibility.

A naive runtime may allocate a new device-memory object for every buffer.

This can create substantial overhead and fragmentation.

## 4.1 Allocation versus suballocation

Allocation requests memory from the underlying API.

Suballocation divides a larger allocation into smaller regions.

Suppose an application requires four buffers:

\[
[16,32,8,24]\text{ MiB}
\]

A simple allocator might create four separate device-memory allocations.

A suballocator might reserve a larger memory block and assign regions within it.

The latter can reduce allocation frequency but requires bookkeeping and alignment management.

## 4.2 Alignment

Every suballocation must satisfy the resource's alignment requirements.

If an allocation requires alignment \(A\), an offset \(o\) must satisfy:

\[
o\bmod A=0
\]

The next aligned offset can be calculated as:

\[
\operatorname{align\_up}(x,A)
=
\left\lceil\frac{x}{A}\right\rceil A
\]

For a power-of-two alignment, bitwise techniques can implement this efficiently, but the general mathematical definition is more fundamental.

## Worked example: aligned allocation

Suppose the current allocation cursor is:

\[
x=1000
\]

bytes and the next resource requires:

\[
A=256
\]

byte alignment.

Then:

\[
\operatorname{align\_up}(1000,256)
=
1024
\]

The allocator inserts 24 bytes of padding.

If the resource requires 600 bytes, the next cursor becomes:

\[
1024+600=1624
\]

The allocator must still align the following allocation independently.

## 4.3 Linear allocator

A linear allocator maintains a cursor within a memory region.

Each allocation advances the cursor.

Advantages:

- Simple.
- Fast.
- Minimal metadata.

Disadvantages:

- Individual allocations are not easily freed.
- Reclamation often requires resetting the entire region.

Linear allocation is useful for short-lived resources with a common lifetime.

## 4.4 Free-list allocator

A free-list allocator tracks available memory regions.

When a resource is released, its region becomes available for reuse.

Adjacent free regions may be merged.

Challenges include:

- Fragmentation.
- Search overhead.
- Alignment.
- Metadata management.

## 4.5 Buddy allocator

A buddy allocator divides memory into power-of-two blocks.

Larger blocks can be split into smaller blocks.

Free buddy blocks may be merged.

Advantages include structured allocation and coalescing.

Disadvantages include internal fragmentation from rounding allocations to supported block sizes.

## 4.6 Slab allocation

A slab allocator manages fixed-size objects or allocation classes.

It can be effective for many similarly sized resources.

## 4.7 Internal fragmentation

Internal fragmentation occurs when an allocation reserves more space than the caller uses.

For example, allocating a 33 KiB buffer from a 64 KiB size class wastes 31 KiB within the allocation.

## 4.8 External fragmentation

External fragmentation occurs when free memory exists but is divided into regions that cannot satisfy a large contiguous request.

For example:

```text
[Used 16][Free 8][Used 16][Free 8][Used 16]
```

There are 16 units of free memory, but no contiguous free region larger than 8 units.

## 4.9 Fragmentation metrics

One simple external-fragmentation indicator is:

\[
F=1-\frac{L}{T}
\]

where:

- \(L\) is the largest free region.
- \(T\) is total free memory.

For:

\[
L=8,\quad T=16
\]

we obtain:

\[
F=0.5
\]

This metric is useful but does not completely describe allocation performance.

## 4.10 Vulkan memory requirements

Vulkan buffers have memory requirements that include size, alignment, and compatible memory types.

A runtime must honor these requirements when binding resources to memory.

Suballocation also requires attention to Vulkan's buffer-image granularity rules when mixing resource types, applicable dedicated-allocation requirements, and memory-type compatibility.

Do not assume that any two resources can safely share arbitrary regions of one allocation.

## 4.11 Device memory budgets

A runtime should track:

- Total allocation size.
- Active allocation size.
- Free pooled memory.
- Peak usage.
- Memory budget information where supported.

The runtime should not assume that all advertised VRAM is available to the application.

## 4.12 Buffer pooling

A buffer pool retains resources for future reuse.

For example, a workload repeatedly requires a 16 MiB temporary buffer.

Instead of allocating and destroying it for every iteration, the runtime may reuse the same buffer after its previous GPU work completes.

## 4.13 Reuse safety

A buffer cannot be reused merely because the CPU has stopped referencing it.

The runtime must establish that all GPU operations using the buffer have completed.

This connects memory allocation directly to asynchronous execution tracking.

## Lab 2: GPU Memory Allocator

Implement a Rust allocator supporting:

- Aligned suballocation.
- Free-list management.
- Allocation metadata.
- Coalescing.
- Allocation statistics.
- Fragmentation reporting.
- GPU completion-aware reclamation.

Begin with a CPU-only simulated allocator.

After validating the algorithms, integrate with Vulkan memory requirements.

## Exercises

1. Define suballocation.
2. Derive alignment rounding.
3. Compare linear and free-list allocators.
4. Explain buddy allocation.
5. Explain slab allocation.
6. Distinguish internal and external fragmentation.
7. Why does buffer pooling improve performance?
8. Why must GPU completion control reuse?
9. Why must memory types be considered?
10. Why is available VRAM different from total VRAM?
11. What makes allocator benchmarks realistic?
12. Why can allocation strategy affect kernel performance indirectly?

---

# 5. Module Three: Resource Lifetimes and Deferred Destruction

GPU execution is asynchronous.

This makes resource lifetime management more complicated than ordinary Rust ownership.

## 5.1 The lifetime problem

Consider:

```text
CPU creates buffer
       |
CPU submits GPU work
       |
CPU returns from submission
       |
CPU drops buffer
       |
GPU still executing
```

If the buffer is destroyed while the GPU still uses it, execution becomes invalid.

## 5.2 Host ownership versus GPU usage

Rust can track host-side ownership.

But the GPU is not an ordinary Rust thread whose borrow lifetime is automatically visible to the compiler.

A runtime must explicitly represent outstanding GPU usage.

## 5.3 Submission identifiers

Assign each GPU submission an identifier:

\[
S_0,S_1,S_2,\ldots
\]

A resource may record the latest submission that requires it.

The runtime can reclaim the resource after that submission has completed, provided the submission identifier represents a valid completion ordering for all relevant uses.

For multiple queues, a single scalar identifier may not be sufficient.

## 5.4 Timeline semaphores

A timeline semaphore represents progress through monotonically increasing values.

Suppose submissions signal:

\[
1,2,3,4
\]

A resource last used by submission 3 may be eligible for reclamation once the required completion conditions associated with value 3 are satisfied.

The runtime must ensure the timeline accurately represents all relevant uses.

## 5.5 Deferred destruction queue

A deferred destruction queue stores resources awaiting safe reclamation.

Conceptually:

```text
Pending destruction:
  Buffer A -> completion 5
  Buffer B -> completion 7
  Pipeline C -> completion 9
```

As completion advances, the runtime destroys eligible resources.

## 5.6 Multiple queues

If a resource is used by two independent queues, completion on one queue does not prove completion on the other.

The runtime must track the required dependencies or completion values for each relevant queue.

## 5.7 Resource aliasing

Two logical resources may share the same underlying memory.

If their lifetimes overlap, they cannot safely reuse the same physical region unless the memory accesses and aliasing behavior are explicitly valid and synchronized.

## 5.8 Lifetime intervals

For a task graph, define a resource's lifetime as the interval between its first required use and its final required use.

For example:

```text
Task:       A   B   C   D   E
Buffer X:   |-----------|
Buffer Y:       |-----------|
Buffer Z:               |---|
```

Nonoverlapping lifetimes can create memory reuse opportunities.

However, task graph order alone does not imply real-time completion order unless the scheduler enforces the necessary dependencies.

## 5.9 Memory aliasing optimization

A runtime may assign two temporary resources to the same memory region if it can prove that their uses do not overlap in execution and that the aliasing satisfies API requirements.

This resembles register allocation in a compiler.

## 5.10 Safety invariant

A central invariant is:

**A resource must remain valid until every GPU operation that can access it has completed.**

A second invariant is:

**A physical memory region must not be reassigned to a conflicting use while earlier accesses remain outstanding.**

## Lab 3: Completion-Aware Resource Manager

Implement:

- Submission identifiers.
- Resource-use tracking.
- Completion tracking.
- Deferred destruction.
- Safe buffer reuse.
- Multi-submission tests.

Use a simulated asynchronous executor first.

Introduce delayed and out-of-order completions to test assumptions.

Then integrate with Vulkan synchronization.

## Exercises

1. Why does Rust ownership not solve GPU completion?
2. What is a submission identifier?
3. What is a timeline semaphore?
4. What is deferred destruction?
5. Why can a single completion counter be insufficient for multiple queues?
6. What is resource aliasing?
7. How can lifetime analysis reduce memory usage?
8. Why must graph order be connected to actual execution order?
9. What invariants govern safe buffer reuse?
10. How can simulated completion testing expose lifetime bugs?

---

# 6. Module Four: GPU Task Graphs

A task graph represents computation as operations connected by dependencies.

## 6.1 Directed acyclic graphs

A directed acyclic graph, or DAG, contains:

\[
G=(V,E)
\]

where:

- \(V\) represents tasks.
- \(E\) represents directed dependencies.

The graph contains no directed cycles.

## 6.2 Example task graph

Consider:

```text
        Upload A
           |
           v
        Kernel B
         /    \
        v      v
    Kernel C  Kernel D
        \      /
         v    v
        Kernel E
           |
           v
        Readback
```

Kernel C and Kernel D may execute independently after Kernel B completes.

Kernel E depends on both.

## 6.3 Explicit dependencies

An application may specify:

\[
A\rightarrow B
\]

to indicate that B must not execute before A satisfies the required dependency.

## 6.4 Resource-derived dependencies

A runtime can also infer dependencies from resource access.

Suppose:

- Task A writes buffer X.
- Task B reads buffer X.

B depends on A if it needs A's result.

## 6.5 Read-after-write hazard

A read-after-write hazard occurs when a task reads data written by an earlier task.

Example:

\[
A:\text{write }X
\]

\[
B:\text{read }X
\]

B must observe the appropriate result from A.

## 6.6 Write-after-read hazard

A write-after-read hazard occurs when a task overwrites storage that an earlier task still needs to read.

Example:

\[
A:\text{read }X
\]

\[
B:\text{write }X
\]

The runtime must preserve the required ordering.

## 6.7 Write-after-write hazard

Two tasks write overlapping storage.

Their relative order may affect the final result.

Unless the operations have explicitly compatible semantics, the runtime must order them or reject the conflicting execution.

## 6.8 Read-after-read

Two tasks only read the same immutable data.

They generally do not require ordering solely because of those reads.

## 6.9 Resource ranges

Dependencies should ideally consider actual memory ranges.

Two tasks accessing different, nonoverlapping regions of the same buffer may be independent.

However, fine-grained tracking increases runtime complexity.

## 6.10 Topological sorting

A topological ordering places every task after its predecessors.

For every edge:

\[
u\rightarrow v
\]

the ordering must place \(u\) before \(v\).

## 6.11 Kahn's algorithm

One approach repeatedly selects tasks with zero unresolved incoming dependencies.

When a task is removed, its outgoing edges are removed from the remaining graph.

If all tasks are processed, the graph is acyclic.

If tasks remain with unresolved dependencies, a cycle exists.

## 6.12 Complexity

For a graph represented with adjacency lists, topological sorting can run in:

\[
O(|V|+|E|)
\]

time.

## 6.13 Critical path

The critical path is the longest weighted dependency path through the graph.

Suppose:

```text
A (2 ms)
  |
  +--> B (5 ms) --> D (3 ms)
  |
  +--> C (4 ms) --> D
```

The path through B requires:

\[
2+5+3=10\text{ ms}
\]

The path through C requires:

\[
2+4+3=9\text{ ms}
\]

Thus the ideal dependency-limited lower bound is at least:

\[
10\text{ ms}
\]

Even unlimited parallel hardware cannot reduce execution below this bound without changing the algorithm or task durations.

## 6.14 Work and span

For a weighted task graph:

\[
W=\sum_{v\in V}t_v
\]

and:

\[
S=\max_{\text{paths }p}\sum_{v\in p}t_v
\]

The theoretical parallelism is:

\[
\frac WS
\]

For the example:

\[
W=2+5+4+3=14\text{ ms}
\]

\[
S=10\text{ ms}
\]

Therefore:

\[
\frac WS=1.4
\]

The graph has limited parallelism despite containing independent tasks.

## 6.15 Graph validation

A task graph runtime should detect:

- Cycles.
- Invalid task identifiers.
- Missing resources.
- Unsupported resource accesses.
- Conflicting operations.
- Invalid execution configurations.

## Lab 4: CPU Task Graph Engine

Before integrating Vulkan, build a CPU-only task graph engine.

Required features:

- Task creation.
- Dependency edges.
- Cycle detection.
- Topological sorting.
- Critical-path calculation.
- Resource-access metadata.
- Dependency inference.

Test with randomly generated DAGs and deliberately invalid graphs.

## Exercises

1. Define a DAG.
2. Explain explicit dependencies.
3. Explain resource-derived dependencies.
4. Distinguish RAW, WAR, and WAW hazards.
5. Why can read-only tasks overlap?
6. What is topological sorting?
7. Derive its complexity.
8. Define critical path.
9. Calculate work and span for a task graph.
10. Why does a DAG not guarantee useful GPU overlap?
11. Why is range-based dependency analysis difficult?
12. How can cycles be detected?

---

# 7. Module Five: GPU Scheduling

A task graph describes which operations are permitted to execute.

A scheduler decides when and where they execute.

## 7.1 Scheduling objectives

A scheduler may attempt to minimize:

- Total completion time.
- CPU overhead.
- GPU idle time.
- Memory usage.
- Synchronization overhead.

These objectives can conflict.

## 7.2 Serial scheduling

The simplest scheduler executes tasks in topological order.

This is easy to validate but may fail to exploit independent work.

## 7.3 Ready-task scheduling

A ready task has no unresolved dependencies.

The scheduler maintains a set of ready tasks and submits them when resources permit.

## 7.4 Critical-path scheduling

A scheduler may prioritize tasks on the longest remaining dependency path.

The motivation is to avoid delaying tasks that determine total completion time.

However, estimated task durations may be inaccurate.

## 7.5 Batching

Submitting many small tasks individually can create CPU overhead.

Batching combines compatible operations into fewer command submissions.

## 7.6 Batching trade-offs

Larger batches may reduce submission overhead.

But they can also:

- Delay ready work.
- Increase latency.
- Increase temporary memory lifetime.
- Reduce scheduling flexibility.

## 7.7 Multiple queues

Vulkan may expose multiple queues.

A runtime can potentially submit independent work to different queues.

However, multiple queues do not guarantee concurrent hardware execution.

Actual overlap depends on device architecture, queue capabilities, workload, and synchronization.

## 7.8 Queue ownership

Some Vulkan resources require explicit queue-family ownership transfers when accessed across queue families.

A runtime must distinguish queue submission ordering from queue-family ownership.

## 7.9 Scheduling model

A simplified runtime cost model is:

\[
T_{\text{total}}
=
T_{\text{CPU submission}}
+
T_{\text{GPU execution}}
+
T_{\text{waiting}}
\]

This is an accounting model, not necessarily a sum of nonoverlapping intervals.

With overlap, wall-clock completion time depends on the actual execution timeline.

## 7.10 Worked example: submission overhead

Suppose 100 tiny kernels each require:

\[
10\ \mu s
\]

of GPU execution.

Assume each submission incurs:

\[
20\ \mu s
\]

of CPU-side overhead.

Submitting every kernel separately requires approximately:

\[
100\times20=2000\ \mu s
\]

of submission work.

The kernels themselves require:

\[
100\times10=1000\ \mu s
\]

of aggregate GPU execution.

If compatible commands can be batched, submission overhead may fall substantially.

But the actual wall-clock improvement depends on whether CPU submission and GPU execution overlap.

## 7.11 Scheduling and memory pressure

Scheduling more tasks concurrently may increase memory consumption.

For example, independent tasks may each require large temporary buffers.

A scheduler may deliberately limit concurrency to remain within a memory budget.

## 7.12 Backpressure

Backpressure prevents producers from generating work faster than the runtime can process it.

A bounded submission queue is one mechanism.

Without backpressure, a runtime may accumulate:

- Pending commands.
- Temporary buffers.
- Descriptor sets.
- Completion handles.
- Host-side metadata.

## Lab 5: Task Graph Scheduler

Implement three scheduling policies:

1. Serial topological execution.
2. Ready-task execution.
3. Critical-path-priority execution.

Benchmark them using a simulated executor with configurable task durations and resource constraints.

Then integrate at least one policy with Vulkan.

## Exercises

1. Define ready-task scheduling.
2. What is critical-path scheduling?
3. Why is batching useful?
4. Why can batching increase latency?
5. Why do multiple queues not guarantee overlap?
6. What is queue-family ownership?
7. Why does memory pressure constrain scheduling?
8. What is backpressure?
9. How can task duration estimates affect scheduling?
10. Why should scheduling policies be benchmarked?

---

# 8. Module Six: Vulkan Synchronization in a Runtime

A runtime must translate task dependencies into actual GPU synchronization.

This is one of the most safety-critical parts of the project.

## 8.1 Graph dependencies are not enough

A graph edge expresses a logical ordering requirement.

The Vulkan implementation still needs appropriate execution and memory dependencies.

For example:

```text
Kernel A writes Buffer X
          |
          v
Kernel B reads Buffer X
```

The runtime must ensure B observes A's writes.

## 8.2 Execution dependencies

Execution dependencies constrain when operations may execute.

## 8.3 Memory dependencies

Memory dependencies govern availability and visibility of memory accesses.

Correct execution ordering and correct memory visibility are related but distinct requirements.

## 8.4 Pipeline stages

Synchronization specifies relevant producer and consumer pipeline stages.

For compute workloads, these may include compute shader and transfer stages.

## 8.5 Access masks

Access masks describe the types of memory accesses involved.

Examples include shader reads, shader writes, transfer reads, and transfer writes.

## 8.6 Barrier generation

A runtime can generate barriers based on task resource metadata.

For example:

- Producer: compute shader write.
- Consumer: compute shader read.
- Resource: storage buffer X.

The runtime can determine an appropriate dependency for the transition.

## 8.7 Barrier coalescing

Multiple compatible dependencies may be combined into fewer synchronization operations.

However, combining barriers must preserve the required ordering and visibility guarantees.

## 8.8 Over-synchronization

A conservative runtime might insert a broad barrier between every task.

This can simplify initial correctness reasoning.

But it may prevent independent operations from overlapping and introduce unnecessary stalls.

## 8.9 Under-synchronization

Insufficient synchronization can cause:

- Stale reads.
- Conflicting accesses.
- Invalid resource reuse.
- Incorrect results.
- Intermittent failures.

## 8.10 Cross-queue dependencies

When dependent tasks execute on different queues, synchronization may require semaphores or other appropriate cross-queue mechanisms.

The runtime must also account for queue-family ownership rules where applicable.

## 8.11 Timeline-based completion

Timeline semaphores can represent progress through submitted work.

A runtime may use them to:

- Track completion.
- Release resources.
- Wake waiting tasks.
- Coordinate queues.

## 8.12 Synchronization correctness proof

For each dependency, identify:

1. Producer operation.
2. Producer memory access.
3. Consumer operation.
4. Consumer memory access.
5. Required execution ordering.
6. Required memory visibility.
7. Synchronization primitive.
8. Completion condition.

This discipline helps prevent synchronization from becoming a collection of unexplained barriers.

## Lab 6: Synchronization Planner

Build a component that accepts task resource-access metadata and produces a synchronization plan.

Test:

- Read-after-write.
- Write-after-read.
- Write-after-write.
- Independent reads.
- Nonoverlapping buffer ranges.
- Cross-queue dependencies.
- Transfer-to-compute transitions.
- Compute-to-readback transitions.

Compare conservative and more precise synchronization strategies.

## Exercises

1. Why are DAG edges insufficient by themselves?
2. Distinguish execution and memory dependencies.
3. What are pipeline stages?
4. What are access masks?
5. How can a runtime generate barriers?
6. What is barrier coalescing?
7. Why is over-synchronization expensive?
8. Why is under-synchronization dangerous?
9. How are cross-queue dependencies handled?
10. What must a synchronization correctness argument establish?

---

# 9. Module Seven: Command Buffer Management

Command buffers are the units through which Vulkan records GPU commands.

## 9.1 Recording overhead

Recording commands requires CPU work.

A runtime should avoid repeating unnecessary setup.

## 9.2 Command pools

Command pools manage command-buffer allocation.

Their use must follow Vulkan's threading and lifetime rules.

## 9.3 Command buffer reuse

Reusable command buffers may reduce recording overhead.

However, reuse must respect:

- Command-buffer state.
- Submission completion.
- Resource validity.
- Pipeline compatibility.
- Descriptor validity.

## 9.4 Primary and secondary command buffers

Vulkan supports different command-buffer levels.

Secondary command buffers can support certain forms of command recording reuse and composition, subject to API constraints.

## 9.5 Threaded command recording

Some command-recording work can be distributed across CPU threads.

However, command pools and related Vulkan objects have externally synchronized usage requirements.

A runtime must design thread ownership carefully.

## 9.6 Recording versus submission

Recording a command does not execute it.

A command buffer must be submitted to an appropriate queue.

## 9.7 Submission batching

A runtime may group multiple command buffers into one submission.

This can reduce CPU overhead.

## 9.8 Completion tracking

A submitted command buffer cannot necessarily be reset or reused immediately.

The runtime must track completion and permitted reuse.

## Lab 7: Command Buffer Pool

Implement:

- Command-buffer allocation.
- Recording.
- Submission.
- Completion tracking.
- Safe reuse.
- Pool statistics.

Benchmark repeated small compute workloads.

## Exercises

1. What is a command pool?
2. Why is command recording expensive?
3. When can command buffers be reused?
4. What are primary and secondary command buffers?
5. Why is threaded recording difficult?
6. Why must submission completion be tracked?
7. What is submission batching?
8. How does command reuse affect CPU overhead?

---

# 10. Module Eight: Pipeline and Descriptor Management

A GPU runtime must manage executable compute pipelines and their resource bindings.

## 10.1 Pipeline creation

Compute pipeline creation may involve driver compilation and validation.

It can be significantly more expensive than dispatching an already-created pipeline.

## 10.2 Pipeline caching

A runtime may cache pipelines using keys derived from:

- Shader identity.
- Entry point.
- Specialization constants.
- Pipeline layout.
- Relevant pipeline configuration.

The cache key must include all properties that affect pipeline compatibility and behavior.

## 10.3 Shader variants

A runtime may maintain multiple shader variants for:

- Workgroup sizes.
- Tile sizes.
- Data types.
- Algorithm variants.
- Hardware capabilities.

## 10.4 Specialization constants

Vulkan specialization constants can parameterize shader behavior at pipeline creation time.

They can support certain optimization choices without maintaining completely separate source files.

## 10.5 Descriptor sets

Descriptor sets bind resources to shaders.

A runtime may cache or pool descriptor sets.

## 10.6 Descriptor lifetime

Descriptor validity and update rules depend on descriptor types, binding flags, and the state of command buffers and submissions.

A runtime must not assume that every descriptor set can be modified freely while GPU work is outstanding.

## 10.7 Pipeline layout compatibility

Pipeline layouts define resource interfaces.

The runtime must ensure that bound descriptor sets are compatible with the pipeline's layout requirements.

## 10.8 Cache invalidation

Pipeline caches may become invalid or ineffective when:

- Shader code changes.
- Pipeline configuration changes.
- Driver versions change.
- Device compatibility changes.

Persistent caches require careful compatibility handling.

## Lab 8: Pipeline Manager

Implement:

- Pipeline creation.
- Pipeline cache keys.
- Pipeline reuse.
- Shader variant selection.
- Descriptor pooling.
- Cache statistics.

Measure pipeline creation and reuse costs separately from GPU execution.

## Exercises

1. Why is pipeline creation expensive?
2. What belongs in a pipeline cache key?
3. What are shader variants?
4. What are specialization constants?
5. Why is descriptor pooling useful?
6. Why is descriptor lifetime complicated?
7. What is pipeline layout compatibility?
8. Why must persistent caches be validated?

---

# 11. Module Nine: Error Handling and Reliability

GPU runtimes must handle failure without corrupting their own state.

## 11.1 Error categories

Failures may include:

- Unsupported device features.
- Allocation failure.
- Invalid shader modules.
- Pipeline creation failure.
- Invalid command usage.
- Submission failure.
- Device loss.
- Synchronization timeout.
- Validation failure.

## 11.2 Recoverable versus terminal errors

Some failures may be handled locally.

For example, an allocation request may fail while existing resources remain valid.

Other failures may invalidate the device or runtime state.

The API must distinguish these cases.

## 11.3 Device loss

A GPU device may become unusable.

A runtime must not assume that all pending work can still complete normally.

## 11.4 Failure propagation

If task A fails, dependent task B may no longer be executable.

The scheduler must define how failure propagates through the task graph.

## 11.5 Cancellation

Cancellation is difficult once GPU commands have been submitted.

A runtime may be able to cancel work that has not yet been submitted.

Submitted work generally requires different handling and cannot be assumed to support arbitrary immediate cancellation.

## 11.6 Timeouts

A timeout does not necessarily mean the GPU operation has stopped.

Destroying resources solely because a wait timed out can be unsafe.

## 11.7 Runtime state machines

A runtime may model states such as:

```text
Uninitialized
     |
     v
   Ready
     |
     v
  Running
     |
     +------> Failed
     |
     v
  Shutdown
```

The precise state machine should reflect actual permitted operations and recovery guarantees.

## 11.8 Error reporting

Error messages should provide useful context:

- Operation.
- Resource identifier.
- Device.
- Relevant configuration.
- Underlying API error.
- Runtime state.

## Lab 9: Runtime Fault Injection

Build tests that simulate:

- Allocation failure.
- Pipeline creation failure.
- Invalid graph dependencies.
- Submission failure.
- Completion timeout.
- Device loss.
- Resource exhaustion.

Verify that the runtime reports errors and does not incorrectly reclaim resources.

## Exercises

1. What is device loss?
2. Why is failure propagation necessary?
3. Why is cancellation difficult after submission?
4. Why does timeout not imply completion?
5. What is a runtime state machine?
6. Why should errors contain resource context?
7. How can fault injection improve reliability?
8. Why must failed submissions be handled differently from completed submissions?

---

# 12. Module Ten: Multi-Queue and Multi-GPU Runtime Design

This module introduces more advanced runtime architecture.

A single AMD GPU is sufficient for conceptual study and some multi-queue experiments.

Actual multi-GPU execution requires compatible hardware.

## 12.1 Multiple queues

A device may expose multiple queues with compute or transfer capabilities.

A runtime can schedule operations across them.

## 12.2 Queue concurrency

Multiple queues do not guarantee physical concurrency.

Some operations may compete for the same hardware resources.

## 12.3 Cross-queue synchronization

Dependent operations on different queues require appropriate synchronization.

Timeline semaphores can help represent progress.

## 12.4 Queue-family ownership

Resources may require ownership transfers between queue families.

This is distinct from ordinary memory visibility.

## 12.5 Multi-GPU execution

A multi-GPU runtime must decide:

- Which GPU owns each resource.
- Where each task executes.
- How data moves between GPUs.
- How dependencies are enforced.
- How failures are handled.

## 12.6 Data partitioning

A workload may be divided across GPUs.

For example, a large matrix operation may partition rows or tiles.

## 12.7 Communication costs

Multi-GPU performance depends on:

- Transfer bandwidth.
- Transfer latency.
- Interconnect topology.
- Synchronization.
- Data partitioning.
- Load balance.

## 12.8 Strong scaling

Strong scaling measures performance when a fixed workload is distributed across more devices.

## 12.9 Weak scaling

Weak scaling measures performance when workload size increases with the number of devices.

## 12.10 Amdahl's law

If fraction \(s\) of a workload is serial, the ideal speedup on \(P\) processors is:

\[
S(P)=\frac{1}{s+\frac{1-s}{P}}
\]

This ignores communication and scheduling overhead.

## Worked example

Suppose:

\[
s=0.1
\]

and:

\[
P=4
\]

Then:

\[
S(4)=\frac{1}{0.1+0.9/4}
\]

\[
S(4)\approx3.08
\]

Even with ideal parallel execution, speedup is below four.

Real multi-GPU speedup may be lower because of communication and synchronization.

## Lab 10: Multi-Device Scheduler Simulation

Build a simulator that models:

- Multiple GPUs.
- Device memory capacities.
- Task durations.
- Transfer costs.
- Task dependencies.
- Scheduling policies.

Compare:

- Single-device execution.
- Static partitioning.
- Load-aware scheduling.
- Communication-aware scheduling.

Do not claim simulated results represent actual multi-GPU hardware performance.

## Exercises

1. Why do multiple queues not guarantee overlap?
2. What is cross-queue synchronization?
3. What is queue-family ownership?
4. What is data partitioning?
5. Why does communication affect scaling?
6. Define strong scaling.
7. Define weak scaling.
8. Derive Amdahl's law.
9. Why does device topology matter?
10. Why must simulated performance be distinguished from measured performance?

---

# 13. Module Eleven: Runtime Observability and Profiling

A runtime is difficult to optimize if its behavior cannot be observed.

## 13.1 Observability

Observability means exposing enough information to understand internal execution behavior.

Useful metrics include:

- Allocated memory.
- Peak memory usage.
- Pool hit rate.
- Fragmentation.
- Pending submissions.
- Queue depth.
- Task execution time.
- CPU recording time.
- CPU submission time.
- GPU idle intervals.
- Pipeline cache hit rate.

## 13.2 Execution traces

A trace records events over time.

For example:

```text
CPU:  Record A | Submit A | Record B | Submit B
GPU:             Execute A | Execute B
```

A trace can reveal whether CPU preparation, GPU execution, or synchronization dominates.

## 13.3 Task graph visualization

Visualizing dependencies can help identify:

- Critical paths.
- Unnecessary serialization.
- Excessive temporary memory lifetimes.
- Scheduling bottlenecks.

## 13.4 GPU timestamps

GPU timestamp queries can measure execution intervals when supported and used correctly.

Their availability, precision, and interpretation depend on the device and API.

## 13.5 CPU timestamps

CPU timestamps can measure:

- Graph construction.
- Dependency analysis.
- Command recording.
- Submission overhead.
- Completion processing.

## 13.6 Timeline analysis

Combining CPU and GPU traces can help explain end-to-end latency.

However, timestamps from different clock domains require care.

Do not assume CPU and GPU timestamps are directly comparable without appropriate calibration or synchronization.

## 13.7 Performance counters

Hardware profiling tools may expose:

- Memory bandwidth.
- Cache behavior.
- Occupancy.
- Wavefront activity.
- Instruction throughput.
- Stalls.

Counter availability varies by GPU, driver, and tool.

## 13.8 Performance regression testing

A runtime should detect unexpected performance changes.

Regression tests should use stable workloads and report uncertainty.

A small timing difference should not automatically be treated as a meaningful regression.

## Lab 11: Runtime Profiler

Implement:

- CPU event tracing.
- GPU timing where supported.
- Task graph visualization.
- Memory statistics.
- Submission statistics.
- Pipeline cache statistics.
- Structured benchmark output.

Generate a report identifying the critical path and major runtime overheads.

## Exercises

1. Define observability.
2. What is an execution trace?
3. Why is task graph visualization useful?
4. What do GPU timestamps measure?
5. Why must CPU and GPU clocks be treated carefully?
6. What is a performance counter?
7. Why is queue depth useful?
8. What is a performance regression?
9. Why does benchmark noise matter?
10. How can profiling reveal unnecessary synchronization?

---

# 14. Module Twelve: Testing GPU Runtime Systems

Testing a runtime requires more than verifying arithmetic results.

You must test scheduling, ownership, synchronization, and failure behavior.

## 14.1 Unit testing

Unit-test components independently:

- Allocator.
- Graph builder.
- Dependency analyzer.
- Scheduler.
- Completion tracker.
- Resource manager.
- Pipeline cache.

## 14.2 Model-based testing

Create a simplified reference model of runtime behavior.

Compare actual state transitions against the model.

## 14.3 Property-based testing

Useful properties include:

- Every submitted task has valid dependencies.
- No task executes before required predecessors complete.
- No resource is reclaimed while still in use.
- No allocation overlaps an incompatible live allocation.
- Every completed submission is eventually processed.
- Every valid acyclic graph has a topological ordering.

## 14.4 Randomized testing

Generate random:

- DAGs.
- Allocation sequences.
- Resource lifetimes.
- Submission orders.
- Completion orders.
- Failure events.

## 14.5 Stress testing

Stress scenarios should include:

- Thousands of small tasks.
- Large temporary buffers.
- High allocation churn.
- Many dependent tasks.
- Many independent tasks.
- Repeated pipeline creation.
- Concurrent CPU submissions.
- Simulated delayed completions.

## 14.6 Validation layers

Vulkan validation layers can detect many API misuse cases.

They do not prove all application-level correctness or eliminate the need for runtime invariants.

## 14.7 Concurrency testing

Host-side Rust concurrency can be tested using appropriate concurrency-testing techniques and tools.

However, CPU race detectors do not automatically detect GPU-side synchronization mistakes.

## 14.8 Differential testing

Compare runtime-executed GPU algorithms against independent CPU reference implementations.

## 14.9 Deterministic simulation

A deterministic simulator can make difficult asynchronous bugs easier to reproduce.

It can control:

- Task completion order.
- Delays.
- Allocation failures.
- Queue capacity.
- Dependency resolution.

## 14.10 Safety audits

Review every unsafe Rust boundary.

For each one, document:

1. Preconditions.
2. Invariants.
3. Resource lifetime assumptions.
4. Synchronization assumptions.
5. Failure behavior.
6. Why callers cannot violate the contract through safe APIs.

## Lab 12: Runtime Validation Suite

Build a comprehensive test suite for:

- Memory allocation.
- Resource lifetime.
- Task graph correctness.
- Scheduling.
- Synchronization.
- Error handling.
- Device loss.
- Concurrent host access.
- GPU numerical correctness.

Use both deterministic and randomized tests.

---

# 15. Major Assessment A: GPU Memory Runtime

Build a reusable GPU memory-management subsystem.

## Required features

- Aligned allocation.
- Suballocation.
- Free-list or buddy allocation.
- Buffer pooling.
- Fragmentation tracking.
- Memory usage reporting.
- Completion-aware reuse.
- Deferred destruction.

## Required benchmarks

Compare:

- Direct allocation.
- Pooled allocation.
- Suballocation.

Measure:

- Allocation latency.
- Reuse latency.
- Peak memory usage.
- Fragmentation.
- Throughput under allocation churn.

## Technical defense

Explain allocator invariants, memory compatibility, fragmentation, and safe reclamation.

---

# 16. Major Assessment B: GPU Task Graph Scheduler

Build a task graph engine.

## Required features

- DAG representation.
- Explicit dependencies.
- Resource-derived dependencies.
- Cycle detection.
- Topological scheduling.
- Critical-path calculation.
- Ready-task scheduling.
- Resource lifetime tracking.
- Vulkan command generation.

## Required tests

- Linear chains.
- Independent tasks.
- Diamond-shaped graphs.
- Wide graphs.
- Deep graphs.
- Cyclic graphs.
- Resource hazards.
- Random DAGs.

## Technical defense

Explain dependency correctness, synchronization, scheduling overhead, and critical-path behavior.

---

# 17. Major Assessment C: Asynchronous Execution Runtime

Build a completion-aware asynchronous execution layer.

## Required features

- Submission handles.
- Completion tracking.
- Fence or timeline-based synchronization.
- Deferred resource destruction.
- Buffer reuse.
- Error propagation.
- Backpressure.
- Timeout handling.

## Required tests

- Immediate completion.
- Delayed completion.
- Out-of-order simulated completion.
- Multiple outstanding submissions.
- Resource sharing.
- Allocation failure.
- Device loss simulation.

## Technical defense

Explain why no resource can be reclaimed before its final GPU use completes.

---

# 18. Final Capstone: Rust GPU Task Graph Runtime

Build a reusable GPU runtime in Rust that can execute complex compute pipelines.

## 18.1 Required architecture

### Device manager

- Device discovery.
- Capability reporting.
- Queue selection.
- Device lifetime management.

### Memory manager

- Allocation.
- Suballocation.
- Buffer pooling.
- Deferred destruction.
- Memory statistics.

### Pipeline manager

- Shader loading.
- Pipeline creation.
- Pipeline caching.
- Descriptor management.

### Task graph engine

- DAG representation.
- Dependency inference.
- Cycle detection.
- Topological scheduling.
- Critical-path analysis.

### Execution engine

- Command recording.
- Submission.
- Synchronization.
- Completion tracking.
- Resource reclamation.

### Reliability layer

- Error propagation.
- Failure states.
- Timeout handling.
- Device loss handling.

### Observability layer

- CPU tracing.
- GPU timing.
- Memory statistics.
- Queue statistics.
- Task graph visualization.

## 18.2 Required workloads

Execute at least five algorithms from earlier phases:

1. Vector addition.
2. Reduction.
3. Prefix scan.
4. Matrix multiplication.
5. One advanced algorithm from Phase 11.

## 18.3 Required task graphs

### Graph A: Linear pipeline

```text
Upload → Transform → Reduce → Readback
```

### Graph B: Branching pipeline

```text
          Transform A
         /           \
Upload                  Merge → Readback
         \           /
          Transform B
```

### Graph C: Multi-stage algorithm

Use an advanced algorithm requiring multiple dependent dispatches.

### Graph D: Independent workloads

Submit several independent tasks and investigate whether the runtime reduces overhead or permits useful overlap.

### Graph E: Memory-constrained workload

Execute tasks whose combined temporary-memory requirements exceed a configured runtime budget.

The scheduler must preserve correctness while respecting the budget.

## 18.4 Required performance comparisons

Compare:

- Manual Vulkan execution.
- Runtime-managed execution.
- Different scheduling policies.
- Direct allocation versus pooling.
- Individual submission versus batching.
- Conservative versus more precise synchronization.

## 18.5 Required correctness report

Document:

- Resource ownership.
- Memory allocation invariants.
- Resource lifetime tracking.
- Task dependency correctness.
- Synchronization correctness.
- Error propagation.
- Device loss behavior.

## 18.6 Required performance report

Measure:

- CPU graph construction time.
- Dependency analysis time.
- Command recording time.
- Submission overhead.
- GPU execution time.
- End-to-end latency.
- Peak memory usage.
- Buffer pool hit rate.
- Pipeline cache hit rate.

## 18.7 Required architecture report

Include:

- Module diagrams.
- Public API contracts.
- Ownership diagrams.
- Submission lifecycle diagrams.
- Memory allocation diagrams.
- Synchronization diagrams.
- Task graph examples.

## 18.8 Technical defense

You must explain:

1. How the runtime prevents use-after-free.
2. How it identifies dependencies.
3. How it schedules ready tasks.
4. How it translates dependencies into Vulkan synchronization.
5. How it manages memory fragmentation.
6. How it reuses buffers safely.
7. How it handles failed submissions.
8. How it avoids unnecessary CPU overhead.
9. How it measures runtime performance.
10. Which design choices remain suboptimal and why.

---

# 19. Phase 13 Quiz Bank

## Runtime architecture

1. What is a GPU runtime?
2. How does a runtime differ from a driver?
3. What is mechanism versus policy?
4. What is an asynchronous execution API?
5. What is a completion handle?
6. Why are resource metadata important?
7. What is explicit synchronization?
8. What is implicit synchronization?
9. Why should runtime policies be configurable?
10. What makes a runtime reusable?

## Memory management

11. What is suballocation?
12. What is alignment?
13. What is a linear allocator?
14. What is a free-list allocator?
15. What is a buddy allocator?
16. What is slab allocation?
17. What is internal fragmentation?
18. What is external fragmentation?
19. What is buffer pooling?
20. Why must buffer reuse wait for GPU completion?
21. Why do Vulkan memory types matter?
22. Why is a memory budget necessary?

## Resource lifetimes

23. Why does Rust ownership not automatically track GPU execution?
24. What is a submission identifier?
25. What is a timeline semaphore?
26. What is deferred destruction?
27. What is resource aliasing?
28. What is lifetime analysis?
29. Why can a single completion counter be insufficient?
30. Why is safe resource reuse difficult?
31. What is a completion-aware allocator?
32. What is a resource lifetime invariant?

## Task graphs

33. What is a DAG?
34. What is a dependency edge?
35. What is a RAW hazard?
36. What is a WAR hazard?
37. What is a WAW hazard?
38. Why can independent reads overlap?
39. What is topological sorting?
40. What is cycle detection?
41. What is a critical path?
42. What is work?
43. What is span?
44. Why does a DAG not guarantee GPU overlap?

## Scheduling

45. What is ready-task scheduling?
46. What is critical-path scheduling?
47. What is submission batching?
48. Why can batching reduce CPU overhead?
49. Why can batching increase latency?
50. Why do multiple queues not guarantee concurrency?
51. What is queue-family ownership?
52. What is backpressure?
53. Why does memory pressure constrain scheduling?
54. Why must scheduling policies be measured?

## Synchronization

55. What is an execution dependency?
56. What is a memory dependency?
57. What is a pipeline stage?
58. What is an access mask?
59. What is a buffer memory barrier?
60. What is barrier coalescing?
61. What is over-synchronization?
62. What is under-synchronization?
63. What is cross-queue synchronization?
64. Why are logical graph edges insufficient by themselves?

## Command and pipeline management

65. What is a command pool?
66. What is command-buffer reuse?
67. Why must command-buffer completion be tracked?
68. What is pipeline caching?
69. What belongs in a pipeline cache key?
70. What is descriptor pooling?
71. What is pipeline layout compatibility?
72. Why can pipeline creation dominate small workloads?

## Reliability

73. What is device loss?
74. What is failure propagation?
75. Why is cancellation difficult after submission?
76. Why does timeout not imply completion?
77. What is a runtime state machine?
78. What is fault injection?
79. Why must errors include contextual information?
80. Why is device loss a runtime-wide concern?

## Multi-GPU and observability

81. What is multi-GPU scheduling?
82. What is data partitioning?
83. Why does communication affect scaling?
84. What is strong scaling?
85. What is weak scaling?
86. What is Amdahl's law?
87. What is runtime observability?
88. What is an execution trace?
89. What is a GPU timestamp?
90. Why must CPU and GPU clock domains be handled carefully?

## Testing and correctness

91. What is model-based testing?
92. What is property-based testing?
93. What is deterministic simulation?
94. Why are randomized DAG tests useful?
95. Why are allocation stress tests important?
96. Why do validation layers not prove all correctness?
97. Why are unsafe Rust audits necessary?
98. What is a resource ownership invariant?
99. Why must every benchmark validate correctness?
100. What makes a GPU runtime production-ready?

---

# 20. Final Examination

**Recommended duration:** 8–12 hours.

| Section | Weight |
|---|---:|
| Runtime architecture | 10% |
| Memory management | 15% |
| Resource lifetimes | 15% |
| Task graphs and scheduling | 20% |
| Synchronization | 20% |
| Reliability | 10% |
| Performance and observability | 10% |

## Part A: Architecture

Design a runtime for a multi-stage GPU application.

Explain module boundaries and API contracts.

## Part B: Memory allocator

Analyze an allocation sequence and calculate fragmentation.

Design a safe reclamation strategy.

## Part C: Resource lifetime

Analyze asynchronous submissions and determine when resources can be reused.

## Part D: Task graph

Construct a DAG, detect hazards, calculate critical path, and produce a valid schedule.

## Part E: Synchronization

Explain the execution and memory dependencies required for a multi-dispatch Vulkan pipeline.

## Part F: Reliability

Analyze allocation failures, timeouts, cancellation, and device loss.

## Part G: Performance

Design experiments to distinguish CPU runtime overhead from GPU execution cost.

## Part H: Technical defense

Defend the final Rust GPU runtime.

## Passing criteria

Recommended:

- At least 80% overall.
- At least 85% in resource lifetime and synchronization.
- No unresolved use-after-free risks.
- No known unsynchronized resource hazards.
- Complete automated correctness tests.
- Reproducible performance measurements.
- Clear documentation of unsafe invariants.

---

# 21. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

**Tutor restriction:** The tutor must never provide answers, code, pseudocode, implementation instructions, debugging fixes, file edits, completed derivations, or assessment solutions.

The tutor must guide reasoning using exactly one question per response.

## Master Socratic Tutor Prompt

```text
You are my Socratic GPU runtime systems tutor.

I am studying Phase 13 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   derivations, projects, or examinations.

2. Never write code, pseudocode, algorithms, or implementations.

3. Never modify files, execute code, debug programs, or perform
   project work on my behalf.

4. Never reveal answer keys, completed calculations, hidden tests,
   or solutions.

5. Never provide hints that effectively disclose a solution.

6. Guide my reasoning exclusively through questions.

7. Ask exactly one guiding question per response.

8. If I am incorrect, ask a question that exposes the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to explain GPU memory management,
    asynchronous resource lifetimes, task graphs,
    scheduling, synchronization, Rust safety invariants,
    reliability, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness,
    ownership, synchronization, failure handling,
    and performance. Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only one guiding question.
```

## Start a lesson

```text
Tutor me through Phase 13, Module [NUMBER].

Start from first principles.
Ask exactly one question at a time.
Require me to reason about runtime architecture,
ownership, scheduling, synchronization, and performance.

Never provide answers or solutions.
```

## Generate a quiz

```text
Administer a 20-question Phase 13 quiz on [TOPIC].

Include conceptual reasoning, memory management,
resource lifetimes, task graphs, scheduling,
synchronization, and runtime reliability.

Ask exactly one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text
Administer Phase 13 Coding Assessment [A/B/C].

Present requirements and constraints only.

Ask me to explain my architecture,
resource lifetime model, scheduling policy,
synchronization strategy, safety invariants,
and correctness tests.

Do not provide code, pseudocode, implementations,
debugging fixes, or solutions.

Ask exactly one question at a time.
```

## Review runtime code

```text
Review my GPU runtime code using only questions.

Focus on:
- Resource ownership
- Memory allocation
- Buffer aliasing
- Asynchronous lifetimes
- Dependency correctness
- Synchronization
- Error propagation
- Unsafe Rust invariants
- Performance

Do not provide fixes or replacement code.
Ask exactly one guiding question at a time.
```

## Memory allocator examination

```text
Examine my understanding of GPU memory allocators.

Ask questions about:
- Alignment
- Suballocation
- Free lists
- Buddy allocation
- Fragmentation
- Buffer pooling
- Memory budgets
- Completion-aware reuse

Never provide solutions.
Ask exactly one question at a time.
```

## Task graph examination

```text
Examine my understanding of GPU task graphs.

Ask questions about:
- DAGs
- Dependencies
- RAW, WAR, and WAW hazards
- Topological sorting
- Critical paths
- Scheduling
- Resource lifetimes
- Synchronization

Never provide answers or implementation guidance.
Ask exactly one question at a time.
```

## Synchronization examination

```text
Examine my understanding of GPU runtime synchronization.

Ask questions about:
- Execution dependencies
- Memory dependencies
- Pipeline stages
- Access masks
- Buffer barriers
- Fences
- Timeline semaphores
- Cross-queue synchronization
- Resource reclamation

Never provide solutions.
Ask exactly one question at a time.
```

## Reliability examination

```text
Examine my understanding of GPU runtime reliability.

Ask questions about:
- Allocation failures
- Submission failures
- Device loss
- Timeouts
- Cancellation
- Failure propagation
- Runtime state machines
- Resource cleanup

Never provide answers or fixes.
Ask exactly one question at a time.
```

## Performance investigation

```text
Help me investigate GPU runtime performance
using only questions.

Ask about:
- Allocation overhead
- Buffer reuse
- Command recording
- Submission overhead
- Pipeline creation
- Synchronization
- Queue utilization
- GPU execution time
- Memory pressure
- Benchmark methodology

Do not interpret results for me.
Do not provide conclusions or optimizations.
Ask exactly one question at a time.
```

## Final oral examination

```text
Conduct a rigorous Phase 13 oral examination.

Challenge my understanding of:
- Runtime architecture
- GPU memory allocators
- Asynchronous resource lifetimes
- Task graphs
- Scheduling
- Vulkan synchronization
- Reliability
- Multi-queue execution
- Observability
- Rust safety

Ask exactly one question at a time.
Never provide answers, solutions, code, or hints
that reveal the answer.
```

---

# 22. Mastery Checklist

## Runtime architecture

- [ ] I understand GPU runtime responsibilities.
- [ ] I can separate mechanisms from policies.
- [ ] I can design asynchronous APIs.
- [ ] I can define resource ownership contracts.
- [ ] I can explain runtime module boundaries.

## Memory management

- [ ] I can implement aligned suballocation.
- [ ] I understand linear allocators.
- [ ] I understand free-list allocators.
- [ ] I understand buddy allocators.
- [ ] I understand fragmentation.
- [ ] I can implement buffer pooling.
- [ ] I can track memory usage.
- [ ] I understand memory budgets.

## Resource lifetimes

- [ ] I understand asynchronous GPU lifetimes.
- [ ] I can track submission completion.
- [ ] I understand timeline semaphores.
- [ ] I can implement deferred destruction.
- [ ] I can prevent premature resource reuse.
- [ ] I understand resource aliasing.
- [ ] I can analyze resource lifetime intervals.

## Task graphs

- [ ] I can represent DAGs.
- [ ] I can detect cycles.
- [ ] I can perform topological sorting.
- [ ] I understand RAW hazards.
- [ ] I understand WAR hazards.
- [ ] I understand WAW hazards.
- [ ] I can calculate critical paths.
- [ ] I can infer resource dependencies.

## Scheduling

- [ ] I understand ready-task scheduling.
- [ ] I understand critical-path scheduling.
- [ ] I understand batching.
- [ ] I understand queue constraints.
- [ ] I understand backpressure.
- [ ] I can compare scheduling policies.

## Synchronization

- [ ] I understand execution dependencies.
- [ ] I understand memory dependencies.
- [ ] I understand pipeline stages.
- [ ] I understand access masks.
- [ ] I can generate synchronization plans.
- [ ] I understand cross-queue synchronization.
- [ ] I understand queue-family ownership.
- [ ] I can justify every synchronization primitive.

## Command and pipeline management

- [ ] I can manage command pools.
- [ ] I can reuse command buffers safely.
- [ ] I understand pipeline caching.
- [ ] I understand descriptor pooling.
- [ ] I understand shader variants.
- [ ] I can measure CPU runtime overhead.

## Reliability

- [ ] I can handle allocation failures.
- [ ] I can propagate task failures.
- [ ] I understand cancellation limitations.
- [ ] I understand timeout behavior.
- [ ] I understand device loss.
- [ ] I can design runtime state machines.
- [ ] I can implement fault-injection tests.

## Observability

- [ ] I can trace CPU runtime activity.
- [ ] I can measure GPU execution.
- [ ] I can visualize task graphs.
- [ ] I can report memory statistics.
- [ ] I can measure queue utilization.
- [ ] I can detect performance regressions.

## Capstone

- [ ] My GPU runtime executes task graphs.
- [ ] My allocator is correct.
- [ ] My resource lifetimes are safe.
- [ ] My synchronization is correct.
- [ ] My scheduler handles dependencies.
- [ ] My runtime handles failures.
- [ ] My tests cover concurrency and edge cases.
- [ ] My benchmarks are reproducible.
- [ ] I can defend my architecture independently.

---

# 23. Recommended Resources

## GPU runtime and Vulkan

**Vulkan Specification**

https://registry.khronos.org/vulkan/

**Vulkan Guide**

https://docs.vulkan.org/guide/latest/

**Vulkan Synchronization Examples**

https://github.com/KhronosGroup/Vulkan-Docs/wiki/Synchronization-Examples

Study these examples alongside the current Vulkan specification.

## Rust GPU ecosystem

**ash**

https://docs.rs/ash/

**Vulkano**

https://vulkano.rs/

**wgpu**

https://docs.rs/wgpu/

## Memory allocation

**Vulkan Memory Allocator**

https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator

Study its design and documentation as an example of production-oriented Vulkan memory management.

## Parallel scheduling

**The Art of Multiprocessor Programming — Herlihy and Shavit**

Useful for synchronization and concurrent data structures.

**Introduction to Parallel Computing — Grama, Gupta, Karypis, and Kumar**

Useful for scheduling and parallel performance models.

## GPU performance

**AMD GPUOpen**

https://gpuopen.com/

**Radeon GPU Profiler**

https://gpuopen.com/rgp/

**Programming Massively Parallel Processors — Kirk and Hwu**

## Further research

Study research and documentation on:

- GPU task graphs.
- Vulkan timeline semaphores.
- Asynchronous GPU execution.
- GPU memory allocation.
- Command submission overhead.
- Resource lifetime analysis.
- Multi-GPU scheduling.
- GPU runtime observability.
- Safe Rust GPU abstractions.
- Compute graph execution systems.

---

# 24. Completion Standard

You have completed Phase 13 when you can independently:

1. Design a GPU runtime architecture.
2. Implement a GPU memory allocator.
3. Implement buffer pooling.
4. Track asynchronous resource lifetimes.
5. Implement deferred destruction.
6. Represent and validate task graphs.
7. Detect resource dependencies.
8. Calculate critical paths.
9. Implement task scheduling.
10. Translate dependencies into Vulkan synchronization.
11. Manage command buffers and pipelines.
12. Track submission completion.
13. Handle failures and device loss.
14. Build runtime profiling tools.
15. Test asynchronous execution systematically.
16. Build and defend a reusable Rust GPU task graph runtime.

The defining achievement is **correct coordination of asynchronous GPU work and resources**, not simply launching kernels through a convenient API.

---

# 25. What Comes Next

**Phase 14: GPU Compilers**

The next phase will investigate how GPU programs are translated, analyzed, optimized, and lowered into executable instructions.

Topics include:

- Compiler architecture.
- Lexing and parsing.
- Abstract syntax trees.
- Intermediate representations.
- Static single assignment.
- Control-flow graphs.
- Data-flow analysis.
- Dominance.
- Type checking.
- GPU address spaces.
- SPIR-V.
- LLVM IR.
- Compiler optimization passes.
- Loop transformations.
- Vectorization.
- Memory optimization.
- Register allocation.
- Instruction scheduling.
- GPU code generation.
- Shader validation.
- Compiler correctness.
- Rust compiler integration.

You will move from building systems that execute GPU programs to understanding the compilers that transform those programs into hardware instructions.
