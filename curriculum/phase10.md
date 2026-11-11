# Phase 10: GPU Optimization and Performance Engineering

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 10 of 16  
**Estimated duration:** 16–20 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** Rust, WGSL, C++20, CUDA C++  
**Primary GPU:** AMD Radeon RX 9070, 16 GB  
**Primary platform:** Fedora Linux  
**Optional environment:** NVIDIA GPU for CUDA profiling  
**Prerequisites:** Phases 0–9

---

# 1. Mission

A working GPU kernel is not necessarily an efficient GPU kernel.

A kernel may be limited by:

- Global-memory bandwidth.
- Memory access inefficiency.
- Arithmetic throughput.
- Register pressure.
- Low occupancy.
- Warp or wavefront divergence.
- Synchronization.
- Atomic contention.
- Kernel launch overhead.
- Data transfers.
- Poor data reuse.
- Insufficient parallelism.

These bottlenecks require different optimization strategies.

An effective GPU engineer does not blindly apply optimization techniques.

Instead, they follow a repeatable process:

1. Establish correctness.
2. Measure a baseline.
3. Construct a performance hypothesis.
4. Identify the limiting resource.
5. Design a controlled experiment.
6. Measure the result.
7. Explain the observed behavior.
8. Preserve improvements with regression tests.

This process is the central skill of Phase 10.

## Learning objectives

By the end of this phase, you should be able to:

1. Design reproducible GPU benchmarks.
2. Distinguish kernel latency from end-to-end latency.
3. Calculate arithmetic intensity.
4. Apply the Roofline model.
5. Estimate memory traffic.
6. Diagnose memory bandwidth bottlenecks.
7. Analyze memory coalescing.
8. Optimize data reuse.
9. Understand occupancy limitations.
10. Analyze register pressure.
11. Explain instruction-level parallelism.
12. Diagnose branch divergence.
13. Understand synchronization overhead.
14. Analyze atomic contention.
15. Apply kernel fusion.
16. Understand asynchronous execution.
17. Design parameterized kernels.
18. Build an autotuning framework.
19. Compare GPU implementations fairly.
20. Produce defensible performance reports.

---

# 2. Study Schedule

| Module | Subject | Duration |
|---|---|---|
| 1 | Benchmarking and experimental methodology | 2 weeks |
| 2 | Roofline and performance modeling | 2 weeks |
| 3 | Memory bandwidth and access patterns | 2–3 weeks |
| 4 | Occupancy and register pressure | 2 weeks |
| 5 | Instruction-level parallelism and divergence | 2 weeks |
| 6 | Synchronization, atomics, and fusion | 2 weeks |
| 7 | Autotuning and architecture-specific optimization | 2–3 weeks |
| 8 | Capstone and final examination | 2–4 weeks |

---

# 3. Module One: Scientific GPU Benchmarking

## 3.1 Why benchmarking is difficult

GPU performance measurements can be affected by:

- Shader compilation.
- Pipeline creation.
- Driver initialization.
- GPU clock variation.
- Thermal behavior.
- Background GPU activity.
- Cache state.
- Data transfer.
- Synchronization.
- Measurement overhead.

A single timing measurement is rarely sufficient.

## 3.2 Kernel execution time

Kernel execution time measures the GPU's execution interval for a particular workload.

It should be distinguished from the time required to submit commands from the CPU.

## 3.3 End-to-end time

End-to-end time includes the complete operation experienced by the application.

For example:

\[
T_{\text{total}}=
T_{\text{prepare}}+
T_{\text{upload}}+
T_{\text{execute}}+
T_{\text{readback}}
\]

This expression assumes sequential stages.

With overlap, the total time is not necessarily the sum of the individual stage durations.

## 3.4 Warm-up

Initial runs may include:

- Shader compilation.
- Pipeline creation.
- Memory allocation.
- Driver initialization.

Steady-state benchmarks should separate these costs when appropriate.

## 3.5 Repeated measurements

Use repeated trials.

Report statistics such as:

- Median.
- Minimum.
- Maximum.
- Standard deviation.
- Percentiles.

The median is often useful because it is less sensitive to isolated slow runs than the arithmetic mean.

## 3.6 Controlled experiments

A controlled experiment changes one important variable at a time.

For example:

