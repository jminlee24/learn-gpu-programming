# Phase 8: Portable GPU Programming with Rust, wgpu, and WGSL

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 8 of 16  
**Estimated duration:** 14–18 weeks  
**Study commitment:** 10–12 hours per week  
**Primary language:** Rust  
**GPU shader language:** WGSL  
**GPU API:** wgpu  
**Platform:** Fedora Linux  
**Hardware:** AMD Radeon RX 9070, 16 GB VRAM  
**Prerequisites:** Phases 0–7

---

# 1. Purpose of This Phase

Until now, you've studied GPU execution theoretically.

In this phase, you will actually execute your own programs on GPU hardware.

You will learn to manage the boundary between CPU and GPU execution, construct compute pipelines, move data, dispatch workloads, and measure performance.

The goal is not simply to produce working shaders.

The goal is to understand every major stage between a Rust function and execution on GPU hardware.

## Learning outcomes

By the end of Phase 8, you should be able to:

1. Initialize a GPU device through wgpu.
2. Explain adapters, devices, and queues.
3. Create and manage GPU buffers.
4. Understand buffer usage flags.
5. Write WGSL compute shaders.
6. Create compute pipelines.
7. Configure bind groups.
8. Dispatch compute workloads.
9. Map global invocation IDs to data indices.
10. Understand GPU workgroup organization.
11. Use workgroup memory.
12. Synchronize workgroup threads.
13. Transfer data between CPU and GPU.
14. Implement parallel GPU reductions.
15. Implement parallel prefix scans.
16. Implement tiled matrix multiplication.
17. Benchmark GPU kernels.
18. Analyze memory bandwidth and arithmetic intensity.
19. Diagnose common correctness and performance problems.
20. Build a reusable Rust GPU computing framework.

---

# 2. Study Schedule

| Module | Subject | Time |
|---|---|---|
| 1 | wgpu environment and device initialization | 1 week |
| 2 | Buffers and GPU memory | 2 weeks |
| 3 | WGSL compute shaders | 2 weeks |
| 4 | Compute pipelines and dispatch | 2 weeks |
| 5 | Workgroup memory and synchronization | 2 weeks |
| 6 | Reductions and scans | 2–3 weeks |
| 7 | Matrix multiplication and optimization | 2–3 weeks |
| 8 | Capstone and examination | 2–3 weeks |

Do not advance until each module's implementation works correctly and you can explain its execution model.

---

# 3. Module One: The GPU Software Stack

## 3.1 From Rust to GPU hardware

A typical Rust compute application using wgpu interacts with several layers.

Conceptually:

```text
Rust Application
      |
      v
wgpu
      |
      v
Graphics/Compute API Backend
      |
      v
GPU Driver
      |
      v
GPU Hardware
```

On Fedora with an AMD GPU, a common execution path is:

```text
Rust Application
      |
      v
wgpu
      |
      v
Vulkan
      |
      v
Mesa RADV
      |
      v
Linux amdgpu driver
      |
      v
AMD Radeon GPU
```

The precise backend depends on platform configuration and adapter selection.

## 3.2 What is wgpu?

wgpu is a Rust GPU abstraction designed to support multiple backends.

It provides interfaces for:

- Device discovery.
- Resource creation.
- Buffer management.
- Compute pipelines.
- Render pipelines.
- Command encoding.
- Command submission.
- GPU synchronization.

For this phase, focus on compute functionality rather than graphics rendering.

## 3.3 What is WGSL?

WGSL stands for WebGPU Shading Language.

It is used to write shaders that execute on GPU hardware through WebGPU-compatible implementations.

WGSL provides:

- Scalar types.
- Vector types.
- Matrix types.
- Arrays.
- Structures.
- Compute entry points.
- Built-in invocation identifiers.
- Storage buffers.
- Workgroup memory.
- Synchronization primitives.

## 3.4 Adapter

An adapter represents a candidate physical or logical GPU implementation.

It exposes capabilities such as:

- Supported features.
- Resource limits.
- Device information.
- Backend information.

## 3.5 Device

A device is a logical interface used to create GPU resources and pipelines.

It provides access to:

- Buffers.
- Bind group layouts.
- Bind groups.
- Shader modules.
- Compute pipelines.
- Command encoders.

## 3.6 Queue

A queue is used to submit commands and perform supported data uploads.

GPU commands execute asynchronously relative to ordinary CPU execution.

## 3.7 Asynchronous execution

