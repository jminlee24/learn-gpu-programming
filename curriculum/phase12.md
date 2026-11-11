# Phase 12: Native Rust GPU Programming — From Rust Source to GPU Machine Code

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 12 of 16  
**Estimated duration:** 16–22 weeks  
**Study commitment:** 10–12 hours per week  
**Primary language:** Rust  
**Supporting languages:** WGSL, C++20, SPIR-V assembly  
**Primary platform:** Fedora Linux  
**Primary GPU:** AMD Radeon RX 9070, 16 GB  
**Primary graphics/compute API:** Vulkan  
**Prerequisites:** Phases 0–11

---

# 1. Mission

In earlier phases, you wrote Rust programs that submitted GPU work through wgpu. The actual compute kernels were written in WGSL.

This phase investigates a different question:

**What does it take to write GPU programs in Rust itself, compile them for GPU execution, and integrate them with a low-level GPU runtime?**

That question has several distinct parts.

First, Rust source must be translated into a representation a GPU driver understands. Second, the code must obey restrictions imposed by GPU execution, memory, and synchronization. Third, host-side Rust must allocate resources, submit commands, and maintain correct ownership and lifetime relationships. Finally, the resulting program must be validated and benchmarked.

Native Rust GPU development is not one standardized technology. It encompasses experimental Rust-to-SPIR-V projects, vendor-specific compilation approaches, low-level Vulkan host programming, and Rust bindings to native GPU APIs.

For this phase, separate three layers:

1. **Host-side native Rust:** Stable Rust controlling Vulkan resources and command submission.
2. **Device-side Rust:** A supported Rust subset compiled for GPU execution, when the selected compiler and hardware support it.
3. **GPU intermediate representations:** SPIR-V and architecture-specific code connecting source programs to hardware.

The first layer is a reliable foundation for the phase. The second is an experimental specialization that must be verified against current toolchain compatibility.

## Learning objectives

By the end, you should be able to:

- Explain the host/device compilation boundary.
- Describe the relationship between Rust, LLVM, SPIR-V, Vulkan, and native GPU instructions.
- Understand Rust-to-SPIR-V toolchain constraints.
- Write a Vulkan compute host application in Rust.
- Enumerate Vulkan physical devices and queue families.
- Create buffers and manage device memory.
- Construct compute pipelines and descriptor sets.
- Record and submit command buffers.
- Use Vulkan synchronization correctly.
- Understand buffer ownership and lifetime requirements.
- Define GPU-compatible data layouts.
- Analyze alignment, padding, and ABI issues.
- Explain which Rust features are difficult to support on GPUs.
- Inspect SPIR-V modules.
- Understand shader capabilities and validation.
- Build safe Rust abstractions over unsafe GPU APIs.
- Compare Rust GPU approaches with WGSL and CUDA.
- Profile and benchmark native Vulkan compute.
- Build a reusable Rust GPU computing framework.

---

# 2. The Toolchain Reality

Before studying implementations, distinguish what is stable from what is experimental.

## 2.1 Stable Rust on the host

Rust is well suited for building GPU host applications.

You can use Rust to:

- Initialize Vulkan.
- Discover devices.
- Manage buffers.
- Compile or load shaders.
- Submit compute commands.
- Synchronize CPU and GPU work.
- Validate results.
- Construct safe resource-management abstractions.

Crates such as `ash` provide access to Vulkan.

Higher-level libraries such as `vulkano` offer different abstraction and safety trade-offs.

Their precise APIs and supported Vulkan versions evolve, so pin versions and use the documentation matching those versions.

## 2.2 Rust on the device

Compiling ordinary Rust code into a GPU shader is substantially harder.

GPU environments differ from conventional operating-system processes.

Common restrictions involve:

- Dynamic allocation.
- Standard library support.
- Thread-local assumptions.
- Stack behavior.
- Recursion.
- Panic handling.
- Exceptions and unwinding.
- Unsupported atomics.
- Function pointers.
- Target-specific instructions.
- Memory address spaces.

A Rust-to-GPU toolchain must define which language features and library facilities are supported.

## 2.3 Rust-GPU and SPIR-V

The Rust-GPU ecosystem has explored compiling Rust code into SPIR-V for GPU execution.

Its compatibility with specific Rust compiler versions, drivers, and platforms must be verified before use.

Do not assume that a current stable Rust compiler can compile arbitrary Rust code into SPIR-V.

Treat the Rust-to-SPIR-V portion of this phase as a compatibility-verified laboratory track.

If the toolchain is unavailable or incompatible, continue with stable Rust host-side Vulkan development and SPIR-V analysis.

## 2.4 Why this distinction matters

You are studying GPU engineering, not merely learning a particular crate.

The enduring skills are:

- Compilation boundaries.
- Device memory.
- Synchronization.
- ABI design.
- Pipeline construction.
- Runtime ownership.
- Performance analysis.

These remain useful even if a particular Rust GPU project changes substantially.

---

# 3. Study Schedule

| Module | Subject | Duration |
|---|---|---|
| 1 | GPU compilation pipeline and toolchains | 2 weeks |
| 2 | Vulkan compute fundamentals | 2–3 weeks |
| 3 | Vulkan memory and Rust ownership | 2–3 weeks |
| 4 | GPU data layout and ABI | 2 weeks |
| 5 | Device-side Rust and SPIR-V | 2–3 weeks |
| 6 | Synchronization and asynchronous execution | 2 weeks |
| 7 | Safe Rust GPU abstractions | 2–3 weeks |
| 8 | Profiling, integration, and capstone | 2–4 weeks |

## Study requirements

Maintain:

- A reproducible toolchain manifest.
- A Vulkan device-capability report.
- A shader compilation notebook.
- A resource-lifetime diagram.
- A synchronization journal.
- A benchmark database.
- A list of unsupported features and verified workarounds.

---

# 4. Module One: From Rust Source to GPU Instructions

## 4.1 Compilation stages

A conventional Rust program follows a pipeline resembling:

\[
\text{Rust Source}
\rightarrow
\text{Rust Frontend}
\rightarrow
\text{Intermediate Representation}
\rightarrow
\text{Machine Code}
\]

A GPU-targeted Rust pipeline may instead resemble:

\[
\text{Rust Source}
\rightarrow
\text{Compiler IR}
\rightarrow
\text{SPIR-V}
\rightarrow
\text{GPU Driver Compiler}
\rightarrow
\text{Native GPU Instructions}
\]

The exact stages depend on the compiler.

For example, a particular compiler may produce SPIR-V through a specialized backend rather than a conventional LLVM pipeline.

Do not assume every GPU compiler uses LLVM in the same way.

## 4.2 Intermediate representations

An intermediate representation, or IR, expresses a program in a form suitable for compiler analysis and transformation.

An IR may expose:

- Control flow.
- Data dependencies.
- Types.
- Memory operations.
- Address spaces.
- Synchronization operations.
- Function calls.

