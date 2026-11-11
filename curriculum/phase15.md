# Phase 15: GPU Engineering Specialization and Final Capstone

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 15 of 16 — Final Phase  
**Duration:** 24–36 weeks  
**Study commitment:** 10–12 hours per week  
**Total estimated effort:** 240–432 hours  
**Primary language:** Rust  
**Supporting languages:** C++20, WGSL, SPIR-V, LLVM IR  
**Primary environment:** Fedora Linux, AMD Radeon RX 9070 16 GB  
**Prerequisites:** Phases 0–14

---

# 1. Purpose of the Final Phase

Over the previous fifteen phases, you studied:

- Mathematical and computational foundations.
- Rust systems programming.
- Modern C++.
- Computer architecture.
- Operating systems.
- Numerical computing.
- Concurrency and parallel algorithms.
- GPU architecture.
- Portable GPU programming.
- CUDA.
- GPU optimization.
- Advanced parallel algorithms.
- Native Rust GPU programming.
- GPU runtime systems.
- GPU compilers.

These topics are deeply interconnected.

A kernel's performance depends on architecture, numerical behavior, memory access, and scheduling. A runtime's correctness depends on resource ownership, asynchronous execution, and synchronization. A compiler's optimization opportunities depend on data flow, aliasing, execution semantics, and hardware constraints.

The purpose of Phase 15 is to integrate these disciplines into one substantial engineering project.

## 1.1 What this phase is not

This phase is not:

- Another collection of unrelated tutorials.
- A sequence of isolated toy kernels.
- A project that simply wraps an existing library.
- An exercise in maximizing benchmark numbers without validating correctness.
- A requirement to invent an entirely new GPU architecture or compiler.

The project must have enough depth to require genuine systems-level decisions.

## 1.2 What this phase is

This phase is an extended engineering practicum.

You will:

1. Choose a technical specialization.
2. Identify a meaningful engineering problem.
3. Research existing approaches.
4. Define functional and nonfunctional requirements.
5. Establish baselines.
6. Design an architecture.
7. Implement a working system.
8. Test its correctness.
9. Investigate performance.
10. Improve the design.
11. Document trade-offs.
12. Present and defend the result.

## 1.3 The final standard

Your work should demonstrate that you can independently answer four questions:

**Does it work?**

You must provide correctness evidence.

**Why does it work?**

You must explain the underlying algorithms, semantics, and invariants.

**How well does it work?**

You must measure performance with a reproducible methodology.

**Why was it designed this way?**

You must justify architectural decisions and alternatives.

---

# 2. Specialization Tracks

Choose one primary specialization.

You may incorporate ideas from other tracks, but your project must have a coherent technical focus.

## Track A: GPU Runtime and Systems Engineering

Build infrastructure for scheduling and executing GPU workloads.

Potential project:

**Asynchronous Rust GPU Task Graph Runtime**

Core responsibilities:

- Task graph representation.
- Dependency analysis.
- GPU memory management.
- Buffer pooling.
- Command recording.
- Asynchronous submission.
- Synchronization.
- Resource lifetime tracking.
- Profiling.

Research questions might include:

- How much CPU overhead can task batching eliminate?
- How does buffer pooling affect latency?
- When does automatic synchronization reduce application complexity?
- How does graph scheduling affect memory pressure?
- Can resource lifetime analysis reduce peak allocation size?

This track builds directly on Phase 13.

### Required depth

A strong runtime project should demonstrate at least:

- One substantial memory-management subsystem.
- One dependency or scheduling subsystem.
- One asynchronous execution subsystem.
- Multiple real GPU workloads.
- Explicit correctness invariants.
- Quantitative performance comparisons.

---

## Track B: GPU Compiler Engineering

Build a compiler or optimization framework for GPU programs.

Potential project:

**Rust GPU Compute Compiler with Optimization and Autotuning**

Core responsibilities:

- Language front end.
- Typed intermediate representation.
- Control-flow analysis.
- SSA.
- Optimization passes.
- GPU-specific lowering.
- Shader generation.
- Validation.
- Performance measurement.

Research questions:

- Which optimization passes improve generated GPU kernels?
- How do workgroup-size choices interact with compiler transformations?
- Can a simple cost model predict profitable fusion?
- How does loop unrolling affect register pressure?
- Which source-level patterns lead to poor GPU code generation?

This track builds directly on Phase 14.

### Required depth

A strong compiler project should demonstrate:

- A well-defined source language or input IR.
- Multiple analyses or transformations.
- An executable backend.
- Differential correctness testing.
- A performance study.
- Clear transformation legality conditions.

---

## Track C: High-Performance GPU Algorithms

Implement and optimize a family of related GPU algorithms.

Potential project:

**High-Performance Parallel Primitives Library**

Algorithms might include:

- Reductions.
- Prefix scans.
- Histograms.
- Sorting.
- Stream compaction.
- Segmented reductions.
- Sparse operations.

Research questions:

- How do different workgroup sizes affect throughput?
- When is hierarchical decomposition necessary?
- How does data distribution affect performance?
- Which algorithms are memory-bound?
- What is the cost of synchronization?
- When does CPU execution outperform GPU execution?

### Required depth

A strong algorithm project should contain:

- Several related algorithms.
- Multiple implementation variants.
- Mathematical correctness arguments.
- Edge-case testing.
- Performance models.
- Hardware-aware optimization.

---

