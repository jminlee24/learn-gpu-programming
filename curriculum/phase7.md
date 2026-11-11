# Phase 7: GPU Architecture

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 7 of 16  
**Estimated duration:** 12–16 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** Rust and C++20  
**Primary environment:** Fedora Linux  
**GPU:** AMD Radeon RX 9070, 16 GB VRAM  
**Prerequisites:** Phases 0–6

---

# 1. What You Will Learn

A GPU is not simply a CPU with more cores.

GPUs are designed to execute large numbers of related operations while sharing instruction control and amortizing hardware costs across many execution lanes.

They make different architectural trade-offs than general-purpose CPUs.

By the end of this phase, you should be able to:

1. Explain why GPUs are architecturally different from CPUs.
2. Describe GPU compute units and execution resources.
3. Understand SIMD and SIMT execution.
4. Explain AMD wavefronts and NVIDIA warps.
5. Understand thread and workgroup organization.
6. Explain GPU register allocation and pressure.
7. Understand GPU memory hierarchies.
8. Explain coalesced and uncoalesced memory access.
9. Understand local/shared memory and synchronization.
10. Explain occupancy and latency hiding.
11. Understand branch divergence.
12. Analyze GPU arithmetic throughput and memory bandwidth.
13. Apply the Roofline model to GPU workloads.
14. Explain asynchronous GPU command execution.
15. Understand GPU scheduling and resource contention.
16. Compare AMD and NVIDIA GPU architectures.
17. Investigate your own GPU's architecture and software stack.
18. Predict likely performance bottlenecks from kernel structure.

## Central question

**How does a GPU keep thousands of operations in flight while hiding the latency of memory access and arithmetic dependencies?**

Everything in this phase contributes to answering that question.

---

# 2. Study Schedule

| Module | Subject | Estimated time |
|---|---|---|
| 1 | CPU versus GPU architecture | 1 week |
| 2 | GPU execution model | 2 weeks |
| 3 | SIMD, SIMT, warps, and wavefronts | 2 weeks |
| 4 | GPU memory hierarchy | 2 weeks |
| 5 | Occupancy and latency hiding | 2 weeks |
| 6 | Divergence and control flow | 1 week |
| 7 | Performance models and bottlenecks | 2 weeks |
| 8 | AMD/NVIDIA architecture and capstone | 2–4 weeks |

---

# 3. Module One: Why GPUs Exist

## 3.1 CPU design priorities

Modern CPUs are optimized for a broad range of workloads.

They commonly devote significant hardware resources to:

- Sophisticated branch prediction.
- Out-of-order execution.
- Large caches.
- Low-latency single-thread execution.
- Speculative execution.
- Complex instruction scheduling.

These mechanisms help individual instruction streams execute quickly.

## 3.2 GPU design priorities

GPUs emphasize throughput.

They devote substantial hardware resources to executing many operations concurrently.

Common design priorities include:

- Large numbers of arithmetic execution lanes.
- High memory bandwidth.
- Hardware scheduling of many execution groups.
- Efficient execution of related operations.
- Specialized arithmetic units.
- High aggregate throughput.

The trade-off is that a GPU may perform poorly on small, serial, highly irregular workloads.

## 3.3 Latency versus throughput

Latency measures how long one operation takes.

Throughput measures how many operations complete per unit time.

A GPU may have high latency for an individual operation but high throughput when many independent operations execute concurrently.

## 3.4 Worked example: vector addition

Suppose we compute:

\[
C_i=A_i+B_i
\]

for one million elements.

Each output element is independent.

A CPU can use SIMD instructions and multiple cores.

A GPU can distribute the computation across many execution groups.

The GPU may achieve higher throughput for sufficiently large inputs, but total execution time also depends on:

- Data transfer.
- Command submission.
- Synchronization.
- Memory bandwidth.
- Kernel launch overhead.

## 3.5 Why GPUs need massive parallelism

GPU memory operations can have substantial latency.

Instead of relying primarily on large caches and aggressive out-of-order execution to hide this latency, GPUs often switch among ready groups of threads.

While one group waits for data, another can execute.

This is latency hiding through concurrency.

## Exercises

1. Why are CPUs optimized for low-latency execution?
2. Why are GPUs optimized for throughput?
3. Explain the difference between latency and throughput.
4. Why is vector addition naturally parallel?
5. Why might a GPU be slower than a CPU for a small workload?
6. Why does memory latency matter?
7. Explain how concurrent execution helps hide latency.
8. Why is a GPU not simply a CPU with more cores?

---

# 4. Module Two: GPU Execution Hardware

## 4.1 Compute units