IRs are important because GPU compiler optimizations often operate below the source-language level.

## 4.3 SPIR-V

SPIR-V is a binary intermediate representation used in ecosystems including Vulkan.

It represents shader modules through structured instructions, types, functions, and metadata.

A SPIR-V module may describe:

- Entry points.
- Execution models.
- Workgroup sizes.
- Storage classes.
- Capabilities.
- Memory operations.
- Control flow.

## 4.4 Entry points

A GPU entry point identifies a function that can be invoked through the execution environment.

In Vulkan compute, a compute shader entry point is associated with a pipeline.

The runtime dispatches workgroups that execute the entry point.

## 4.5 Storage classes

SPIR-V distinguishes memory storage classes.

Conceptually, these represent different kinds of storage, such as:

- Private thread-local values.
- Workgroup-shared memory.
- Device-visible storage buffers.
- Input and output interfaces.
- Uniform data.

The exact available classes and rules depend on the execution environment and capabilities.

## 4.6 Capabilities

SPIR-V modules declare capabilities required by their instructions and types.

A shader that requires a capability unsupported by the device or execution environment cannot simply be assumed to work.

## 4.7 Validation

SPIR-V validation checks structural and semantic constraints.

Validation is distinct from proving algorithmic correctness.

A shader can be valid SPIR-V and still:

- Read the wrong element.
- Produce incorrect output.
- Contain synchronization mistakes.
- Perform poorly.

## 4.8 Compilation versus execution

A shader may compile successfully but fail pipeline creation because of:

- Unsupported capabilities.
- Invalid interfaces.
- Unsupported execution requirements.
- Driver restrictions.

Successful pipeline creation also does not guarantee correct execution.

## Lab 1: GPU Compilation Pipeline Investigation

Investigate three compilation paths:

1. WGSL to SPIR-V through a compatible compiler.
2. GLSL compute shader to SPIR-V.
3. Rust to SPIR-V, if a verified compatible toolchain is available.

For each path, document:

- Source language.
- Compiler.
- Intermediate representation.
- Validation tools.
- Target execution environment.
- Device compatibility.
- Compiler version.

Inspect generated SPIR-V and identify entry points, storage classes, and workgroup configuration.

## Exercises

1. What is an intermediate representation?
2. Why do GPU compilers use IRs?
3. What is SPIR-V?
4. What is a shader entry point?
5. What is a storage class?
6. What is a SPIR-V capability?
7. Why does validation not prove correctness?
8. Why can valid SPIR-V fail pipeline creation?
9. How does a driver compiler differ from a source-language compiler?
10. Why must Rust GPU toolchain versions be pinned?

---

# 5. Module Two: Vulkan Compute Fundamentals

Vulkan is a low-level API for graphics and compute workloads.

Compared with wgpu, Vulkan exposes more explicit control over resource management, synchronization, and execution.

## 5.1 Vulkan objects

Important Vulkan objects include:

- Instance.
- Physical device.
- Logical device.
- Queue.
- Buffer.
- Device memory.
- Descriptor set layout.
- Descriptor set.
- Pipeline layout.
- Compute pipeline.
- Command pool.
- Command buffer.
- Fence.
- Semaphore.

Each object has specific creation, ownership, and destruction rules.

## 5.2 Vulkan instance

A Vulkan instance establishes access to the Vulkan implementation and relevant extensions and layers.

It does not represent a particular GPU.

## 5.3 Physical devices

A physical device represents a Vulkan-capable device exposed by the implementation.

A system may expose multiple physical devices.

Device selection should consider:

- Compute support.
- Memory heaps.
- Required features.
- Queue families.
- Device limits.
- Supported extensions.

## 5.4 Logical devices

A logical device provides access to enabled features and queues of a selected physical device.

Creating a logical device requires specifying the capabilities and queues your application needs.

## 5.5 Queue families

Vulkan groups queues into families with defined capabilities.

A queue family may support:

- Graphics.
- Compute.
- Transfer.
- Other specialized operations.

A compute application needs a queue that supports compute operations.

## 5.6 Command buffers

A command buffer records GPU commands.

Commands may include:

- Pipeline binding.
- Descriptor binding.
- Compute dispatch.
- Buffer copies.
- Memory barriers.

Recording commands is distinct from executing them.

## 5.7 Compute pipelines

A compute pipeline combines shader code with pipeline configuration.

A pipeline identifies the compute shader entry point and compatible resource interfaces.

## 5.8 Descriptor sets

Descriptor sets connect shader-visible resource bindings to actual buffers or other resources.

For example, a shader may declare two input storage buffers and one output storage buffer.

The host application must bind compatible resources at the expected locations.

## 5.9 Dispatch

A compute dispatch specifies a number of workgroups.

If the dispatch dimensions are:

\[
(G_x,G_y,G_z)
\]

and the shader's local workgroup dimensions are:

\[
(L_x,L_y,L_z)
\]

then the logical invocation domain covers:

\[
(G_xL_x,G_yL_y,G_zL_z)
\]

invocations, subject to the execution model and bounds checks.

## 5.10 Worked example: vector addition

Suppose:

\[
N=10{,}000
\]

and each workgroup contains:

\[
L_x=256
\]

invocations.

The required workgroup count is:

\[
G_x=\left\lceil\frac{10{,}000}{256}\right\rceil=40
\]

The dispatch covers:

\[
40\times256=10{,}240
\]

invocations.

The shader must prevent the excess 240 invocations from accessing invalid elements.

## 5.11 Vulkan versus wgpu

wgpu simplifies many responsibilities that Vulkan exposes explicitly.

A Vulkan application typically requires more direct management of:

- Memory allocation.
- Resource synchronization.
- Descriptor lifetimes.
- Command buffers.
- Queue submission.
- Feature negotiation.

This provides greater control but also more opportunities for mistakes.

## Lab 2: Minimal Vulkan Compute Application

Build a Rust Vulkan application that:

1. Creates a Vulkan instance.
2. Selects a compatible physical device.
3. Creates a logical device.
4. Obtains a compute-capable queue.
5. Creates buffers.
6. Loads a compute shader.
7. Creates a compute pipeline.
8. Binds resources.
9. Records a dispatch.
10. Submits the command buffer.
11. Waits for completion.
12. Reads and validates results.
13. Releases resources correctly.

Use vector addition as the initial workload.

## Exercises

1. Distinguish instance, physical device, and logical device.
2. Explain queue families.
3. Explain command buffers.
4. Explain compute pipelines.
5. Explain descriptor sets.
6. Derive dispatch dimensions.
7. Why are bounds checks necessary?
8. What responsibilities does wgpu abstract away?
9. Why must Vulkan features be queried?
10. Why does command recording differ from execution?

---

# 6. Module Three: Vulkan Memory and Rust Ownership

## 6.1 The host/device memory boundary

CPU and GPU execution use different memory-access mechanisms.