- Workgroup size.
- Tile size.
- Input size.
- Memory layout.
- Number of dispatches.

If several variables change simultaneously, explaining the result becomes harder.

## 3.7 Statistical significance

A measured difference may reflect noise rather than a genuine performance improvement.

Small changes should be repeated and evaluated against measurement variability.

## 3.8 Benchmark correctness

Every benchmark must verify that the optimized implementation produces acceptable results.

A kernel that omits required work is not faster in a meaningful sense.

## Exercises

1. Why are GPU benchmarks noisy?
2. Why is warm-up important?
3. Why must kernel time and end-to-end time be separated?
4. Why are repeated measurements necessary?
5. What is a controlled experiment?
6. Why should correctness be validated before benchmarking?
7. Why can a single fast measurement be misleading?
8. Why should benchmark environments be documented?

## Lab 1: GPU Benchmark Harness

Build a reusable Rust GPU benchmark framework.

Requirements:

- Configurable warm-up.
- Repeated trials.
- GPU timing where supported.
- End-to-end timing.
- Statistical summaries.
- Correctness validation.
- Structured output.
- Device metadata.
- Reproducible configurations.

Use your Phase 8 kernels as benchmark subjects.

---

# 4. Module Two: Roofline Performance Modeling

## 4.1 Why performance models matter

A performance model provides a way to reason about hardware limits.

Without a model, optimization becomes guesswork.

## 4.2 Arithmetic intensity

Arithmetic intensity is:

\[
I=\frac{\text{Floating-point operations}}{\text{Bytes transferred}}
\]

The denominator must be defined carefully.

For Roofline analysis, it typically refers to traffic at a chosen memory hierarchy level.

## 4.3 Roofline model

The Roofline model gives:

\[
P\leq\min(P_{\text{peak}},B_{\text{peak}}I)
\]

where:

- \(P\) is achievable performance.
- \(P_{\text{peak}}\) is peak arithmetic throughput.
- \(B_{\text{peak}}\) is peak memory bandwidth.
- \(I\) is arithmetic intensity.

## 4.4 Ridge point

The ridge point is the arithmetic intensity at which the compute and bandwidth ceilings intersect.

\[
I_{\text{ridge}}=\frac{P_{\text{peak}}}{B_{\text{peak}}}
\]

Below the ridge point, a workload is potentially bandwidth-limited.

Above the ridge point, it is potentially compute-limited.

This is a simplified model.

Real performance can be limited by other resources.

## 4.5 Worked example: vector addition

For:

\[
C_i=A_i+B_i
\]

each element requires:

- One load from A.
- One load from B.
- One store to C.
- One floating-point addition.

For 32-bit floats, the nominal traffic is:

\[
3\times4=12\text{ bytes}
\]

The arithmetic intensity is approximately:

\[
I=\frac{1}{12}\approx0.0833\text{ FLOPs/byte}
\]

This is a low-arithmetic-intensity operation.

It is therefore likely to be memory-bandwidth-limited for sufficiently large arrays, assuming the data traffic reaches the memory level being modeled.

## 4.6 Worked example: matrix multiplication

For square matrices of dimension \(N\), conventional dense matrix multiplication performs approximately:

\[
2N^3
\]

floating-point operations.

If each matrix is read or written once in an idealized model, the minimum traffic for three FP32 matrices is approximately:

\[
12N^2\text{ bytes}
\]

Thus:

\[
I_{\text{ideal}}\approx\frac{2N^3}{12N^2}
=\frac{N}{6}
\]

This idealized intensity increases with matrix size.

Actual memory traffic depends on implementation and cache reuse.

## 4.7 Model limitations

The Roofline model does not fully capture:

- Synchronization.
- Instruction dependencies.
- Divergence.
- Register spilling.
- Launch overhead.
- Cache behavior.
- Specialized arithmetic hardware.
- Scheduling inefficiencies.

It is a useful upper-bound model, not a complete GPU simulator.

## Exercises

1. Define arithmetic intensity.
2. Derive the Roofline model.
3. Calculate the ridge point.
4. Derive vector-addition arithmetic intensity.
5. Derive idealized matrix-multiplication arithmetic intensity.
6. Explain memory-bound behavior.
7. Explain compute-bound behavior.
8. Explain why the Roofline model is not a complete performance predictor.

