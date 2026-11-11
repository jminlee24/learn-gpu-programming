# Phase 6: Concurrency and Parallel Algorithms

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 6 of 16  
**Estimated duration:** 12–16 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** Rust and C++20  
**Primary hardware:** CPU on Fedora Linux  
**GPU:** AMD Radeon RX 9070, used in later phases  
**Prerequisites:** Phases 0–5

---

# 1. Phase Objectives

By the end of this phase, you should be able to:

1. Distinguish concurrency from parallelism.
2. Decompose algorithms into independent tasks.
3. Analyze parallel algorithms using work and span.
4. Apply Amdahl's Law and Gustafson's Law.
5. Understand the C++ and Rust memory models.
6. Implement correct synchronization using mutexes, condition variables, and atomics.
7. Reason about deadlock, livelock, starvation, and contention.
8. Implement thread pools and task schedulers.
9. Understand work stealing and load balancing.
10. Implement parallel reductions and prefix scans.
11. Understand parallel sorting and partitioning.
12. Analyze memory locality and false sharing.
13. Understand lock-free data structures conceptually.
14. Design repeatable parallel benchmarks.
15. Explain how CPU parallel algorithms translate into GPU execution models.

## What this phase is not

This is not simply a course about spawning threads.

Creating threads is straightforward. Designing correct and efficient parallel algorithms is substantially harder.

A program can use every available CPU core and still perform worse than its sequential equivalent.

Your objective is to understand why that happens and how to investigate it.

---

# 2. Study Schedule

| Module | Subject | Time |
|---|---|---|
| 1 | Parallel computation fundamentals | 1–2 weeks |
| 2 | Work, span, and scaling laws | 1–2 weeks |
| 3 | Memory models and synchronization | 2 weeks |
| 4 | Thread pools and task scheduling | 2 weeks |
| 5 | Parallel reductions and scans | 2 weeks |
| 6 | Sorting, partitioning, and irregular parallelism | 2 weeks |
| 7 | Lock-free concepts and performance engineering | 2 weeks |
| 8 | Capstone and examination | 1–2 weeks |

**Study rule:** Every parallel implementation must have a correct sequential reference implementation.

Parallelizing an incorrect algorithm does not make it correct.

---

# 3. Module One: Concurrency Versus Parallelism

## 3.1 Concurrency

Concurrency describes a system in which multiple activities can make progress over overlapping periods.

They do not necessarily execute at the same instant.

A single CPU core can support concurrency by switching between threads.

For example:

- Thread A performs a small amount of work.
- The scheduler pauses A.
- Thread B executes.
- The scheduler resumes A.

The activities overlap in time but may not execute simultaneously.

## 3.2 Parallelism

Parallelism means multiple operations execute simultaneously.

For example, on a multicore CPU:

- Core 0 processes one portion of an array.
- Core 1 processes another portion.
- Core 2 processes a third portion.

These computations may execute at the same time.

## 3.3 Why the distinction matters

Concurrency is often used to manage independent activities, responsiveness, and I/O.

Parallelism is often used to reduce the execution time of computational work.

A system may be concurrent without being parallel.

A system may also be both concurrent and parallel.

## 3.4 Task parallelism

Task parallelism divides work into distinct tasks.

For example:

- Task A loads data.
- Task B parses metadata.
- Task C computes statistics.

The tasks may perform different operations.

## 3.5 Data parallelism

Data parallelism applies similar operations to different portions of data.

For example:

\[
y_i=2x_i+1
\]

Each output element can be computed independently.

Data parallelism is particularly important for GPU programming.

## 3.6 Pipeline parallelism

Pipeline parallelism divides processing into stages.

For example:

```text
Input → Decode → Transform → Encode → Output
```

Different stages can operate on different data items simultaneously.

The throughput of the pipeline depends on the stages' capacities and coordination.

## 3.7 Dependencies

Parallelism is limited by dependencies.

Consider:

\[
a=b+c
\]

\[
d=a+e
\]

The second operation depends on the first.

Now consider:

\[
a=b+c
\]

\[
d=e+f
\]

These operations are independent.

A central skill in parallel programming is identifying the dependency graph of a computation.

## Worked example: processing one million pixels

Suppose you apply the same brightness transformation to one million independent pixels.

If each pixel depends only on its own input, the computation is naturally data-parallel.

Now suppose each output pixel also depends on neighboring pixels.

The computation may still be parallelizable, but the algorithm must account for neighboring data and possible boundaries between work partitions.

This distinction becomes important in image processing, simulations, and GPU stencil computations.

## Exercises

1. Explain concurrency without using the word parallelism.
2. Explain parallelism without using the word concurrency.
3. Give an example of task parallelism.
4. Give an example of data parallelism.
5. Give an example of pipeline parallelism.
6. Identify dependencies in a simple arithmetic expression.
7. Explain why dependencies limit parallel execution.
8. Explain why creating more threads does not automatically increase performance.

---

# 4. Module Two: Work, Span, and Parallel Complexity

Big-O complexity describes how algorithmic costs scale, but parallel algorithms require additional measures.

Two particularly important quantities are work and span.

## 4.1 Work

Work is the total amount of computation performed.

We denote it by:

\[
T_1
\]

It is often interpreted as the time required by an idealized single processor to execute the algorithm's operations.

For a sequential sum of \(N\) numbers:

\[
T_1=O(N)
\]

## 4.2 Span

Span is the length of the longest dependency chain.

We denote it by:

\[
T_\infty
\]

