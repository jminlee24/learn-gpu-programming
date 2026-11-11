# Phase 9: CUDA Programming

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 9 of 16  
**Estimated duration:** 14–18 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** C++20 and CUDA C++  
**Supporting language:** Rust  
**Platform:** Fedora Linux for local development; NVIDIA-compatible Linux environment for execution  
**Local GPU:** AMD Radeon RX 9070, 16 GB  
**Required execution hardware:** Supported NVIDIA GPU, locally or remotely  
**Prerequisites:** Phases 0–8

---

# 1. Purpose and Learning Objectives

CUDA provides direct access to NVIDIA's GPU programming ecosystem.

Unlike portable APIs such as wgpu, CUDA exposes NVIDIA-specific programming constructs, hardware capabilities, runtime behavior, and optimization facilities.

Understanding CUDA is important even if you ultimately specialize in portable GPU computing, AMD GPUs, or Rust GPU runtimes.

CUDA provides a widely used reference point for:

- GPU execution models.
- High-performance numerical computing.
- Memory hierarchy optimization.
- Asynchronous execution.
- Parallel algorithms.
- GPU runtime design.
- Compiler-generated GPU code.
- Performance analysis.

By the end of this phase, you should be able to:

1. Explain the CUDA software stack.
2. Write CUDA C++ kernels.
3. Understand grids, blocks, threads, and warps.
4. Calculate thread indices and launch dimensions.
5. Manage device and host memory.
6. Explain global, shared, local, constant, and register storage.
7. Implement coalesced memory access.
8. Use shared memory and barriers correctly.
9. Implement parallel reductions and scans.
10. Understand CUDA atomics and memory ordering.
11. Use streams and events.
12. Understand asynchronous transfers.
13. Explain pinned memory and unified memory.
14. Implement tiled matrix multiplication.
15. Understand warp-level primitives.
16. Use CUDA profiling tools.
17. Analyze occupancy, divergence, and register pressure.
18. Compare CUDA with WGSL/wgpu.
19. Build a reusable CUDA computing library.
20. Defend performance decisions using measurements and architectural reasoning.

## Central question

**How can a programmer express parallel work so that NVIDIA GPU hardware executes it correctly and efficiently?**

---

# 2. Hardware Strategy

Your Radeon RX 9070 is suitable for continuing your portable GPU work, but it does not provide native CUDA execution.

You have three practical study modes.

| Mode | What you can do |
|---|---|
| Local Fedora without NVIDIA hardware | Read CUDA code, derive algorithms, study PTX, write architecture notes |
| Remote NVIDIA GPU | Compile, execute, test, benchmark, and profile CUDA kernels |
| Local NVIDIA GPU | Full CUDA development and profiling |

For this phase, actual NVIDIA execution is strongly recommended.

A remote GPU is sufficient for the majority of the curriculum if you can install or use an appropriate CUDA toolkit and profiling tools.

Do not assume that a CUDA installation alone makes CUDA kernels executable on AMD hardware.

## Hardware verification

Before selecting a remote environment, verify:

- GPU model.
- Compute capability.
- Driver version.
- CUDA toolkit compatibility.
- Available device memory.
- Profiling permissions.
- Whether multiple users share the GPU.
- Whether the environment permits long-running benchmarks.

Record the environment in your laboratory notebook.

---

# 3. Study Schedule

| Module | Subject | Time |
|---|---|---|
| 1 | CUDA software stack and first kernel | 1–2 weeks |
| 2 | CUDA execution model | 2 weeks |
| 3 | Device memory and transfers | 2 weeks |
| 4 | Shared memory and synchronization | 2 weeks |
| 5 | Reductions, scans, and atomics | 2–3 weeks |
| 6 | Streams and asynchronous execution | 1–2 weeks |
| 7 | Matrix multiplication and warp primitives | 2–3 weeks |
| 8 | Profiling, optimization, and capstone | 2–3 weeks |

Maintain a laboratory notebook with:

- Hypotheses.
- Device specifications.
- Kernel configurations.
- Correctness results.
- Timing distributions.
- Performance explanations.
- Failed experiments.
- Revised hypotheses.

---

# 4. Module One: The CUDA Software Stack

## 4.1 CUDA C++

CUDA extends C++ with constructs for expressing GPU execution.

CUDA source files commonly use the `.cu` extension.

The CUDA toolchain compiles host-side code and GPU device code.

## 4.2 CUDA runtime

The CUDA runtime provides functionality for:

- Device discovery.
- Memory allocation.
- Memory transfers.
- Kernel launches.
- Streams.
- Events.
- Synchronization.
- Error reporting.

## 4.3 CUDA driver

The CUDA driver provides lower-level functionality for interacting with NVIDIA GPUs.

The runtime API is generally more convenient for introductory development.

## 4.4 PTX

PTX stands for Parallel Thread Execution.

It is NVIDIA's virtual GPU instruction-set representation.