## Lab 2: Roofline Analyzer

Build a program that:

- Accepts operation counts.
- Accepts estimated memory traffic.
- Accepts compute throughput.
- Accepts memory bandwidth.
- Calculates arithmetic intensity.
- Calculates theoretical performance bounds.
- Compares measured performance.
- Produces reports.

Document assumptions explicitly.

---

# 5. Module Three: Memory Bandwidth Optimization

## 5.1 Memory bandwidth as a limiting resource

Many GPU kernels perform little arithmetic per byte transferred.

Such kernels may be limited by memory bandwidth.

Examples include:

- Vector addition.
- Simple reductions.
- Large array transformations.
- Certain sparse operations.

## 5.2 Coalesced memory access

GPU memory systems are designed to service groups of nearby memory accesses efficiently.

When neighboring lanes access neighboring addresses, the hardware can often combine requests into fewer memory transactions.

## 5.3 Strided access

Strided accesses may generate more transactions.

For example:

```text id="b8c4m2"
Lane 0 → A[0]
Lane 1 → A[64]
Lane 2 → A[128]
Lane 3 → A[192]
```

These accesses may be less efficient than contiguous accesses.

## 5.4 Alignment

Aligned accesses may map more efficiently to memory transactions.

Misalignment can increase transaction count.

## 5.5 Structure of arrays

For some data-parallel workloads, a structure-of-arrays layout allows neighboring lanes to access contiguous values.

## 5.6 Array of structures

An array-of-structures layout groups related fields into records.

This may improve locality when each thread consumes most fields of one record.

The best layout depends on the access pattern.

## 5.7 Data reuse

Data reuse reduces repeated memory traffic.

Common techniques include:

- Register reuse.
- Shared-memory tiling.
- Cache-friendly access patterns.
- Kernel fusion.
- Reusing GPU-resident buffers.

## 5.8 Memory traffic accounting

When estimating traffic, distinguish:

- Logical bytes requested by the program.
- Actual memory transactions.
- Cache hits.
- Cache misses.
- Read and write traffic.
- Intermediate buffer traffic.

These quantities are not always equal.

## Exercises

1. Why does memory bandwidth limit performance?
2. What is coalescing?
3. Why are strided accesses expensive?
4. Why does alignment matter?
5. Compare AoS and SoA.
6. What is data reuse?
7. Why does tiling reduce memory traffic?
8. Why can logical bytes differ from physical memory traffic?

## Lab 3: Memory Access Laboratory

Implement kernels using:

- Contiguous access.
- Strided access.
- Aligned access.
- Misaligned access.
- AoS layout.
- SoA layout.

Measure effective bandwidth.

Explain observed differences.

---

# 6. Module Four: Occupancy and Register Pressure

## 6.1 Occupancy

Occupancy describes the fraction of the hardware's maximum resident warps or wavefronts that are active on an execution unit.

## 6.2 Why occupancy matters

GPUs hide latency by scheduling ready execution groups while others wait.

More resident execution groups may provide more opportunities for latency hiding.

## 6.3 Register pressure

Each thread consumes registers.

High register usage can reduce the number of concurrently resident threads.

## 6.4 Shared-memory pressure

Workgroups consume shared/local memory.

Large workgroup-memory allocations may reduce the number of resident workgroups.

## 6.5 Workgroup size

Workgroup size affects:

- Scheduling.
- Register allocation.
- Shared-memory usage.
- Synchronization.
- Occupancy.

## 6.6 Occupancy is not performance

Maximum occupancy is not necessarily optimal.

A kernel may benefit from more registers per thread even if occupancy decreases.

## 6.7 Register spilling

When register allocation cannot keep all required values in registers, some values may be stored in slower memory.

This can introduce additional memory traffic.

## 6.8 Occupancy tuning

A useful occupancy experiment varies:

- Workgroup size.
- Tile size.
- Register usage.
- Shared-memory usage.

The goal is to identify the best measured performance, not the highest occupancy percentage.

## Exercises

1. Define occupancy.
2. Explain latency hiding.
3. What is register pressure?
4. What is shared-memory pressure?
5. Why does workgroup size affect occupancy?
6. What is register spilling?
7. Why is maximum occupancy not always optimal?
8. Why must occupancy tuning be measured?

## Lab 4: Occupancy Investigation