## Track D: Numerical and Scientific GPU Computing

Build a numerical computation system.

Potential project:

**GPU-Accelerated Scientific Simulation Engine**

Possible domains:

- Particle simulation.
- Fluid simulation.
- Heat diffusion.
- Numerical integration.
- Iterative linear solvers.
- Finite-difference methods.

Research questions:

- How does precision affect convergence?
- Which data layouts improve memory throughput?
- How does numerical stability change under parallel execution?
- When does computation become bandwidth-bound?
- How does scaling vary with problem size?

### Required depth

A strong numerical project should include:

- Mathematical formulation.
- Discretization or numerical algorithm.
- Error analysis.
- CPU reference implementation.
- GPU implementation.
- Stability investigation.
- Performance investigation.

---

## Track E: Machine Learning Systems

Build a GPU execution component used in machine learning.

Potential project:

**Minimal GPU Tensor Execution Engine**

Possible features:

- Tensor storage.
- Elementwise operations.
- Reductions.
- Matrix multiplication.
- Operation graphs.
- Memory reuse.
- Kernel fusion.
- Automatic differentiation as an advanced extension.

Research questions:

- How does kernel fusion affect memory traffic?
- When does batching improve throughput?
- How does tensor shape affect performance?
- What are the costs of intermediate allocations?
- Which operations dominate execution time?

### Required depth

A strong ML systems project should focus on the execution infrastructure rather than merely training a model with an existing framework.

---

# 3. Choosing Your Specialization

You should choose based on the kind of engineering work you want to pursue.

| Primary interest | Recommended track |
|---|---|
| GPU drivers, execution systems, memory management | A — Runtime |
| Compilers, programming languages, code generation | B — Compiler |
| Kernel optimization, HPC, performance engineering | C — Algorithms |
| Scientific computing, simulation, numerical methods | D — Numerical |
| ML infrastructure, tensor compilers, inference engines | E — ML Systems |

## 3.1 Recommended project for your curriculum

A particularly strong integrative choice is:

**A Rust GPU Compute Graph Engine with Automatic Memory Management and Kernel Optimization**

This combines:

- Rust systems engineering.
- Vulkan execution.
- GPU memory allocation.
- Task graphs.
- Synchronization.
- Parallel algorithms.
- Compiler optimization.
- Benchmarking.

The project could accept a computation graph, select kernels, schedule operations, reuse temporary buffers, and measure execution.

It is broad enough to demonstrate systems expertise while still allowing a carefully constrained implementation.

However, it must be scoped realistically.

A well-tested graph executor supporting five operations is more valuable than an unfinished framework claiming to support fifty.

---

# 4. Capstone Project Selection Criteria

Before beginning implementation, evaluate candidate projects against six criteria.

## 4.1 Technical depth

The project should require substantial reasoning about at least three of:

- Memory management.
- Concurrency.
- GPU architecture.
- Numerical correctness.
- Algorithms.
- Scheduling.
- Compilation.
- Synchronization.
- Performance.

## 4.2 Measurable outcomes

The project must support objective evaluation.

Examples:

- Execution latency.
- Throughput.
- Memory consumption.
- Compilation time.
- Kernel launch overhead.
- Numerical error.
- Resource utilization.

## 4.3 Correctness oracle

You need a way to determine whether the system is correct.

Possible methods:

- CPU reference implementations.
- Mathematical invariants.
- Differential testing.
- Known analytical solutions.
- Independent implementations.
- Property-based testing.

## 4.4 Feasibility

The project must fit your hardware and time budget.

For your Radeon RX 9070, prioritize workloads supported through Vulkan and Rust tooling.

Do not make native CUDA execution a prerequisite.

## 4.5 Reproducibility

Another engineer should be able to:

- Build the project.
- Run tests.
- Execute benchmarks.
- Inspect results.
- Understand environment requirements.

## 4.6 Portfolio value

The project should demonstrate transferable engineering skills.

A reviewer should be able to understand:

- The problem.
- The architecture.
- The technical challenges.
- The results.
- Your individual contributions.

---

# 5. Project Proposal

Before writing production code, prepare a formal proposal.

## Required proposal sections

### 5.1 Problem statement

Describe the engineering problem in precise terms.

Avoid vague goals such as:

"Make GPU programs faster."

Prefer measurable goals such as:

"Investigate whether automatic temporary-buffer reuse reduces peak GPU memory consumption for multi-stage compute graphs."

### 5.2 Motivation

Explain why the problem matters.

### 5.3 Existing approaches

Research relevant libraries, frameworks, or academic work.

### 5.4 Scope

Define what the project will and will not support.

### 5.5 Functional requirements

Describe required behavior.

### 5.6 Nonfunctional requirements

Describe:

- Performance.
- Reliability.
- Portability.
- Maintainability.
- Memory limits.
- Testing requirements.

### 5.7 Success criteria

Define measurable completion conditions.

### 5.8 Risks

Identify technical risks and mitigation strategies.

### 5.9 Timeline

Break work into milestones.

### 5.10 Evaluation plan

Specify correctness and performance experiments before implementation.

---

# 6. Research and Literature Review

Engineering decisions should be informed by existing work.

## 6.1 Sources

Use a combination of:

- Official specifications.
- Technical documentation.
- Research papers.
- Open-source implementations.
- Compiler source code.
- Performance studies.
- Hardware documentation.

## 6.2 Evaluate sources critically