When the CPU submits GPU work, it does not necessarily wait for that work to complete.

This allows CPU and GPU operations to overlap.

However, reading GPU-generated results requires appropriate synchronization.

## 3.8 Environment setup

Install Rust using rustup if necessary.

Verify:

```bash
rustc --version
cargo --version
```

Check GPU visibility:

```bash
lspci | grep -Ei 'vga|display|3d'
```

Check Vulkan support if the utility is installed:

```bash
vulkaninfo --summary
```

On Fedora, the Vulkan utility package is commonly named `vulkan-tools`.

## Exercises

1. Explain the role of wgpu.
2. Explain the role of WGSL.
3. Explain the difference between an adapter and a device.
4. Explain the purpose of a GPU queue.
5. Explain why GPU execution is asynchronous.
6. Describe the software path from Rust to your AMD GPU.
7. Explain why portable APIs do not guarantee identical performance across GPUs.
8. Explain why device limits must be queried rather than assumed.

## Coding Lab 1: GPU Device Inspector

Build a Rust command-line application that:

- Initializes wgpu.
- Enumerates available adapters.
- Selects an appropriate GPU.
- Creates a logical device.
- Prints adapter information.
- Prints supported limits.
- Prints relevant features.
- Reports initialization errors clearly.

Document the selected backend and hardware.

---

# 4. Module Two: GPU Buffers and Memory

## 4.1 CPU memory versus GPU memory

A discrete GPU has its own memory resources.

The CPU and GPU may access different memory regions with different performance characteristics.

Data movement between CPU and GPU can be expensive.

## 4.2 GPU buffers

A GPU buffer represents a region of memory accessible through supported GPU operations.

Buffers may contain:

- Input arrays.
- Output arrays.
- Matrices.
- Parameters.
- Intermediate results.

## 4.3 Buffer usage flags

Buffers are created with usage flags describing how they may be accessed.

Common wgpu buffer usages include:

- `STORAGE`
- `UNIFORM`
- `COPY_SRC`
- `COPY_DST`
- `MAP_READ`
- `MAP_WRITE`

These flags determine which operations are permitted.

## 4.4 Storage buffers

Storage buffers allow shaders to read and, when declared appropriately, write structured data.

They are useful for general-purpose GPU computing.

## 4.5 Uniform buffers

Uniform buffers are commonly used for relatively small read-only parameter data.

They have different layout and access constraints from storage buffers.

## 4.6 Buffer alignment

GPU APIs impose alignment requirements on certain buffer operations and bindings.

Misaligned accesses may be invalid.

Correct buffer layouts require understanding:

- Element sizes.
- Alignment.
- Padding.
- Structure layout.
- Binding constraints.

## 4.7 Host-to-device transfer

Input data must be made available to the GPU.

A typical workflow involves:

1. Preparing CPU data.
2. Creating or updating GPU buffers.
3. Submitting GPU work.
4. Reading results if necessary.

## 4.8 Device-to-host transfer

GPU results are often copied into a buffer suitable for CPU mapping.

The CPU must wait until the relevant GPU operations complete before safely consuming the mapped result.

## 4.9 Persistent GPU data

Repeatedly transferring large arrays between CPU and GPU can dominate execution time.

Whenever possible, GPU workloads should reuse data already resident on the device.

## Exercises

1. What is a GPU buffer?
2. What is a storage buffer?
3. What is a uniform buffer?
4. Why do usage flags matter?
5. Why does alignment matter?
6. Why are transfers expensive?
7. Why is asynchronous readback important?
8. Why should GPU data sometimes remain resident across multiple kernels?
9. Explain the difference between allocation, upload, execution, and readback.

## Coding Lab 2: Buffer Transfer Laboratory

Build a program that:

- Creates GPU buffers.
- Uploads arrays.
- Copies data between GPU buffers.
- Reads data back.
- Validates correctness.
- Measures transfer times.
- Measures transfer throughput.

Test several buffer sizes.

Document whether measured performance is dominated by fixed overhead or bulk data transfer.

---

# 5. Module Three: WGSL Compute Shaders

## 5.1 Compute shaders

A compute shader is a program executed by GPU threads.

Unlike graphics shaders, compute shaders are designed for general computation.

## 5.2 Entry points

WGSL compute shaders use an entry-point annotation.

Conceptually:

```wgsl
@compute @workgroup_size(64)
fn main(
    @builtin(global_invocation_id) global_id: vec3<u32>
) {
    // GPU computation
}
```