Even when memory is physically shared or host-visible, accessibility, visibility, coherence, and synchronization must be treated separately.

A pointer valid in CPU address space is not automatically a usable GPU pointer.

## 6.2 Buffers

A Vulkan buffer represents a linear resource.

It does not necessarily own its backing allocation.

A buffer's usage flags describe permitted operations.

Examples include:

- Storage-buffer access.
- Transfer source.
- Transfer destination.
- Uniform-buffer access.

## 6.3 Memory allocation

Vulkan exposes memory requirements and memory types.

A buffer must be associated with a compatible allocation.

Memory properties may include:

- Device-local.
- Host-visible.
- Host-coherent.
- Host-cached.

These properties describe different aspects of memory access.

## 6.4 Host-visible versus host-coherent

Host-visible memory can be mapped for CPU access.

Host-coherent memory simplifies certain host/device visibility requirements.

For noncoherent mapped memory, explicit flushing or invalidation may be required, with attention to the relevant alignment and synchronization rules.

Host coherence does not eliminate the need for execution synchronization.

## 6.5 Staging buffers

A common transfer strategy uses:

1. A host-visible staging buffer.
2. A device-local compute buffer.
3. A copy command.
4. Appropriate synchronization.

This can be useful when the preferred device-local memory is not directly CPU-mappable.

## 6.6 Rust ownership

Rust ownership is valuable for managing host-side resource lifetimes.

For example, a buffer wrapper may own:

- A Vulkan buffer handle.
- Its backing allocation.
- Device references or ownership relationships.
- Metadata describing size and usage.

However, ordinary Rust ownership does not automatically prove that a GPU has finished using a resource.

## 6.7 The asynchronous lifetime problem

Consider this sequence:

1. The CPU submits a GPU command referencing a buffer.
2. The submission returns.
3. The CPU drops the Rust buffer wrapper.
4. The GPU has not yet completed the command.

If the wrapper destroys the underlying Vulkan resources immediately, the GPU may access resources whose lifetime has ended.

This is a fundamental mismatch between synchronous host-side destruction and asynchronous GPU execution.

## 6.8 Lifetime tracking

A safe runtime must ensure that resources remain alive until the GPU no longer needs them.

Possible mechanisms include:

- Fence-associated ownership.
- Submission-tracked resources.
- Deferred destruction.
- Explicit completion handles.
- Reference-counted lifetime management combined with completion tracking.

Reference counting alone does not establish GPU completion.

## 6.9 Resource aliasing

Two buffer views may reference overlapping memory.

Even if their Rust wrappers are distinct, the underlying GPU memory can alias.

A safe abstraction must account for actual memory ranges, not merely wrapper identity.

## 6.10 Unsafe Rust

Low-level Vulkan bindings commonly require `unsafe` operations.

Unsafe code must uphold invariants that the Rust compiler cannot verify.

These may include:

- Valid Vulkan handles.
- Correct buffer sizes.
- Proper alignment.
- Correct memory binding.
- Valid pointer lifetimes.
- Synchronization.
- Resource destruction order.

The goal is not to eliminate all unsafe code, but to isolate it behind carefully justified interfaces.

## Worked example: asynchronous resource lifetime

Suppose a GPU copy uses a source buffer and a destination buffer.

The command buffer is submitted, and the CPU immediately returns from the submission function.

The Rust API must determine whether the caller is allowed to destroy either buffer before the copy completes.

A correct abstraction might retain ownership of both buffers until a completion event confirms the operation has finished.

This example illustrates why a safe-looking function signature can still conceal an unsound asynchronous lifetime contract.

## Lab 3: GPU Buffer Ownership Library

Design a Rust library that manages:

- Buffer creation.
- Allocation.
- Mapping.
- Transfers.
- Deferred destruction.
- Submission completion.
- Buffer metadata.

Write a safety argument for each public abstraction that wraps unsafe Vulkan operations.

## Exercises

1. Distinguish buffers from memory allocations.
2. Explain device-local memory.
3. Explain host-visible memory.
4. Explain host-coherent memory.
5. Why are staging buffers useful?
6. Why does Rust ownership not automatically prove GPU completion?
7. What is deferred destruction?
8. Why does reference counting alone not solve synchronization?
9. Explain GPU memory aliasing.
10. What invariants must an unsafe buffer wrapper uphold?

---

# 7. Module Four: GPU Data Layout and ABI

## 7.1 Why data layout matters

The CPU and GPU must agree on the representation of shared data.

This includes:

- Field offsets.
- Alignment.
- Padding.
- Element size.
- Array stride.
- Matrix layout.
- Integer widths.
- Boolean representation.

A mismatch can produce incorrect results without causing an obvious crash.

## 7.2 Rust representation

Rust's default struct representation does not promise a stable C-compatible field layout.

For data shared across an FFI or GPU ABI boundary, explicit representation is often necessary.

Consider:

```rust
#[repr(C)]
struct Particle {
    position: [f32; 4],
    velocity: [f32; 4],
    mass: f32,
}
```

`#[repr(C)]` gives the struct a C-compatible layout according to its field types and target ABI.

However, that alone does not guarantee compatibility with every Vulkan shader layout rule.

The shader interface must still be checked.

## 7.3 Alignment

An object may require its address to be a multiple of some alignment.

For example, a 16-byte-aligned object must begin at an address divisible by 16.

Alignment affects:

- Struct offsets.
- Array strides.
- Buffer offsets.
- Descriptor requirements.
- Memory transactions.

## 7.4 Padding

Padding is unused space inserted to satisfy alignment or layout rules.

Consider a conceptual structure with:

- One 32-bit integer.
- One 64-bit integer.

The fields require 4 and 8 bytes respectively.

Under a layout where the 64-bit field requires 8-byte alignment, padding may be inserted between the fields.

The exact final layout must be established using the applicable ABI.

## 7.5 GPU vector layouts

A three-component vector may have a size of 12 bytes but an alignment requirement larger than 4 bytes in certain shader layout contexts.

Do not assume that three consecutive `f32` values have identical alignment and stride behavior in every host and shader representation.

## 7.6 Array stride

An array element's stride may be larger than the logical size of its data.

For example, a shader interface might require an array element to occupy 16 bytes even when its useful payload contains only 12 bytes.

## 7.7 Booleans

Boolean representations are especially dangerous across ABI boundaries.

A Rust `bool` should not be assumed to match a shader boolean's storage representation.

A safer interface may use a clearly specified integer representation, subject to the shader layout rules.

## 7.8 Endianness

Byte order matters when interpreting raw binary data.

Although common desktop CPU and GPU configurations use little-endian representations, a portable serialization or interface contract should not rely on unstated assumptions.

## 7.9 Compile-time checks

Rust can use compile-time assertions or tests to verify:

- Struct size.
- Alignment.
- Field offsets.
- Array stride assumptions.

These checks should be paired with shader-interface validation.

## 7.10 Typed GPU buffers

A typed buffer abstraction may associate a Rust element type with a GPU buffer.