For each source, identify:

1. The problem addressed.
2. The proposed method.
3. The assumptions.
4. The hardware used.
5. The evaluation methodology.
6. The limitations.
7. Its relevance to your project.

## 6.3 Avoid superficial comparisons

Two GPU systems may differ in:

- Hardware.
- Drivers.
- Input sizes.
- Precision.
- Synchronization.
- Memory residency.
- Benchmark methodology.

A performance number without context is not a reliable basis for comparison.

## 6.4 Literature review deliverable

Produce a technical review containing:

- At least five substantial sources.
- A comparison of approaches.
- A discussion of limitations.
- A statement of your project's contribution.

Your contribution does not need to be academically novel.

A rigorous implementation and evaluation of existing ideas is appropriate.

---

# 7. Architecture and Design

Before implementation, define the system's major components.

## 7.1 Architecture diagram

For a GPU graph engine:

```text id="k9e2sp"
          User Application
                 |
                 v
           Graph Builder
                 |
                 v
          Graph Validator
                 |
                 v
         Dependency Analyzer
                 |
                 v
        Optimization Planner
                 |
                 v
             Scheduler
                 |
        +--------+--------+
        |                 |
        v                 v
   Memory Manager    Pipeline Manager
        |                 |
        +--------+--------+
                 |
                 v
          Vulkan Backend
                 |
                 v
             AMD GPU
```

## 7.2 Define invariants

Examples:

- Every resource reference identifies a valid resource.
- Every task dependency references a valid task.
- The graph contains no cycles.
- Every buffer access is within bounds.
- No resource is reclaimed before its final GPU use completes.
- Every required memory dependency is enforced.
- Failed operations do not silently produce successful completion handles.

## 7.3 Define ownership

For each resource, identify:

- Creator.
- Owner.
- Borrowers.
- GPU users.
- Destruction conditions.

## 7.4 Define failure behavior

What happens when:

- Memory allocation fails?
- Shader compilation fails?
- A task graph is invalid?
- Submission fails?
- A wait times out?
- The device is lost?

## 7.5 Define observability

Decide what must be measurable.

Examples:

- CPU graph construction time.
- Scheduling overhead.
- GPU execution time.
- Peak memory usage.
- Buffer pool hit rate.
- Compilation time.

---

# 8. Engineering Foundations: Correctness Contracts

The most important architectural improvement you can make is to define correctness before optimization.

## 8.1 Functional correctness

For each operation, specify:

- Input types.
- Input shapes.
- Output types.
- Output shapes.
- Valid parameter ranges.
- Error conditions.
- Numerical behavior.

## 8.2 Numerical correctness

Floating-point operations are not exact real arithmetic.

For example:

\[
(a+b)+c
\]

may differ from:

\[
a+(b+c)
\]

because intermediate results are rounded.

Define a numerical comparison policy.

One common criterion is:

\[
|x-y|\leq \epsilon_{\text{abs}}+
\epsilon_{\text{rel}}|y|
\]

where \(y\) is the reference value.

This criterion must be adapted to the operation and magnitude of expected results.

## 8.3 Memory safety

GPU execution introduces asynchronous memory lifetimes.

A buffer must remain valid while any outstanding GPU operation may access it.

## 8.4 Synchronization correctness

For every conflicting access pair, identify:

- Producer.
- Consumer.
- Memory region.
- Required ordering.
- Required visibility.
- Synchronization mechanism.

## 8.5 Compiler correctness

If your project transforms programs, define the behavior preserved by each transformation.

For example, kernel fusion must preserve:

- Data dependencies.
- Observable memory effects.
- Required synchronization.
- Numerical semantics under the chosen contract.

## 8.6 Testable invariants

Translate important invariants into automated tests wherever possible.

---

# 9. Performance Engineering Methodology

Performance claims must be supported by controlled experiments.

## 9.1 Establish a baseline

A baseline is the reference against which improvements are measured.

Possible baselines:

- Sequential CPU implementation.
- Multithreaded CPU implementation.
- Simple GPU implementation.
- Existing library.
- Unoptimized compiler output.
- Manual Vulkan execution.

Choose a baseline relevant to the research question.

## 9.2 Speedup

Define speedup as:

\[
S=\frac{T_{\text{baseline}}}{T_{\text{candidate}}}
\]

If:

\[
T_{\text{baseline}}=20\text{ ms}
\]

and:

\[
T_{\text{candidate}}=5\text{ ms}
\]

then:

\[
S=4
\]

The candidate is four times as fast for the measured workload.

## 9.3 Throughput

Throughput measures work completed per unit time.

For processing \(N\) elements in time \(T\):

\[
R=\frac NT
\]

## 9.4 Effective memory bandwidth

If an operation transfers approximately \(B\) bytes and takes time \(T\):

\[
BW_{\text{effective}}=\frac BT
\]

The byte count must reflect the traffic model used.

## 9.5 Arithmetic intensity

\[
I=\frac{\text{Floating-point operations}}{\text{Bytes transferred}}
\]

Arithmetic intensity helps distinguish memory-bound from compute-bound workloads.

## 9.6 Roofline model

A simplified performance bound is:

\[
P\leq\min(P_{\text{peak}},BW_{\text{peak}}I)
\]

where:

- \(P\) is achieved performance.
- \(P_{\text{peak}}\) is peak compute throughput.
- \(BW_{\text{peak}}\) is peak memory bandwidth.
- \(I\) is arithmetic intensity.