CUDA device code may be compiled into PTX before further translation into hardware-specific instructions.

PTX is useful for studying:

- Thread indexing.
- Memory operations.
- Register usage.
- Control flow.
- Atomic operations.
- Compiler transformations.

PTX is not identical to the GPU's final machine instruction set.

## 4.5 SASS

SASS refers to NVIDIA GPU machine instructions.

Studying SASS can help explain how compiled kernels execute on a particular architecture.

The relationship between CUDA C++, PTX, and SASS is not always one-to-one.

## 4.6 Toolchain

A typical CUDA development environment includes:

- NVIDIA GPU driver.
- CUDA toolkit.
- `nvcc`.
- CUDA runtime libraries.
- Profiling tools.

Verify an available toolkit:

```bash
nvcc --version
```

Verify an accessible NVIDIA GPU:

```bash
nvidia-smi
```

These commands answer different questions.

`nvcc` checks the compiler installation.

`nvidia-smi` checks whether the NVIDIA driver can communicate with a supported GPU.

Neither command alone proves that a particular application will execute successfully.

## 4.7 First CUDA program

Consider a simple vector addition kernel:

```cpp
#include <cuda_runtime.h>

__global__ void vector_add(
    const float* a,
    const float* b,
    float* c,
    int n
) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;

    if (i < n) {
        c[i] = a[i] + b[i];
    }
}
```

The `__global__` qualifier identifies a kernel callable from the host and executed on the GPU.

Each thread calculates its own global index.

The bounds check prevents threads beyond the logical input length from accessing invalid elements.

A kernel launch might use:

```cpp
int threads_per_block = 256;
int blocks = (n + threads_per_block - 1) / threads_per_block;

vector_add<<<blocks, threads_per_block>>>(
    device_a,
    device_b,
    device_c,
    n
);
```

The launch syntax specifies the execution grid.

The example assumes valid device pointers, successful allocation, and an appropriate CUDA environment.

It omits the surrounding error handling and resource management.

## Worked example: launch dimensions

Suppose:

\[
N=1000
\]

and:

\[
B=256
\]

threads per block.

The number of blocks is:

\[
G=\left\lceil\frac{1000}{256}\right\rceil=4
\]

The launch creates:

\[
4\times256=1024
\]

logical threads.

The final 24 threads have indices outside the valid input range and must not access the arrays.

## Exercises

1. Explain the CUDA runtime.
2. Explain the CUDA driver.
3. What is PTX?
4. What is SASS?
5. What does `__global__` mean?
6. What is `threadIdx.x`?
7. What is `blockIdx.x`?
8. What is `blockDim.x`?
9. Derive the one-dimensional global thread index.
10. Why must kernels handle excess threads?

## Lab 1: CUDA Environment and Vector Addition

Implement a complete CUDA vector addition program.

Requirements:

- Discover the device.
- Allocate host and device memory.
- Initialize inputs.
- Transfer inputs.
- Launch the kernel.
- Transfer results.
- Validate against a CPU reference.
- Check CUDA errors.
- Release resources.

Test input lengths that are smaller than, equal to, and larger than one block.

---

# 5. Module Two: CUDA Execution Model

## 5.1 Threads

A CUDA thread is one logical execution instance of a kernel.

Threads may access their own registers and other memory spaces.

## 5.2 Thread blocks

Threads are grouped into blocks.

Threads within a block can cooperate using shared memory and block-level synchronization.

## 5.3 Grids

A grid contains the blocks launched for a kernel.

Grids and blocks can be organized in one, two, or three dimensions.

## 5.4 Thread indexing

For a two-dimensional launch, an element's coordinates may be calculated as:

\[
x=\text{blockIdx.x}\cdot\text{blockDim.x}+\text{threadIdx.x}
\]

\[
y=\text{blockIdx.y}\cdot\text{blockDim.y}+\text{threadIdx.y}
\]

For a row-major array of width \(W\):

\[
i=yW+x
\]

The programmer must ensure the index is within the array bounds.

## 5.5 Warps

A CUDA warp consists of 32 threads.

Warp execution is a central feature of NVIDIA GPU architecture.

However, programmers should not assume all warp operations are automatically synchronized without using the required synchronization semantics.

## 5.6 Warp divergence

When threads within a warp follow different control-flow paths, execution may require different active-thread masks.

This can reduce effective throughput.

## 5.7 Block scheduling

Thread blocks are scheduled onto streaming multiprocessors according to resource availability and hardware scheduling behavior.

Ordinary CUDA kernels should not assume a particular execution order among blocks.

## 5.8 Resource limits

The number of concurrently resident blocks may be constrained by:

- Registers.
- Shared memory.
- Threads per block.
- Maximum resident blocks.
- Architecture-specific limits.

## 5.9 Occupancy

Occupancy describes the ratio of active resident warps to the maximum supported resident warps on an SM.