However, the abstraction must distinguish:

- Types safe for bytewise transfer.
- Types with padding.
- Types containing pointers.
- Types with invalid bit patterns.
- Types whose shader layout differs from Rust layout.

A blanket assumption that every `Copy` type is GPU-compatible is unsafe.

## Worked example: buffer size

Suppose a GPU buffer stores:

\[
N=4096
\]

elements, each with a verified stride of:

\[
S=32\text{ bytes}
\]

Then the required storage is:

\[
4096\times32=131{,}072\text{ bytes}
\]

Using the sum of logical field sizes instead of the actual stride could underallocate the buffer.

## Lab 4: GPU ABI Verification Suite

Build tests for:

- Scalar layouts.
- Vector layouts.
- Struct layouts.
- Array strides.
- Alignment.
- Padding.
- Nested structures.

Compare Rust-side expectations with shader-side declarations.

## Exercises

1. What is an ABI?
2. Why is default Rust struct layout unsuitable as an implicit GPU contract?
3. What does `#[repr(C)]` guarantee?
4. Why is `#[repr(C)]` not sufficient for every shader interface?
5. Define alignment.
6. Define padding.
7. Define array stride.
8. Why are three-component vectors potentially tricky?
9. Why are booleans risky across host/device boundaries?
10. Why is `Copy` not a sufficient GPU-buffer safety condition?

---

# 8. Module Five: Device-Side Rust

This module explores writing GPU kernels in Rust.

**Prerequisite:** Verify an actual supported Rust-to-GPU toolchain before beginning execution labs.

If no compatible toolchain is available, perform source and SPIR-V analysis while continuing the practical GPU work through Vulkan and WGSL.

## 8.1 Host Rust versus device Rust

Host Rust executes in an operating-system process.

Device Rust executes under GPU execution constraints.

The same source-language syntax does not imply identical runtime capabilities.

## 8.2 Standard library restrictions

GPU kernels often cannot use the full Rust standard library.

Restrictions may affect:

- Heap allocation.
- Filesystem access.
- Networking.
- Operating-system threads.
- Panic handling.
- Dynamic dispatch.
- Standard I/O.

A GPU-targeted Rust environment may provide only a subset of `core` or additional specialized libraries.

## 8.3 No-standard-library programming

Rust supports `#![no_std]` for environments without the conventional standard library.

This is relevant to embedded systems and certain GPU compilation environments.

However, `no_std` alone does not make a Rust program GPU-compatible.

The target compiler must still support the program's instructions, types, and runtime assumptions.

## 8.4 Entry-point attributes

GPU-targeted Rust projects may use special attributes or macros to mark shader entry points.

The syntax depends on the compiler ecosystem and version.

Do not treat an example from one toolchain release as a universal Rust GPU interface.

## 8.5 Address spaces

GPU memory is not one homogeneous address space.

Device code may distinguish:

- Private storage.
- Workgroup storage.
- Storage buffers.
- Uniform buffers.
- Other specialized memory regions.

The compiler must map source-level references and pointers into the appropriate GPU memory model.

## 8.6 References and aliasing

Rust references have strong validity and aliasing expectations.

A GPU compiler must preserve those semantics while mapping references onto GPU address spaces.

This becomes difficult when:

- Multiple lanes access shared memory.
- Multiple invocations update the same buffer.
- Raw pointers are used.
- Atomic operations coordinate ownership.
- Memory aliases across descriptors.

## 8.7 Atomics

GPU atomics require support from:

- The source-language abstraction.
- The compiler.
- The SPIR-V environment.
- The device.
- The relevant memory storage class.

Atomic types and operations may be more restricted than on the CPU.

## 8.8 Barriers

A workgroup barrier coordinates invocations within a workgroup according to its execution and memory semantics.

A barrier cannot be replaced by a Rust mutex intended for operating-system threads.

The device compiler must expose appropriate GPU synchronization primitives.

## 8.9 Panic behavior

A GPU kernel cannot assume ordinary host-side panic unwinding.

The target environment must define how unsupported operations, assertions, or panic paths behave.

Avoid assuming that a GPU assertion behaves like a desktop Rust assertion.

## 8.10 Recursion

Recursion may be restricted or unsupported depending on the compiler and GPU execution environment.

Even when recursion is technically possible, stack consumption and performance can be problematic.

## 8.11 Dynamic allocation

GPU-side heap allocation is not universally available or desirable.

Many kernels instead use:

- Preallocated buffers.
- Fixed-size local storage.
- Workgroup memory.
- Atomic allocation within bounded buffers.

## 8.12 Generic programming

Rust generics are attractive for reusable GPU kernels.

However, generic specialization can increase generated code size and compiler complexity.

A GPU-targeted compiler must also support the monomorphized operations and types.

## 8.13 Error handling

A kernel cannot generally return a conventional host-side `Result` to the CPU directly.

Error reporting may require:

- Status buffers.
- Atomic flags.
- Per-element error codes.
- Host-side validation.
- Defined fallback behavior.

## 8.14 Portability

A Rust GPU kernel that compiles for one target may not compile or run on another.

Portability depends on:

- Compiler support.
- SPIR-V capabilities.
- Vulkan features.
- Shader interface rules.
- Driver behavior.

## Lab 5: Rust-to-SPIR-V Feasibility Study

Using a verified toolchain, attempt to compile progressively more complex Rust GPU kernels.

Suggested progression:

1. Scalar arithmetic.
2. Vector addition.
3. Bounds-checked buffer access.
4. Workgroup-local operations.
5. Atomic counters.
6. Parallel reduction.

For each experiment, document:

- Compiler version.
- Required language restrictions.
- Compilation result.
- SPIR-V validation result.
- Vulkan pipeline creation result.
- Runtime correctness.
- Unsupported features.

If compilation fails due to toolchain incompatibility, document the limitation rather than silently replacing the target or assuming success.

## Exercises

1. Why is GPU Rust different from host Rust?
2. What does `no_std` mean?
3. Why is `no_std` not sufficient for GPU compilation?
4. Why are address spaces important?
5. How can aliasing complicate GPU Rust?
6. Why are atomics target-dependent?
7. Why are barriers different from CPU mutexes?
8. Why is panic handling difficult on GPUs?
9. Why may recursion be restricted?
10. Why does generic code increase compiler complexity?
11. How can kernels report errors?
12. Why does successful compilation not guarantee portability?

---

# 9. Module Six: Vulkan Synchronization

Synchronization is one of the most demanding parts of low-level GPU programming.

## 9.1 Execution dependencies

An execution dependency constrains when operations may execute relative to one another.

For example, a compute shader must not consume the result of a transfer before the transfer's writes are available under the required synchronization.

## 9.2 Memory dependencies

A memory dependency controls availability and visibility of memory accesses between operations.

Execution ordering alone does not always establish the required memory visibility.

## 9.3 Pipeline stages