This model provides a useful upper bound, not an exact prediction.

## 9.7 Benchmark warmup

Warmup may account for:

- Pipeline creation.
- Shader compilation.
- Cache initialization.
- Memory allocation.
- GPU clock changes.

Separate initialization costs from steady-state execution unless end-to-end latency is the intended metric.

## 9.8 Repeated measurements

Use multiple trials.

Report:

- Median.
- Minimum.
- Maximum.
- Standard deviation or another dispersion measure.
- Number of trials.

## 9.9 Experimental controls

Record:

- GPU model.
- Driver version.
- Kernel version.
- Compiler version.
- Optimization flags.
- Input sizes.
- Workgroup sizes.
- Precision.
- Timing method.

## 9.10 Avoid misleading conclusions

Do not claim an optimization is universally better based on one input size.

Investigate performance across relevant workloads.

---

# 10. Worked Example: Evaluating Kernel Fusion

Suppose a computation performs:

\[
y_i=2x_i
\]

followed by:

\[
z_i=y_i+3
\]

Assume all arrays contain \(N\) 32-bit floating-point values.

## 10.1 Unfused execution

The first kernel:

- Reads \(x_i\).
- Writes \(y_i\).

The second kernel:

- Reads \(y_i\).
- Writes \(z_i\).

Ignoring cache effects and other traffic, the total logical memory traffic is approximately:

\[
4N+4N+4N+4N=16N\text{ bytes}
\]

## 10.2 Fused execution

A fused kernel computes:

\[
z_i=2x_i+3
\]

It:

- Reads \(x_i\).
- Writes \(z_i\).

The logical traffic is approximately:

\[
4N+4N=8N\text{ bytes}
\]

## 10.3 Theoretical traffic reduction

The ratio is:

\[
\frac{16N}{8N}=2
\]

The fused kernel halves the modeled logical memory traffic.

## 10.4 Why speedup may differ

The actual speedup is not guaranteed to be two.

Reasons include:

- Cache behavior.
- Launch overhead.
- Arithmetic throughput.
- Memory bandwidth.
- Register pressure.
- GPU occupancy.
- Compiler optimization.
- Input size.

## 10.5 Experimental design

Compare fused and unfused implementations for several input sizes.

Measure:

- Correctness.
- GPU execution time.
- End-to-end latency.
- Estimated memory traffic.
- Resource usage where available.

## 10.6 Interpretation

For small inputs, reduced launch overhead may dominate.

For large memory-bound inputs, reduced memory traffic may dominate.

For more complex fused kernels, register pressure may offset the benefits.

This is the kind of reasoning expected in the final capstone.

---

# 11. Testing Strategy

A capstone is incomplete without a rigorous validation system.

## 11.1 Unit tests

Test individual modules.

Examples:

- Buffer allocator.
- Dependency analyzer.
- Graph validator.
- Shader generator.
- Numerical operation.
- Scheduler.

## 11.2 Integration tests

Test interactions between components.

For example:

- Graph creation.
- Buffer allocation.
- GPU execution.
- Synchronization.
- Readback.
- Resource reclamation.

## 11.3 Differential tests

Compare against independent reference implementations.

## 11.4 Property-based tests

Generate random valid inputs and check invariants.

Examples:

- Prefix scan output satisfies the prefix-sum definition.
- Reduction agrees with the reference under the numerical contract.
- Allocator regions do not overlap incompatibly.
- Task graphs respect dependencies.
- Compiler transformations preserve defined behavior.

## 11.5 Stress tests

Use:

- Large workloads.
- Small workloads.
- Odd sizes.
- Zero-length cases where supported.
- Repeated execution.
- Allocation churn.
- Many concurrent tasks.
- Delayed completion.
- Resource exhaustion.

## 11.6 Failure injection

Simulate or induce failures where practical.

Examples:

- Allocation failure.
- Invalid shader.
- Unsupported feature.
- Invalid graph.
- Timeout.
- Device loss simulation.

## 11.7 Regression tests

Every discovered bug should produce a reproducible regression test.

---

# 12. Reliability and Systems Engineering

Performance is not the only quality attribute.

## 12.1 Error handling

Errors should identify:

- The failed operation.
- The relevant resource or task.
- The underlying cause.
- Whether the runtime remains usable.

## 12.2 Resource cleanup

Ensure resources are released safely.

Asynchronous GPU usage may require deferred destruction.

## 12.3 Device loss

Define how the system behaves if the GPU device becomes unusable.

## 12.4 Timeouts

A timeout does not imply GPU execution has stopped.

Avoid reclaiming resources merely because a wait timed out.

## 12.5 Configuration validation

Reject unsupported configurations before execution.

## 12.6 Reproducible builds

Record dependency versions and build configuration.

For Rust, retain the project's lockfile where appropriate.

## 12.7 Unsafe Rust

If unsafe Rust is necessary, document:

- Preconditions.
- Invariants.
- Lifetime assumptions.
- Synchronization assumptions.
- Failure behavior.

Minimize unsafe code and isolate it behind carefully designed interfaces.

---

# 13. Project Management and Milestones

Treat the capstone as a real engineering project.

## 13.1 Milestone structure

Each milestone should have:

- Scope.
- Deliverables.
- Acceptance criteria.
- Tests.
- Risks.
- Estimated effort.