This defines a compute entry point with a workgroup size of 64 invocations.

## 5.3 Global invocation ID

Each invocation has a global identifier.

For a one-dimensional workload, the x component is commonly used as an array index.

## 5.4 Workgroup size

The workgroup size determines how many invocations belong to each workgroup.

For example:

```wgsl
@workgroup_size(64)
```

defines a workgroup with 64 invocations.

## 5.5 Dispatch size

The CPU specifies how many workgroups to dispatch.

For \(N\) elements and workgroup size \(W\), the required number of workgroups is:

\[
G=\left\lceil\frac{N}{W}\right\rceil
\]

## 5.6 Bounds checking

The total number of launched invocations may exceed the number of valid data elements.

Shaders must avoid invalid accesses.

## 5.7 Storage buffer bindings

WGSL resources are associated with binding locations.

For example:

```wgsl
@group(0) @binding(0)
var<storage, read> input: array<f32>;

@group(0) @binding(1)
var<storage, read_write> output: array<f32>;
```

These declarations specify resource access modes and binding locations.

## 5.8 Vector addition shader

A basic vector addition shader conceptually computes:

\[
C_i=A_i+B_i
\]

Each invocation processes one output element.

## 5.9 WGSL data types

Important types include:

- `i32`
- `u32`
- `f32`
- `bool`
- Vectors.
- Matrices.
- Arrays.
- Structures.

Additional types and capabilities depend on supported language features and device capabilities.

## Exercises

1. What is a compute shader?
2. What is a compute entry point?
3. What is a workgroup?
4. What is a global invocation ID?
5. How is dispatch size calculated?
6. Why is bounds checking necessary?
7. What is a storage buffer binding?
8. What does read-only storage access mean?
9. What does read-write storage access mean?
10. Why is vector addition naturally data-parallel?

## Coding Lab 3: First Compute Shader

Implement GPU vector addition.

Requirements:

- Configurable input size.
- CPU reference implementation.
- GPU implementation.
- Correct bounds checking.
- Result validation.
- Timing measurements.

Test small, medium, and large inputs.

---

# 6. Module Four: Compute Pipelines and Command Submission

## 6.1 Shader modules

A shader module contains shader code compiled or validated by the GPU API implementation.

## 6.2 Bind group layouts

A bind group layout describes the resources a shader expects.

It defines properties such as:

- Binding indices.
- Resource types.
- Access modes.
- Shader visibility.

## 6.3 Bind groups

A bind group associates actual GPU resources with the bindings described by a layout.

## 6.4 Compute pipelines

A compute pipeline combines the shader entry point with the resource interface and relevant execution configuration.

## 6.5 Command encoders

A command encoder records GPU commands.

Commands are later submitted for execution.

## 6.6 Compute passes

A compute pass records compute operations, including:

- Pipeline selection.
- Resource binding.
- Dispatch commands.

## 6.7 Submission

Encoded commands are submitted through a queue.

Execution is asynchronous relative to the CPU.

## 6.8 Synchronization

GPU synchronization ensures operations execute in a valid order and that data is available before it is consumed.

The API manages many resource hazards, but the application must still structure commands and readbacks correctly.

## Exercises

1. What is a shader module?
2. What is a bind group layout?
3. What is a bind group?
4. What is a compute pipeline?
5. What is a command encoder?
6. What is a compute pass?
7. What is command submission?
8. Why does GPU synchronization matter?
9. Why is CPU-side submission time different from GPU execution time?

## Coding Lab 4: Reusable Compute Pipeline

Refactor your vector addition application into a reusable GPU compute framework.

Requirements:

- Device initialization.
- Buffer creation.
- Shader loading.
- Pipeline creation.
- Resource binding.
- Dispatch.
- Readback.
- Validation.
- Error handling.

The framework should support additional compute shaders without duplicating the entire setup.

---

# 7. Module Five: Workgroup Memory and Synchronization

## 7.1 Workgroup memory

Workgroup memory is shared among invocations in the same workgroup.

It is useful for:

- Data reuse.
- Reductions.
- Scans.
- Tiled matrix multiplication.

## 7.2 Why workgroup memory matters

Global memory accesses can be expensive.

If multiple threads need the same data, loading it into workgroup memory may reduce repeated global-memory traffic.

## 7.3 Workgroup barriers

A workgroup barrier coordinates invocations within a workgroup.

It can ensure required memory operations are ordered appropriately before subsequent work.