Vulkan identifies pipeline stages where operations occur.

Examples relevant to compute include:

- Transfer operations.
- Compute shader execution.
- Host interactions.

Synchronization specifies which operations are ordered and which memory accesses are made available or visible.

## 9.4 Access masks

Access masks describe categories of memory access.

Examples include:

- Transfer writes.
- Shader reads.
- Shader writes.

Correct synchronization requires matching the actual producer and consumer accesses.

## 9.5 Buffer memory barriers

A buffer memory barrier can establish memory dependencies for a specified buffer range.

It is not a generic instruction to make every GPU operation globally sequential.

## 9.6 Queue submission

Submitting a command buffer schedules work for execution.

Submission returning does not imply that the GPU has completed the work.

## 9.7 Fences

Fences can allow the host to determine when a queue submission has completed.

They are commonly used for CPU/GPU synchronization and resource reclamation.

## 9.8 Semaphores

Semaphores coordinate GPU operations, including across queue submissions.

Binary and timeline semaphores provide different synchronization models.

## 9.9 Timeline semaphores

Timeline semaphores use monotonically increasing values to represent synchronization progress.

They can be useful for tracking asynchronous work and managing resource lifetimes.

## 9.10 Host visibility

After GPU writes complete, the host must observe the results through the appropriate synchronization and memory-coherence mechanisms.

For noncoherent mapped memory, invalidation may be necessary before the CPU reads updated data.

## 9.11 Synchronization2

Modern Vulkan provides synchronization mechanisms commonly referred to as Synchronization2.

These provide more explicit stage and access descriptions than older synchronization interfaces.

Use the API version and extensions supported by the selected device.

## Worked example: compute followed by readback

Consider:

1. A compute shader writes a storage buffer.
2. A copy operation transfers its contents into a host-readable buffer.
3. The CPU reads the copied data.

Correctness requires:

- Ordering the shader writes before the copy reads.
- Making the shader writes available to the copy.
- Waiting for the GPU copy to complete before CPU consumption.
- Handling host memory coherence as required.

A single CPU-side wait cannot retroactively repair an omitted GPU memory dependency between the shader and copy stages.

## Lab 6: Synchronization Laboratory

Implement multi-stage GPU workflows:

- Transfer followed by compute.
- Compute followed by compute.
- Compute followed by readback.
- Multiple queue submissions.
- Fence-based completion.
- Timeline-based completion, where supported.

For each workflow, document the producer, consumer, stage, access type, and synchronization mechanism.

## Exercises

1. Distinguish execution and memory dependencies.
2. What is a pipeline stage?
3. What is an access mask?
4. What is a buffer memory barrier?
5. Why does queue submission not imply completion?
6. What is a fence?
7. What is a semaphore?
8. What is a timeline semaphore?
9. Why can noncoherent host memory require invalidation?
10. Why can a CPU wait be insufficient to replace a GPU memory barrier?
11. Why is synchronization scope important?
12. How can over-synchronization reduce performance?

---

# 10. Module Seven: Safe Rust Abstractions for GPU Resources

Low-level Vulkan APIs are powerful but difficult to use correctly.

A major Rust engineering opportunity is building abstractions that make invalid states harder to express.

## 10.1 Resource ownership

A resource wrapper should clearly define:

- Who owns the resource.
- When it may be destroyed.
- Which operations are permitted.
- Whether it may be shared.
- Whether the GPU is currently using it.

## 10.2 Typed buffers

A typed buffer may represent:

\[
\text{GpuBuffer}\langle T\rangle
\]

where \(T\) is a GPU-compatible element type.

The abstraction must enforce size, alignment, and representation constraints.

## 10.3 Buffer states

A buffer may be conceptually:

- Available for host writes.
- Ready for GPU reading.
- In use by a submission.
- Awaiting completion.
- Available for readback.

A type-state design can encode some transitions.

However, Vulkan synchronization and aliasing remain more complex than a simple linear state machine.

## 10.4 Submission handles

A submission handle can represent GPU work that has been queued but not necessarily completed.

The handle may own or retain references to resources required by the submission.

## 10.5 Deferred destruction

Resource destruction can be delayed until the GPU has completed relevant work.

A runtime may associate pending resources with completion tokens or timeline values.

## 10.6 RAII

Resource Acquisition Is Initialization is useful for Vulkan resource management.

Rust's `Drop` trait can release resources when wrappers are destroyed.

But destruction must respect outstanding GPU work and dependencies between Vulkan objects.

## 10.7 Error handling

A robust GPU library should distinguish:

- Device initialization failures.
- Unsupported features.
- Allocation failures.
- Shader compilation failures.
- Pipeline creation failures.
- Submission failures.
- Device loss.
- Validation failures.

## 10.8 Safe versus unsafe interfaces

An unsafe implementation may expose a safe public API only when all required invariants are enforced internally.

Documentation should explain why the interface is sound.

A wrapper is not safe merely because its public methods omit the `unsafe` keyword.

## 10.9 Example: resource ownership

Consider a conceptual Rust type:

```rust
struct GpuAllocation {
    size_bytes: usize,
    // Vulkan allocation and ownership metadata.
}
```

The type's existence does not establish whether:

- The allocation is bound to a buffer.
- The GPU is using it.
- Its memory is mapped.
- A descriptor references it.
- Another view aliases it.

These are runtime and API invariants that must be modeled or enforced.

## 10.10 Thread safety

Rust's `Send` and `Sync` traits describe host-side thread-safety properties.

They do not automatically establish GPU memory synchronization.

A GPU resource wrapper may be safely shared among CPU threads while still requiring explicit coordination of GPU accesses.

## Lab 7: Safe GPU Resource Layer

Build a Rust abstraction layer over Vulkan.

Required features:

- Buffer ownership.
- Allocation ownership.
- Typed buffer metadata.
- Submission tracking.
- Deferred destruction.
- Completion handles.
- Error propagation.
- Debug validation.

Write a safety contract for each public API.

## Exercises

1. What is a resource ownership invariant?
2. What is a typed GPU buffer?
3. What is type-state?
4. Why can type-state only model part of Vulkan synchronization?
5. What is a submission handle?
6. Why is deferred destruction useful?
7. Why is RAII insufficient without GPU completion tracking?
8. Why must safe wrappers document their invariants?
9. Why do `Send` and `Sync` not solve GPU synchronization?
10. How should a runtime handle device loss?

---

# 11. Module Eight: Native GPU Performance Engineering

Low-level control is useful only when it leads to a measurable engineering benefit.

## 11.1 Native versus portable performance

A native Vulkan implementation may expose controls unavailable through a higher-level portable API.

However, lower-level APIs do not automatically produce faster kernels.

Performance depends on:

- Shader code.
- Compiler optimizations.
- Memory access.
- Dispatch configuration.
- Synchronization.
- Driver behavior.
- Resource management.
- Workload size.

## 11.2 Shader compilation cost

Native applications may incur costs for:

- Loading SPIR-V.
- Pipeline creation.
- Driver compilation.
- Pipeline cache misses.

These costs should be distinguished from steady-state kernel execution.

## 11.3 Pipeline caching

Pipeline caches can reduce repeated compilation work under supported conditions.

Their effectiveness and compatibility depend on implementation details and environment.

## 11.4 Command submission overhead

Small GPU workloads may be dominated by CPU-side setup and submission overhead.

Batching and reusing resources can improve end-to-end throughput.

## 11.5 Memory allocation overhead

Frequent low-level allocations can be expensive.

A GPU runtime may benefit from:

- Allocation pooling.
- Suballocation.
- Buffer reuse.
- Staging-buffer reuse.

## 11.6 Descriptor management

Descriptor allocation and updates can contribute overhead.

A runtime should consider descriptor reuse and resource-binding strategy.

## 11.7 GPU profiling

Use supported profiling tools to investigate:

- GPU execution time.
- Memory bandwidth.
- Workgroup occupancy.
- Shader instruction behavior.
- Queue idle time.
- Synchronization stalls.

Tool compatibility must be checked for the GPU, driver, and application API.

## 11.8 Comparing Rust and WGSL

A fair comparison should control:

- GPU hardware.
- Input data.
- Algorithm.
- Precision.
- Workgroup configuration.
- Memory layout.
- Warm-up.
- Timing methodology.

Differences in generated machine code can make apparently equivalent source kernels perform differently.

## 11.9 Comparing CUDA

CUDA comparisons require NVIDIA hardware.

Do not compare your AMD Vulkan timings against NVIDIA CUDA timings and attribute the difference solely to the programming language.

Hardware architecture, drivers, clocks, and memory bandwidth are confounding variables.

## Lab 8: Native Vulkan Performance Study

Compare equivalent kernels implemented through:

- Rust/wgpu/WGSL.
- Rust/Vulkan/SPIR-V.
- Rust-to-SPIR-V, if supported.

Measure:

- Initialization time.
- Pipeline creation time.
- Submission overhead.
- Kernel execution time.
- Readback time.
- End-to-end latency.

Explain the results using evidence rather than assumptions about API overhead.

---

# 12. Major Assessment A: Native Vulkan Compute Engine

Build a Rust Vulkan compute engine.

## Required functionality

- Device enumeration.
- Capability reporting.
- Logical-device creation.
- Compute queue selection.
- Buffer creation.
- Memory allocation.
- Shader loading.
- Descriptor binding.
- Compute pipeline creation.
- Command recording.
- Submission.
- Synchronization.
- Readback.
- Resource destruction.

## Required workloads

- Vector addition.
- Reduction.
- Matrix-vector multiplication.

## Technical defense

Explain the lifetime and synchronization of every major resource.

---

# 13. Major Assessment B: GPU ABI and Safety Library

Build a library that manages GPU-compatible data types and resource ownership.

## Required functionality

- Verified buffer element layouts.
- Alignment checks.
- Size calculations.
- Typed buffer abstractions.
- Resource lifetime tracking.
- Deferred destruction.
- Error reporting.

## Required tests

- Invalid sizes.
- Alignment boundaries.
- Empty buffers.
- Multiple outstanding submissions.
- Resource reuse.
- Repeated allocation and destruction.

## Technical defense

Explain which safety properties Rust enforces statically and which remain runtime obligations.

---

# 14. Major Assessment C: Rust-to-GPU Compiler Investigation

Investigate a Rust-to-GPU compilation toolchain.

## Required research questions

1. Which Rust compiler versions are supported?
2. Which targets are supported?
3. Which Rust language features work?
4. Which standard-library features work?
5. How are shader entry points declared?
6. How are address spaces represented?
7. How are atomics represented?
8. How is synchronization expressed?
9. What SPIR-V capabilities are required?
10. What unsupported features remain?

## Required deliverables

- Toolchain manifest.
- Compatibility matrix.
- Compilation experiments.
- SPIR-V inspection.
- Validation results.
- Runtime tests where supported.
- Limitations report.

If no compatible compiler can be installed, submit a reproducible feasibility investigation and use the native Vulkan track for the execution requirements.

---

# 15. Final Capstone: Rust Native GPU Runtime

Build a reusable native GPU computing runtime in Rust.

## 15.1 Architecture

Separate the system into:

### Device layer

- Instance and device management.
- Feature discovery.
- Queue selection.
- Device metadata.

### Memory layer

- Buffer allocation.
- Staging buffers.
- Typed buffers.
- Mapping.
- Lifetime tracking.

### Pipeline layer

- Shader loading.
- SPIR-V validation.
- Pipeline creation.
- Descriptor management.
- Pipeline caching where appropriate.

### Execution layer

- Command recording.
- Submission.
- Synchronization.
- Completion tracking.
- Resource reclamation.

### Algorithm layer

- Vector operations.
- Reduction.
- Scan.
- Matrix multiplication.
- At least one Phase 11 algorithm.

### Validation layer

- CPU reference implementations.
- Correctness tests.
- Numerical error checks.
- Stress tests.

### Benchmark layer

- Initialization timing.
- Pipeline creation timing.
- Kernel timing.
- End-to-end timing.
- Structured result reporting.

## 15.2 Required safety analysis

Document:

- Ownership.
- Resource lifetimes.
- Buffer aliasing.
- Mapping.
- Synchronization.
- Device loss.
- Unsafe code boundaries.

## 15.3 Required performance analysis

Compare against equivalent wgpu implementations.

For each comparison:

1. Establish correctness.
2. Match workloads.
3. Control experimental conditions.
4. Measure repeated trials.
5. Report timing distributions.
6. Explain observed differences.
7. Identify remaining uncertainties.

## 15.4 Required architecture document

Include diagrams showing:

- Rust host application.
- Vulkan runtime.
- Driver.
- Shader compiler.
- SPIR-V module.
- GPU execution.
- Memory resources.
- Submission lifecycle.

## 15.5 Final technical defense

Be prepared to explain:

- Why native Vulkan requires explicit synchronization.
- Why Rust ownership alone cannot model GPU completion.
- How shader ABI compatibility is established.
- How the driver transforms SPIR-V into executable code.
- Which unsafe operations remain necessary.
- How your runtime prevents premature resource destruction.
- What benefits native Vulkan provides over wgpu.
- Which Rust-to-GPU capabilities are experimental.
- How you verified performance claims.

---

# 16. Phase 12 Quiz Bank

## Compilation and SPIR-V

1. What is an intermediate representation?
2. What is SPIR-V?
3. What is a shader entry point?
4. What is a storage class?
5. What is a capability?
6. Why does SPIR-V validation matter?
7. Why does validation not prove correctness?
8. Why can pipeline creation fail after compilation?
9. What is a driver compiler?
10. Why must Rust GPU toolchain compatibility be verified?

## Vulkan fundamentals