GPUs contain multiple execution units capable of running groups of threads.

AMD commonly refers to major programmable execution blocks as **Compute Units**, or CUs.

NVIDIA commonly uses the term **Streaming Multiprocessor**, or SM.

These are not identical hardware structures, but they serve broadly comparable roles.

## 4.2 Execution resources

A compute unit or SM typically contains resources such as:

- Arithmetic execution lanes.
- Register storage.
- Scheduling hardware.
- Load/store capabilities.
- Shared or local memory.
- Instruction execution machinery.

The exact organization depends on the architecture.

## 4.3 Threads

A GPU thread represents one logical execution instance of a program.

Each thread may have:

- Its own logical index.
- Private variables.
- Register state.
- Access to shared and global memory.

## 4.4 Workgroups

Threads are organized into groups.

In Vulkan and WGSL, these are commonly called **workgroups**.

In CUDA, the corresponding concept is a **thread block**.

Workgroups allow threads to cooperate using shared memory and synchronization primitives.

## 4.5 Dispatch grids

A GPU dispatch launches many workgroups.

Conceptually:

```text id="j8b5wf"
Dispatch Grid
|
+-- Workgroup 0
|   +-- Thread 0
|   +-- Thread 1
|   +-- Thread 2
|
+-- Workgroup 1
|   +-- Thread 0
|   +-- Thread 1
|   +-- Thread 2
|
+-- Workgroup 2
    +-- Thread 0
    +-- Thread 1
    +-- Thread 2
```

The number of threads per workgroup and the number of workgroups are chosen by the application, subject to API and hardware limits.

## 4.6 Logical versus physical threads

A GPU application may launch millions of logical threads.

That does not mean millions of threads execute physically at the same instant.

The hardware schedules groups of threads onto available execution resources.

## 4.7 Workgroup scheduling

Workgroups are assigned to hardware execution units according to the device's scheduling mechanisms and resource availability.

The application should not assume a particular execution order between independent workgroups.

## Exercises

1. What is a compute unit?
2. What is a streaming multiprocessor?
3. What is a GPU thread?
4. What is a workgroup?
5. What is a dispatch grid?
6. Why are logical threads different from physical execution lanes?
7. Why should workgroups not assume a particular scheduling order?
8. What hardware resources limit the number of concurrently resident workgroups?

---

# 5. Module Three: SIMD and SIMT

## 5.1 SIMD

SIMD stands for Single Instruction, Multiple Data.

One instruction operates on multiple data elements.

For example:

\[
[a_0,a_1,a_2,a_3]
+
[b_0,b_1,b_2,b_3]
\]

produces:

\[
[a_0+b_0,a_1+b_1,a_2+b_2,a_3+b_3]
\]

using a vector-style operation.

## 5.2 SIMT

SIMT stands for Single Instruction, Multiple Threads.

A SIMT programming model presents many logical threads executing the same program.

Hardware groups these threads and executes their instructions together where possible.

Each logical thread may have its own:

- Index.
- Register values.
- Control-flow state.
- Memory addresses.

## 5.3 Warps and wavefronts

NVIDIA groups threads into warps.

A common NVIDIA warp size is 32 threads.

AMD groups lanes into wavefronts.

AMD architectures may support different wavefront sizes, commonly 32 or 64 lanes depending on the architecture and execution mode.

Do not assume every AMD GPU always uses wave64.

## 5.4 Why grouping matters

Threads in a warp or wavefront share aspects of instruction execution.

If all lanes perform similar work, the hardware can execute efficiently.

If lanes follow different control-flow paths, execution may become less efficient.

## 5.5 Lane masks

A lane mask indicates which lanes are active for a particular instruction or execution path.

Inactive lanes may not contribute useful work during that instruction.

## 5.6 Worked example: conditional computation

Suppose a wavefront executes:

```text id="8p2z9d"
if (index is even):
    perform operation A
else:
    perform operation B
```

If neighboring lanes take different paths, the hardware may need to execute the paths with different active-lane masks.

This can reduce effective throughput.

The exact behavior depends on the GPU architecture and compiler.

## 5.7 SIMD versus SIMT

SIMD is primarily an execution model in which instructions operate on vectors of data.

SIMT is a programming model exposing many logical threads.

GPU hardware often implements SIMT using SIMD-like execution resources.

These concepts are related but not identical.

## Exercises

1. Define SIMD.
2. Define SIMT.
3. What is a warp?
4. What is a wavefront?
5. Why does lane grouping matter?
6. What is an active-lane mask?
7. Explain the relationship between SIMD and SIMT.
8. Why can divergent control flow reduce throughput?
9. Why should programmers avoid assuming a fixed wavefront size across all AMD GPUs?