It represents a lower bound on execution time even with arbitrarily many processors, under the chosen computational model.

## 4.3 Parallelism

The average available parallelism is:

\[
\mathcal{P}=\frac{T_1}{T_\infty}
\]

This ratio indicates how much parallel execution the dependency structure potentially supports.

## 4.4 Worked example: sequential summation

Consider:

\[
S=x_1+x_2+\cdots+x_N
\]

A sequential implementation accumulates values one at a time.

Its work is:

\[
T_1=O(N)
\]

Its span is:

\[
T_\infty=O(N)
\]

Therefore:

\[
\mathcal{P}=O(1)
\]

The sequential dependency chain limits parallelism.

## 4.5 Worked example: balanced tree reduction

A balanced reduction combines pairs of values.

For eight inputs:

```text
Level 0:  a   b   c   d   e   f   g   h
           \ /     \ /     \ /     \ /
Level 1:   ab      cd      ef      gh
             \    /          \    /
Level 2:      abcd            efgh
                   \          /
Level 3:               total
```

For \(N\) inputs:

\[
T_1=O(N)
\]

\[
T_\infty=O(\log N)
\]

The available parallelism is approximately:

\[
\mathcal{P}=O\left(\frac{N}{\log N}\right)
\]

The reduction performs similar total work but exposes much more parallelism.

## 4.6 Brent's theorem

For a computation with work \(T_1\) and span \(T_\infty\), a useful scheduling bound is:

\[
T_P=O\left(\frac{T_1}{P}+T_\infty\right)
\]

where \(P\) is the number of processors.

The two terms represent different limitations:

- Work distributed across processors.
- Unavoidable dependency-chain length.

This model assumes an appropriate scheduler and abstracts away many real hardware costs.

## 4.7 Amdahl's Law

Suppose a fraction \(s\) of a program is inherently serial.

With \(P\) processors, the idealized speedup is:

\[
S(P)=\frac{1}{s+\frac{1-s}{P}}
\]

### Worked example

Suppose 10% of a program is serial.

With eight processors:

\[
S(8)=\frac{1}{0.1+\frac{0.9}{8}}
\]

\[
S(8)\approx4.71
\]

Even with eight processors, the maximum idealized speedup is less than eight.

## 4.8 Gustafson's Law

Amdahl's Law examines speedup for a fixed workload.

Gustafson's Law considers scaling the workload with the number of processors.

One common expression is:

\[
S(P)=P-s(P-1)
\]

where \(s\) is the serial fraction under the scaled-workload interpretation.

This helps explain why large scientific workloads can use many processors effectively even when smaller workloads do not.

## 4.9 Strong scaling

Strong scaling measures performance when the problem size stays fixed and the processor count increases.

## 4.10 Weak scaling

Weak scaling measures performance when the workload per processor remains approximately constant as processor count increases.

## 4.11 Parallel efficiency

Parallel efficiency is:

\[
E(P)=\frac{S(P)}{P}
\]

An efficiency of 1 represents ideal linear speedup under the model.

Real applications commonly achieve lower efficiency because of communication, synchronization, imbalance, and other overhead.

## Exercises

1. Define work.
2. Define span.
3. Derive the work of a sequential sum.
4. Derive the span of a tree reduction.
5. Calculate average parallelism.
6. Apply Brent's theorem to a hypothetical computation.
7. Calculate speedup using Amdahl's Law.
8. Explain Gustafson's Law.
9. Distinguish strong scaling from weak scaling.
10. Calculate parallel efficiency.
11. Explain why work and span do not fully predict real hardware performance.

---

# 5. Module Three: The Memory Model and Synchronization

Correct parallel programming requires understanding what operations other threads are allowed to observe.

## 5.1 Why source-code order is insufficient

A programmer writes statements in a particular order.

However:

- Compilers optimize instructions.
- Processors execute instructions out of order.
- Caches and store buffers affect visibility.
- Threads execute independently.

A concurrent program cannot assume that every thread observes operations in the same order without appropriate synchronization.

## 5.2 The happens-before relationship

The happens-before relationship describes ordering constraints that determine when effects of one operation must be visible to another.

It is central to the C++ memory model and to reasoning about synchronization.

## 5.3 Data races

In C++, conflicting non-atomic accesses to the same memory location without the required happens-before relationship can create a data race.

A data race results in undefined behavior.

In safe Rust, the type system and synchronization abstractions prevent many data races.

However, Rust does not prevent all logical race conditions.

## 5.4 Mutexes

A mutex provides exclusive access to a protected critical section.

A correct mutex-based design must define:

- Which state is protected.
- Which lock protects it.
- Which operations require the lock.
- How long the lock is held.
- Whether multiple locks may be acquired together.

## 5.5 Lock contention

Contention occurs when multiple threads compete for the same lock.

A highly contended lock can serialize an otherwise parallel workload.

Reducing lock contention may require:

- Partitioning data.
- Reducing critical-section duration.
- Combining operations.
- Using thread-local accumulation.
- Changing the algorithm.

## 5.6 Atomics

Atomic operations provide indivisible access to supported memory objects.

Common operations include:

- Load.
- Store.
- Exchange.
- Fetch-add.
- Compare-and-exchange.

Atomicity does not automatically make a multi-step algorithm correct.

## 5.7 Memory ordering

C++ provides memory-ordering modes such as:

- Relaxed.
- Acquire.
- Release.
- Acquire-release.
- Sequentially consistent.

### Relaxed ordering