Occupancy can affect latency hiding.

Maximum occupancy does not necessarily produce maximum performance.

## Worked example: two-dimensional indexing

Suppose an image has dimensions:

\[
W=1920,\quad H=1080
\]

and each block contains:

\[
16\times16=256
\]

threads.

The grid dimensions are:

\[
G_x=\left\lceil\frac{1920}{16}\right\rceil=120
\]

\[
G_y=\left\lceil\frac{1080}{16}\right\rceil=68
\]

The launched grid covers 1920 columns and 1088 rows.

Threads corresponding to the extra eight rows must be excluded through bounds checking.

## Exercises

1. Explain CUDA grids.
2. Explain thread blocks.
3. Explain warps.
4. Derive two-dimensional indexing.
5. Explain block scheduling.
6. Explain why blocks cannot assume execution order.
7. Explain occupancy.
8. Explain register pressure.
9. Explain warp divergence.
10. Explain why block size affects performance.

## Lab 2: Two-Dimensional GPU Processing

Implement a CUDA kernel for a two-dimensional numerical array.

Requirements:

- Two-dimensional launch configuration.
- Correct row-major indexing.
- Arbitrary width and height.
- Boundary handling.
- CPU reference validation.
- Multiple block configurations.

Compare one-dimensional and two-dimensional mappings.

---

# 6. Module Three: CUDA Memory Hierarchy

## 6.1 Global memory

Global memory is accessible across threads and blocks.

It typically resides in GPU device memory.

Its performance depends on access patterns, caching, and hardware characteristics.

## 6.2 Registers

Registers store thread-private values.

Register usage influences occupancy and may affect instruction scheduling.

## 6.3 Shared memory

Shared memory is a fast storage space shared among threads within a block.

It is commonly used for:

- Tiling.
- Reductions.
- Scans.
- Data reuse.
- Cooperative computation.

## 6.4 Local memory

CUDA local memory is logically private to a thread.

Despite its name, it is not equivalent to on-chip shared memory.

It may be backed by device memory and cached according to the architecture.

Compiler decisions such as register spilling can cause local-memory traffic.

## 6.5 Constant memory

Constant memory provides a read-only address space with specialized caching behavior.

It can be effective for certain access patterns, particularly when threads access common locations.

## 6.6 Memory coalescing

A warp's memory requests may be combined into memory transactions.

Contiguous access patterns often improve transaction efficiency.

For example:

```cpp
int i = blockIdx.x * blockDim.x + threadIdx.x;
float value = input[i];
```

Neighboring threads typically access neighboring elements.

## 6.7 Strided access

Consider:

```cpp
int i = blockIdx.x * blockDim.x + threadIdx.x;
float value = input[i * stride];
```

For large strides, accesses may span many memory transactions.

## 6.8 Structure of arrays versus array of structures

Consider particles with position and velocity.

An array-of-structures layout stores complete particle records together.

A structure-of-arrays layout stores each field in a separate array.

The better layout depends on which fields neighboring threads access.

## 6.9 Host and device memory

CUDA supports several memory-management mechanisms.

Common categories include:

- Explicit device allocation.
- Pageable host memory.
- Pinned host memory.
- Managed memory.

These mechanisms have different performance and synchronization implications.

## 6.10 Pinned memory

Pinned host memory can support efficient asynchronous transfers under appropriate conditions.

However, excessive pinned-memory allocation can negatively affect the host system.

## 6.11 Managed memory

Managed memory provides a unified programming abstraction over memory accessible to the CPU and GPU.

It does not eliminate the cost of data movement or page migration.

Performance still depends on access patterns and residency.

## Exercises

1. Explain global memory.
2. Explain registers.
3. Explain shared memory.
4. Explain CUDA local memory.
5. Explain constant memory.
6. What is coalescing?
7. Why can strided access be expensive?
8. Compare AoS and SoA.
9. What is pinned memory?
10. What is managed memory?
11. Why can managed memory still incur transfer overhead?
12. Why does register spilling matter?

## Lab 3: Memory Access Benchmark

Implement kernels with:

- Contiguous accesses.
- Strided accesses.
- Different alignment conditions.
- AoS-style data access.
- SoA-style data access.

Measure effective bandwidth.

Explain the observed differences using memory transactions and cache behavior.

---

# 7. Module Four: Shared Memory and Synchronization

## 7.1 Shared-memory declaration

CUDA supports statically allocated shared memory.

For example:

```cpp
__shared__ float tile[256];
```

The allocation is shared among threads in the block.

## 7.2 Block synchronization

CUDA provides:

```cpp
__syncthreads();
```

This primitive provides block-level synchronization and associated memory-ordering guarantees.

It must be used according to CUDA's participation and control-flow rules.

## 7.3 Why barriers matter

Suppose threads cooperatively load data into shared memory.

A thread must not consume another thread's data before that write is appropriately synchronized.