## 7.4 Barrier correctness

Barriers must be used according to the language's uniformity and synchronization rules.

A barrier reached by only some invocations can be invalid or unsafe depending on the control-flow structure.

## 7.5 Workgroup-local reductions

A workgroup can cooperate to reduce a collection of values.

Each invocation may contribute one or more values.

Intermediate results can be combined in workgroup memory.

## 7.6 Synchronization overhead

Barriers are not free.

Excessive synchronization can reduce performance.

## 7.7 Workgroup memory limits

Each workgroup consumes a portion of the hardware's limited shared/local memory capacity.

Large allocations may reduce the number of resident workgroups.

## Exercises

1. What is workgroup memory?
2. Why is workgroup memory useful?
3. What is a workgroup barrier?
4. Why must barriers be used carefully?
5. How can workgroup memory improve data reuse?
6. Why can excessive synchronization hurt performance?
7. How does workgroup memory usage affect occupancy?
8. Why can workgroups not generally synchronize with each other inside an ordinary dispatch?

## Coding Lab 5: Workgroup Reduction

Implement a GPU reduction that computes partial sums within workgroups.

Requirements:

- Correct workgroup synchronization.
- Correct handling of arbitrary input lengths.
- CPU reference implementation.
- Numerical error analysis.
- Performance measurement.

---

# 8. Module Six: GPU Reductions

## 8.1 Reduction structure

A GPU reduction combines many values into fewer values.

For example:

\[
S=\sum_{i=1}^{N}x_i
\]

## 8.2 Hierarchical reduction

A common GPU strategy uses multiple stages.

First stage:

- Each workgroup computes a partial result.

Second stage:

- Partial results are combined.

Additional stages may be required for large inputs.

## 8.3 Why multiple stages are needed

Ordinary workgroup barriers synchronize invocations within a workgroup.

They do not provide a general grid-wide barrier across independently scheduled workgroups.

Multiple dispatches can establish the necessary global ordering.

## 8.4 Numerical behavior

GPU reductions may produce results different from sequential CPU summation.

This is expected when the accumulation order changes.

## 8.5 Performance considerations

Important factors include:

- Workgroup size.
- Memory coalescing.
- Register usage.
- Shared-memory usage.
- Synchronization overhead.
- Number of dispatches.

## Exercises

1. Why are reductions important?
2. Why use hierarchical reduction?
3. Why are multiple dispatches sometimes necessary?
4. How does reduction order affect floating-point results?
5. Why does workgroup size affect reduction performance?
6. Why does memory coalescing matter?
7. Why can synchronization limit performance?

## Coding Lab 6: Complete GPU Reduction

Implement a complete GPU sum reduction.

Support:

- Arbitrary input lengths.
- Multiple reduction stages.
- CPU reference validation.
- Configurable workgroup sizes.
- Performance benchmarking.
- Numerical error reporting.

Compare against your Phase 6 CPU reduction.

---

# 9. Module Seven: GPU Prefix Scans

Prefix scans are among the most important parallel primitives.

## 9.1 Inclusive scan

Given:

\[
[1,2,3,4]
\]

the inclusive scan is:

\[
[1,3,6,10]
\]

## 9.2 Exclusive scan

The exclusive scan is:

\[
[0,1,3,6]
\]

## 9.3 Workgroup-local scan

A workgroup can compute a scan over a portion of the input.

## 9.4 Multi-workgroup scan

For large arrays, a hierarchical scan may:

1. Compute scans within workgroups.
2. Compute aggregates of workgroup totals.
3. Propagate offsets to each workgroup's results.

## 9.5 GPU scan challenges

Important challenges include:

- Synchronization.
- Shared-memory layout.
- Workgroup size.
- Non-power-of-two input lengths.
- Multiple dispatch stages.
- Memory bandwidth.

## 9.6 Scan applications

Scans support:

- Stream compaction.
- Radix sorting.
- Histograms.
- Sparse data processing.
- Parallel allocation.
- Graph algorithms.

## Exercises

1. Define inclusive scan.
2. Define exclusive scan.
3. Why is scan harder than reduction?
4. Why are hierarchical scans useful?
5. How do workgroup totals contribute to a global scan?
6. Why do non-power-of-two lengths matter?
7. Why are scans important for GPU sorting?
8. Why does synchronization matter?

## Coding Lab 7: GPU Scan

Implement:

- Inclusive scan.
- Exclusive scan.
- Arbitrary-length support.
- Multi-workgroup support.
- CPU validation.
- Performance benchmarks.

Compare with your Phase 6 CPU scan.

---

# 10. Module Eight: GPU Matrix Multiplication

Matrix multiplication is a major GPU workload.

## 10.1 Mathematical definition

For:

\[
A\in\mathbb{R}^{M\times K}
\]

and:

\[
B\in\mathbb{R}^{K\times N}
\]

the product is:

\[
C=AB
\]

where:

\[
C_{ij}=\sum_{k=0}^{K-1}A_{ik}B_{kj}
\]

using zero-based indexing.

## 10.2 Naive GPU implementation

A straightforward GPU implementation assigns output elements to invocations.

Each invocation computes a dot product.

This is easy to understand but may repeatedly load the same data.

## 10.3 Tiled matrix multiplication

Tiling divides matrices into smaller blocks.

Threads cooperate to load tiles into workgroup memory.

The loaded values are reused for multiple arithmetic operations.

## 10.4 Why tiling helps

Without reuse, many global-memory accesses are repeated.

Tiling increases arithmetic work performed per byte loaded.

This increases arithmetic intensity.

## 10.5 Workgroup tile size

Tile size affects:

- Workgroup memory consumption.
- Register usage.
- Synchronization.
- Occupancy.
- Memory access patterns.
- Arithmetic intensity.

## 10.6 Matrix layout

Row-major and column-major layouts affect memory access patterns.

The layout must be considered when mapping lanes to matrix elements.

## 10.7 Numerical correctness

Different matrix multiplication implementations may accumulate products in different orders.

Floating-point results may differ slightly.

Validation must account for numerical error.

## 10.8 Performance measurement

Measure:

- Kernel execution time.
- End-to-end execution time.
- FLOPs per second.
- Effective memory traffic.
- Arithmetic intensity.
- Scaling with matrix dimensions.

## Exercises

1. Derive matrix multiplication.
2. Explain naive GPU matrix multiplication.
3. Why does naive matrix multiplication repeat memory accesses?
4. What is tiling?
5. Why does tiling increase data reuse?
6. How does tile size affect occupancy?
7. Why does memory layout matter?
8. Why can floating-point results differ across implementations?
9. Why must GPU matrix multiplication be benchmarked across multiple sizes?

## Coding Lab 8: GPU Matrix Multiplication

Implement:

1. Naive matrix multiplication.
2. Tiled matrix multiplication.
3. Configurable tile sizes.
4. CPU reference validation.
5. Performance benchmarks.

Compare results with your Phase 5 CPU matrix multiplication.

---

# 11. Module Nine: GPU Benchmarking and Profiling

## 11.1 Why benchmarking is difficult

GPU execution is asynchronous.

CPU-side timing may not represent GPU execution time.

## 11.2 Timing categories

Distinguish:

- CPU preparation time.
- Buffer allocation time.
- Upload time.
- Pipeline creation time.
- Command submission time.
- GPU execution time.
- Readback time.
- Total end-to-end time.

## 11.3 Warm-up

Initial runs may include:

- Shader compilation.
- Pipeline creation.
- Driver initialization.
- Resource allocation.

Steady-state benchmarks should separate these costs when appropriate.

## 11.4 Repeated measurements

Run benchmarks repeatedly.

Report:

- Median.
- Minimum.
- Maximum.
- Variability.
- Relevant percentiles.

## 11.5 GPU timestamp queries

GPU timestamp queries can measure elapsed GPU execution intervals when supported and enabled.

Availability and behavior depend on device features and backend support.

## 11.6 Profiling tools

Useful tools may include:

- RenderDoc.
- AMD Radeon GPU Profiler.
- AMD GPUOpen performance tools.
- Vulkan validation layers.
- Rust profiling tools.

Tool compatibility depends on the GPU, driver, API, and operating system.

Verify support before relying on a particular profiler.

## 11.7 Performance models

Use the Roofline model to reason about bottlenecks.

Estimate:

\[
I=\frac{\text{FLOPs}}{\text{Bytes transferred}}
\]

Then compare expected compute and memory limits.

## Exercises

1. Why is CPU timing insufficient for GPU execution?
2. What is GPU timestamp profiling?
3. Why is warm-up important?
4. Why are repeated measurements necessary?
5. Why should end-to-end performance be measured?
6. What is arithmetic intensity?
7. How does the Roofline model help?
8. Why must profiling tool compatibility be verified?