Choose a compute-heavy kernel.

Vary:

- Workgroup size.
- Per-thread work.
- Tile size.
- Shared-memory allocation.

Measure performance.

Use available profiling tools to investigate resource usage.

---

# 7. Module Five: Instruction-Level Parallelism

## 7.1 What is instruction-level parallelism?

Instruction-level parallelism describes independent instructions that can execute concurrently or overlap in a processor's execution machinery.

## 7.2 Dependency chains

Consider:

\[
a_1=a_0+1
\]

\[
a_2=a_1+1
\]

\[
a_3=a_2+1
\]

Each operation depends on the previous result.

This creates a serial dependency chain.

## 7.3 Independent operations

Now consider:

\[
a=a_0+1
\]

\[
b=b_0+1
\]

\[
c=c_0+1
\]

These operations may execute independently.

## 7.4 GPU instruction scheduling

GPU hardware can exploit instruction-level parallelism within a thread or execution group.

However, the available parallelism depends on the generated machine instructions and hardware execution resources.

## 7.5 Loop unrolling

Loop unrolling can reduce loop-control overhead and expose additional independent operations.

However, it can also increase:

- Code size.
- Register usage.
- Instruction-cache pressure.

## 7.6 Per-thread work

Assigning multiple elements to each thread may improve reuse or reduce overhead.

But excessive per-thread work can reduce occupancy.

## Exercises

1. Define instruction-level parallelism.
2. Explain dependency chains.
3. Why do independent operations matter?
4. What is loop unrolling?
5. Why can loop unrolling improve performance?
6. Why can loop unrolling hurt performance?
7. How does per-thread work affect occupancy?
8. Why does compiler-generated code matter?

## Lab 5: Instruction-Level Parallelism

Implement several variants of a compute-heavy kernel.

Vary:

- Dependency-chain length.
- Independent operations.
- Loop unrolling.
- Work per thread.

Measure performance and investigate generated code where practical.

---

# 8. Module Six: Warp and Wavefront Divergence

## 8.1 Divergence

Divergence occurs when lanes in a warp or wavefront follow different control-flow paths.

## 8.2 Why divergence matters

The hardware may execute different paths using different active-lane masks.

This can reduce effective lane utilization.

## 8.3 Branchless transformations

Some branches can be replaced with arithmetic or selection operations.

However, branchless code is not automatically faster.

It may perform unnecessary work.

## 8.4 Data reorganization

Grouping similar data can reduce divergence.

For example, processing elements with similar conditions together may improve execution coherence.

## 8.5 Irregular workloads

Irregular workloads include:

- Graph traversal.
- Sparse computation.
- Ray tracing.
- Adaptive algorithms.

These workloads often combine divergence with unpredictable memory access.

## Exercises

1. What is divergence?
2. Why does divergence reduce throughput?
3. What is predication?
4. Why can branchless code be slower?
5. How can data reorganization reduce divergence?
6. Why are irregular workloads difficult?
7. How does divergence interact with memory access?

## Lab 6: Divergence Benchmark

Implement kernels with:

- Uniform branches.
- Alternating branches.
- Randomized branches.
- Branchless alternatives.

Measure performance.

Explain how control-flow structure affects execution.

---

# 9. Module Seven: Synchronization and Atomic Contention

## 9.1 Synchronization overhead

Synchronization coordinates concurrent execution.

It can introduce waiting and reduce parallelism.

## 9.2 Workgroup barriers

Workgroup barriers synchronize threads within a workgroup.

Excessive barriers can reduce throughput.

## 9.3 Atomics

Atomic operations provide indivisible updates to supported memory locations.

## 9.4 Atomic contention

If many threads update the same location, operations may serialize or experience significant contention.

## 9.5 Privatization

Privatization reduces contention by allowing threads or workgroups to accumulate partial results independently.

Partial results are later combined.

## 9.6 Histograms

Histograms are a useful case study.

A naive histogram may use global atomics.

An optimized histogram may use workgroup-local accumulation followed by merging.

## 9.7 Synchronization scope

Synchronization must be appropriate for the threads and memory involved.

A workgroup barrier does not generally synchronize all workgroups in a dispatch.

## Exercises