Relaxed atomics guarantee atomicity for the operation but do not establish general synchronization ordering for unrelated memory operations.

### Release and acquire

A release operation can publish preceding writes.

A corresponding acquire operation that observes the appropriate release sequence can establish synchronization.

### Sequential consistency

Sequential consistency provides a stronger global ordering model for sequentially consistent atomic operations.

It is often easier to reason about, though the exact guarantees still require understanding the memory model.

## 5.8 Condition variables

A condition variable allows a thread to wait until a shared condition may have changed.

Correct waiting requires a predicate.

Spurious wakeups are possible.

The condition must be checked again after waking.

## 5.9 Deadlock

A deadlock occurs when a collection of threads cannot make progress because of circular resource dependencies.

Common prevention strategies include:

- Consistent lock ordering.
- Avoiding unnecessary nested locks.
- Reducing shared state.
- Using scoped locking abstractions.
- Designing operations to avoid holding locks while waiting.

## 5.10 Livelock and starvation

Livelock occurs when threads remain active but make no useful progress.

Starvation occurs when a thread repeatedly fails to obtain the resources required to proceed.

## 5.11 False sharing

Two threads can modify different variables that occupy the same cache line.

Their operations may create coherence traffic even though the variables are logically independent.

This is false sharing.

It can cause severe performance degradation.

## Worked example: shared counter

Suppose four threads each increment a shared counter 100,000 times.

The expected final count is:

\[
4\times100000=400000
\]

A naive non-atomic increment is not a safe concurrent operation.

The read-modify-write sequence may overlap with other threads, and in C++ the unsynchronized conflicting accesses produce undefined behavior.

Possible correct designs include:

- Mutex protection.
- Atomic increments.
- Thread-local counters followed by a reduction.

These designs have different performance characteristics.

## Exercises

1. Explain the happens-before relationship.
2. Explain why source-code order does not guarantee cross-thread visibility.
3. What is a data race?
4. What does a mutex guarantee?
5. What is lock contention?
6. What is an atomic operation?
7. What does relaxed ordering guarantee?
8. What is the purpose of acquire-release synchronization?
9. What is sequential consistency?
10. Why must condition-variable waits use predicates?
11. Explain deadlock.
12. Explain false sharing.
13. Compare a mutex-protected counter with thread-local accumulation.

## Coding Lab 1: Synchronization Laboratory

Implement several versions of a shared counter:

1. Mutex-protected counter.
2. Atomic counter.
3. Thread-local counters with final reduction.

For each implementation:

- Verify correctness.
- Measure runtime.
- Vary thread count.
- Vary work per thread.
- Investigate contention.
- Document trade-offs.

Do not use a data-racing implementation as a performance baseline.

---

# 6. Module Four: Thread Pools and Task Scheduling

Creating a new operating-system thread for every small task can be expensive.

Thread pools allow a fixed collection of worker threads to execute many tasks.

## 6.1 Thread pool architecture

A basic thread pool contains:

- Worker threads.
- A task queue.
- A synchronization mechanism.
- A shutdown mechanism.

Conceptually:

```text
Producer
   |
   v
Task Queue
   |
   +---- Worker 1
   |
   +---- Worker 2
   |
   +---- Worker 3
   |
   +---- Worker 4
```

## 6.2 Task granularity

Task granularity describes how much work each task performs.

Very small tasks may incur excessive scheduling overhead.

Very large tasks may limit parallelism and load balancing.

The ideal granularity depends on:

- Task execution cost.
- Scheduler overhead.
- Processor count.
- Memory behavior.
- Workload variability.

## 6.3 Static scheduling

Static scheduling assigns work to workers in advance.

It can be efficient when tasks have predictable costs.

However, uneven workloads may leave some workers idle.

## 6.4 Dynamic scheduling

Dynamic scheduling assigns work as workers become available.

It can improve load balancing when task durations vary.

However, dynamic scheduling introduces coordination overhead.

## 6.5 Work stealing

Work stealing allows idle workers to obtain tasks from other workers.

A common design uses:

- Per-worker task queues.
- Local task execution.
- Stealing when local work is exhausted.

Work stealing can improve utilization for irregular workloads.

## 6.6 Work stealing versus a global queue

A global queue is simple but may become a contention point.

Per-worker queues can reduce contention and improve locality.

However, they introduce complexity involving stealing, queue ownership, and scheduling fairness.

## 6.7 Structured concurrency

Structured concurrency organizes concurrent tasks within well-defined lifetimes and scopes.

The goal is to make task ownership, cancellation, completion, and error propagation easier to reason about.

This is especially relevant in Rust, where ownership and lifetimes can help express safe task relationships.

## 6.8 Cancellation

Task cancellation is not simply terminating a thread at an arbitrary instruction.

A robust design must consider:

- Which operations can be interrupted.
- Which resources must be released.
- Whether partially completed work is valid.
- How cancellation propagates.
- How shutdown completes.

## 6.9 Backpressure

Backpressure prevents producers from overwhelming consumers.

For example, a bounded task queue can prevent unbounded memory growth when tasks arrive faster than workers can execute them.

## Exercises

1. Why use a thread pool?
2. What is task granularity?
3. Explain static scheduling.
4. Explain dynamic scheduling.
5. What is work stealing?
6. Why might a global queue become a bottleneck?
7. What is structured concurrency?
8. What is backpressure?
9. Why is cancellation difficult?
10. Explain how task scheduling affects cache locality.

## Coding Lab 2: Thread Pool