## 13.2 Recommended 30-week plan

| Weeks | Focus | Deliverable |
|---|---|---|
| 1–2 | Specialization and problem selection | Project proposal |
| 3–4 | Research and requirements | Literature review |
| 5–6 | Architecture and correctness contracts | Design document |
| 7–10 | Minimal working system | End-to-end prototype |
| 11–14 | Core implementation | Functional subsystem |
| 15–17 | Correctness and reliability | Validation suite |
| 18–21 | Performance investigation | Baseline and profiling |
| 22–25 | Optimization and refinement | Improved implementation |
| 26–28 | Documentation and reproducibility | Engineering report |
| 29–30 | Final defense and review | Capstone presentation |

## 13.3 Weekly engineering log

Maintain a log containing:

- Goals.
- Work completed.
- Problems encountered.
- Experiments.
- Measurements.
- Decisions.
- Next steps.

## 13.4 Decision records

For significant architectural choices, document:

1. Context.
2. Alternatives.
3. Decision.
4. Rationale.
5. Consequences.

## 13.5 Scope management

Separate features into:

- Required.
- Optional.
- Stretch goals.

Do not allow optional features to prevent completion of the core system.

---

# 14. Recommended Repository Structure

A representative Rust workspace might contain:

```text id="0kq9ra"
gpu-capstone/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── LICENSE
├── crates/
│   ├── core/
│   ├── runtime/
│   ├── backend/
│   ├── algorithms/
│   └── profiling/
├── shaders/
├── tests/
├── benches/
├── examples/
├── docs/
│   ├── architecture.md
│   ├── correctness.md
│   ├── performance.md
│   └── limitations.md
├── scripts/
└── results/
```

Adapt this structure to your specialization.

Do not create modules merely to make the project appear large.

Module boundaries should correspond to meaningful responsibilities.

---

# 15. Required Engineering Artifacts

The final project must include the following.

## 15.1 Project proposal

Define the problem, scope, and success criteria.

## 15.2 Architecture document

Explain the system's major components.

## 15.3 Correctness specification

Define expected behavior and invariants.

## 15.4 Implementation

Provide a working, documented system.

## 15.5 Automated tests

Demonstrate functional correctness.

## 15.6 Benchmark suite

Provide reproducible performance measurements.

## 15.7 Performance report

Explain results and limitations.

## 15.8 Technical presentation

Summarize the problem, design, implementation, and results.

## 15.9 Final engineering retrospective

Explain:

- What worked.
- What failed.
- What you changed.
- What you learned.
- What remains incomplete.
- What you would redesign.

---

# 16. Capstone Evaluation Rubric

| Category | Weight |
|---|---:|
| Problem definition and research | 10% |
| Architecture and design | 15% |
| Implementation quality | 20% |
| Correctness and testing | 20% |
| Performance engineering | 15% |
| Reliability and maintainability | 10% |
| Documentation and technical defense | 10% |

## 16.1 Problem definition

A strong submission has:

- Clear requirements.
- Measurable objectives.
- Appropriate scope.
- Relevant prior-work research.

## 16.2 Architecture

A strong submission has:

- Clear module boundaries.
- Explicit ownership.
- Well-defined interfaces.
- Documented trade-offs.

## 16.3 Implementation

A strong submission has:

- Working functionality.
- Appropriate abstractions.
- Minimal unnecessary complexity.
- Consistent error handling.

## 16.4 Correctness

A strong submission has:

- Reference implementations.
- Automated tests.
- Edge-case coverage.
- Explicit invariants.
- Documented numerical behavior.

## 16.5 Performance

A strong submission has:

- Valid baselines.
- Controlled experiments.
- Repeated measurements.
- Hardware-aware interpretation.
- Reproducible results.

## 16.6 Reliability

A strong submission has:

- Failure handling.
- Resource cleanup.
- Safe asynchronous behavior.
- Clear operational limitations.

## 16.7 Communication

A strong submission explains the project clearly enough for another engineer to understand and evaluate it.

---

# 17. Final Technical Defense

The defense should resemble an engineering design review.

## Part A: Problem

Explain:

- What problem you solved.
- Why it matters.
- What existing systems do.
- What your contribution is.

## Part B: Architecture

Explain:

- Major components.
- Data flow.
- Ownership.
- Synchronization.
- Failure handling.

## Part C: Algorithms

Explain:

- Mathematical foundations.
- Complexity.
- Parallel decomposition.
- Correctness.

## Part D: GPU execution

Explain:

- Workgroup mapping.
- Memory access.
- Synchronization.
- Resource usage.
- Hardware constraints.

## Part E: Performance

Explain:

- Baselines.
- Measurement methodology.
- Bottlenecks.
- Optimizations.
- Trade-offs.

## Part F: Reliability

Explain:

- Error handling.
- Resource lifetimes.
- Device loss.
- Test coverage.

## Part G: Future work

Explain:

- Remaining limitations.
- Potential improvements.
- Alternative architectures.
- What would be required for production use.

---

# 18. Comprehensive Final Quiz Bank

## Systems fundamentals

1. What is the relationship between CPU and GPU execution?
2. Why are GPUs throughput-oriented?
3. What is a cache hierarchy?
4. What is memory bandwidth?
5. What is latency hiding?
6. What is false sharing?
7. What is virtual memory?
8. What is DMA?
9. What is an IOMMU?
10. What is an ABI?