1. Why is synchronization necessary?
2. Why does synchronization introduce overhead?
3. What is atomic contention?
4. What is privatization?
5. Why are histograms challenging?
6. Why can workgroup-local accumulation help?
7. Why does synchronization scope matter?

## Lab 7: Histogram Optimization

Implement:

- Global atomic histogram.
- Workgroup-local histogram.
- Hierarchical histogram.

Compare performance under different input distributions.

Investigate contention.

---

# 10. Module Eight: Kernel Fusion

## 10.1 What is kernel fusion?

Kernel fusion combines multiple GPU operations into a single kernel.

## 10.2 Why fusion helps

Fusion may reduce:

- Kernel launch overhead.
- Intermediate memory traffic.
- Synchronization between kernels.

## 10.3 Example

Suppose a workload computes:

\[
B_i=2A_i
\]

followed by:

\[
C_i=B_i+1
\]

A two-kernel implementation may write B to global memory and later read it.

A fused implementation may compute C directly.

## 10.4 Fusion trade-offs

Fusion can increase:

- Register pressure.
- Code complexity.
- Resource usage.
- Divergence.
- Compilation cost.

## 10.5 Fusion and arithmetic intensity

Fusion may increase arithmetic intensity by reducing intermediate memory traffic.

## Exercises

1. What is kernel fusion?
2. Why can fusion reduce memory traffic?
3. Why can fusion reduce launch overhead?
4. Why can fusion increase register pressure?
5. Why is fusion not always beneficial?
6. How does fusion affect arithmetic intensity?

## Lab 8: Kernel Fusion

Implement a multi-stage array-processing pipeline.

Compare:

- Separate kernels.
- Fused kernels.
- Different fusion boundaries.

Measure kernel time and end-to-end time.

---

# 11. Module Nine: Asynchronous Execution and Overlap

## 11.1 GPU execution is asynchronous

CPU and GPU work can overlap.

However, synchronization and resource dependencies constrain overlap.

## 11.2 Data transfers

Host-device transfers can be expensive.

Reducing transfers may improve end-to-end performance.

## 11.3 Overlap

Some systems can overlap:

- CPU work.
- GPU execution.
- Data transfers.

Actual overlap depends on hardware and API capabilities.

## 11.4 Double buffering

Double buffering uses separate buffers to allow different pipeline stages to operate concurrently.

## 11.5 Pipeline throughput

For a pipeline with independent stages, steady-state throughput may be limited by the slowest stage.

However, dependencies and resource contention can prevent ideal overlap.

## Exercises

1. Why is asynchronous execution useful?
2. Why can data transfers dominate?
3. What is overlap?
4. What is double buffering?
5. Why does overlap depend on hardware capabilities?
6. Why can synchronization prevent overlap?
7. Why must end-to-end throughput be measured?

## Lab 9: Asynchronous Pipeline

Build a GPU data-processing pipeline.

Compare:

- Sequential execution.
- Batched execution.
- Overlapped execution where supported.

Measure throughput and latency.

---

# 12. Module Ten: Autotuning

## 12.1 What is autotuning?

Autotuning systematically explores implementation parameters to identify configurations that perform well on a target device.

## 12.2 Common parameters

Parameters may include:

- Workgroup size.
- Tile size.
- Work per thread.
- Unroll factor.
- Shared-memory usage.
- Algorithm variant.

## 12.3 Search spaces

A search space contains candidate parameter combinations.

Exhaustive search may be expensive.

Practical autotuners may use structured or heuristic search strategies.

## 12.4 Correctness constraints

Every candidate must satisfy correctness requirements.

Invalid configurations must not be treated as performance improvements.

## 12.5 Overfitting

A configuration optimized for one input size may perform poorly on another.

Autotuning should use representative workloads.

## 12.6 Reproducibility

Autotuning results should record:

- Device.
- Driver.
- Compiler.
- Input size.
- Parameter values.
- Timing methodology.
- Correctness results.

## Exercises

1. What is autotuning?
2. What is a parameter search space?
3. Why is exhaustive search expensive?
4. Why must candidates be validated?
5. What is benchmark overfitting?
6. Why must tuning results be reproducible?
7. Why may different GPUs prefer different parameters?

## Lab 10: GPU Autotuner

Build an autotuner for one of your kernels.

Requirements:

- Parameterized kernel variants.
- Correctness validation.
- Automated benchmarking.
- Search-space exploration.
- Result recording.
- Best-configuration selection.
- Reproducibility.