11. What is a Vulkan instance?
12. What is a physical device?
13. What is a logical device?
14. What is a queue family?
15. What is a command buffer?
16. What is a compute pipeline?
17. What is a descriptor set?
18. What is a pipeline layout?
19. What is a dispatch?
20. Why are bounds checks necessary?

## Memory

21. What is a Vulkan buffer?
22. What is a memory allocation?
23. What is device-local memory?
24. What is host-visible memory?
25. What is host-coherent memory?
26. What is a staging buffer?
27. Why can host mapping require synchronization?
28. What is resource aliasing?
29. Why is deferred destruction necessary?
30. Why does CPU-side ownership not prove GPU completion?

## Data layout

31. What is an ABI?
32. What does `#[repr(C)]` mean?
33. Why is `#[repr(C)]` not sufficient for every shader layout?
34. What is alignment?
35. What is padding?
36. What is array stride?
37. Why can vector layouts differ?
38. Why are booleans risky?
39. Why are raw pointers generally unsuitable as portable GPU buffer data?
40. Why is `Copy` not enough to establish GPU compatibility?

## Device-side Rust

41. What is device-side Rust?
42. What does `no_std` mean?
43. Why does `no_std` not guarantee GPU compatibility?
44. Why are address spaces important?
45. Why is heap allocation difficult on GPUs?
46. Why may recursion be restricted?
47. Why is panic handling different?
48. Why are GPU atomics target-dependent?
49. Why are GPU barriers different from CPU mutexes?
50. How can GPU kernels report errors?

## Synchronization

51. What is an execution dependency?
52. What is a memory dependency?
53. What is a pipeline stage?
54. What is an access mask?
55. What is a buffer memory barrier?
56. What is a fence?
57. What is a semaphore?
58. What is a timeline semaphore?
59. Why does queue submission not imply completion?
60. Why may noncoherent memory require invalidation?
61. Why is synchronization scope important?
62. Why can over-synchronization reduce performance?

## Safe abstractions

63. What is RAII?
64. What is type-state?
65. What is a typed GPU buffer?
66. What is a submission handle?
67. What is deferred resource destruction?
68. Why does reference counting not establish GPU completion?
69. Why are `Send` and `Sync` insufficient for GPU synchronization?
70. Why must safe wrappers enforce unsafe invariants?
71. What is resource lifetime tracking?
72. How should a runtime handle device loss?

## Performance

73. Why is lower-level API access not automatically faster?
74. What is pipeline compilation overhead?
75. What is pipeline caching?
76. Why can command submission overhead dominate small workloads?
77. Why is buffer reuse useful?
78. Why must GPU benchmarks separate initialization from execution?
79. Why must equivalent kernels be compared on the same hardware?
80. Why can compiler-generated machine code affect performance?
81. Why is numerical validation necessary?
82. Why should benchmark distributions be reported?
83. Why can synchronization affect end-to-end latency?
84. Why must performance claims distinguish observation from hypothesis?

---

# 17. Final Examination

**Recommended duration:** 8–10 hours.

| Section | Weight |
|---|---:|
| Compilation and SPIR-V | 15% |
| Vulkan execution model | 15% |
| Memory and ownership | 20% |
| ABI and data layout | 10% |
| Device-side Rust | 10% |
| Synchronization | 20% |
| Performance and runtime design | 10% |

## Part A: Compilation pipeline

Explain how a Rust GPU program could be transformed into executable GPU instructions.

Identify toolchain and compatibility constraints.

## Part B: Vulkan architecture

Design the object relationships for a minimal compute application.

## Part C: Resource ownership

Analyze asynchronous resource lifetime problems.

## Part D: ABI

Determine whether host and shader data representations are compatible.

## Part E: Device Rust

Explain which Rust language features require special handling.

## Part F: Synchronization

Analyze dependencies between transfer, compute, and readback operations.

## Part G: Performance

Design a fair comparison between native Vulkan and wgpu.

## Part H: Technical defense

Defend the safety, correctness, and performance of your Rust native GPU runtime.

## Passing standard

Recommended:

- At least 80% overall.
- At least 80% in memory ownership and synchronization.
- No unresolved resource-lifetime violations.
- No unresolved GPU data-layout mismatches.
- Complete correctness testing for all capstone algorithms.
- A reproducible toolchain and benchmark environment.

---

# 18. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

**The tutor must never provide answers, code, pseudocode, implementation instructions, debugging fixes, file edits, completed derivations, or assessment solutions.**

The tutor's role is to guide your reasoning using exactly one question per response.

## Master Socratic Tutor Prompt

```text
You are my Socratic native Rust GPU programming tutor.

I am studying Phase 12 of a rigorous GPU engineering curriculum.

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

10. Require me to explain GPU compilation, Vulkan execution,
    resource ownership, memory layout, synchronization,
    Rust safety invariants, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness,
    lifetimes, unsafe invariants, synchronization, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only one guiding question.
```

## Start a lesson

```text
Tutor me through Phase 12, Module [NUMBER].

Start from first principles.
Ask exactly one question at a time.
Require me to reason about Rust, GPU compilation,
Vulkan resources, synchronization, and safety.

Never provide answers or solutions.
```

## Generate a quiz

```text
Administer a 20-question Phase 12 quiz on [TOPIC].

Include conceptual reasoning, Vulkan execution,
GPU memory, Rust ownership, ABI, and synchronization.

Ask exactly one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text
Administer Phase 12 Coding Assessment [A/B/C].

Present requirements and constraints only.

Ask me to explain my architecture,
resource lifetime model, synchronization strategy,
safety invariants, and correctness tests.

Do not provide code, pseudocode, implementations,
debugging fixes, or solutions.

Ask exactly one question at a time.
```

## Review unsafe Rust GPU code

```text
Review my Rust GPU code using only questions.

Focus on:
- Unsafe invariants
- Resource ownership
- Buffer lifetimes
- Aliasing
- Alignment
- ABI compatibility
- Submission completion
- Synchronization
- Error handling

Do not provide fixes or replacement code.
Ask exactly one guiding question at a time.
```

## Vulkan synchronization examination

```text
Examine my understanding of Vulkan synchronization.

Ask questions about:
- Execution dependencies
- Memory dependencies
- Pipeline stages
- Access masks
- Buffer barriers
- Queue submissions
- Fences
- Semaphores
- Host visibility

Never provide answers or implementation guidance.
Ask exactly one question at a time.
```

## Rust-to-SPIR-V examination

```text
Examine my understanding of Rust-to-SPIR-V compilation.

Ask questions about:
- Compiler architecture
- Intermediate representations
- SPIR-V capabilities
- Address spaces
- Entry points
- Rust language restrictions
- Validation
- Driver compilation
- Toolchain compatibility

Never provide solutions.
Ask exactly one question at a time.
```

## Performance investigation