---

# 6. Module Four: GPU Memory Hierarchy

GPU performance depends heavily on how data moves through memory.

## 6.1 Global memory

Global memory is the device-visible memory address space used for large buffers and other resources.

On a discrete GPU, much of this data may reside in dedicated VRAM.

Global memory generally offers high capacity and bandwidth but relatively high access latency.

## 6.2 Registers

Registers are among the fastest storage resources available to executing threads.

They commonly hold:

- Temporary values.
- Loop variables.
- Intermediate arithmetic results.
- Addresses.
- Per-thread state.

Register capacity is limited.

## 6.3 Shared or local memory

Workgroups may use a fast memory region shared among their threads.

Terminology differs:

- CUDA: shared memory.
- WGSL: workgroup memory.
- AMD architecture documentation: local data share, or LDS.

These concepts are related, although their exact hardware and API semantics differ.

## 6.4 Cache hierarchy

Modern GPUs commonly include multiple cache levels.

Caches reduce the cost of repeated memory accesses.

The organization, coherence properties, and cache policies differ across architectures.

## 6.5 Memory hierarchy overview

```text id="a9y3ve"
Per-thread registers
        |
        v
Workgroup shared/local memory
        |
        v
GPU cache hierarchy
        |
        v
Global memory / VRAM
        |
        v
Host memory and external transfers
```

This is a conceptual illustration, not a literal path every memory access follows.

For example, workgroup memory is a separately addressed storage space rather than a transparent cache through which all global-memory traffic passes.

## 6.6 Memory coalescing

Memory coalescing refers to combining memory requests from multiple lanes into efficient memory transactions.

Suppose neighboring lanes access neighboring elements:

```text id="j3x8qv"
Lane 0 → A[0]
Lane 1 → A[1]
Lane 2 → A[2]
Lane 3 → A[3]
```

These accesses are often easier to service efficiently than widely scattered accesses.

## 6.7 Strided access

Consider:

```text id="k5f2um"
Lane 0 → A[0]
Lane 1 → A[128]
Lane 2 → A[256]
Lane 3 → A[384]
```

These accesses may require more memory transactions.

The exact cost depends on cache-line size, transaction granularity, and hardware behavior.

## 6.8 Data reuse

A GPU kernel can improve performance by reusing data already loaded into registers or workgroup memory.

This reduces repeated global-memory traffic.

## 6.9 Memory alignment

Aligned accesses may be more efficient or required for certain operations.

Misalignment can increase memory transaction cost or violate API constraints.

## 6.10 Host-to-device transfers

Discrete GPUs communicate with the CPU through an interconnect such as PCIe.

Transfers between host memory and GPU memory can be expensive relative to on-device arithmetic.

A GPU algorithm must account for data movement, not merely kernel execution time.

## Exercises

1. What is global memory?
2. What are registers?
3. What is workgroup memory?
4. What is LDS?
5. Why are caches important?
6. What is memory coalescing?
7. What is a strided memory access?
8. Why does data reuse matter?
9. Why can host-device transfers dominate a workload?
10. Explain why memory layout is important for GPU performance.

---

# 7. Module Five: Occupancy and Latency Hiding

Occupancy is one of the most important GPU performance concepts.

It is also frequently misunderstood.

## 7.1 What is occupancy?

Occupancy generally refers to the fraction of the hardware's maximum supported resident warps or wavefronts that are active on an execution unit.

The exact definition and measurement depend on the architecture and profiling tool.

## 7.2 Why occupancy matters

When one wavefront stalls, the scheduler may execute another ready wavefront.

More resident wavefronts can provide more opportunities to hide latency.

## 7.3 Occupancy is not utilization

A GPU can have high occupancy but low useful arithmetic utilization.

For example, many resident threads may spend most of their time waiting on memory.

Conversely, a compute-heavy kernel may achieve excellent performance without maximum occupancy.

## 7.4 Register pressure

Each thread consumes registers.

If a kernel uses many registers per thread, fewer threads may fit concurrently within a compute unit's register capacity.

This can reduce occupancy.

## 7.5 Workgroup memory pressure

Each workgroup consumes some amount of shared/local memory.

If a workgroup uses a large amount, fewer workgroups may reside simultaneously on the same execution unit.

## 7.6 Workgroup size

Workgroup size affects:

- Scheduling.
- Register allocation.
- Shared-memory usage.
- Synchronization.
- Available parallelism.
- Occupancy.

There is no universally optimal workgroup size.

## 7.7 Occupancy limits