---

# 13. Major Assessment A: GPU Memory Optimization Suite

Build a suite of GPU kernels demonstrating:

1. Coalesced memory access.
2. Strided memory access.
3. Aligned and misaligned access.
4. AoS and SoA layouts.
5. Shared-memory tiling.
6. Data reuse.

## Required report

Explain:

- Memory traffic.
- Effective bandwidth.
- Cache behavior.
- Arithmetic intensity.
- Performance differences.

---

# 14. Major Assessment B: GPU Compute Optimization Suite

Build a suite demonstrating:

1. Workgroup-size tuning.
2. Register-pressure trade-offs.
3. Loop unrolling.
4. Instruction-level parallelism.
5. Divergence.
6. Kernel fusion.

## Required report

Explain:

- Occupancy.
- Register usage.
- Instruction dependencies.
- Divergence.
- Synchronization.
- Performance trade-offs.

---

# 15. Major Assessment C: GPU Autotuning Framework

Build a reusable GPU autotuning framework.

## Required features

- Parameter search.
- Correctness validation.
- Automated benchmarking.
- Structured results.
- Device metadata.
- Reproducible experiments.
- Performance regression detection.

## Technical defense

Explain:

- Search strategy.
- Benchmark methodology.
- Correctness criteria.
- Statistical uncertainty.
- Hardware-specific tuning.
- Limitations.

---

# 16. Final Capstone: GPU Performance Engineering Laboratory

Build a comprehensive GPU performance laboratory.

## Required kernels

1. Vector addition.
2. Reduction.
3. Prefix scan.
4. Histogram.
5. Matrix multiplication.
6. Fused array-processing pipeline.

## Required optimization dimensions

- Workgroup size.
- Memory access pattern.
- Data reuse.
- Shared-memory usage.
- Register pressure.
- Synchronization.
- Divergence.
- Kernel fusion.

## Required reports

### Report 1: Baselines

Document original implementations and performance.

### Report 2: Performance models

Calculate arithmetic intensity and theoretical bounds.

### Report 3: Optimization experiments

Document hypotheses, changes, and measurements.

### Report 4: Final results

Compare optimized implementations against baselines.

### Report 5: Architectural analysis

Explain how GPU hardware characteristics influenced performance.

### Report 6: Reproducibility

Document the environment and procedures required to reproduce the results.

## Required technical defense

You must explain:

- Why each optimization was attempted.
- What bottleneck it targeted.
- What evidence supported the hypothesis.
- Whether the optimization succeeded.
- Why some optimizations failed.
- Whether improvements generalize to other inputs.

---

# 17. Phase 10 Quiz Bank

## Benchmarking

1. Why are GPU benchmarks noisy?
2. Why is warm-up important?
3. What is kernel execution time?
4. What is end-to-end time?
5. Why are repeated measurements necessary?
6. What is a controlled experiment?
7. Why must correctness be validated?
8. Why should benchmark environments be documented?

## Performance modeling

9. What is arithmetic intensity?
10. What is the Roofline model?
11. What is the ridge point?
12. What is memory-bound execution?
13. What is compute-bound execution?
14. Why is vector addition often memory-bound?
15. Why can matrix multiplication be compute-bound?
16. What are the limitations of the Roofline model?

## Memory

17. What is coalescing?
18. Why are strided accesses expensive?
19. Why does alignment matter?
20. What is AoS?
21. What is SoA?
22. What is data reuse?
23. Why does tiling improve performance?
24. Why can logical bytes differ from physical memory traffic?

## Occupancy

25. What is occupancy?
26. What is latency hiding?
27. What is register pressure?
28. What is shared-memory pressure?
29. What is register spilling?
30. Why does workgroup size affect occupancy?
31. Why is maximum occupancy not always optimal?
32. Why must occupancy tuning be measured?

## Instruction-level parallelism

33. What is instruction-level parallelism?
34. What is a dependency chain?
35. Why do independent operations matter?
36. What is loop unrolling?
37. Why can unrolling improve performance?
38. Why can unrolling hurt performance?
39. How does per-thread work affect occupancy?
40. Why does compiler-generated code matter?

## Divergence