Implement a thread pool in Rust or C++.

Requirements:

- Configurable worker count.
- Task submission.
- Worker coordination.
- Graceful shutdown.
- Error handling.
- No lost tasks.
- No data races.
- Automated tests.

Measure throughput for different task sizes.

---

# 7. Module Five: Parallel Reductions

Reductions are among the most common building blocks in GPU algorithms.

## 7.1 Reduction definition

A reduction combines a collection of values using an operation.

Examples include:

\[
\sum_i x_i
\]

\[
\max_i x_i
\]

\[
\prod_i x_i
\]

## 7.2 Sequential reduction

A sequential reduction performs one combination after another.

For \(N\) inputs:

\[
W=O(N)
\]

\[
S=O(N)
\]

## 7.3 Tree reduction

A balanced reduction combines pairs in parallel.

For \(N\) inputs:

\[
W=O(N)
\]

\[
S=O(\log N)
\]

This assumes the operation can be regrouped appropriately.

For floating-point addition, regrouping can change the numerical result.

## 7.4 Partitioned reduction

A practical multicore reduction may:

1. Divide input into chunks.
2. Compute a partial result for each chunk.
3. Combine the partial results.

This reduces shared-state contention.

## 7.5 Reduction correctness

A reduction must consider:

- Identity elements.
- Empty inputs.
- Operation associativity.
- Overflow.
- Floating-point error.
- Determinism.
- Synchronization.

## 7.6 Numerical reproducibility

Different thread counts or scheduling orders may change floating-point reduction order.

This can produce slightly different results.

Reproducibility may require controlling reduction structure or using specialized numerical techniques.

## 7.7 GPU connection

GPU reductions often use:

- Workgroups.
- Shared memory.
- Barriers.
- Subgroup operations.
- Hierarchical reduction stages.

The CPU algorithms in this phase establish the conceptual foundation.

## Exercises

1. Derive reduction work and span.
2. Explain a tree reduction.
3. Explain a partitioned reduction.
4. Why are identity elements important?
5. Why can floating-point reduction results vary?
6. Why is thread-local accumulation often faster than frequent atomic updates?
7. Explain how reduction chunk size affects performance.
8. Explain how a reduction could be divided across GPU workgroups.

## Coding Lab 3: Parallel Reduction Suite

Implement:

- Sequential sum.
- Tree-based reduction.
- Thread-pool reduction.
- Parallel minimum and maximum.
- Parallel dot product.

Measure:

- Runtime.
- Speedup.
- Efficiency.
- Numerical error.
- Thread-count scaling.

---

# 8. Module Six: Prefix Sums and Scans

Prefix scans are foundational parallel algorithms.

They appear in:

- Stream compaction.
- Sorting.
- Graph algorithms.
- Memory allocation.
- GPU data processing.

## 8.1 Prefix sum definition

Given:

\[
x=[1,2,3,4]
\]

an inclusive prefix sum is:

\[
[1,3,6,10]
\]

Each output contains the sum of all preceding elements, including the current element.

## 8.2 Exclusive prefix sum

An exclusive prefix sum excludes the current element.

For the same input:

\[
[0,1,3,6]
\]

The initial value is the additive identity.

## 8.3 Sequential scan

A sequential scan processes elements from left to right.

Its work is:

\[
O(N)
\]

Its span is:

\[
O(N)
\]

## 8.4 Parallel scan

Parallel scan algorithms use structured combinations of partial results.

A common family of algorithms uses:

1. An upsweep phase.
2. A downsweep phase.

The upsweep computes partial aggregates.

The downsweep distributes prefix information.

## 8.5 Work-efficient scan

A work-efficient parallel scan can achieve:

\[
W=O(N)
\]

\[
S=O(\log N)
\]

under an appropriate parallel model.

## 8.6 Why scans are harder than reductions

A reduction produces one result.

A scan produces an output for every input position.

Each output depends on a prefix of the input.

The algorithm must propagate information efficiently without performing a separate reduction for every output.

## 8.7 Stream compaction

Stream compaction removes elements that do not satisfy a predicate.

For example:

```text
Input:      [3, 8, 1, 6, 9]
Predicate:  [0, 1, 0, 1, 1]
Output:     [8, 6, 9]
```

A prefix scan can help determine the destination index for each retained element.

This is a common GPU algorithmic pattern.

## Exercises

1. Compute an inclusive prefix sum by hand.
2. Compute an exclusive prefix sum by hand.
3. Explain the difference between scan and reduction.
4. Explain the purpose of upsweep.
5. Explain the purpose of downsweep.
6. Derive the work and span of a work-efficient scan.
7. Explain why a naive scan may perform unnecessary work.
8. Explain how scan supports stream compaction.
9. Explain why scan is important in GPU programming.

## Coding Lab 4: Parallel Scan

Implement:

- Sequential inclusive scan.
- Sequential exclusive scan.
- Parallel inclusive scan.
- Parallel exclusive scan.

Validate against sequential reference implementations.

Test:

- Empty input.
- One-element input.
- Non-power-of-two lengths.
- Large arrays.
- Different numeric types where supported.

---

# 9. Module Seven: Parallel Sorting and Partitioning

Sorting is a useful case study because it combines comparisons, data movement, and coordination.

## 9.1 Sequential sorting

You should already understand algorithms such as:

- Insertion sort.
- Merge sort.
- Quicksort.
- Heap sort.

This phase focuses on how sorting changes under parallel execution.

## 9.2 Parallel merge sort