---

# 12. Major Coding Assessment A: GPU Compute Framework

Build a reusable Rust GPU compute framework.

## Required features

- GPU initialization.
- Adapter selection.
- Buffer creation.
- Buffer uploads.
- Buffer readback.
- Shader loading.
- Compute pipeline creation.
- Resource binding.
- Dispatch.
- Error handling.
- Benchmarking.
- Result validation.

## Engineering requirements

- Clear module boundaries.
- Reusable abstractions.
- Minimal duplicated setup.
- Documented resource ownership.
- Explicit synchronization.
- Automated tests.

## Technical defense

Explain:

- How resources are created.
- How data reaches the GPU.
- How shaders access buffers.
- How commands are submitted.
- How results are synchronized.
- How errors are handled.
- How execution time is measured.

---

# 13. Major Coding Assessment B: GPU Parallel Primitives

Implement a GPU primitives library.

## Required algorithms

1. Vector addition.
2. Element-wise multiplication.
3. Reduction.
4. Inclusive scan.
5. Exclusive scan.
6. Dot product.

## Requirements

- CPU reference implementations.
- Arbitrary input sizes.
- Correct bounds handling.
- Numerical validation.
- Configurable workgroup sizes.
- Performance benchmarks.
- Documentation of limitations.

## Technical defense

Explain:

- Parallel decomposition.
- Workgroup organization.
- Memory access patterns.
- Synchronization.
- Numerical error.
- Expected bottlenecks.
- Performance scaling.

---

# 14. Major Coding Assessment C: Matrix Multiplication Optimization

Build a GPU matrix multiplication benchmark suite.

## Required implementations

- Naive matrix multiplication.
- Tiled matrix multiplication.
- At least three tile-size configurations.
- CPU reference implementation.

## Required experiments

Vary:

- Matrix dimensions.
- Tile size.
- Workgroup size.
- Input distributions.

Measure:

- Correctness.
- GPU execution time.
- End-to-end time.
- FLOPs per second.
- Arithmetic intensity estimates.

## Required analysis

Explain:

- Why tiling improves performance.
- Why some tile sizes perform worse.
- How workgroup memory affects occupancy.
- How register pressure may matter.
- How memory access patterns affect throughput.
- Why performance changes with matrix dimensions.

---

# 15. Final Capstone: Rust GPU Computing Library

Build a reusable Rust GPU computing library using wgpu and WGSL.

## Required capabilities

### GPU infrastructure

- Device initialization.
- Resource management.
- Pipeline management.
- Command submission.
- Synchronization.
- Readback.

### Compute algorithms

- Vector operations.
- Reductions.
- Prefix scans.
- Matrix multiplication.

### Correctness

- CPU reference implementations.
- Numerical error analysis.
- Edge-case tests.
- Arbitrary-length support.
- Reproducible test inputs.

### Performance

- GPU timing.
- End-to-end timing.
- Throughput measurement.
- Workgroup-size experiments.
- Tile-size experiments.
- CPU versus GPU comparisons.

## Required reports

### Report 1: Architecture

Explain the library's design and resource ownership model.

### Report 2: Correctness

Explain how results are validated and how numerical tolerances are selected.

### Report 3: Performance

Explain the relationship between:

- Arithmetic intensity.
- Memory bandwidth.
- Occupancy.
- Synchronization.
- Workgroup size.
- Data transfers.

### Report 4: GPU Architecture Connections

Explain how the implementation maps onto:

- Compute units.
- Wavefronts.
- Registers.
- Workgroup memory.
- Global memory.
- GPU scheduling.

---

# 16. Phase 8 Quiz Bank

## GPU software stack

1. What is wgpu?
2. What is WGSL?
3. What is a GPU adapter?
4. What is a GPU device?
5. What is a GPU queue?
6. Why is GPU execution asynchronous?
7. What is a GPU backend?
8. Why does device capability discovery matter?

## Buffers

9. What is a GPU buffer?
10. What is a storage buffer?
11. What is a uniform buffer?
12. What are buffer usage flags?
13. Why does alignment matter?
14. Why are host-device transfers expensive?
15. What is buffer mapping?
16. Why must readback be synchronized?

## WGSL

17. What is a compute shader?
18. What is a compute entry point?
19. What is a workgroup?
20. What is a global invocation ID?
21. How is dispatch size calculated?
22. Why is bounds checking necessary?
23. What is a storage binding?
24. What is workgroup size?