41. What is divergence?
42. Why does divergence reduce throughput?
43. What is predication?
44. Why can branchless code be slower?
45. How can data reorganization reduce divergence?
46. Why are irregular workloads difficult?

## Synchronization

47. Why is synchronization necessary?
48. Why does synchronization introduce overhead?
49. What is atomic contention?
50. What is privatization?
51. Why are histograms challenging?
52. Why does synchronization scope matter?

## Kernel fusion

53. What is kernel fusion?
54. Why can fusion reduce memory traffic?
55. Why can fusion reduce launch overhead?
56. Why can fusion increase register pressure?
57. Why is fusion not always beneficial?
58. How does fusion affect arithmetic intensity?

## Asynchronous execution

59. Why is asynchronous execution useful?
60. What is double buffering?
61. What is pipeline overlap?
62. Why can synchronization prevent overlap?
63. Why must end-to-end throughput be measured?

## Autotuning

64. What is autotuning?
65. What is a parameter search space?
66. Why must candidates be validated?
67. What is benchmark overfitting?
68. Why may different GPUs prefer different configurations?
69. Why must tuning results be reproducible?
70. What is performance regression testing?

---

# 18. Final Examination

**Recommended duration:** 6–8 hours.

| Section | Weight |
|---|---:|
| Benchmarking methodology | 15% |
| Performance modeling | 15% |
| Memory optimization | 20% |
| Occupancy and register pressure | 15% |
| Instruction-level parallelism and divergence | 10% |
| Synchronization and fusion | 10% |
| Autotuning and reproducibility | 15% |

## Part A: Benchmark design

Design a reproducible GPU benchmark.

## Part B: Performance modeling

Calculate arithmetic intensity and Roofline bounds.

## Part C: Memory optimization

Analyze memory access patterns and data reuse.

## Part D: Occupancy

Analyze resource constraints and expected occupancy trade-offs.

## Part E: Divergence

Analyze control-flow behavior.

## Part F: Kernel fusion

Evaluate hypothetical fusion opportunities.

## Part G: Autotuning

Design a parameter search and validation strategy.

## Part H: Technical defense

Defend your GPU performance laboratory.

---

# 19. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

The tutor must never provide answers, code, pseudocode, implementations, debugging fixes, or completed work.

## Master Socratic Tutor Prompt

```text id="u6w4q2"
You are my Socratic GPU performance engineering tutor.

I am studying Phase 10 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   derivations, or examinations.

2. Never write code, pseudocode, algorithms, or implementations.

3. Never modify files, execute code, debug programs, or complete
   assessment work on my behalf.

4. Never reveal answer keys, hidden tests, or completed solutions.

5. Never give hints that effectively disclose solutions.

6. Guide my reasoning exclusively through questions.

7. Ask exactly one guiding question per response.

8. If I am incorrect, ask a question that exposes the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to explain performance models, memory traffic,
    occupancy, register pressure, divergence, synchronization,
    and benchmarking methodology.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness,
    bottlenecks, assumptions, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only guiding questions.
```

## Start a lesson

```text id="j9k7m3"
Tutor me through Phase 10, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to explain GPU performance trade-offs.
Never provide answers or solutions.
```

## Generate a quiz

```text id="v3n8r5"
Administer a 20-question Phase 10 quiz on [TOPIC].

Include conceptual reasoning, performance modeling,
memory optimization, occupancy, and benchmarking.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text id="x5d2q8"
Administer Phase 10 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my performance hypothesis
and correctness argument.

Guide me through questions about measurement,
memory traffic, occupancy, synchronization,
and optimization trade-offs.

Do not write code, pseudocode, or implementations.
```

## Review GPU performance code

```text id="p7m4z2"
Review my GPU performance code using only questions.

Focus on:
- Correctness
- Memory access patterns
- Arithmetic intensity
- Occupancy
- Register pressure
- Divergence
- Synchronization
- Benchmark methodology

Do not provide fixes or replacement code.
Ask one guiding question at a time.
```

## Performance debugging examination

```text id="h2c9w6"
Help me investigate a slow GPU kernel using only questions.

Ask me to examine:
- Measured runtime
- Memory bandwidth
- Arithmetic intensity
- Occupancy
- Register pressure
- Divergence
- Synchronization
- Launch overhead
- Data transfers

Never identify the exact optimization.
Never write code.
```