Merge sort recursively divides data into smaller subproblems.

Independent halves can be sorted in parallel.

The merge step combines sorted results.

Parallelizing the recursive sorting is straightforward conceptually, but parallel merging introduces additional complexity.

## 9.3 Parallel quicksort

Quicksort partitions data around a pivot.

The resulting partitions can be processed independently.

However, partition sizes may be uneven.

This creates load-balancing challenges.

## 9.4 Sorting networks

Sorting networks use a predetermined sequence of comparisons and exchanges.

They are useful for understanding data-independent parallel structure.

Examples include:

- Bitonic sorting networks.
- Odd-even merge networks.

Their regular structure can be attractive on parallel hardware.

## 9.5 Radix sort

Radix sort processes keys according to digits or groups of bits.

Parallel radix sorting often relies on:

- Histograms.
- Prefix sums.
- Scatter operations.
- Multiple passes.

This is an important connection between scans and sorting.

## 9.6 Partitioning

Partitioning divides data according to a predicate or key.

Parallel partitioning often requires determining output positions without write conflicts.

Prefix scans are useful for this.

## 9.7 Irregular parallelism

Not all workloads divide evenly.

Examples include:

- Graph traversal.
- Recursive algorithms.
- Sparse matrix operations.
- Variable-length records.

Irregular workloads can suffer from:

- Load imbalance.
- Unpredictable memory access.
- Synchronization overhead.
- Poor locality.

## Exercises

1. Explain parallel merge sort.
2. Explain why parallel quicksort may suffer from imbalance.
3. What is a sorting network?
4. What is bitonic sort?
5. What is radix sort?
6. Why are prefix scans useful in radix sort?
7. What is parallel partitioning?
8. Why is irregular parallelism difficult?
9. Explain why GPU sorting algorithms often differ from CPU sorting algorithms.

## Coding Lab 5: Parallel Sorting

Implement at least two parallel sorting approaches.

Compare them against:

- A correct sequential implementation.
- A standard-library sorting implementation.

Measure:

- Runtime.
- Scaling.
- Memory usage.
- Sensitivity to input distributions.
- Load imbalance.

---

# 10. Module Eight: Lock-Free Programming

Lock-free programming uses atomic operations and carefully designed algorithms to coordinate concurrent access without relying on ordinary mutual-exclusion locks for every operation.

This is an advanced topic.

## 10.1 Why lock-free algorithms exist

Locks can introduce:

- Contention.
- Blocking.
- Priority inversion.
- Deadlock risks.
- Scheduling delays.

Lock-free algorithms attempt to provide progress guarantees under concurrent execution.

## 10.2 Progress guarantees

Important categories include:

### Obstruction-free

A thread can complete its operation if it eventually executes without interference.

### Lock-free

The system as a whole continues making progress under contention, even if individual threads may starve.

### Wait-free

Each operation completes within a bounded number of its own steps, under the algorithm's assumptions.

These guarantees are different.

## 10.3 Compare-and-exchange

Compare-and-exchange conditionally updates a value if it matches an expected value.

It is a fundamental building block for many lock-free algorithms.

## 10.4 The ABA problem

The ABA problem occurs when a location changes:

\[
A\rightarrow B\rightarrow A
\]

A thread may observe the final value A and incorrectly assume that nothing changed.

This can cause subtle correctness problems.

## 10.5 Memory reclamation

Lock-free data structures must handle memory reclamation carefully.

A node cannot be freed while another thread may still access it.

Common approaches include:

- Hazard pointers.
- Epoch-based reclamation.
- Reference counting in suitable designs.

Memory reclamation is often harder than the basic atomic update algorithm.

## 10.6 Lock-free does not mean faster

A lock-free algorithm can perform worse than a lock-based algorithm because of:

- Repeated retries.
- Cache-line contention.
- Complex memory ordering.
- Additional bookkeeping.
- Poor locality.

Progress guarantees and performance are different properties.

## Exercises

1. Define lock-free progress.
2. Define wait-free progress.
3. Define obstruction-free progress.
4. Explain compare-and-exchange.
5. Explain the ABA problem.
6. Explain why memory reclamation is difficult.
7. Explain why lock-free does not imply high performance.
8. Explain why lock-free programming requires careful memory-model reasoning.

## Coding Lab 6: Atomic Data Structure Investigation

Study an atomic counter and a bounded concurrent queue.

Begin with a correct, mutex-protected queue.

Then investigate a bounded atomic queue design.

Do not attempt an unrestricted lock-free linked structure until you can explain:

- Its progress guarantee.
- Its memory ordering.
- Its ownership model.
- Its reclamation strategy.
- Its correctness argument.

For this phase, a conceptual analysis of a published lock-free queue is an acceptable alternative to implementing one.

---

# 11. Module Nine: Parallel Performance Engineering

Correctness is only the first requirement.

A parallel program must also use hardware efficiently.

## 11.1 Speedup

Speedup is:

\[
S(P)=\frac{T_1}{T_P}
\]

where:

- \(T_1\) is sequential runtime.
- \(T_P\) is parallel runtime using \(P\) processors.

## 11.2 Efficiency

Efficiency is:

\[
E(P)=\frac{S(P)}{P}
\]

## 11.3 Synchronization overhead

Synchronization may require:

- Atomic operations.
- Lock acquisition.
- Waiting.
- Cache coherence traffic.
- Memory fences.

These operations can reduce parallel speedup.

## 11.4 Communication overhead