## 7.4 Shared-memory bank conflicts

Shared memory is organized into banks.

Certain simultaneous access patterns can create bank conflicts.

Bank conflicts may serialize or otherwise reduce the efficiency of accesses, depending on the architecture and access pattern.

## 7.5 Tiling

Tiling allows threads to reuse data loaded into shared memory.

A tile may be processed through repeated phases:

- Load.
- Synchronize.
- Compute.
- Synchronize before reuse.

The exact synchronization requirements depend on the algorithm.

## 7.6 Barrier limitations

Ordinary `__syncthreads()` synchronizes threads within one block.

It does not provide a general grid-wide barrier.

Global synchronization commonly requires separate kernel launches or specialized execution mechanisms with additional constraints.

## Worked example: shared-memory reuse

Suppose 256 threads each need the same 256 values.

A naive design might repeatedly read the same values from global memory.

A cooperative design can load the values into shared memory and reuse them.

The benefit depends on:

- The number of reused values.
- Global-memory cache behavior.
- Shared-memory access cost.
- Synchronization overhead.
- Register and shared-memory pressure.

Shared memory is not automatically faster in every case.

## Exercises

1. What is shared memory?
2. What does `__syncthreads()` provide?
3. Why are barriers necessary?
4. What is a bank conflict?
5. What is tiling?
6. Why can shared-memory usage reduce occupancy?
7. Why are grid-wide barriers difficult?
8. Why can excessive synchronization hurt performance?

## Lab 4: Shared-Memory Tiling

Implement a tiled numerical operation.

Compare:

- Direct global-memory access.
- Shared-memory tiling.
- Multiple tile sizes.

Measure performance and explain the effects of data reuse, synchronization, and occupancy.

---

# 8. Module Five: Reductions, Scans, and Atomics

## 8.1 GPU reductions

A reduction combines many values into one result.

For example:

\[
S=\sum_{i=0}^{N-1}x_i
\]

A common CUDA implementation uses block-local reductions followed by additional aggregation.

## 8.2 Work and span

A balanced tree reduction over \(N\) elements has:

\[
W=O(N)
\]

\[
S=O(\log N)
\]

under the abstract parallel model.

Real GPU execution additionally depends on scheduling, memory access, and synchronization.

## 8.3 Hierarchical reduction

A typical hierarchical design produces one partial result per block.

A later stage combines partial results.

## 8.4 Atomic operations

CUDA supports atomic operations on supported types and memory spaces.

Atomics are useful for:

- Counters.
- Histograms.
- Work queues.
- Certain reductions.

However, contention can limit performance.

## 8.5 Atomic correctness

Atomicity does not automatically establish all required ordering between unrelated memory operations.

Correct algorithms must use the appropriate synchronization and memory-ordering semantics.

## 8.6 Prefix scans

A scan produces prefix aggregates.

For example:

\[
[2,3,5,7]
\]

has inclusive sum scan:

\[
[2,5,10,17]
\]

A hierarchical scan may use:

- Block-local scans.
- Block totals.
- Scans of block totals.
- Offset propagation.

## 8.7 Warp-level primitives

CUDA provides warp-level operations for communication and collective computation.

These include shuffle and vote operations.

For example, shuffle operations can exchange values between participating lanes.

Correct use requires attention to:

- Active-lane masks.
- Participation.
- Source-lane validity.
- Synchronization assumptions.

## 8.8 Numerical reproducibility

Parallel floating-point reductions may produce different results depending on accumulation order.

Different block sizes can change the reduction tree.

Numerical validation should distinguish expected rounding differences from algorithmic errors.

## Exercises

1. Derive reduction work and span.
2. Explain hierarchical reduction.
3. Explain the purpose of atomics.
4. Why can atomic contention hurt performance?
5. Why does atomicity not guarantee complete algorithm correctness?
6. Explain inclusive scan.
7. Explain exclusive scan.
8. Explain block-local scans.
9. Explain warp shuffles.
10. Why does floating-point reduction order matter?

## Lab 5: CUDA Reduction Library

Implement:

- Sum.
- Minimum.
- Maximum.
- Dot product.

Compare:

- Global atomic aggregation.
- Hierarchical block reductions.
- Different block sizes.

Validate against CPU reference implementations.

## Lab 6: CUDA Prefix Scan

Implement inclusive and exclusive scans.

Requirements:

- Arbitrary input lengths.
- Multi-block support.
- Correct synchronization.
- CPU validation.
- Performance benchmarking.

---

# 9. Module Six: CUDA Streams and Asynchronous Execution

## 9.1 Why asynchronous execution matters

CPU and GPU operations can overlap.

A program may:

- Prepare data on the CPU.
- Transfer data.
- Execute a kernel.
- Transfer results.

Careful scheduling can reduce idle time.

## 9.2 CUDA streams

A CUDA stream represents an ordered sequence of submitted operations.