## Autotuning examination

```text id="n8v5k1"
Examine my GPU autotuning design.

Ask questions about:
- Search spaces
- Parameter selection
- Correctness validation
- Benchmark noise
- Reproducibility
- Overfitting
- Performance regressions

Ask one question at a time.
Never provide solutions.
```

---

# 20. Mastery Checklist

## Benchmarking

- [ ] I can design reproducible GPU benchmarks.
- [ ] I understand warm-up effects.
- [ ] I can measure kernel time.
- [ ] I can measure end-to-end time.
- [ ] I can report timing distributions.
- [ ] I can design controlled experiments.

## Performance modeling

- [ ] I can calculate arithmetic intensity.
- [ ] I understand the Roofline model.
- [ ] I can calculate the ridge point.
- [ ] I can identify memory-bound workloads.
- [ ] I can identify compute-bound workloads.
- [ ] I understand model limitations.

## Memory optimization

- [ ] I understand coalescing.
- [ ] I understand alignment.
- [ ] I understand AoS and SoA.
- [ ] I understand data reuse.
- [ ] I understand tiling.
- [ ] I can estimate memory traffic.

## Occupancy

- [ ] I understand occupancy.
- [ ] I understand register pressure.
- [ ] I understand shared-memory pressure.
- [ ] I understand register spilling.
- [ ] I understand occupancy trade-offs.

## Instruction execution

- [ ] I understand instruction-level parallelism.
- [ ] I understand dependency chains.
- [ ] I understand loop unrolling.
- [ ] I understand divergence.
- [ ] I understand predication.

## Synchronization

- [ ] I understand synchronization overhead.
- [ ] I understand atomic contention.
- [ ] I understand privatization.
- [ ] I understand synchronization scope.

## Optimization

- [ ] I understand kernel fusion.
- [ ] I understand asynchronous execution.
- [ ] I understand double buffering.
- [ ] I can design an autotuner.
- [ ] I can detect performance regressions.

## Capstone

- [ ] My GPU performance laboratory works.
- [ ] My results are reproducible.
- [ ] My optimizations are validated.
- [ ] My reports explain performance trade-offs.
- [ ] I can defend my conclusions.

---

# 21. Recommended Resources

## GPU performance

**AMD GPUOpen**

https://gpuopen.com/

**AMD GPU Performance Engineering Resources**

https://gpuopen.com/performance/

**NVIDIA CUDA Best Practices Guide**

https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/

## Profiling

**Nsight Compute**

https://developer.nvidia.com/nsight-compute

**Nsight Systems**

https://developer.nvidia.com/nsight-systems

**AMD Radeon GPU Profiler**

https://gpuopen.com/rgp/

## Research

**Roofline: An Insightful Visual Performance Model for Multicore Architectures**

A foundational paper on performance modeling.

**Programming Massively Parallel Processors — Kirk and Hwu**

A foundational GPU programming textbook.

## Numerical performance

**Accuracy and Stability of Numerical Algorithms — Nicholas J. Higham**

Useful for understanding numerical correctness during optimization.

---

# 22. Completion Standard

You have completed Phase 10 when you can independently:

1. Benchmark GPU kernels reproducibly.
2. Calculate arithmetic intensity.
3. Apply the Roofline model.
4. Diagnose memory bandwidth limitations.
5. Analyze memory access patterns.
6. Explain occupancy trade-offs.
7. Explain register pressure.
8. Analyze divergence.
9. Evaluate synchronization overhead.
10. Apply kernel fusion appropriately.
11. Build an autotuner.
12. Detect performance regressions.
13. Explain why optimizations succeed or fail.
14. Defend performance claims using measurements.

The most important skill is the ability to form and test performance hypotheses.

---

# 23. What Comes Next

**Phase 11: Advanced Parallel Algorithms**

The next phase will study:

- Advanced parallel scans.
- Segmented scans.
- Radix sorting.
- Parallel graph algorithms.
- Sparse matrix operations.
- Parallel hash tables.
- Stream compaction.
- GPU work queues.
- Load balancing.
- Irregular parallelism.
- Graph traversal.
- Parallel dynamic programming.
- Advanced synchronization patterns.
- GPU algorithm design.

You will move from optimizing individual kernels to designing sophisticated parallel algorithms and data structures.