Parallel workers may need to exchange data.

Communication can involve:

- Shared memory.
- Cache coherence.
- Inter-thread queues.
- NUMA memory transfers.

Communication costs can dominate small tasks.

## 11.5 Load imbalance

If some workers finish much earlier than others, available processors may remain idle.

Load balancing attempts to distribute work more evenly.

## 11.6 Memory bandwidth saturation

A program may reach the memory system's bandwidth limit before all CPU cores are fully utilized.

Adding threads may then produce little benefit.

This is particularly common in low-arithmetic-intensity workloads.

## 11.7 False sharing

False sharing can create cache coherence traffic even when threads update different logical variables.

Padding or changing data layout may improve performance, but changes must be measured.

## 11.8 NUMA effects

On NUMA systems, memory access cost depends on the location of the memory relative to the executing processor.

Thread placement and memory allocation policies can influence performance.

## 11.9 Benchmarking parallel programs

A good parallel benchmark should:

1. Validate correctness.
2. Establish a sequential baseline.
3. Use representative inputs.
4. Measure repeated runs.
5. Record thread counts.
6. Record hardware characteristics.
7. Control background load where practical.
8. Report speedup and efficiency.
9. Investigate bottlenecks.
10. Avoid unsupported conclusions.

## Exercises

1. Calculate speedup.
2. Calculate efficiency.
3. Explain synchronization overhead.
4. Explain communication overhead.
5. Explain load imbalance.
6. Explain memory bandwidth saturation.
7. Explain false sharing.
8. Explain NUMA effects.
9. Explain why a parallel program may become slower as threads are added.
10. Explain why a benchmark must validate correctness.

---

# 12. Major Coding Assessment A: Parallel Algorithms Library

Build a reusable Rust parallel algorithms library.

## Required algorithms

Implement:

1. Parallel map.
2. Parallel reduction.
3. Parallel inclusive scan.
4. Parallel exclusive scan.
5. Parallel partition.
6. Parallel sorting.

## Requirements

- Correct sequential reference implementations.
- Configurable worker count.
- Correct handling of empty inputs.
- Correct handling of uneven partitions.
- No data races.
- No lost work.
- Meaningful error handling.
- Automated correctness tests.
- Performance benchmarks.

## Technical defense

You must explain:

- The dependency structure of each algorithm.
- Work and span.
- Synchronization requirements.
- Memory access patterns.
- Load balancing.
- Numerical behavior.
- Expected scaling limitations.

---

# 13. Major Coding Assessment B: Work-Stealing Scheduler

Build a task scheduler in Rust or C++.

## Requirements

The scheduler must:

1. Create a configurable number of workers.
2. Accept dynamically generated tasks.
3. Support local task queues.
4. Support work stealing.
5. Detect completion.
6. Support graceful shutdown.
7. Handle task failures.
8. Prevent lost or duplicated tasks.
9. Collect scheduling statistics.
10. Support performance experiments.

## Required experiments

Compare:

- A global task queue.
- Static task assignment.
- Work stealing.

Use workloads with:

- Uniform task costs.
- Highly variable task costs.
- Recursive task generation.

## Technical defense

Explain:

- How tasks are owned.
- How task completion is detected.
- How work stealing avoids duplicate execution.
- What synchronization is required.
- How contention is reduced.
- How load balancing is measured.
- Why different workloads favor different scheduling strategies.

---

# 14. Major Coding Assessment C: Parallel Performance Investigation

Choose three algorithms from your library.

For each algorithm:

1. Measure sequential runtime.
2. Measure parallel runtime.
3. Calculate speedup.
4. Calculate efficiency.
5. Vary input size.
6. Vary thread count.
7. Identify bottlenecks.
8. Inspect relevant hardware behavior.
9. Compare observations with theoretical predictions.
10. Produce a technical report.

## Required discussion

Your report must explain:

- Whether scaling is strong or weak.
- Whether the workload is compute-bound or memory-bound.
- Whether synchronization limits performance.
- Whether load imbalance is significant.
- Whether task granularity is appropriate.
- Whether cache locality affects performance.

---

# 15. Final Capstone: Parallel Computing Runtime

Build a small CPU parallel computing runtime.

This is the culminating project of Phase 6.

## Required components

### Task execution

- Worker threads.
- Task submission.
- Task completion tracking.
- Graceful shutdown.

### Parallel algorithms

- Map.
- Reduce.
- Scan.
- Sort or partition.

### Performance measurement

- Runtime.
- Speedup.
- Efficiency.
- Worker utilization.
- Task execution statistics.

### Correctness

- Unit tests.
- Stress tests.
- Edge-case tests.
- Concurrency tests.
- Documented invariants.

## Required reports

Produce:

1. Architecture document.
2. Correctness argument.
3. Benchmark report.
4. Scaling analysis.
5. Comparison with a mature parallel library.
6. GPU translation analysis.

## GPU translation analysis

For each algorithm, explain:

- What constitutes a unit of work.
- Which operations are independent.
- Which operations require synchronization.
- What data must be shared.
- What memory access patterns occur.
- How the algorithm might map to GPU workgroups.
- What challenges GPU execution introduces.

You are not yet required to implement GPU kernels.

The goal is to develop the reasoning needed to design them.

---

# 16. Phase 6 Quiz Bank

## Parallelism fundamentals

1. What is concurrency?
2. What is parallelism?
3. What is task parallelism?
4. What is data parallelism?
5. What is pipeline parallelism?
6. What is a dependency graph?
7. Why do dependencies limit parallel execution?
8. Why does more concurrency not necessarily imply more throughput?