Resident execution capacity may be limited by:

- Registers.
- Shared/local memory.
- Maximum resident workgroups.
- Maximum resident warps or wavefronts.
- Architecture-specific constraints.

## 7.8 Latency hiding

Suppose a wavefront issues a memory load and must wait for data.

The scheduler may issue instructions from another ready wavefront.

If enough independent work is available, the hardware can keep execution resources busy.

## 7.9 Instruction-level parallelism

Latency can also be hidden through independent instructions within a thread.

A kernel with multiple independent arithmetic operations may provide useful instruction-level parallelism.

## 7.10 Occupancy trade-offs

Increasing occupancy may require reducing register usage or shared-memory allocation.

But doing so can increase memory traffic or arithmetic overhead.

The correct goal is not maximum occupancy.

The goal is minimum execution time.

## Exercises

1. Define occupancy.
2. Explain why occupancy matters.
3. Why is occupancy different from utilization?
4. What is register pressure?
5. How does shared-memory usage affect occupancy?
6. How does workgroup size affect scheduling?
7. Explain latency hiding.
8. Explain instruction-level parallelism.
9. Why can maximum occupancy produce worse performance?
10. Why must occupancy optimizations be measured?

---

# 8. Module Six: Branch Divergence and Control Flow

## 8.1 Branch divergence

Branch divergence occurs when lanes in the same execution group follow different control-flow paths.

For example:

```text id="m9q4dc"
if (value > threshold):
    operation A
else:
    operation B
```

If all lanes take the same path, execution can be efficient.

If lanes take different paths, the hardware may need to execute both paths with different active-lane masks.

## 8.2 Divergence versus branch prediction

CPU performance often depends on branch prediction.

GPU performance frequently depends on how control flow interacts with grouped lane execution.

These are different architectural mechanisms.

## 8.3 Divergence is not always avoidable

Some algorithms inherently contain irregular control flow.

Examples include:

- Graph traversal.
- Ray tracing.
- Sparse computations.
- Adaptive numerical methods.

The goal is not to eliminate every branch.

It is to understand the cost and evaluate alternatives.

## 8.4 Predication

Predication allows conditional behavior to be expressed through masked operations.

It can reduce certain control-flow costs but may execute instructions that do not contribute useful results.

## 8.5 Data reorganization

Sometimes data can be reorganized so that nearby lanes perform similar work.

This may reduce divergence.

However, data reorganization itself has a cost.

## 8.6 Early exits

An early exit may save work for one thread.

But if other lanes in the execution group continue, the overall benefit may be smaller than expected.

## Exercises

1. What is branch divergence?
2. Why does divergence matter?
3. How does GPU divergence differ from CPU branch prediction?
4. What is predication?
5. Why is branchless code not always faster?
6. How can data organization affect divergence?
7. Why can early exits have unexpected performance behavior?
8. Give examples of workloads with irregular control flow.

---

# 9. Module Seven: GPU Performance Models

Performance models help predict bottlenecks.

They do not replace measurement.

## 9.1 Arithmetic throughput

Arithmetic throughput measures the number of arithmetic operations completed per unit time.

A common unit is FLOPs per second.

## 9.2 Memory bandwidth

Memory bandwidth measures the amount of data transferred per unit time.

A common unit is GB/s.

## 9.3 Arithmetic intensity

Arithmetic intensity is:

\[
I=\frac{\text{Floating-point operations}}{\text{Bytes transferred}}
\]

For example, a kernel performing two floating-point operations for every eight bytes transferred has:

\[
I=\frac{2}{8}=0.25\text{ FLOPs/byte}
\]

## 9.4 Roofline model

The Roofline model estimates an upper bound on performance:

\[
P\leq\min(P_{\text{peak}},B_{\text{peak}}\times I)
\]

where:

- \(P\) is achievable performance.
- \(P_{\text{peak}}\) is peak compute throughput.
- \(B_{\text{peak}}\) is peak memory bandwidth.
- \(I\) is arithmetic intensity.

## 9.5 Memory-bound workloads

A workload is memory-bound when memory movement limits performance.

Examples often include:

- Simple vector operations.
- Large streaming reductions.
- Certain sparse computations.

## 9.6 Compute-bound workloads

A workload is compute-bound when arithmetic execution limits performance.

Examples may include:

- Well-optimized dense matrix multiplication.
- Certain numerical simulations.
- Compute-intensive transformations.

## 9.7 Kernel launch overhead

Launching a GPU workload has overhead.

For very small workloads, this overhead can dominate execution time.

## 9.8 Transfer overhead