Operations in the same stream obey the relevant stream-ordering guarantees.

Operations in different streams may overlap when dependencies, hardware capabilities, and resource availability permit.

## 9.3 CUDA events

CUDA events can record points in stream execution.

They are useful for:

- Timing.
- Synchronization.
- Dependency management.

## 9.4 Asynchronous copies

CUDA provides asynchronous memory-transfer APIs.

Actual overlap depends on conditions such as:

- Host memory properties.
- Hardware copy engines.
- Stream usage.
- Transfer direction.
- Workload dependencies.

## 9.5 Double buffering

Double buffering uses separate storage regions so one data chunk can be processed while another is prepared or transferred.

## 9.6 Pipeline overlap

Suppose a workload contains:

- Upload time \(T_H\).
- Kernel time \(T_K\).
- Download time \(T_D\).

A purely sequential pipeline takes approximately:

\[
T_{\text{serial}}=T_H+T_K+T_D
\]

per item, ignoring fixed overhead.

With sufficient independent work and supported overlap, steady-state throughput may approach a limit determined by the slowest pipeline stage rather than the sum of all stages.

This is an idealized bound, not a guarantee.

## 9.7 Synchronization mistakes

Common mistakes include:

- Reading results before completion.
- Reusing buffers too early.
- Assuming different streams automatically overlap.
- Adding unnecessary global synchronization.
- Measuring submission time instead of execution time.

## Exercises

1. What is a CUDA stream?
2. What is a CUDA event?
3. Why is asynchronous execution useful?
4. What conditions affect transfer overlap?
5. What is double buffering?
6. What is pipeline parallelism?
7. Why can synchronization reduce throughput?
8. Why do different streams not guarantee overlap?
9. How can CUDA events help measure execution?

## Lab 7: Streaming Pipeline

Build a pipeline that processes multiple data chunks.

Compare:

- Sequential transfer and execution.
- Stream-based execution.
- Double-buffered execution.

Measure total throughput and explain whether overlap occurs.

---

# 10. Module Seven: CUDA Matrix Multiplication

## 10.1 Matrix multiplication

For matrices:

\[
A\in\mathbb{R}^{M\times K}
\]

\[
B\in\mathbb{R}^{K\times N}
\]

the output is:

\[
C_{ij}=\sum_{k=0}^{K-1}A_{ik}B_{kj}
\]

## 10.2 Naive kernel

A naive kernel may assign one output element to each thread.

Each thread computes a dot product.

## 10.3 Tiled kernel

A tiled kernel cooperatively loads blocks of input matrices into shared memory.

Threads reuse the loaded values.

## 10.4 Arithmetic intensity

Arithmetic intensity is:

\[
I=\frac{\text{FLOPs}}{\text{Bytes transferred}}
\]

Tiling can increase arithmetic intensity by reducing repeated global-memory traffic.

## 10.5 Tile-size trade-offs

Larger tiles may improve reuse but increase:

- Shared-memory consumption.
- Register pressure.
- Synchronization complexity.
- Occupancy constraints.

## 10.6 Specialized matrix hardware

Many NVIDIA GPUs provide specialized matrix arithmetic capabilities.

Their supported types, shapes, and programming interfaces vary by architecture.

Advanced use may involve:

- Tensor Cores.
- Warp-level matrix operations.
- Architecture-specific instructions.
- Specialized libraries.

This phase introduces the concepts; later optimization work will study them more deeply.

## 10.7 cuBLAS

cuBLAS provides optimized linear algebra routines for NVIDIA GPUs.

It is a useful performance reference.

A hand-written introductory kernel should not be expected to match mature library performance.

## 10.8 Validation

Matrix multiplication validation should account for:

- Input dimensions.
- Leading dimensions and strides.
- Floating-point precision.
- Accumulation order.
- Numerical error.
- Edge cases.

## Exercises

1. Derive matrix multiplication.
2. Explain naive CUDA GEMM.
3. Explain tiled CUDA GEMM.
4. Why does tiling increase arithmetic intensity?
5. Why does tile size affect occupancy?
6. What are Tensor Cores?
7. What is cuBLAS?
8. Why may cuBLAS outperform a handwritten kernel?
9. Why does numerical validation require tolerances?

## Lab 8: CUDA GEMM

Implement:

- Naive GEMM.
- Tiled GEMM.
- Multiple tile sizes.
- CPU reference validation.
- Performance benchmarking.

Compare against cuBLAS where available.

---

# 11. Module Eight: CUDA Profiling and Performance Engineering

## 11.1 Performance measurement

A CUDA benchmark should distinguish:

- Host preparation.
- Allocation.
- Data transfer.
- Kernel execution.
- Synchronization.
- Readback.
- Total application time.

## 11.2 CUDA events

CUDA events can measure elapsed GPU execution intervals.

They are generally more appropriate than timing asynchronous kernel submission with an ordinary CPU timer.