## Work and span

9. What is work?
10. What is span?
11. What is average parallelism?
12. What is Brent's theorem?
13. What is Amdahl's Law?
14. What is Gustafson's Law?
15. What is strong scaling?
16. What is weak scaling?
17. What is speedup?
18. What is parallel efficiency?

## Synchronization

19. What is a data race?
20. What is happens-before?
21. What is a mutex?
22. What is lock contention?
23. What is an atomic operation?
24. What is relaxed memory ordering?
25. What is acquire-release synchronization?
26. What is sequential consistency?
27. What is a condition variable?
28. What is deadlock?
29. What is livelock?
30. What is starvation?
31. What is false sharing?

## Scheduling

32. What is a thread pool?
33. What is task granularity?
34. What is static scheduling?
35. What is dynamic scheduling?
36. What is work stealing?
37. Why can a global queue become a bottleneck?
38. What is structured concurrency?
39. What is cancellation?
40. What is backpressure?

## Reductions and scans

41. What is a reduction?
42. What is a tree reduction?
43. What is a partitioned reduction?
44. Why does reduction order matter for floating-point arithmetic?
45. What is an inclusive scan?
46. What is an exclusive scan?
47. What is upsweep?
48. What is downsweep?
49. What is a work-efficient scan?
50. What is stream compaction?

## Sorting and irregular parallelism

51. What is parallel merge sort?
52. What is parallel quicksort?
53. What is a sorting network?
54. What is bitonic sort?
55. What is radix sort?
56. Why does radix sort use prefix scans?
57. What is parallel partitioning?
58. What is irregular parallelism?
59. Why is load balancing difficult for irregular workloads?

## Lock-free programming

60. What is lock-free progress?
61. What is wait-free progress?
62. What is obstruction-free progress?
63. What is compare-and-exchange?
64. What is the ABA problem?
65. What is memory reclamation?
66. Why is lock-free programming difficult?
67. Why does lock-free not necessarily mean faster?

## Performance

68. What is synchronization overhead?
69. What is communication overhead?
70. What is load imbalance?
71. What is memory bandwidth saturation?
72. What is NUMA?
73. Why can additional threads reduce performance?
74. Why must parallel benchmarks validate correctness?
75. How do CPU parallel algorithms connect to GPU programming?

---

# 17. Final Examination

**Recommended duration:** 5–6 hours.

| Section | Weight |
|---|---:|
| Parallel computation fundamentals | 10% |
| Work, span, and scaling laws | 15% |
| Memory models and synchronization | 20% |
| Task scheduling | 15% |
| Reductions and scans | 15% |
| Sorting and irregular parallelism | 10% |
| Lock-free concepts | 5% |
| Performance engineering | 10% |

## Part A: Parallel algorithm analysis

Analyze hypothetical algorithms for:

- Work.
- Span.
- Available parallelism.
- Synchronization requirements.
- Expected scaling.

## Part B: Concurrency correctness

Analyze multithreaded programs for:

- Data races.
- Deadlocks.
- Incorrect synchronization.
- Resource ownership.
- Progress guarantees.

## Part C: Parallel algorithm design

Design:

- A parallel reduction.
- A parallel scan.
- A parallel partition.
- A parallel sorting strategy.

Explain the dependency structure of each.

## Part D: Scheduling

Compare:

- Static scheduling.
- Dynamic scheduling.
- Work stealing.

Explain the workloads for which each approach is appropriate.

## Part E: Performance

Analyze:

- Speedup.
- Efficiency.
- Load imbalance.
- Synchronization overhead.
- Memory bandwidth.
- Cache locality.

## Part F: Technical defense

Defend your parallel computing runtime.

You must explain the correctness and performance of your design without relying on an AI to solve problems for you.

---

# 18. Copy-Ready AI Tutor Prompts

All prompts are grouped here.

The tutor must never provide answers, code, pseudocode, implementations, fixes, or completed work.

## Master Socratic Tutor Prompt

```text id="l4v3a1"
You are my Socratic concurrency and parallel algorithms tutor.

I am studying Phase 6 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   derivations, or examinations.

2. Never write code, pseudocode, algorithms, or implementations.

3. Never modify files, execute code, debug programs, or complete
   assessment work on my behalf.

4. Never reveal answer keys, hidden tests, or completed solutions.

5. Never give hints that effectively disclose solutions.

6. Guide my reasoning exclusively through questions.

7. Ask one primary question per response.

8. If I am incorrect, ask a question that exposes the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to explain dependencies, invariants, synchronization,
    memory ordering, correctness, work, span, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about races, deadlocks,
    ownership, progress guarantees, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only guiding questions.
```

## Start a lesson

```text id="g2e8s9"
Tutor me through Phase 6, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to explain dependency structures and parallelism.
Never provide answers or solutions.
```

## Generate a quiz

```text id="w8p3r2"
Administer a 20-question Phase 6 quiz on [TOPIC].

Include conceptual reasoning, concurrency correctness,
work/span analysis, and performance questions.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text id="k7n5b4"
Administer Phase 6 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my design and correctness argument.

Guide me through questions about synchronization,
ownership, dependencies, load balancing, testing,
and performance.

Do not write code, pseudocode, or implementations.
```

## Review concurrent code