Data transfer between CPU and GPU memory may dominate the total time.

A fast GPU kernel does not guarantee a fast end-to-end application.

## 9.9 Synchronization overhead

Synchronization may prevent independent operations from overlapping.

Unnecessary synchronization can reduce throughput.

## 9.10 Performance measurement

A useful GPU benchmark should distinguish:

- CPU preparation time.
- Data transfer time.
- GPU execution time.
- Synchronization time.
- Total end-to-end time.

## Exercises

1. Define arithmetic throughput.
2. Define memory bandwidth.
3. Define arithmetic intensity.
4. Derive the Roofline bound.
5. Explain memory-bound execution.
6. Explain compute-bound execution.
7. Why does launch overhead matter?
8. Why do data transfers matter?
9. Why does synchronization affect throughput?
10. Why must GPU benchmarks measure end-to-end performance?

---

# 10. Module Eight: AMD and NVIDIA Architecture

You should understand both major GPU ecosystems.

Your own hardware provides a practical AMD development environment.

CUDA will be studied later using suitable NVIDIA hardware.

## 10.1 AMD architecture

Modern AMD GPUs use the RDNA family for many consumer graphics products.

Your Radeon RX 9070 belongs to the RDNA 4 generation.

Important architectural concepts include:

- Compute Units.
- Wavefronts.
- SIMD execution.
- Registers.
- Local Data Share.
- Cache hierarchy.
- VRAM.
- Command queues.
- Hardware scheduling.

## 10.2 NVIDIA architecture

Modern NVIDIA GPUs commonly use:

- Streaming Multiprocessors.
- Warps.
- CUDA cores.
- Registers.
- Shared memory.
- Cache hierarchy.
- Global memory.
- Specialized matrix arithmetic units.

## 10.3 Terminology comparison

| Concept | AMD terminology | NVIDIA terminology |
|---|---|---|
| Main execution block | Compute Unit | Streaming Multiprocessor |
| Group of execution lanes | Wavefront | Warp |
| Cooperative thread group | Workgroup | Thread block |
| Fast shared storage | LDS | Shared memory |
| Device memory | Global memory / VRAM | Global memory |
| Thread-private fast storage | Registers | Registers |

These terms are useful conceptual analogies, not guarantees of identical behavior.

## 10.4 Architectural portability

A GPU algorithm may be mathematically portable but perform differently across architectures.

Differences include:

- Wavefront or warp size.
- Register allocation.
- Cache organization.
- Shared-memory capacity.
- Scheduling behavior.
- Specialized arithmetic hardware.
- Compiler optimization.

## 10.5 Why portable APIs matter

Portable APIs such as Vulkan and wgpu allow programs to target multiple GPU vendors.

However, portability does not guarantee identical performance.

Performance tuning often requires understanding the underlying architecture.

## Exercises

1. What is an AMD Compute Unit?
2. What is an NVIDIA Streaming Multiprocessor?
3. What is a wavefront?
4. What is a warp?
5. What is LDS?
6. What is shared memory?
7. Why do architectural differences affect performance?
8. Why does API portability not guarantee performance portability?
9. Why should GPU engineers understand multiple architectures?

---

# 11. Practical Lab: Investigate Your GPU

Use your Fedora system to inspect your AMD GPU.

## 11.1 Identify the GPU

```bash id="af6dx7"
lspci | grep -Ei 'vga|display|3d'
```

## 11.2 Inspect the kernel driver

```bash id="6qzt2c"
lsmod | grep amdgpu
```

## 11.3 Inspect device nodes

```bash id="x0rt6e"
ls -l /dev/dri/
```

## 11.4 Inspect Vulkan support

If the relevant tools are installed:

```bash id="z9t4vq"
vulkaninfo --summary
```

You may need to install Vulkan utilities.

## 11.5 Inspect available system information

Explore:

```bash id="y8f5ma"
ls /sys/class/drm/
```

Record the information you can verify.

## Required report

Document:

- GPU model.
- GPU architecture.
- VRAM capacity.
- Kernel driver.
- Vulkan device information.
- Relevant driver stack.
- Available hardware capabilities.
- Unknown or unverified details.

Distinguish observed values from documentation-derived specifications.

---

# 12. Major Coding Assessment A: GPU Execution Simulator

Build a CPU-based simulator that models a simplified GPU execution system.

Use Rust.

## Required concepts

The simulator must represent:

- Logical threads.
- Workgroups.
- Wavefronts.
- Execution lanes.
- A simplified scheduler.
- A simplified memory system.
- Branch divergence.
- Resource limits.

## Required experiments

Investigate:

1. Different workgroup sizes.
2. Different wavefront sizes.
3. Different register limits.
4. Different shared-memory limits.
5. Different memory access patterns.
6. Different branch divergence patterns.

## Important limitation

Your simulator is an educational model.

It is not expected to reproduce the exact behavior of your AMD GPU.

You must document its assumptions.

## Technical defense

Explain:

- What the simulator models.
- What it intentionally ignores.
- How workgroups are scheduled.
- How divergence is represented.
- How occupancy is estimated.
- Why its results may differ from real hardware.

---

# 13. Major Coding Assessment B: Memory Access Pattern Analyzer

Build a program that analyzes memory access patterns.

## Required features

The program must:

1. Accept a simulated lane count.
2. Accept an address-generation pattern.
3. Generate memory addresses.
4. Group accesses into hypothetical memory transactions.
5. Estimate transaction efficiency.
6. Compare contiguous and strided accesses.
7. Compare aligned and misaligned accesses.
8. Produce structured reports.

## Required analysis

Explain:

- Why contiguous accesses are often efficient.
- Why strided accesses may be expensive.
- How alignment affects transactions.
- How transaction granularity influences results.
- Why actual hardware may behave differently from your model.

---

# 14. Major Coding Assessment C: GPU Performance Model

Build a small GPU performance estimator.

## Required inputs

- Estimated operation count.
- Estimated memory traffic.
- Peak compute throughput.
- Peak memory bandwidth.
- Launch overhead estimate.
- Transfer overhead estimate.

## Required outputs

- Arithmetic intensity.
- Compute-bound time estimate.
- Bandwidth-bound time estimate.
- Roofline performance bound.
- Estimated end-to-end time under clearly stated assumptions.

## Requirements

- Separate theoretical limits from measured values.
- Document assumptions.
- Handle invalid inputs.
- Support multiple hypothetical GPU configurations.
- Include tests.

## Technical defense

Explain:

- The Roofline model.
- Arithmetic intensity.
- Memory-bound behavior.
- Compute-bound behavior.
- Why the estimator is not a cycle-accurate simulator.
- Why real performance may differ.

---

# 15. Final Capstone: GPU Architecture Investigation

Produce a detailed technical report on your AMD Radeon RX 9070.

## Required sections

### 1. Hardware overview

Identify:

- Architecture generation.
- Compute resources.
- Memory capacity.
- Relevant memory hierarchy.
- Available architectural documentation.

### 2. Execution model

Explain:

- Logical threads.
- Workgroups.
- Wavefronts.
- Execution lanes.
- Scheduling.

### 3. Memory model

Explain:

- Registers.
- LDS.
- Caches.
- Global memory.
- Host-device transfers.

### 4. Performance model

Estimate:

- Arithmetic intensity for selected workloads.
- Memory bandwidth limitations.
- Compute throughput limitations.
- Expected bottlenecks.

### 5. Comparative analysis

Compare major architectural concepts with a contemporary NVIDIA GPU.

Use documented architectural information rather than assuming equivalent terminology implies equivalent hardware.

### 6. Limitations

Clearly identify:

- What you verified experimentally.
- What you obtained from documentation.
- What remains uncertain.
- Which architectural details are not publicly documented.

## Required defense

Be able to explain why:

- Occupancy matters.
- Maximum occupancy is not always optimal.
- Memory coalescing matters.
- Divergence matters.
- Register pressure matters.
- Arithmetic intensity matters.
- GPU execution is asynchronous.
- Different GPU architectures require different tuning decisions.

---

# 16. Phase 7 Quiz Bank

## CPU versus GPU

1. Why are CPUs optimized for low latency?
2. Why are GPUs optimized for throughput?
3. What is latency hiding?
4. Why do GPUs need many concurrent threads?
5. Why can a GPU be slower for small workloads?
6. What is the difference between latency and throughput?

## Execution hardware

7. What is a Compute Unit?
8. What is a Streaming Multiprocessor?
9. What is a GPU thread?
10. What is a workgroup?
11. What is a dispatch grid?
12. What limits resident workgroups?
13. Why are logical threads different from physical lanes?
14. Why is workgroup execution order not guaranteed?

## SIMD and SIMT

15. What is SIMD?
16. What is SIMT?
17. What is a warp?
18. What is a wavefront?
19. What is an active-lane mask?
20. Why does divergence matter?
21. Why can AMD wavefront size vary?
22. How are SIMD and SIMT related?

## Memory hierarchy

23. What is global memory?
24. What are registers?
25. What is LDS?
26. What is workgroup memory?
27. What is memory coalescing?
28. What is a strided access?
29. Why does alignment matter?
30. Why is data reuse important?
31. Why do host-device transfers matter?
32. Why are GPU caches important?