## 11.3 Nsight Systems

Nsight Systems is used for system-level performance investigation.

It can help examine:

- CPU activity.
- GPU activity.
- Kernel launches.
- Memory transfers.
- Stream behavior.
- Synchronization.

## 11.4 Nsight Compute

Nsight Compute provides kernel-level performance analysis on supported hardware.

It can help investigate:

- Memory throughput.
- Occupancy.
- Instruction execution.
- Warp behavior.
- Resource utilization.

Tool support varies by GPU generation and environment.

## 11.5 Roofline analysis

The Roofline model estimates:

\[
P\leq\min(P_{\text{peak}},B_{\text{peak}}I)
\]

where:

- \(P_{\text{peak}}\) is peak compute throughput.
- \(B_{\text{peak}}\) is peak memory bandwidth.
- \(I\) is arithmetic intensity.

## 11.6 Register pressure

High register usage may reduce occupancy.

Register spilling may introduce additional memory traffic.

## 11.7 Warp divergence

Divergent control flow can reduce effective execution-lane utilization.

## 11.8 Memory bandwidth

Memory-bound kernels may saturate available bandwidth before fully utilizing arithmetic resources.

## 11.9 Benchmark methodology

For each experiment:

1. Validate correctness.
2. Warm up appropriately.
3. Repeat measurements.
4. Record device details.
5. Record kernel configuration.
6. Report timing distributions.
7. Compare against a baseline.
8. Explain likely bottlenecks.
9. Separate observation from hypothesis.
10. Reproduce important results.

## Exercises

1. What is Nsight Systems?
2. What is Nsight Compute?
3. What is occupancy?
4. What is register pressure?
5. What is warp divergence?
6. What is memory bandwidth saturation?
7. What is arithmetic intensity?
8. Why are CUDA events useful?
9. Why is warm-up important?
10. Why must performance claims be reproducible?

---

# 12. Major Assessment A: CUDA Parallel Primitives Library

Build a CUDA C++ library implementing:

1. Vector addition.
2. Element-wise multiplication.
3. Dot product.
4. Sum reduction.
5. Minimum and maximum reduction.
6. Inclusive scan.
7. Exclusive scan.
8. Histogram.

## Requirements

- Correct CPU reference implementations.
- Arbitrary input sizes.
- Error handling.
- Automated tests.
- Multiple launch configurations.
- GPU timing.
- End-to-end timing.
- Numerical validation.

## Technical defense

Explain:

- Thread indexing.
- Work decomposition.
- Synchronization.
- Memory coalescing.
- Shared-memory use.
- Atomic contention.
- Numerical behavior.
- Expected scaling.

---

# 13. Major Assessment B: CUDA Execution Pipeline

Build a reusable asynchronous GPU processing pipeline.

## Requirements

- Multiple CUDA streams.
- CUDA events.
- Host-device transfers.
- Multiple independent data chunks.
- Correct dependency management.
- Optional pinned-memory experiments.
- Graceful error handling.
- Performance measurements.

## Required experiments

Compare:

- Sequential execution.
- Multiple streams.
- Double buffering.
- Different chunk sizes.

## Technical defense

Explain:

- Which operations can overlap.
- Which operations must remain ordered.
- Why overlap may not occur.
- How dependencies are enforced.
- How timing is measured.

---

# 14. Major Assessment C: CUDA GEMM Optimization

Build a matrix multiplication benchmark suite.

## Required implementations

- Naive GEMM.
- Shared-memory tiled GEMM.
- Multiple tile configurations.
- CPU reference implementation.

## Required comparison

Compare against cuBLAS where available.

## Required analysis

Discuss:

- Arithmetic intensity.
- Global-memory traffic.
- Shared-memory reuse.
- Register pressure.
- Occupancy.
- Synchronization.
- Numerical error.
- Performance scaling.

---

# 15. Final Capstone: CUDA Computing Runtime

Build a reusable CUDA C++ computing library.

## Required components

### Device infrastructure

- Device discovery.
- Device memory management.
- Error handling.
- Resource ownership.
- Stream management.
- Event management.

### Compute algorithms

- Vector operations.
- Reductions.
- Scans.
- Histograms.
- Matrix multiplication.

### Performance

- GPU timing.
- End-to-end timing.
- Throughput measurement.
- Scaling analysis.
- Occupancy investigation.
- Memory bandwidth investigation.

### Correctness

- CPU reference implementations.
- Numerical validation.
- Stress testing.
- Edge-case testing.
- Reproducible inputs.

## Required reports

1. Architecture document.
2. Correctness report.
3. Performance report.
4. CUDA versus WGSL comparison.
5. Hardware utilization analysis.

## CUDA versus WGSL comparison

Compare your Phase 8 and Phase 9 implementations.

Discuss:

- Execution models.
- Resource binding.
- Memory management.
- Synchronization.
- Portability.
- Profiling.
- Optimization opportunities.