## Rust and C++

11. What is Rust ownership?
12. What is borrowing?
13. What are lifetimes?
14. What is unsafe Rust?
15. What is RAII?
16. What is move semantics?
17. What is undefined behavior?
18. What is the C++ memory model?
19. What are atomics?
20. What is FFI?

## Parallel algorithms

21. What is work?
22. What is span?
23. What is Amdahl's law?
24. What is a reduction?
25. What is a prefix scan?
26. What is stream compaction?
27. What is parallel sorting?
28. What is work stealing?
29. What is a race condition?
30. What is a synchronization barrier?

## GPU architecture

31. What is a compute unit?
32. What is a wavefront?
33. What is divergence?
34. What is occupancy?
35. What is register pressure?
36. What is workgroup-local memory?
37. What is memory coalescing?
38. What is arithmetic intensity?
39. What is the Roofline model?
40. What is a memory-bound kernel?

## GPU programming

41. What is a compute shader?
42. What is a workgroup?
43. What is a dispatch?
44. What is a storage buffer?
45. What is a uniform buffer?
46. What is a pipeline?
47. What is a descriptor set?
48. What is a command buffer?
49. What is a queue submission?
50. What is a fence?

## Numerical computing

51. What is floating-point rounding?
52. What is machine epsilon?
53. What is numerical stability?
54. What is conditioning?
55. What is cancellation?
56. What is a residual?
57. What is numerical convergence?
58. What is pairwise summation?
59. What is mixed precision?
60. Why can parallel reductions produce different results?

## GPU optimization

61. What is tiling?
62. What is loop unrolling?
63. What is kernel fusion?
64. What is register spilling?
65. What is a performance counter?
66. What is a GPU timestamp?
67. What is effective bandwidth?
68. What is a cache miss?
69. What is autotuning?
70. Why must optimizations be measured?

## Runtime systems

71. What is a GPU runtime?
72. What is suballocation?
73. What is buffer pooling?
74. What is deferred destruction?
75. What is a task graph?
76. What is a critical path?
77. What is dependency analysis?
78. What is a timeline semaphore?
79. What is queue-family ownership?
80. What is device loss?

## Compiler engineering

81. What is an AST?
82. What is semantic analysis?
83. What is an IR?
84. What is a CFG?
85. What is SSA?
86. What is a phi node?
87. What is dominance?
88. What is liveness?
89. What is alias analysis?
90. What is dead-code elimination?
91. What is constant propagation?
92. What is loop-invariant code motion?
93. What is uniformity analysis?
94. What is SPIR-V?
95. What is LLVM IR?

## Engineering judgment

96. How do you define a useful baseline?
97. How do you prove an optimization preserves correctness?
98. How do you distinguish CPU overhead from GPU execution time?
99. How do you decide whether an abstraction is worthwhile?
100. How do you identify the true bottleneck?
101. How do you evaluate competing architectures?
102. How do you decide when to stop optimizing?
103. How do you handle uncertainty in benchmark results?
104. How do you scope a complex systems project?
105. How do you defend a technical decision?

---

# 19. Final Written Examination

**Recommended duration:** 12–16 hours.

The examination is separate from the capstone implementation.

## Section A: Architecture analysis

Analyze a GPU system architecture.

Identify:

- Data flow.
- Ownership.
- Synchronization.
- Performance bottlenecks.
- Failure risks.

## Section B: Parallel algorithm design

Design a parallel algorithm.

Explain:

- Correctness.
- Work.
- Span.
- Memory access.
- Synchronization.
- Expected performance.

## Section C: Numerical analysis

Analyze floating-point behavior.

Explain:

- Error sources.
- Stability.
- Conditioning.
- Precision trade-offs.
- Validation methods.

## Section D: GPU runtime design

Design a runtime subsystem.

Explain:

- Resource management.
- Submission.
- Completion.
- Dependencies.
- Failure handling.

## Section E: Compiler optimization

Analyze an optimization.

Explain:

- Preconditions.
- Semantics.
- Correctness.
- Performance implications.

## Section F: Performance investigation

Interpret benchmark results and propose experiments to test competing explanations.

## Section G: Engineering defense

Defend a major architectural choice from your capstone.

---

# 20. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

**Strict tutor rule:** The tutor must never provide answers, code, pseudocode, implementation instructions, debugging fixes, file edits, completed derivations, or assessment solutions.

It must ask exactly one guiding question per response.

## Master Capstone Tutor Prompt

```text id="g05o3t"
You are my Socratic GPU engineering capstone tutor.

I am completing Phase 15 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises,
   assessments, derivations, projects, or examinations.

2. Never write code, pseudocode, algorithms,
   implementations, or project solutions.

3. Never modify files, execute code, debug programs,
   or perform project work on my behalf.

4. Never reveal answer keys, completed calculations,
   hidden tests, or solutions.

5. Never provide hints that effectively disclose a solution.

6. Guide my reasoning exclusively through questions.

7. Ask exactly one guiding question per response.

8. If I am incorrect, ask a question that exposes
   the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to justify architecture, algorithms,
    correctness, synchronization, numerical behavior,
    performance, reliability, and engineering trade-offs.

11. For project planning, ask questions that help me
    establish requirements, scope, risks, and milestones.
    Do not create my plan for me.

12. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

13. For code review, ask questions about correctness,
    invariants, ownership, synchronization, failure handling,
    and performance. Never provide exact fixes.

14. For performance analysis, ask questions about
    measurement methodology and competing explanations.
    Never interpret results or provide conclusions for me.

15. For oral examinations, ask one question at a time
    and adapt difficulty based on my responses.

16. Never provide a final answer during an assessment.

17. Keep responses concise and focused.

Your responses must contain only one guiding question.
```