```text id="c9m2t6"
Review my concurrent code using only questions.

Focus on:
- Data races
- Deadlocks
- Memory ordering
- Resource ownership
- Progress guarantees
- Task completion
- Synchronization overhead
- Load balancing

Do not provide fixes or replacement code.
Ask one guiding question at a time.
```

## Parallel algorithm examination

```text id="q6x1j8"
Examine my understanding of parallel algorithms.

Ask me to derive work, span, and expected scaling.
Challenge my assumptions about dependencies and synchronization.

Do not provide calculations, answers, or solutions.
```

## Performance investigation

```text id="v3d7h5"
Help me investigate parallel performance using only questions.

Ask about:
- Speedup
- Efficiency
- Synchronization
- Load imbalance
- Task granularity
- Memory bandwidth
- Cache locality
- NUMA
- Benchmark methodology

Do not interpret results for me or provide conclusions.
```

---

# 19. Mastery Checklist

## Parallelism

- [ ] I understand concurrency and parallelism.
- [ ] I understand task and data parallelism.
- [ ] I can identify dependency graphs.
- [ ] I can distinguish independent and dependent operations.

## Complexity

- [ ] I can calculate work.
- [ ] I can calculate span.
- [ ] I understand Brent's theorem.
- [ ] I understand Amdahl's Law.
- [ ] I understand Gustafson's Law.
- [ ] I can calculate speedup and efficiency.

## Synchronization

- [ ] I understand data races.
- [ ] I understand happens-before.
- [ ] I understand mutexes.
- [ ] I understand atomic operations.
- [ ] I understand memory ordering conceptually.
- [ ] I understand deadlock and starvation.
- [ ] I understand false sharing.

## Scheduling

- [ ] I can implement a thread pool.
- [ ] I understand task granularity.
- [ ] I understand static scheduling.
- [ ] I understand dynamic scheduling.
- [ ] I understand work stealing.
- [ ] I understand backpressure.

## Parallel algorithms

- [ ] I can implement a parallel reduction.
- [ ] I can implement an inclusive scan.
- [ ] I can implement an exclusive scan.
- [ ] I understand parallel partitioning.
- [ ] I understand parallel sorting.
- [ ] I understand irregular parallelism.

## Advanced concurrency

- [ ] I understand lock-free progress.
- [ ] I understand wait-free progress.
- [ ] I understand compare-and-exchange.
- [ ] I understand the ABA problem.
- [ ] I understand memory reclamation conceptually.

## Performance

- [ ] I can benchmark parallel algorithms.
- [ ] I can calculate speedup and efficiency.
- [ ] I can identify synchronization bottlenecks.
- [ ] I can identify load imbalance.
- [ ] I can identify memory bandwidth limitations.
- [ ] I can explain why additional threads may hurt performance.

---

# 20. Recommended Resources

## Parallel algorithms

**Introduction to Parallel Algorithms — Joseph JáJá**

A foundational treatment of parallel algorithm design.

**Introduction to Parallel Computing — Ananth Grama, Anshul Gupta, George Karypis, and Vipin Kumar**

Useful for parallel models, scheduling, and performance analysis.

**The Art of Multiprocessor Programming — Maurice Herlihy and Nir Shavit**

A rigorous treatment of concurrency, synchronization, and nonblocking algorithms.

## Concurrency

**C++ Concurrency in Action — Anthony Williams**

Useful for C++ threading and memory-model concepts.

**Rust Atomics and Locks — Mara Bos**

https://marabos.nl/atomics/

An excellent resource for understanding Rust atomics, locks, and memory ordering.

## Practical parallel programming

**Rayon**

https://docs.rs/rayon/

Study how Rust expresses data-parallel computations.

**Intel oneTBB**

https://github.com/uxlfoundation/oneTBB

Study task-based parallel programming and scheduling.

**OpenMP**

https://www.openmp.org/

Useful for understanding compiler-supported CPU parallelism.

## GPU preparation

**GPU Gems**

https://developer.nvidia.com/gpugems/gpugems/contributors

Contains foundational GPU algorithm discussions.

**NVIDIA CUDA Programming Guide**

https://docs.nvidia.com/cuda/

Read selectively for conceptual connections to GPU execution.

You do not need an NVIDIA GPU to study the conceptual material, but CUDA execution requires an appropriate NVIDIA environment.

---

# 21. Completion Standard

You have completed Phase 6 when you can independently:

1. Analyze a parallel algorithm using work and span.
2. Explain its dependency graph.
3. Design a correct synchronization strategy.
4. Implement a thread pool.
5. Implement parallel reductions and scans.
6. Explain parallel sorting approaches.
7. Reason about lock-free progress guarantees.
8. Identify performance bottlenecks.
9. Measure scaling and efficiency.
10. Defend a parallel runtime's correctness.
11. Explain how its algorithms could map onto GPU execution hardware.

**Do not advance until you can implement and explain a work-efficient parallel prefix scan.** It is one of the most valuable algorithmic building blocks for later GPU programming.

---

# 22. What Comes Next

**Phase 7: GPU Architecture**

The next phase will cover:

- GPU execution hardware.
- Compute units and streaming multiprocessors.
- Warps and wavefronts.
- SIMD and SIMT.
- Thread scheduling.
- GPU registers.
- Shared/local memory.
- Global memory.
- Memory coalescing.
- GPU caches.
- Occupancy.
- Latency hiding.
- Branch divergence.
- GPU synchronization.
- GPU performance models.
- Architectural differences between AMD and NVIDIA GPUs.

This is where the mathematical, systems, and parallel programming foundations begin to converge into GPU engineering.