```text
Help me investigate native Rust GPU performance
using only questions.

Ask about:
- Shader compilation
- Pipeline creation
- Memory allocation
- Command submission
- Synchronization
- GPU execution time
- Data transfers
- Benchmark methodology
- Compiler-generated code

Do not interpret results for me.
Do not provide conclusions or optimizations.
Ask exactly one question at a time.
```

## Final oral examination

```text
Conduct a rigorous Phase 12 oral examination.

Challenge my understanding of:
- Rust GPU compilation
- SPIR-V
- Vulkan compute
- Memory ownership
- GPU ABI
- Unsafe Rust
- Synchronization
- Runtime design
- Performance engineering

Ask exactly one question at a time.
Never provide answers, solutions, code, or hints
that reveal the answer.
```

---

# 19. Mastery Checklist

## Compilation

- [ ] I understand GPU compilation pipelines.
- [ ] I understand SPIR-V.
- [ ] I understand shader entry points.
- [ ] I understand storage classes.
- [ ] I understand SPIR-V capabilities.
- [ ] I can inspect and validate SPIR-V.
- [ ] I understand Rust GPU toolchain limitations.

## Vulkan

- [ ] I can create a Vulkan instance.
- [ ] I can enumerate physical devices.
- [ ] I can create a logical device.
- [ ] I can select a compute queue.
- [ ] I can create compute pipelines.
- [ ] I can bind descriptors.
- [ ] I can record command buffers.
- [ ] I can submit compute work.

## Memory

- [ ] I understand Vulkan buffers.
- [ ] I understand memory allocation.
- [ ] I understand device-local memory.
- [ ] I understand host-visible memory.
- [ ] I understand staging buffers.
- [ ] I understand mapping and coherence.
- [ ] I understand resource aliasing.
- [ ] I understand deferred destruction.

## ABI

- [ ] I understand `#[repr(C)]`.
- [ ] I understand alignment.
- [ ] I understand padding.
- [ ] I understand array stride.
- [ ] I can verify host/shader layouts.
- [ ] I understand GPU-compatible data types.

## Device-side Rust

- [ ] I understand `no_std`.
- [ ] I understand GPU address spaces.
- [ ] I understand GPU atomics.
- [ ] I understand GPU barriers.
- [ ] I understand GPU error reporting.
- [ ] I understand unsupported Rust features.
- [ ] I can explain toolchain compatibility.

## Synchronization

- [ ] I understand execution dependencies.
- [ ] I understand memory dependencies.
- [ ] I understand pipeline stages.
- [ ] I understand access masks.
- [ ] I understand buffer barriers.
- [ ] I understand fences.
- [ ] I understand semaphores.
- [ ] I understand timeline semaphores.
- [ ] I understand host memory visibility.

## Rust safety

- [ ] I can design resource ownership abstractions.
- [ ] I understand GPU submission lifetimes.
- [ ] I understand deferred destruction.
- [ ] I understand type-state limitations.
- [ ] I can document unsafe invariants.
- [ ] I understand why `Send` and `Sync` do not solve GPU synchronization.

## Performance

- [ ] I can benchmark native Vulkan compute.
- [ ] I can separate compilation from execution time.
- [ ] I can measure submission overhead.
- [ ] I can compare Vulkan and wgpu fairly.
- [ ] I can explain compiler-generated performance differences.

## Capstone

- [ ] My Rust native GPU runtime works.
- [ ] My resource lifetimes are correct.
- [ ] My synchronization is correct.
- [ ] My ABI layouts are verified.
- [ ] My algorithms are validated.
- [ ] My benchmarks are reproducible.
- [ ] I can defend my unsafe code.
- [ ] I can explain the runtime architecture.

---

# 20. Recommended Resources

## Vulkan

**Vulkan Specification**

https://registry.khronos.org/vulkan/

**Vulkan Guide**

https://docs.vulkan.org/guide/latest/

**Vulkan Tutorial**

https://vulkan-tutorial.com/

Use tutorials as practical introductions, but verify details against the current specification and your selected Vulkan version.

## Rust Vulkan ecosystem

**ash**

https://docs.rs/ash/

**Vulkano**

https://vulkano.rs/

**Rust-GPU**

https://github.com/Rust-GPU/rust-gpu

Treat Rust-GPU compatibility and maintenance status as version-dependent.

## SPIR-V

**SPIR-V Registry**

https://registry.khronos.org/SPIR-V/

**SPIRV-Tools**

https://github.com/KhronosGroup/SPIRV-Tools

**SPIRV-Cross**

https://github.com/KhronosGroup/SPIRV-Cross

## Rust language and systems programming

**The Rustonomicon**

https://doc.rust-lang.org/nomicon/

**Rust Reference**

https://doc.rust-lang.org/reference/

**Rust Embedded Book**

https://docs.rust-embedded.org/book/

The Embedded Book is useful for understanding constrained Rust execution environments, although embedded CPU programming and GPU programming are not interchangeable.

## GPU engineering

**AMD GPUOpen**

https://gpuopen.com/

**Mesa**

https://www.mesa3d.org/

**Programming Massively Parallel Processors — Kirk and Hwu**

## Further research

Study:

- Vulkan synchronization and memory models.
- SPIR-V compilation.
- GPU-safe Rust abstractions.
- Shader ABI design.
- GPU memory allocators.
- Asynchronous resource lifetime management.
- GPU runtime architecture.
- Rust compiler backends.

---

# 21. Completion Standard

You have completed Phase 12 when you can independently:

1. Explain how Rust source can reach GPU execution.
2. Explain SPIR-V's role.
3. Build a native Vulkan compute application in Rust.
4. Manage GPU buffers and allocations.
5. Create and use compute pipelines.
6. Record and submit command buffers.
7. Synchronize transfers and compute.
8. Establish host/device ABI compatibility.
9. Explain GPU resource lifetime problems.
10. Design safe Rust abstractions over unsafe Vulkan APIs.
11. Investigate Rust-to-SPIR-V compatibility.
12. Benchmark native Vulkan compute.
13. Compare Vulkan and wgpu implementations fairly.
14. Build and defend a reusable Rust native GPU runtime.

The decisive competency is not whether one experimental Rust-to-GPU compiler works on a particular date.

It is whether you understand the full chain from language semantics to GPU execution and can build correct systems around that chain.

---

# 22. What Comes Next

**Phase 13: GPU Runtime Systems**

The next phase focuses on building the infrastructure that makes GPU applications usable, efficient, and reliable.

Topics include:

- GPU runtime architecture.
- Device and queue management.
- Memory allocators.
- Buffer pooling.
- Asynchronous execution.
- Command scheduling.
- Dependency graphs.
- Event and fence management.
- Resource lifetime tracking.
- GPU task graphs.
- Multi-GPU execution.
- Error recovery.
- Device loss.
- Profiling infrastructure.
- Runtime performance optimization.
- Rust API design.
- Safe abstractions over native GPU APIs.

You will move from controlling individual native GPU operations to designing the runtime systems that coordinate complete GPU workloads.