## Project selection

```text id="cl3v6k"
Help me choose my Phase 15 GPU engineering specialization.

Ask questions about my interests, existing skills,
hardware constraints, project feasibility,
and desired technical depth.

Do not choose the project for me.
Do not provide a project plan or implementation.

Ask exactly one question at a time.
```

## Proposal review

```text id="3d1uwu"
Review my capstone proposal using only questions.

Focus on:
- Problem definition
- Scope
- Requirements
- Success criteria
- Risks
- Correctness
- Performance
- Feasibility

Do not rewrite my proposal or provide solutions.

Ask exactly one guiding question at a time.
```

## Architecture review

```text id="xq7r4m"
Review my capstone architecture using only questions.

Focus on:
- Module boundaries
- Ownership
- Data flow
- Synchronization
- Failure handling
- Resource management
- Extensibility
- Performance

Do not provide alternative designs or implementation instructions.

Ask exactly one guiding question at a time.
```

## Correctness examination

```text id="w8yt4h"
Examine the correctness of my GPU system.

Ask questions about:
- Functional contracts
- Numerical correctness
- Memory safety
- Resource lifetimes
- Synchronization
- Compiler semantics
- Edge cases
- Testing

Do not provide fixes or solutions.

Ask exactly one question at a time.
```

## Performance investigation

```text id="m3rr59"
Help me investigate GPU performance using only questions.

Ask about:
- Baselines
- Workload characteristics
- CPU overhead
- GPU execution time
- Memory bandwidth
- Arithmetic intensity
- Occupancy
- Register pressure
- Synchronization
- Benchmark methodology
- Statistical uncertainty

Do not interpret my results for me.
Do not suggest exact optimizations.

Ask exactly one question at a time.
```

## Reliability review

```text id="1t8n65"
Examine the reliability of my GPU system.

Ask questions about:
- Error propagation
- Allocation failure
- Device loss
- Timeouts
- Resource cleanup
- Concurrent execution
- Unsafe Rust
- Recovery behavior
- Stress testing

Never provide fixes or implementation guidance.

Ask exactly one question at a time.
```

## Technical defense

```text id="j1h2ms"
Conduct a rigorous technical defense of my GPU capstone.

Challenge me on:
- Problem definition
- Prior work
- Architecture
- Algorithms
- Correctness
- GPU execution
- Memory management
- Synchronization
- Compiler behavior
- Performance
- Reliability
- Trade-offs
- Limitations

Ask exactly one question at a time.

Never provide answers, solutions, code,
or hints that reveal the answer.
```

## Final comprehensive oral examination

```text id="p5e9fr"
Conduct a comprehensive final GPU engineering examination
covering Phases 0 through 15.

Test:
- Mathematics
- Rust
- C++
- Computer architecture
- Operating systems
- Numerical computing
- Concurrency
- GPU architecture
- GPU programming
- CUDA concepts
- GPU optimization
- Parallel algorithms
- GPU runtimes
- GPU compilers
- Engineering judgment

Ask exactly one question at a time.

Adapt difficulty based on my responses.

Never provide answers, solutions, code,
or implementation guidance.
```

---

# 21. Final Mastery Checklist

## Mathematical foundations

- [ ] I understand floating-point arithmetic.
- [ ] I understand numerical error.
- [ ] I understand conditioning and stability.
- [ ] I can analyze algorithmic complexity.
- [ ] I can reason about parallel work and span.

## Systems programming

- [ ] I can design safe Rust APIs.
- [ ] I understand ownership and lifetimes.
- [ ] I can reason about unsafe Rust.
- [ ] I understand C++ object lifetime.
- [ ] I understand concurrency and memory ordering.
- [ ] I understand operating-system resource management.

## GPU architecture

- [ ] I understand GPU execution hierarchy.
- [ ] I understand wavefront execution.
- [ ] I understand divergence.
- [ ] I understand memory coalescing.
- [ ] I understand occupancy.
- [ ] I understand register pressure.
- [ ] I understand memory bandwidth.
- [ ] I can use performance models.

## GPU programming

- [ ] I can write GPU compute kernels.
- [ ] I can manage GPU buffers.
- [ ] I can dispatch GPU work.
- [ ] I can synchronize GPU operations.
- [ ] I can validate GPU results.
- [ ] I can benchmark GPU execution.

## Parallel algorithms

- [ ] I can implement reductions.
- [ ] I can implement prefix scans.
- [ ] I understand parallel sorting.
- [ ] I understand stream compaction.
- [ ] I can analyze work and span.
- [ ] I can optimize memory access.

## GPU runtime systems

- [ ] I understand memory allocation.
- [ ] I understand resource lifetimes.
- [ ] I understand task graphs.
- [ ] I understand scheduling.
- [ ] I understand synchronization.
- [ ] I understand asynchronous execution.
- [ ] I understand device loss.

## GPU compilers

- [ ] I understand compiler front ends.
- [ ] I understand intermediate representations.
- [ ] I understand CFGs.
- [ ] I understand SSA.
- [ ] I understand data-flow analysis.
- [ ] I understand optimization correctness.
- [ ] I understand GPU-specific lowering.
- [ ] I understand SPIR-V and shader validation.