Avoid claiming that differences in performance are caused solely by the programming API when the benchmarks run on different GPU hardware.

---

# 16. Phase 9 Quiz Bank

## CUDA fundamentals

1. What is CUDA?
2. What is CUDA C++?
3. What is the CUDA runtime?
4. What is the CUDA driver?
5. What is PTX?
6. What is SASS?
7. What does `__global__` mean?
8. What is a kernel launch?

## Execution model

9. What is a CUDA thread?
10. What is a thread block?
11. What is a grid?
12. What is a warp?
13. What is `threadIdx`?
14. What is `blockIdx`?
15. What is `blockDim`?
16. What is `gridDim`?
17. How is a global index calculated?
18. Why is bounds checking necessary?
19. What is warp divergence?
20. What limits occupancy?

## Memory

21. What is global memory?
22. What are registers?
23. What is shared memory?
24. What is CUDA local memory?
25. What is constant memory?
26. What is memory coalescing?
27. What is register spilling?
28. What is pinned memory?
29. What is managed memory?
30. Why do memory access patterns matter?

## Synchronization

31. What is `__syncthreads()`?
32. Why are barriers necessary?
33. What is a shared-memory bank conflict?
34. Why can shared-memory usage reduce occupancy?
35. Why are grid-wide barriers difficult?
36. What is an atomic operation?
37. Why can atomics create contention?
38. Why does atomicity not guarantee all synchronization requirements?

## Parallel algorithms

39. What is a hierarchical reduction?
40. Why do reductions use multiple stages?
41. What is an inclusive scan?
42. What is an exclusive scan?
43. Why are scans useful for stream compaction?
44. What are warp shuffle operations?
45. Why does floating-point reduction order matter?
46. Why are histograms challenging on GPUs?

## Streams

47. What is a CUDA stream?
48. What is a CUDA event?
49. Why is asynchronous execution useful?
50. What is double buffering?
51. What is pipeline overlap?
52. Why do multiple streams not guarantee overlap?
53. Why does pinned memory matter for some transfers?
54. Why can unnecessary synchronization hurt performance?

## Matrix multiplication

55. What is naive GEMM?
56. What is tiled GEMM?
57. Why does tiling improve reuse?
58. What is arithmetic intensity?
59. Why does tile size affect occupancy?
60. What are Tensor Cores?
61. What is cuBLAS?
62. Why may cuBLAS outperform a handwritten kernel?

## Profiling

63. What is Nsight Systems?
64. What is Nsight Compute?
65. What is occupancy?
66. What is register pressure?
67. What is memory bandwidth saturation?
68. What is the Roofline model?
69. Why are CUDA events useful for timing?
70. Why must benchmarks be reproducible?

---

# 17. Final Examination

**Recommended duration:** 6–8 hours.

| Section | Weight |
|---|---:|
| CUDA software stack | 10% |
| Execution model | 15% |
| Memory hierarchy | 20% |
| Synchronization and atomics | 15% |
| Reductions and scans | 15% |
| Streams and asynchronous execution | 10% |
| Matrix multiplication and profiling | 15% |

## Part A: CUDA execution

Analyze thread indexing, grids, blocks, and warp execution.

## Part B: Memory hierarchy

Analyze global-memory accesses, shared-memory use, and register pressure.

## Part C: Synchronization

Identify synchronization requirements in hypothetical kernels.

## Part D: Parallel primitives

Design reductions, scans, and histograms.

## Part E: Asynchronous execution

Analyze stream dependencies and potential overlap.

## Part F: Matrix multiplication

Explain tiled GEMM and its performance trade-offs.

## Part G: Technical defense

Defend your CUDA computing runtime.

---

# 18. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

The tutor must never provide answers, code, pseudocode, implementations, debugging fixes, or completed work.

## Master Socratic Tutor Prompt

```text
You are my Socratic CUDA programming tutor.

I am studying Phase 9 of a rigorous GPU engineering curriculum.

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

10. Require me to explain CUDA execution, memory hierarchy,
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

```text
Tutor me through Phase 9, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to explain CUDA execution and resource management.
Never provide answers or solutions.
```

## Generate a quiz

```text
Administer a 20-question Phase 9 quiz on [TOPIC].

Include conceptual reasoning, CUDA memory,
kernel execution, synchronization, and performance.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text
Administer Phase 9 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my design and correctness argument.

Guide me through questions about resource ownership,
kernel execution, synchronization, numerical validation,
and performance.

Do not write code, pseudocode, or implementations.
```

## Review CUDA code

```text
Review my CUDA code using only questions.

Focus on:
- Thread indexing
- Bounds checking
- Memory coalescing
- Shared-memory usage
- Synchronization
- Warp divergence
- Numerical correctness
- Performance

Do not provide fixes or replacement code.
Ask one guiding question at a time.
```

## CUDA debugging examination

```text
Help me debug my CUDA program using only questions.