## Pipelines

25. What is a shader module?
26. What is a bind group layout?
27. What is a bind group?
28. What is a compute pipeline?
29. What is a command encoder?
30. What is a compute pass?
31. What is command submission?
32. Why does synchronization matter?

## Workgroup memory

33. What is workgroup memory?
34. Why is workgroup memory useful?
35. What is a workgroup barrier?
36. Why must barriers be used carefully?
37. How does workgroup memory affect occupancy?
38. Why can synchronization reduce performance?

## Reductions

39. What is a GPU reduction?
40. What is hierarchical reduction?
41. Why are multiple dispatches sometimes required?
42. Why does reduction order affect floating-point results?
43. Why does workgroup size affect reduction performance?
44. Why does memory coalescing matter?

## Scans

45. What is an inclusive scan?
46. What is an exclusive scan?
47. Why is scan harder than reduction?
48. What is a hierarchical scan?
49. Why are scans useful for stream compaction?
50. Why do scans require careful synchronization?

## Matrix multiplication

51. What is naive GPU matrix multiplication?
52. What is tiled matrix multiplication?
53. Why does tiling improve data reuse?
54. What is arithmetic intensity?
55. How does tile size affect occupancy?
56. Why does memory layout matter?
57. Why do floating-point results differ across implementations?
58. Why does matrix size affect performance?

## Benchmarking

59. Why is CPU timing insufficient for GPU kernels?
60. What is GPU timestamp profiling?
61. Why is warm-up important?
62. Why are repeated measurements necessary?
63. What is end-to-end timing?
64. What is the Roofline model?
65. Why must profiling tool compatibility be verified?
66. Why can a GPU kernel be faster while the application is slower?

---

# 17. Final Examination

**Recommended duration:** 6–8 hours.

| Section | Weight |
|---|---:|
| GPU software stack | 10% |
| Buffers and memory | 15% |
| WGSL and compute pipelines | 15% |
| Workgroup memory and synchronization | 15% |
| Reductions and scans | 15% |
| Matrix multiplication | 15% |
| Benchmarking and performance | 15% |

## Part A: GPU software stack

Explain the complete path from Rust application code to GPU execution.

## Part B: Buffer management

Design a buffer management strategy for a hypothetical compute workload.

## Part C: Shader execution

Analyze workgroup sizing, dispatch counts, and bounds checking.

## Part D: Synchronization

Explain workgroup barriers and multi-dispatch coordination.

## Part E: Parallel algorithms

Design GPU reductions and scans.

## Part F: Matrix multiplication

Explain naive and tiled matrix multiplication.

## Part G: Performance

Analyze GPU execution time, memory traffic, arithmetic intensity, and transfer overhead.

## Part H: Technical defense

Defend your Rust GPU computing library.

---

# 18. Copy-Ready AI Tutor Prompts

All prompts are consolidated here.

The tutor must never provide answers, code, pseudocode, implementations, fixes, or completed work.

## Master Socratic Tutor Prompt

```text id="b8p4x2"
You are my Socratic GPU programming tutor.

I am studying Phase 8 of a rigorous GPU engineering curriculum.

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

10. Require me to explain GPU execution, memory access,
    synchronization, numerical correctness, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness,
    resource ownership, synchronization, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only guiding questions.
```

## Start a lesson

```text id="c7n5v3"
Tutor me through Phase 8, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to explain GPU execution and resource management.
Never provide answers or solutions.
```

## Generate a quiz

```text id="m2r8q4"
Administer a 20-question Phase 8 quiz on [TOPIC].

Include conceptual reasoning, GPU memory,
shader execution, synchronization, and performance.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text id="t6h3p9"
Administer Phase 8 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my design and correctness argument.

Guide me through questions about resource ownership,
shader execution, synchronization, numerical validation,
and performance.

Do not write code, pseudocode, or implementations.
```

## Review GPU code

```text id="k4x7d1"
Review my GPU code using only questions.

Focus on:
- Buffer usage
- Resource bindings
- Dispatch sizing
- Bounds checking
- Workgroup synchronization
- Memory access patterns
- Numerical correctness
- Performance

Do not provide fixes or replacement code.
Ask one guiding question at a time.
```

## GPU debugging examination

```text id="v9m2s5"
Help me debug my GPU program using only questions.

Ask me to investigate assumptions, resource states,
dispatch dimensions, synchronization, buffer layouts,
and numerical results.

Never identify the exact fix.
Never write code.
```