## Engineering process

- [ ] I can define requirements.
- [ ] I can research existing approaches.
- [ ] I can design a system architecture.
- [ ] I can document invariants.
- [ ] I can build correctness tests.
- [ ] I can design performance experiments.
- [ ] I can interpret measurements.
- [ ] I can explain trade-offs.
- [ ] I can manage project scope.
- [ ] I can produce reproducible results.

## Final capstone

- [ ] My project has a clear problem statement.
- [ ] My project has measurable success criteria.
- [ ] My project has a documented architecture.
- [ ] My project has a working implementation.
- [ ] My project has automated tests.
- [ ] My project has reproducible benchmarks.
- [ ] My project has a performance report.
- [ ] My project has documented limitations.
- [ ] My project has a technical presentation.
- [ ] I can defend the design independently.

---

# 22. Recommended Final Resources

## Systems engineering

**Computer Systems: A Programmer's Perspective — Bryant and O'Hallaron**

**Operating Systems: Three Easy Pieces — Arpaci-Dusseau and Arpaci-Dusseau**

https://pages.cs.wisc.edu/~remzi/OSTEP/

**The Rust Programming Language**

https://doc.rust-lang.org/book/

## Computer architecture

**Computer Architecture: A Quantitative Approach — Hennessy and Patterson**

## GPU programming

**Programming Massively Parallel Processors — Kirk and Hwu**

**Vulkan Specification**

https://registry.khronos.org/vulkan/

**Vulkan Guide**

https://docs.vulkan.org/guide/latest/

**AMD GPUOpen**

https://gpuopen.com/

## Compiler engineering

**Engineering a Compiler — Cooper and Torczon**

**LLVM Documentation**

https://llvm.org/docs/

**SPIR-V Registry**

https://registry.khronos.org/SPIR-V/

## Performance engineering

**Brendan Gregg — Systems Performance**

**The Art of Computer Systems Performance Analysis — Raj Jain**

## Research literature

**arXiv**

https://arxiv.org/

**ACM Digital Library**

https://dl.acm.org/

**IEEE Xplore**

https://ieeexplore.ieee.org/

Relevant conferences and communities include:

- ASPLOS.
- ISCA.
- MICRO.
- PLDI.
- CGO.
- PPoPP.
- SC.
- HPCA.

---

# 23. Graduation Requirements

You graduate from the curriculum when you have completed:

## Requirement 1: Foundational competence

You can explain the mathematical, architectural, and systems foundations of GPU computing.

## Requirement 2: Independent implementation

You can implement GPU algorithms and systems without relying on AI-generated solutions.

## Requirement 3: Correctness

You can establish and test the correctness of your implementations.

## Requirement 4: Performance engineering

You can measure, analyze, and optimize GPU workloads systematically.

## Requirement 5: Systems integration

You can integrate memory management, execution, synchronization, and algorithms into a coherent system.

## Requirement 6: Technical communication

You can explain your design decisions, results, and limitations clearly.

## Requirement 7: Final capstone

You have produced a substantial, working, tested, benchmarked, and documented GPU engineering project.

---

# 24. What Comes After the Curriculum?

Completing the curriculum is a major technical milestone, but it does not automatically confer senior-level engineering judgment.

That judgment develops through repeated exposure to:

- Production constraints.
- Long-lived codebases.
- Difficult debugging.
- Performance regressions.
- Cross-team design decisions.
- Reliability incidents.
- Code review.
- Maintenance.
- Hardware and software changes.

The next stage is to apply these skills in increasingly demanding environments.

## Possible career directions

### GPU software engineer

Focus on kernels, execution systems, performance, and hardware-aware programming.

### GPU compiler engineer

Focus on intermediate representations, optimizations, lowering, and code generation.

### GPU runtime engineer

Focus on scheduling, memory management, synchronization, and execution infrastructure.

### High-performance computing engineer

Focus on numerical algorithms, parallel systems, and scientific workloads.

### Machine learning systems engineer

Focus on tensor execution, inference infrastructure, kernel optimization, and compilation.

### Graphics and compute driver engineer

Focus on low-level APIs, shader compilation, memory management, and hardware interfaces.

## Continuing development

After graduation:

1. Contribute to an established open-source GPU project.
2. Study production GPU compiler or runtime implementations.
3. Reproduce a published performance result.
4. Investigate a difficult correctness or performance bug.
5. Maintain your capstone across toolchain and driver updates.
6. Participate in technical design and code review.
7. Continue developing deeper expertise in your chosen specialization.

---

# 25. Final Curriculum Completion Standard

The ultimate goal of this curriculum is not memorization.

It is the ability to approach an unfamiliar GPU engineering problem and independently determine:

- What the system is doing.
- What correctness requires.
- What the hardware is capable of.
- Where the bottlenecks are.
- Which abstractions are appropriate.
- Which optimizations are legal.
- How to test the implementation.
- How to measure performance.
- How to communicate the results.

A successful graduate can move between mathematical reasoning, low-level implementation, GPU architecture, runtime systems, compiler infrastructure, and performance analysis.

**The final milestone is a working GPU system whose correctness, architecture, and performance you can explain and defend from first principles.**

**End of the 16-phase GPU Engineering Curriculum.**