## Occupancy

33. What is occupancy?
34. Why is occupancy useful?
35. Why is occupancy not the same as utilization?
36. What is register pressure?
37. How does shared-memory usage affect occupancy?
38. How does workgroup size affect occupancy?
39. What is latency hiding?
40. Why is maximum occupancy not always optimal?

## Divergence

41. What is branch divergence?
42. What is predication?
43. Why can branchless code be slower?
44. Why does data organization affect divergence?
45. Why are irregular workloads difficult?
46. Why can early exits behave unexpectedly?

## Performance

47. What is arithmetic throughput?
48. What is memory bandwidth?
49. What is arithmetic intensity?
50. What is the Roofline model?
51. What is a memory-bound workload?
52. What is a compute-bound workload?
53. What is launch overhead?
54. What is transfer overhead?
55. Why does synchronization matter?
56. Why must end-to-end performance be measured?

## Architecture comparison

57. What is RDNA?
58. What is a Compute Unit?
59. What is a Streaming Multiprocessor?
60. What is the difference between a wavefront and a warp?
61. Why does architectural portability matter?
62. Why does performance portability remain difficult?
63. Why should GPU engineers study both AMD and NVIDIA architectures?
64. Why must architectural claims be verified against documentation?

---

# 17. Final Examination

**Recommended duration:** 5–6 hours.

| Section | Weight |
|---|---:|
| CPU versus GPU architecture | 10% |
| GPU execution model | 15% |
| SIMD/SIMT and wavefronts | 15% |
| Memory hierarchy | 20% |
| Occupancy and latency hiding | 15% |
| Divergence | 10% |
| Performance models | 10% |
| AMD/NVIDIA comparison | 5% |

## Part A: Architecture reasoning

Explain the major architectural trade-offs between CPUs and GPUs.

## Part B: Execution model

Analyze a hypothetical GPU dispatch.

Explain:

- Thread count.
- Workgroup count.
- Wavefront grouping.
- Resource limits.
- Scheduling implications.

## Part C: Memory analysis

Analyze memory access patterns.

Identify likely transaction inefficiencies and opportunities for reuse.

## Part D: Occupancy

Analyze hypothetical register and shared-memory constraints.

Explain which resources limit occupancy.

## Part E: Divergence

Analyze conditional control flow and active-lane behavior.

## Part F: Performance modeling

Calculate arithmetic intensity and Roofline bounds for hypothetical workloads.

## Part G: Technical defense

Defend your GPU architecture capstone.

---

# 18. Copy-Ready AI Tutor Prompts

All prompts are consolidated here.

The tutor must never provide answers, code, pseudocode, implementations, fixes, or completed work.

## Master Socratic Tutor Prompt

```text id="m7r4vq"
You are my Socratic GPU architecture tutor.

I am studying Phase 7 of a rigorous GPU engineering curriculum.

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

10. Require me to explain execution models, memory hierarchy,
    occupancy, divergence, arithmetic intensity, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about assumptions,
    correctness, modeling limitations, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only guiding questions.
```

## Start a lesson

```text id="w4k8p2"
Tutor me through Phase 7, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to explain GPU architectural trade-offs.
Never provide answers or solutions.
```

## Generate a quiz

```text id="x9c3n5"
Administer a 20-question Phase 7 quiz on [TOPIC].

Include conceptual reasoning, architecture analysis,
memory behavior, occupancy, and performance questions.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text id="t2h7m4"
Administer Phase 7 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my architectural model and assumptions.

Guide me through questions about correctness,
execution behavior, resource limits, memory access,
and performance.

Do not write code, pseudocode, or implementations.
```

## Review architectural analysis

```text id="b6q1v8"
Review my GPU architecture analysis using only questions.

Focus on:
- Execution model
- Wavefronts and warps
- Memory hierarchy
- Occupancy
- Register pressure
- Divergence
- Arithmetic intensity
- Performance assumptions

Do not provide corrections or conclusions.
Ask one guiding question at a time.
```

## GPU architecture oral examination

```text id="r8z5d3"
Examine my understanding of GPU architecture.

Ask me to explain how GPUs execute threads,
hide latency, access memory, and schedule work.

Challenge assumptions through questions.
Never provide answers.
```

## Performance modeling assessment

```text id="n3p6y9"
Examine my understanding of GPU performance models.

Ask questions about:
- FLOPs
- Memory bandwidth
- Arithmetic intensity
- Roofline bounds
- Occupancy
- Divergence
- Register pressure
- Launch overhead
- Data transfers

Ask one question at a time.
Never provide calculations or solutions.
```