Ask me to investigate assumptions, resource states,
launch dimensions, synchronization, memory access,
and numerical results.

Never identify the exact fix.
Never write code.
```

## Performance investigation

```text
Help me investigate CUDA performance using only questions.

Ask about:
- Kernel execution time
- End-to-end time
- Memory bandwidth
- Arithmetic intensity
- Occupancy
- Warp divergence
- Register pressure
- Shared-memory usage
- Stream overlap
- Data transfers

Do not interpret results for me or provide conclusions.
```

---

# 19. Mastery Checklist

## Environment

- [ ] I understand the CUDA software stack.
- [ ] I can compile CUDA C++.
- [ ] I can identify a CUDA-capable GPU.
- [ ] I understand driver and toolkit compatibility.
- [ ] I understand PTX and SASS conceptually.

## Execution

- [ ] I can write CUDA kernels.
- [ ] I understand grids and blocks.
- [ ] I understand thread indexing.
- [ ] I understand warps.
- [ ] I understand divergence.
- [ ] I understand occupancy.

## Memory

- [ ] I can allocate device memory.
- [ ] I can transfer data.
- [ ] I understand global memory.
- [ ] I understand shared memory.
- [ ] I understand registers.
- [ ] I understand local memory.
- [ ] I understand coalescing.
- [ ] I understand pinned and managed memory.

## Synchronization

- [ ] I understand block barriers.
- [ ] I understand atomics.
- [ ] I understand synchronization scope.
- [ ] I understand shared-memory bank conflicts.
- [ ] I understand cross-block synchronization limitations.

## Algorithms

- [ ] I can implement reductions.
- [ ] I can implement scans.
- [ ] I can implement histograms.
- [ ] I can implement naive GEMM.
- [ ] I can implement tiled GEMM.

## Asynchronous execution

- [ ] I understand CUDA streams.
- [ ] I understand CUDA events.
- [ ] I understand asynchronous transfers.
- [ ] I understand double buffering.
- [ ] I can reason about stream dependencies.

## Performance

- [ ] I can measure GPU execution time.
- [ ] I can measure end-to-end time.
- [ ] I understand occupancy trade-offs.
- [ ] I understand register pressure.
- [ ] I understand memory bandwidth limitations.
- [ ] I understand the Roofline model.
- [ ] I can explain profiling results.

## Capstone

- [ ] My CUDA computing library works.
- [ ] My implementations are validated.
- [ ] My benchmarks are reproducible.
- [ ] My documentation explains design decisions.
- [ ] I can defend correctness and performance.

---

# 20. Recommended Resources

## CUDA documentation

**CUDA C++ Programming Guide**

https://docs.nvidia.com/cuda/cuda-c-programming-guide/

**CUDA C++ Best Practices Guide**

https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/

**CUDA Runtime API Documentation**

https://docs.nvidia.com/cuda/cuda-runtime-api/

**PTX ISA Documentation**

https://docs.nvidia.com/cuda/parallel-thread-execution/

## NVIDIA profiling

**Nsight Systems**

https://developer.nvidia.com/nsight-systems

**Nsight Compute**

https://developer.nvidia.com/nsight-compute

## Textbooks

**Programming Massively Parallel Processors — David B. Kirk and Wen-mei W. Hwu**

**Professional CUDA C Programming — John Cheng, Max Grossman, and Ty McKercher**

## Parallel algorithms

**The Art of Multiprocessor Programming — Maurice Herlihy and Nir Shavit**

Useful for synchronization and concurrency reasoning.

---

# 21. Completion Standard

You have completed Phase 9 when you can independently:

1. Explain the CUDA execution model.
2. Write and execute CUDA kernels.
3. Manage GPU memory correctly.
4. Use shared memory and synchronization.
5. Implement reductions and scans.
6. Implement asynchronous pipelines.
7. Implement tiled matrix multiplication.
8. Validate numerical results.
9. Benchmark GPU execution.
10. Explain occupancy, divergence, and memory bottlenecks.
11. Compare CUDA with WGSL/wgpu.
12. Build and defend a reusable CUDA computing library.

The most important outcome is not memorizing CUDA syntax.

It is understanding how a native GPU programming environment exposes hardware capabilities and performance trade-offs.

---

# 22. What Comes Next

**Phase 10: GPU Optimization and Performance Engineering**

The next phase will focus on systematically improving GPU performance.

Topics include:

- Performance modeling.
- Roofline analysis.
- Memory bandwidth optimization.
- Occupancy tuning.
- Register pressure.
- Instruction-level parallelism.
- Warp divergence.
- Shared-memory optimization.
- Kernel fusion.
- Asynchronous execution.
- Profiling methodology.
- Autotuning.
- Benchmark reproducibility.
- Architecture-specific optimization.
- Performance regression testing.

You will move from writing correct GPU kernels to understanding how to make them fast.