## Performance investigation

```text id="q3b8n6"
Help me investigate GPU performance using only questions.

Ask about:
- GPU execution time
- End-to-end time
- Memory bandwidth
- Arithmetic intensity
- Occupancy
- Workgroup size
- Tile size
- Synchronization
- Data transfers

Do not interpret results for me or provide conclusions.
```

---

# 19. Mastery Checklist

## Environment

- [ ] I can initialize wgpu.
- [ ] I can select a GPU adapter.
- [ ] I can create a device and queue.
- [ ] I can inspect device limits.
- [ ] I understand the GPU software stack.

## Memory

- [ ] I can create GPU buffers.
- [ ] I understand buffer usage flags.
- [ ] I can upload data.
- [ ] I can read data back.
- [ ] I understand alignment.
- [ ] I understand synchronization.

## WGSL

- [ ] I can write compute shaders.
- [ ] I understand workgroups.
- [ ] I understand global invocation IDs.
- [ ] I can calculate dispatch sizes.
- [ ] I can implement bounds checking.
- [ ] I understand storage buffer bindings.

## Pipelines

- [ ] I can create shader modules.
- [ ] I can create bind group layouts.
- [ ] I can create bind groups.
- [ ] I can create compute pipelines.
- [ ] I can encode commands.
- [ ] I can submit GPU work.

## Parallel algorithms

- [ ] I can implement GPU vector operations.
- [ ] I can implement GPU reductions.
- [ ] I can implement inclusive scans.
- [ ] I can implement exclusive scans.
- [ ] I can implement naive matrix multiplication.
- [ ] I can implement tiled matrix multiplication.

## Performance

- [ ] I can measure GPU execution time.
- [ ] I can measure end-to-end time.
- [ ] I understand warm-up effects.
- [ ] I can calculate arithmetic intensity.
- [ ] I can analyze memory bandwidth.
- [ ] I can compare CPU and GPU performance.
- [ ] I can explain workgroup-size trade-offs.
- [ ] I can explain tile-size trade-offs.

## Capstone

- [ ] My Rust GPU computing library is functional.
- [ ] My implementations are validated.
- [ ] My benchmarks are reproducible.
- [ ] My documentation explains design decisions.
- [ ] I can defend correctness and performance.

---

# 20. Recommended Resources

## wgpu

**wgpu Documentation**

https://docs.rs/wgpu/

**wgpu Examples**

https://github.com/gfx-rs/wgpu/tree/trunk/examples

## WGSL

**WebGPU Shading Language Specification**

https://www.w3.org/TR/WGSL/

**WebGPU Specification**

https://www.w3.org/TR/webgpu/

## Rust GPU programming

**Learn wgpu**

https://sotrh.github.io/learn-wgpu/

Although much of this material focuses on graphics, the resource-management concepts are useful for compute programming.

## GPU architecture and performance

**AMD GPUOpen**

https://gpuopen.com/

**Programming Massively Parallel Processors — Kirk and Hwu**

A foundational GPU programming textbook.

## Numerical validation

**Accuracy and Stability of Numerical Algorithms — Nicholas J. Higham**

Useful for understanding floating-point error in GPU algorithms.

---

# 21. Completion Standard

You have completed Phase 8 when you can independently:

1. Initialize and manage a GPU through wgpu.
2. Write and execute WGSL compute shaders.
3. Manage GPU buffers and bindings.
4. Dispatch workgroups correctly.
5. Synchronize GPU work.
6. Implement reductions and scans.
7. Implement tiled matrix multiplication.
8. Validate numerical results.
9. Benchmark GPU execution.
10. Explain performance bottlenecks.
11. Build and defend a reusable Rust GPU computing library.

This is the first phase where you should have a substantial collection of working GPU programs.

---

# 22. What Comes Next

**Phase 9: CUDA Programming**

The next phase will focus on NVIDIA's CUDA programming model.

You will study:

- CUDA C++.
- CUDA kernels.
- Thread blocks and grids.
- Warps.
- Shared memory.
- CUDA synchronization.
- CUDA streams.
- Asynchronous execution.
- CUDA memory management.
- CUDA reductions and scans.
- CUDA matrix multiplication.
- CUDA profiling.
- NVIDIA-specific optimization.

Your AMD Radeon RX 9070 cannot execute CUDA kernels natively. You will need access to an appropriate NVIDIA GPU, either locally or through a remote development environment.