---

# 19. Mastery Checklist

## Architecture

- [ ] I understand CPU versus GPU design priorities.
- [ ] I understand latency versus throughput.
- [ ] I understand GPU compute units.
- [ ] I understand GPU thread scheduling.

## Execution model

- [ ] I understand GPU threads.
- [ ] I understand workgroups.
- [ ] I understand dispatch grids.
- [ ] I understand warps and wavefronts.
- [ ] I understand SIMD and SIMT.

## Memory

- [ ] I understand registers.
- [ ] I understand workgroup memory.
- [ ] I understand LDS.
- [ ] I understand GPU caches.
- [ ] I understand global memory.
- [ ] I understand memory coalescing.
- [ ] I understand host-device transfers.

## Occupancy

- [ ] I understand occupancy.
- [ ] I understand register pressure.
- [ ] I understand shared-memory pressure.
- [ ] I understand latency hiding.
- [ ] I understand why maximum occupancy is not always optimal.

## Control flow

- [ ] I understand divergence.
- [ ] I understand active-lane masks.
- [ ] I understand predication.
- [ ] I understand irregular workloads.

## Performance

- [ ] I understand arithmetic intensity.
- [ ] I understand memory bandwidth.
- [ ] I understand compute throughput.
- [ ] I understand the Roofline model.
- [ ] I understand launch overhead.
- [ ] I understand transfer overhead.

## Architecture comparison

- [ ] I understand AMD GPU terminology.
- [ ] I understand NVIDIA GPU terminology.
- [ ] I understand major architectural differences.
- [ ] I understand performance portability limitations.

## Practical investigation

- [ ] I can identify my GPU hardware.
- [ ] I can identify the Linux GPU driver.
- [ ] I can inspect Vulkan support.
- [ ] I can document verified architectural properties.
- [ ] I can defend a GPU architecture performance model.

---

# 20. Recommended Resources

## GPU architecture

**Programming Massively Parallel Processors — David B. Kirk and Wen-mei W. Hwu**

A foundational textbook on GPU programming and architecture.

**Computer Architecture: A Quantitative Approach — John L. Hennessy and David A. Patterson**

Useful for broader architectural reasoning.

## AMD documentation

**AMD GPUOpen**

https://gpuopen.com/

**AMD RDNA Architecture Documentation**

https://gpuopen.com/learn/

**AMD GPU ISA Documentation**

https://gpuopen.com/amd-isa-documentation/

Use the documentation matching the architecture generation being studied.

## NVIDIA documentation

**CUDA C++ Programming Guide**

https://docs.nvidia.com/cuda/cuda-c-programming-guide/

**NVIDIA GPU Architecture Whitepapers**

Available through NVIDIA's developer documentation.

## GPU performance

**Roofline: An Insightful Visual Performance Model for Multicore Architectures**

A foundational paper on arithmetic intensity and performance bounds.

**GPU Performance Engineering Resources**

https://gpuopen.com/performance/

## Linux GPU stack

**Linux DRM Documentation**

https://docs.kernel.org/gpu/

**Mesa Documentation**

https://docs.mesa3d.org/

---

# 21. Completion Standard

You have completed Phase 7 when you can independently:

1. Explain CPU and GPU architectural trade-offs.
2. Explain GPU thread and workgroup execution.
3. Explain SIMD and SIMT.
4. Explain warps and wavefronts.
5. Explain GPU memory hierarchy.
6. Analyze memory access patterns.
7. Explain occupancy and latency hiding.
8. Explain branch divergence.
9. Calculate arithmetic intensity.
10. Apply the Roofline model.
11. Explain AMD and NVIDIA architectural differences.
12. Investigate your own GPU's software and hardware environment.
13. Defend your architecture capstone and performance assumptions.

You should now understand the fundamental execution and memory mechanisms that determine GPU performance.

---

# 22. What Comes Next

**Phase 8: Portable GPU Programming with Rust, wgpu, and WGSL**

The next phase will move from architecture theory into real GPU programming.

You will learn:

- Rust GPU application setup.
- wgpu device initialization.
- WGSL compute shaders.
- GPU buffers.
- Bind groups.
- Compute pipelines.
- Dispatching workgroups.
- GPU synchronization.
- Host-device data transfers.
- GPU debugging.
- GPU benchmarking.
- Parallel reductions.
- Prefix scans.
- Matrix multiplication.
- Performance optimization.

You will begin writing and executing GPU compute workloads on your AMD Radeon RX 9070.
