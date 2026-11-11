# Phase 4: Operating Systems and Linux Systems Programming

**Curriculum:** GPU Engineering  
**Phase:** 4 of 16  
**Estimated duration:** 10–14 weeks  
**Study commitment:** 10–12 hours per week  
**Primary environment:** Fedora Linux  
**Languages:** Rust and C++20  
**Prerequisites:** Phases 0–3

---

# 1. The Goal of This Phase

In Phase 3, you studied how processors execute instructions, how memory hierarchies work, and why hardware architecture affects performance.

Now you will learn how an operating system coordinates the hardware.

An operating system must solve several difficult problems:

- Multiple programs want to use the same CPU.
- Programs need memory but must not corrupt each other.
- Programs need to access files and devices.
- Multiple threads may access shared data.
- Devices execute work asynchronously.
- Programs must recover from failures.
- Applications must interact with hardware without having unrestricted control over the entire machine.

The operating system provides mechanisms to address these problems.

## Learning objectives

By the end of this phase, you should be able to:

1. Explain the responsibilities of an operating system.
2. Distinguish user mode from kernel mode.
3. Explain system calls and privilege transitions.
4. Understand processes, threads, and scheduling.
5. Explain virtual memory and memory protection.
6. Understand page faults, demand paging, and memory mapping.
7. Use Linux process and memory inspection tools.
8. Explain synchronization and common concurrency bugs.
9. Understand file descriptors, pipes, sockets, and event-driven I/O.
10. Explain interrupts, device drivers, and DMA.
11. Understand Linux device interfaces relevant to GPU programming.
12. Write robust Linux systems programs in Rust and C++.
13. Investigate performance using tracing and profiling tools.
14. Explain the operating system's role in GPU command execution.

## The central mental model

A useful abstraction is:

```text
Application
    |
    v
Libraries and Runtime
    |
    v
System Call Interface
    |
    v
Operating System Kernel
    |
    v
Device Drivers and Hardware
```

This is a simplified view, not a complete description of every execution path.

For example, ordinary arithmetic instructions execute directly on the CPU without making a system call. Some device operations use memory mappings and shared queues rather than a system call for every individual operation.

Understanding where the kernel participates—and where it does not—is an important part of this phase.

---

# 2. Your Linux Systems Laboratory

Use Fedora as your primary environment.

## 2.1 Install useful tools

```bash
sudo dnf install gcc-c++ clang rust cargo \
  gdb strace ltrace procps-ng \
  util-linux perf
```

Package availability and names can vary by Fedora release. Install any missing tools individually if necessary.

Useful commands:

| Tool | Purpose |
|---|---|
| `ps` | Inspect processes |
| `top` | Observe process activity |
| `htop` | Interactive process inspection, if installed |
| `strace` | Trace system calls |
| `ltrace` | Trace supported library calls |
| `gdb` | Debug native programs |
| `perf` | Analyze CPU performance |
| `lsof` | Inspect open files, if installed |
| `ls /proc` | Explore kernel process information |
| `ls /sys` | Explore kernel device and system interfaces |
| `dmesg` | Inspect kernel messages, subject to permissions |
| `journalctl` | Inspect system logs |

## 2.2 Explore your machine

Run:

```bash
uname -a
```

```bash
cat /proc/cpuinfo
```

```bash
cat /proc/meminfo
```

```bash
ls /proc/self/
```

```bash
ls /sys/class/
```

For each command, determine:

- What information it exposes.
- Whether that information comes from a normal file.
- Whether the information is generated dynamically by the kernel.
- Why the operating system exposes it.

## 2.3 Inspect a program's system calls

Compile a simple C++ program that prints a message.

Then run:

```bash
strace ./program
```

You may see calls related to:

- Opening files.
- Mapping memory.
- Loading libraries.
- Writing output.
- Exiting the process.

**Important observation:** even a trivial application depends on operating-system services.

---

# 3. Lesson One: What Is an Operating System?

## 3.1 The operating system as a resource manager

A computer contains finite resources:

- CPU execution time.
- Physical memory.
- Storage capacity.
- Network bandwidth.
- Device access.
- GPU execution capacity.

The operating system coordinates access to these resources.

Without coordination, multiple applications could interfere with one another.

## 3.2 The operating system as an abstraction layer

Hardware is complicated.

Instead of requiring applications to manage every physical detail, operating systems provide abstractions.

Examples:

| Hardware resource | OS abstraction |
|---|---|
| CPU execution | Process and thread |
| Physical memory | Virtual address space |
| Storage device | Files and directories |
| Network interface | Sockets |
| Hardware devices | Device files and driver interfaces |
| Device events | Interrupt and event mechanisms |

These abstractions simplify application development.

They also introduce costs and constraints.

## 3.3 Mechanism versus policy

This distinction is fundamental.

A **mechanism** describes how something can be done.

A **policy** describes which action should be chosen.

For example:

- Mechanism: The kernel can suspend one thread and resume another.
- Policy: The scheduler decides which thread should run next.

Keeping mechanism and policy separate allows systems to support different scheduling strategies and workloads.

## 3.4 Kernel versus user space

Modern operating systems commonly distinguish between:

- User mode: Restricted execution privileges.
- Kernel mode: Privileged execution with access to protected operations.

Applications normally run in user mode.

The kernel runs privileged code that manages resources and enforces protection.

A user-space program cannot arbitrarily modify kernel memory or directly execute every privileged instruction.

## 3.5 Why protection matters

Imagine two applications running simultaneously.

Application A contains a bug that writes outside an allocated buffer.

Without memory protection, the bug might overwrite Application B's data or kernel memory.

Memory protection helps isolate processes.

It does not eliminate every bug or security vulnerability, but it provides a fundamental boundary.

## Worked example: a file read

Suppose a program wants to read data from a file.

Conceptually:

1. The application requests access to a file.
2. The operating system checks the request.
3. The kernel resolves the file through the filesystem.
4. Data is obtained from a cache or storage device as needed.
5. The application receives the requested data or an error.

The program does not need to understand the physical layout of the storage device.

### Exercises

1. What resources does an operating system manage?
2. Why do applications need abstractions?
3. What is the difference between mechanism and policy?
4. Why is kernel mode privileged?
5. What problems does process isolation prevent?
6. Why does a file operation require operating-system cooperation?

---

# 4. Lesson Two: System Calls and Privilege Transitions

A system call is a controlled way for a user-space program to request an operating-system service.

## 4.1 Why system calls exist

Applications need operations such as:

- Creating processes.
- Reading files.
- Allocating or mapping memory.
- Creating sockets.
- Communicating with devices.

Some of these operations require privileged kernel participation.

A system call provides a defined interface.

## 4.2 System calls versus function calls

A normal function call transfers control within a program's execution environment.

A system call invokes a kernel service through an architecture-defined mechanism.

A system call may involve:

1. Preparing arguments.
2. Executing a system-call instruction.
3. Transitioning to privileged execution.
4. Validating arguments.
5. Performing the requested operation.
6. Returning a result or error.
7. Resuming user-space execution.

The precise details depend on the architecture and operating system.

## 4.3 Common Linux system calls

| System call | Purpose |
|---|---|
| `read` | Read bytes from a file descriptor |
| `write` | Write bytes |
| `openat` | Open a file relative to a directory context |
| `close` | Close a file descriptor |
| `mmap` | Create a memory mapping |
| `munmap` | Remove a memory mapping |
| `clone` | Create execution contexts with specified sharing |
| `execve` | Replace the current process image |
| `wait4` | Wait for child process state changes |
| `futex` | Support certain user-space synchronization operations |
| `ioctl` | Perform device-specific or other control operations |

Library functions are not necessarily one-to-one wrappers around system calls.

For example, a C library may buffer I/O or select different kernel interfaces depending on circumstances.

## 4.4 Errors and return values

System calls can fail.

Reasons include:

- Invalid arguments.
- Insufficient permissions.
- Resource exhaustion.
- Interrupted operations.
- Unavailable devices.
- Filesystem errors.

Systems programs must handle failure explicitly.

## Worked example: tracing output

Consider:

```cpp
#include <unistd.h>

int main() {
    const char message[] = "Hello\n";
    write(STDOUT_FILENO, message, sizeof(message) - 1);
}
```

This program requests a write to standard output.

On Linux, `STDOUT_FILENO` normally identifies file descriptor 1.

Trace it:

```bash
strace -e write ./program
```

Identify the write operation and its arguments.

### Exercises

1. Why is a system call different from an ordinary function call?
2. Why must the kernel validate system-call arguments?
3. What does `write` receive as arguments?
4. Why can a write operation fail?
5. Why might a library function avoid making a system call immediately?
6. Why is system-call overhead relevant to performance?

---

# 5. Lesson Three: Processes

A process is an operating-system abstraction representing an executing program and its associated resources.

## 5.1 What belongs to a process?

A process commonly has:

- A virtual address space.
- One or more threads.
- Open file descriptors.
- Credentials and permissions.
- Memory mappings.
- Process identifiers.
- Signal dispositions.
- Resource limits.

A process is not simply the executable file on disk.

The executable is program data; the process is a running execution environment.

## 5.2 Process creation

On Unix-like systems, process creation commonly involves `fork` or related facilities.

`fork` creates a child process.

The child begins with a process state derived from the parent.

The parent and child have separate process identities.

## 5.3 Copy-on-write

Duplicating every memory page immediately during `fork` would be expensive.

Operating systems commonly use copy-on-write.

Conceptually:

1. Parent and child initially reference shared physical pages.
2. Those mappings are protected as needed.
3. When a process writes to a shared copy-on-write page, the kernel may create a private copy.
4. The processes then have independent writable state.

This avoids unnecessary copying when pages are not modified.

## 5.4 Replacing a process image

The `exec` family of operations replaces the current process image with a new program.

This is commonly used after creating a child process.

The child can then execute another application.

## 5.5 Waiting for children

A parent process may wait for a child to finish.

Waiting allows the parent to retrieve termination status and enables the kernel to release certain process bookkeeping resources.

A terminated child that has not yet been reaped may remain as a zombie process.

## 5.6 Process states

Processes and threads can be:

- Running.
- Runnable.
- Sleeping.
- Waiting.
- Stopped.
- Terminated.

The precise state names depend on the operating system.

## 5.7 Context switching

A context switch changes which execution context runs on a CPU.

It may require saving and restoring architectural state and updating scheduler-related information.

Switching between processes can also affect cache and TLB behavior.

### Exercises

1. What is the difference between a program and a process?
2. What resources belong to a process?
3. What does `fork` create?
4. Why is copy-on-write useful?
5. What does `execve` do?
6. Why must a parent sometimes wait for a child?
7. What is a zombie process?
8. What happens during a context switch?

## Lab: Process Explorer

Write a program that:

1. Prints its process identifier.
2. Creates a child process.
3. Prints the parent and child identifiers.
4. Demonstrates that the parent and child have separate writable state.
5. Waits for the child.
6. Reports its termination status.

Inspect the processes using `ps` and `/proc`.

**Deliverable:** Source code and a short explanation of the observed process behavior.

---

# 6. Lesson Four: Threads and CPU Scheduling

A process can contain multiple threads.

Threads share many process resources but maintain separate execution state.

## 6.1 What is a thread?

A thread is an independently schedulable execution context.

Threads within the same process commonly share:

- Virtual address space.
- Open file descriptors.
- Global variables.
- Heap allocations.

Each thread has its own:

- Instruction pointer.
- Register state.
- Stack.
- Thread-local storage.

## 6.2 Threads versus processes

Processes generally provide stronger address-space isolation.

Threads allow easier shared-memory communication but introduce synchronization challenges.

Neither is universally better.

## 6.3 Scheduling

The scheduler decides which runnable thread receives CPU time.

Scheduling must balance competing goals:

- Fairness.
- Responsiveness.
- Throughput.
- Latency.
- Priority.
- CPU affinity.
- Resource utilization.

Different workloads may benefit from different scheduling policies.

## 6.4 Preemption

A preemptive scheduler can interrupt a running thread and allow another thread to execute.

This helps prevent one ordinary user-space thread from monopolizing the processor indefinitely.

## 6.5 CPU affinity

CPU affinity restricts or influences which CPUs a thread may execute on.

Affinity can affect:

- Cache locality.
- NUMA memory access.
- Scheduling flexibility.
- Benchmark consistency.

It should be used carefully.

## 6.6 Oversubscription

Suppose a machine has eight logical CPUs.

A program creates 1,000 CPU-bound threads.

Only a limited number can execute simultaneously.

The remaining threads must wait for CPU time.

Excessive runnable threads can increase scheduling overhead and contention.

More threads do not automatically mean more throughput.

## 6.7 Blocking versus spinning

A thread waiting for an event can:

- Block, allowing the scheduler to run other work.
- Spin, repeatedly checking whether the event occurred.

Spinning may reduce wake-up latency in some circumstances but consumes CPU resources.

Blocking is often preferable for longer or unpredictable waits.

### Exercises

1. How do threads differ from processes?
2. Which resources do threads share?
3. What is preemption?
4. What is a runnable thread?
5. What is CPU affinity?
6. Why can oversubscription hurt performance?
7. What is the difference between blocking and spinning?
8. Why does thread scheduling matter to performance benchmarking?

## Lab: Thread Scheduling

Build a program that creates different numbers of CPU-bound threads.

Test configurations with:

- Fewer threads than logical CPUs.
- Approximately the same number of threads as logical CPUs.
- More threads than logical CPUs.

Measure total completion time.

Record the CPU model, workload, and thread count.

Explain your observations using scheduling and resource contention.

---

# 7. Lesson Five: Virtual Memory in Practice

Phase 3 introduced virtual memory as a hardware and architectural concept.

This lesson examines how the operating system uses it.

## 7.1 Process address spaces

A process typically has regions used for:

- Executable code.
- Read-only data.
- Writable global data.
- Heap allocations.
- Thread stacks.
- Shared libraries.
- Memory mappings.

The exact arrangement varies across processes and platforms.

## 7.2 Memory allocation

Consider a C++ allocation:

```cpp
auto buffer = std::make_unique<int[]>(1000000);
```

The allocator may obtain memory from existing reserved regions or request additional virtual memory from the operating system.

Allocating virtual address space does not necessarily mean every corresponding physical page is immediately committed and populated.

## 7.3 Demand paging

Demand paging allows the operating system to provide physical memory when pages are actually accessed.

A process may reserve a large virtual region without immediately touching every page.

When the process accesses a page, the kernel may need to establish the appropriate mapping.

## 7.4 Page faults

A page fault can happen because:

- A valid page has not yet been populated.
- A copy-on-write page must be duplicated.
- A memory-mapped file page must be loaded.
- An access violates memory protection.

The kernel determines whether the fault can be resolved.

Not every page fault indicates a bug.

## 7.5 Memory mapping

`mmap` allows a process to map memory regions.

Mappings may be:

- Anonymous.
- File-backed.
- Shared.
- Private.
- Read-only.
- Writable.

These properties influence visibility, persistence, and access permissions.

## 7.6 Shared memory

Processes can communicate through shared memory mappings.

This can avoid copying data through certain other communication mechanisms.

However, shared memory requires synchronization and careful lifetime management.

## 7.7 Memory protection

Pages may have permissions such as:

- Read.
- Write.
- Execute.

The kernel and hardware enforce these permissions.

This is one foundation of process isolation.

## 7.8 Huge pages

Some systems support larger page sizes.

Larger pages can reduce the number of translations required to cover a large memory region.

However, they introduce trade-offs involving allocation, fragmentation, and system configuration.

### Worked example: a large allocation

Suppose a program reserves a large anonymous memory region.

It then touches one byte on each page.

Questions:

- How many pages are involved?
- How many page faults might occur?
- Why might the first pass be slower?
- What changes during the second pass?
- How does page size affect the number of translations?

The exact result depends on the kernel, allocation method, page state, and hardware.

### Exercises

1. Explain the difference between virtual address reservation and physical memory use.
2. What is demand paging?
3. What is copy-on-write?
4. What is a shared mapping?
5. What is a private mapping?
6. Why can first-touch memory access be slower?
7. Why do page permissions matter?
8. What trade-offs do huge pages introduce?

## Lab: Memory Mapping Explorer

Write a program that:

1. Creates a large anonymous mapping.
2. Measures the time required to touch each page.
3. Repeats the same traversal.
4. Reports relevant page-fault statistics where available.
5. Compares different access patterns.
6. Releases the mapping correctly.

Use `/proc/self/maps` to inspect the process address space.

**Deliverable:** Source code and a report explaining the observations.

---

# 8. Lesson Six: Synchronization and Concurrency Correctness

Multiple threads can access shared memory concurrently.

Without appropriate synchronization, their interactions may produce incorrect behavior.

## 8.1 Race conditions

A race condition occurs when behavior depends on the relative timing of concurrent operations.

Some race conditions are harmless or intentional; others cause correctness problems.

A data race has a more specific meaning in languages such as C++ and Rust.

## 8.2 Data races

In C++, a data race generally occurs when conflicting accesses to the same memory location happen without the required happens-before relationship and at least one access is non-atomic.

A data race causes undefined behavior.

Rust's safe concurrency abstractions are designed to prevent many forms of data races, although logical race conditions remain possible.

## 8.3 Atomic operations

Atomic operations provide indivisible operations on supported memory objects.

Examples include:

- Atomic load.
- Atomic store.
- Atomic exchange.
- Atomic fetch-add.
- Compare-and-exchange.

Atomicity does not automatically make an entire algorithm correct.

## 8.4 Memory ordering

Modern processors and compilers may reorder operations under specific constraints.

Concurrent programs must establish the required ordering and visibility relationships.

Important C++ memory orders include:

- Relaxed.
- Acquire.
- Release.
- Acq_rel.
- Sequentially consistent.

You should understand the purpose of these categories.

Do not assume ordinary source-code order always describes the order observed by other threads.

## 8.5 Mutexes

A mutex provides mutual exclusion.

Only one thread can hold the mutex at a time.

Mutexes are useful when multiple operations must be protected as one critical section.

## 8.6 Condition variables

A condition variable allows threads to wait for a shared condition to become true.

Correct condition-variable usage requires checking a predicate because wakeups can occur without the desired condition being satisfied.

## 8.7 Deadlocks

A deadlock occurs when threads wait in a cycle that prevents progress.

For example:

- Thread A holds lock X and waits for lock Y.
- Thread B holds lock Y and waits for lock X.

Neither can proceed.

## 8.8 Livelock and starvation

Livelock occurs when threads continue changing state but fail to make useful progress.

Starvation occurs when a thread is repeatedly denied the resources it needs to progress.

These are different failure modes.

## 8.9 False sharing

You encountered false sharing in Phase 3.

Now connect it to multithreaded systems programming.

Two threads can update separate variables without a data race, yet their writes may contend for the same cache line.

This can substantially reduce throughput.

### Exercises

1. What is a race condition?
2. What is a data race?
3. Why does atomicity not guarantee algorithmic correctness?
4. What does a mutex protect?
5. What is a critical section?
6. What does acquire-release ordering accomplish conceptually?
7. Why must condition-variable waits use a predicate?
8. Explain deadlock.
9. Explain livelock.
10. Explain starvation.
11. Explain false sharing.

## Lab: Concurrent Counter

Implement a counter updated by multiple threads.

Compare:

- An intentionally unsynchronized version used only to demonstrate the problem under controlled conditions.
- A mutex-protected version.
- An atomic version.

Validate correctness and measure performance.

Explain why the results differ.

Do not treat the unsynchronized C++ version's observed output as predictable; a data race is undefined behavior.

---

# 9. Lesson Seven: Files, File Descriptors, and I/O

Operating systems provide abstractions for communicating with files and devices.

On Unix-like systems, the file descriptor is a particularly important abstraction.

## 9.1 File descriptors

A file descriptor is a process-local integer handle used to refer to an open file description or other supported I/O object.

Common descriptors include:

- 0: Standard input.
- 1: Standard output.
- 2: Standard error.

Descriptors can refer to:

- Regular files.
- Pipes.
- Sockets.
- Certain devices.
- Other kernel-supported objects.

## 9.2 Buffered I/O

Libraries often buffer I/O to reduce the frequency of system calls.

For example, writing one byte at a time through a buffered stream may result in fewer kernel writes than one system call per byte.

Buffering improves efficiency but introduces additional state and flush behavior.

## 9.3 Blocking I/O

A blocking operation may wait until data becomes available or an operation completes.

This is often simple to program but can limit responsiveness.

## 9.4 Nonblocking I/O

Nonblocking I/O allows operations to return without waiting for certain conditions.

Applications must then handle partial progress or temporary unavailability.

## 9.5 Pipes

A pipe provides a byte stream between communicating execution contexts.

Pipes are commonly used for:

- Shell pipelines.
- Parent-child communication.
- Streaming data between processes.

## 9.6 Sockets

Sockets provide an interface for communication.

They can be used for:

- Network communication.
- Local interprocess communication.

## 9.7 Event-driven I/O

Linux provides mechanisms such as `epoll` for monitoring many file descriptors for readiness events.

This allows a program to coordinate large numbers of I/O operations without creating one dedicated blocking thread for every connection.

## 9.8 Asynchronous I/O

Asynchronous I/O allows operations to progress independently of the application's immediate execution flow.

Linux also provides interfaces such as `io_uring` for certain asynchronous I/O workloads.

These interfaces have their own capabilities, limitations, and kernel-version requirements.

### Exercises

1. What is a file descriptor?
2. Why are standard input and output represented using descriptors?
3. Why is buffered I/O useful?
4. What is blocking I/O?
5. What is nonblocking I/O?
6. What is a pipe?
7. What is a socket?
8. What problem does `epoll` solve?
9. How does asynchronous I/O differ from synchronous blocking I/O?

## Lab: File Copy Utility

Implement a file-copying program in Rust or C++.

Requirements:

- Accept input and output paths.
- Handle partial reads and writes correctly.
- Handle errors.
- Support configurable buffer sizes.
- Measure throughput.
- Validate that the output matches the input.

Experiment with buffer sizes.

Explain how buffering and system-call frequency affect performance.

---

# 10. Lesson Eight: Interrupts, Drivers, and DMA

This lesson begins connecting operating systems directly to GPU hardware.

## 10.1 Device communication

The CPU communicates with hardware devices through defined interfaces.

Devices may expose:

- Control registers.
- Memory-mapped regions.
- Command queues.
- Status registers.
- Interrupt mechanisms.

The exact interface depends on the device and platform.

## 10.2 Memory-mapped I/O

Some device registers are mapped into the CPU's address space.

Software can access these regions using appropriate memory operations.

However, device memory is not ordinary RAM.

It may have special ordering, caching, and access requirements.

Ordinary C++ or Rust memory operations are not automatically sufficient to implement a correct device driver.

## 10.3 Interrupts

An interrupt allows hardware or the system to request attention from the processor.

For example, a device may generate an interrupt after completing an operation.

The operating system handles the event and performs appropriate follow-up work.

## 10.4 Polling

Instead of waiting for an interrupt, software may repeatedly check a device's status.

Polling can be useful when:

- Events occur frequently.
- Latency requirements are strict.
- The cost of interrupt handling is undesirable.

But polling consumes processing resources.

## 10.5 Direct Memory Access

Direct Memory Access, or DMA, allows a device to transfer data to or from memory without requiring the CPU to perform every individual data movement operation.

Conceptually:

1. Software prepares a transfer.
2. The device receives the necessary configuration.
3. The device performs the transfer.
4. Completion is detected through an interrupt, polling, or another mechanism.

The CPU still participates in setup, synchronization, and coordination.

## 10.6 IOMMU

An Input-Output Memory Management Unit can translate and restrict device memory accesses.

This supports isolation and flexible device memory mappings.

The IOMMU is particularly relevant when devices perform DMA.

## 10.7 Device drivers

A device driver is software that understands a device's interface.

Drivers may be responsible for:

- Initialization.
- Resource allocation.
- Command submission.
- Interrupt handling.
- Memory management.
- Error recovery.
- Device synchronization.

Drivers bridge generic operating-system mechanisms and device-specific behavior.

### Exercises

1. What is memory-mapped I/O?
2. Why is device memory different from ordinary RAM?
3. What is an interrupt?
4. What is polling?
5. What is DMA?
6. Why is DMA useful?
7. What does an IOMMU do?
8. What responsibilities belong to a device driver?
9. Why must CPU and device memory accesses be synchronized?

---

# 11. Lesson Nine: How Linux Interacts With a GPU

This is the most directly GPU-specific lesson in the phase.

Your current GPU is an AMD Radeon RX 9070 with 16 GB of VRAM, and your primary development environment is Fedora Linux.

You will eventually use Vulkan and wgpu to submit compute workloads to it.

Understanding the operating-system layer will help you debug failures and performance problems.

## 11.1 The graphics and compute software stack

A simplified Linux GPU stack looks like:

```text
Rust or C++ Application
          |
          v
wgpu / Vulkan / Other Compute API
          |
          v
User-Space Driver and Runtime
          |
          v
Linux Kernel GPU Driver
          |
          v
GPU Hardware
```

Different APIs use different runtimes and driver components.

The diagram is intentionally simplified.

## 11.2 User-space drivers

User-space driver components translate high-level API operations into lower-level commands and manage API-specific state.

For Vulkan on AMD Linux systems, Mesa's RADV driver is one possible implementation.

The exact stack depends on the installed drivers and selected Vulkan device.

## 11.3 Kernel drivers

Linux kernel GPU drivers manage privileged device operations.

For supported modern AMD GPUs, the `amdgpu` kernel driver is an important component.

It participates in operations such as:

- Device initialization.
- Memory management.
- Scheduling.
- Command submission.
- Synchronization.
- Fault handling.

## 11.4 GPU memory

A GPU has its own memory hierarchy and may have dedicated VRAM.

Applications typically do not directly control arbitrary physical VRAM addresses.

Instead, APIs and drivers provide abstractions for:

- Allocating buffers.
- Mapping memory where supported.
- Transferring data.
- Synchronizing access.
- Managing residency.

## 11.5 Command buffers and queues

A GPU workload is commonly represented as commands submitted to a queue.

The CPU prepares work.

The GPU executes that work asynchronously.

This means the CPU and GPU may be performing different tasks simultaneously.

## 11.6 Synchronization

Because GPU execution is asynchronous, applications need ways to determine when work is complete.

Different APIs expose mechanisms such as:

- Fences.
- Semaphores.
- Barriers.
- Queue synchronization.
- Submission completion events.

These mechanisms have different meanings and should not be treated as interchangeable.

## 11.7 GPU scheduling

Multiple applications may want to use the same GPU.

The driver and hardware coordinate execution and resource use.

Scheduling details depend on the GPU architecture, driver, and operating system.

## 11.8 Why this matters

Suppose a GPU compute program is unexpectedly slow.

Possible causes include:

- CPU-side preparation.
- Driver overhead.
- Command submission overhead.
- Data transfer costs.
- Synchronization.
- GPU kernel execution.
- Memory allocation behavior.
- GPU contention.

You cannot diagnose these issues effectively if you treat the GPU as a single opaque function call.

### Exercises

1. What is the role of a user-space GPU driver?
2. What is the role of a kernel GPU driver?
3. Why are GPU commands submitted asynchronously?
4. Why does GPU synchronization matter?
5. Why is GPU memory allocation more complicated than ordinary CPU heap allocation?
6. What is a command queue?
7. Why might a GPU workload be slow even when the kernel itself is fast?
8. How do the operating system and driver contribute to GPU execution?

---

# 12. Lesson Ten: Linux Observability and Debugging

A systems engineer must be able to investigate a running system.

## 12.1 `/proc`

The `/proc` filesystem exposes information about processes and kernel state.

Useful paths include:

```text
/proc/self/maps
/proc/self/status
/proc/self/stat
/proc/self/fd
/proc/meminfo
/proc/cpuinfo
```

Explore what each exposes.

## 12.2 `/sys`

The `/sys` filesystem exposes information about devices, drivers, and kernel objects.

Useful areas include:

```text
/sys/class/
/sys/devices/
/sys/bus/
/sys/module/
```

These are interfaces to kernel-managed state, not ordinary files stored on disk.

## 12.3 `strace`

`strace` helps answer:

- Which system calls are being made?
- Which calls are failing?
- Is the program repeatedly waiting?
- Is it opening the expected files?
- Is it making unexpectedly many small I/O requests?

## 12.4 `perf`

`perf` can help investigate:

- CPU utilization.
- Cycles.
- Instructions.
- Cache behavior.
- Branch misses.
- Sampling profiles.
- Certain scheduling events.

Available events depend on the processor and permissions.

## 12.5 GDB

GDB can inspect:

- Call stacks.
- Threads.
- Registers.
- Memory.
- Breakpoints.
- Program state.

You should be comfortable debugging a multithreaded program.

## 12.6 Sanitizers

Clang and GCC support useful instrumentation tools.

Examples include:

- AddressSanitizer.
- UndefinedBehaviorSanitizer.
- ThreadSanitizer.

These tools detect different classes of problems.

They do not prove that a program is correct.

## 12.7 Investigating GPU problems

On Fedora, useful starting points may include:

```bash
ls /dev/dri/
```

```bash
lsmod | grep amdgpu
```

```bash
journalctl -k
```

Some systems expose additional GPU-related information through `/sys` or debug interfaces.

Access may be restricted.

Do not modify driver settings or kernel parameters casually. Begin with read-only inspection.

### Exercises

1. What is `/proc`?
2. What is `/sys`?
3. How does `strace` differ from `perf`?
4. What information does GDB provide?
5. What kinds of bugs can sanitizers detect?
6. Why might a profiler report misleading results?
7. How would you begin investigating a process that uses unexpectedly high CPU time?
8. How would you begin investigating a GPU application that appears to hang?

---

# 13. Major Coding Assessment A: Process and Thread Inspector

Build a command-line Linux monitoring tool.

Use Rust as the primary implementation language.

## Required features

Your application must:

1. Accept a process identifier.
2. Read process information from `/proc`.
3. Display process state.
4. Display memory usage information.
5. Display thread information.
6. Display open file descriptor information where permitted.
7. Refresh information periodically.
8. Handle process termination gracefully.
9. Handle missing or inaccessible information.
10. Produce both human-readable and machine-readable output.

## Engineering requirements

Your implementation must:

- Avoid panicking on ordinary operational errors.
- Handle malformed or changing `/proc` data.
- Avoid assuming that process state remains constant between reads.
- Distinguish missing information from zero-valued information.
- Document limitations.

## Technical defense

You must explain:

- Why `/proc` information can change between reads.
- Why a process can disappear during inspection.
- What process identifiers represent.
- Why memory statistics can be difficult to interpret.
- Why file descriptor inspection may fail.
- How your application handles errors.

---

# 14. Major Coding Assessment B: Multithreaded Task Executor

Build a small task execution system in C++20 or Rust.

## Requirements

The system must:

1. Accept tasks from a producer.
2. Execute tasks using a configurable number of worker threads.
3. Support orderly shutdown.
4. Prevent lost tasks.
5. Avoid data races.
6. Avoid deadlocks.
7. Handle task failures according to a documented policy.
8. Collect throughput and latency measurements.
9. Support multiple workload types.
10. Include concurrency tests.

## Required experiments

Compare performance using:

- One worker.
- A moderate number of workers.
- More workers than logical CPUs.
- CPU-bound tasks.
- Tasks containing simulated waits.

## Technical defense

Explain:

- Which state is shared.
- How that state is protected.
- Which synchronization operations are required.
- How workers are awakened.
- What happens during shutdown.
- How you know that no tasks are lost.
- Why throughput changes with worker count.
- Whether the workload is CPU-bound or wait-bound.

---

# 15. Major Coding Assessment C: Memory-Mapped Data Processor

Build a program that processes a binary data file using memory mapping.

Use either Rust or C++.

## Requirements

The program must:

1. Open a binary file.
2. Validate its size and format.
3. Map the file into memory.
4. Process the data.
5. Report summary statistics.
6. Handle invalid input.
7. Release all resources correctly.
8. Compare mapped access with conventional buffered I/O.
9. Measure runtime and throughput.
10. Produce a written analysis.

## Technical defense

Explain:

- How virtual memory mappings work.
- What causes pages to become resident.
- How page faults may influence performance.
- What happens when the mapped file is invalid or unexpectedly changes.
- How you avoid out-of-bounds access.
- Why memory mapping is not automatically faster than buffered I/O.

---

# 16. Final Capstone: A Linux GPU Systems Investigation

This is the final project for Phase 4.

You are not yet expected to write a GPU kernel.

Instead, you will investigate how Linux exposes and manages your AMD GPU.

## Objective

Produce a technical report explaining the software and operating-system components involved in executing a GPU compute workload on your Fedora machine.

## Required investigation

Your report should identify:

1. The detected GPU hardware.
2. The relevant kernel driver.
3. The user-space graphics or compute driver stack.
4. Relevant device nodes.
5. Available Vulkan support.
6. The relationship between the application and the GPU driver.
7. How GPU memory allocation is exposed to applications.
8. How commands are submitted.
9. How execution completion is reported.
10. What information is available for debugging failures.

## Constraints

- Prefer read-only inspection.
- Do not change kernel configuration unless you understand the consequences.
- Do not assume ROCm support merely because the GPU is manufactured by AMD.
- Do not assume CUDA can run natively on your AMD GPU.
- Distinguish verified observations from architectural expectations.

## Deliverables

- A system inventory.
- A diagram of the software stack.
- Relevant command output.
- A technical explanation.
- A troubleshooting checklist.
- A list of unresolved questions to investigate during the GPU programming phases.

This project creates a bridge between operating-system knowledge and actual GPU development.

---

# 17. Phase 4 Quiz Bank

Use these questions for independent practice or AI-administered quizzes.

## Operating-system fundamentals

1. What problem does an operating system solve?
2. What is the difference between mechanism and policy?
3. What is the difference between user mode and kernel mode?
4. Why are system calls necessary?
5. What is the difference between a system call and a function call?
6. Why must the kernel validate application requests?

## Processes and threads

7. What is a process?
8. What is a thread?
9. What resources do threads share?
10. What is a context switch?
11. What is copy-on-write?
12. What does `fork` do?
13. What does `execve` do?
14. What is a zombie process?
15. What is preemption?
16. What is CPU affinity?
17. What is oversubscription?
18. What is the difference between blocking and spinning?

## Virtual memory

19. What is a virtual address space?
20. What is demand paging?
21. What is a page fault?
22. What is memory mapping?
23. What is the difference between shared and private mappings?
24. What is copy-on-write memory?
25. Why might the first access to a page be slower?
26. What is a TLB?
27. What are huge pages?
28. What is NUMA?

## Concurrency

29. What is a race condition?
30. What is a data race?
31. What is a mutex?
32. What is an atomic operation?
33. What is a memory-ordering constraint?
34. What is a condition variable?
35. What is a deadlock?
36. What is a livelock?
37. What is starvation?
38. What is false sharing?

## I/O

39. What is a file descriptor?
40. What is buffered I/O?
41. What is nonblocking I/O?
42. What is a pipe?
43. What is a socket?
44. What problem does `epoll` solve?
45. What is asynchronous I/O?
46. Why do partial reads and writes matter?

## Devices and GPU systems

47. What is an interrupt?
48. What is polling?
49. What is DMA?
50. What does an IOMMU do?
51. What is a device driver?
52. What is memory-mapped I/O?
53. What is a GPU command queue?
54. Why is GPU execution asynchronous?
55. What is the difference between user-space and kernel GPU drivers?
56. Why does GPU memory require synchronization?
57. What does the Linux `amdgpu` driver do conceptually?
58. Why can GPU performance be affected by CPU-side overhead?

## Observability

59. What information does `/proc` expose?
60. What information does `/sys` expose?
61. What does `strace` reveal?
62. What does `perf` reveal?
63. What does GDB reveal?
64. What is a sanitizer?
65. Why must benchmark results be interpreted carefully?

---

# 18. Phase 4 Final Examination

**Recommended duration:** 4–6 hours.

| Section | Weight |
|---|---:|
| OS fundamentals and system calls | 10% |
| Processes and threads | 15% |
| Scheduling | 10% |
| Virtual memory | 15% |
| Synchronization | 15% |
| Files and I/O | 10% |
| Devices and GPU interfaces | 15% |
| Debugging and performance analysis | 10% |

## Examination structure

### Part A: Conceptual reasoning

Explain operating-system mechanisms from first principles.

### Part B: Process analysis

Interpret process states, memory mappings, and system-call traces.

### Part C: Concurrency reasoning

Analyze hypothetical multithreaded programs for correctness and progress.

### Part D: Performance analysis

Reason about context switching, system-call overhead, memory faults, and I/O buffering.

### Part E: GPU systems

Explain the journey from a user-space GPU API call to GPU execution and completion.

### Part F: Practical defense

Defend one of your Phase 4 coding assessments.

Your defense should address:

- Correctness.
- Resource ownership.
- Failure handling.
- Synchronization.
- Performance.
- Limitations.

---

# 19. Copy-Ready AI Tutor Prompts

All prompts are grouped here to reduce scrolling.

The rules below are intentionally strict: the tutor must **never provide answers, code, pseudocode, implementations, fixes, or completed work**. Its only job is to guide your reasoning through questions.

## Master Socratic Tutor Prompt

Copy this once when starting an AI tutoring session.

```text
You are my Socratic operating-systems tutor and assessment examiner.

I am studying Phase 4 of a rigorous GPU engineering curriculum:
Operating Systems and Linux Systems Programming.

NON-NEGOTIABLE RULES:

1. Never give me the answer to a question, quiz, exercise, lab,
   coding assessment, or examination.

2. Never write code or pseudocode for me.

3. Never implement, modify, complete, debug, or fix my work.

4. Never execute commands, edit files, or perform assessment tasks
   on my behalf.

5. Never reveal answer keys, completed solutions, or hidden tests.

6. Never give hints that effectively disclose the solution.

7. Guide me exclusively through questions.

8. Ask one primary question per response.

9. If I am incorrect, do not tell me the correct answer.
   Ask a question that exposes the relevant misconception.

10. If I am stuck, reduce the conceptual difficulty and ask
    a simpler prerequisite question.

11. Prioritize first-principles reasoning over memorization.

12. Require me to explain assumptions, invariants, failure cases,
    correctness, resource ownership, and performance trade-offs.

13. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

14. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

15. For code reviews, ask questions about possible issues.
    Never provide corrected code or exact fixes.

16. Never provide a final answer even if I explicitly ask for one
    during the assessment.

17. Keep your responses concise and focused.

Your response must contain only guiding questions.
```

## Start a lesson

```text
Teach me Phase 4, Lesson [NUMBER], using the Socratic method.

Start from first principles.
Ask one question at a time.
Test my understanding before increasing difficulty.
Never provide answers.
```

## Generate a quiz

```text
Administer a 20-question Phase 4 quiz on [TOPIC].

Include conceptual, practical, debugging, and performance questions.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers or solutions.
```

## Start a coding assessment

```text
Administer Phase 4 Coding Assessment [A/B/C].

Present the requirements and constraints only.
Ask me to explain my design before implementation.

Guide me through questions about correctness,
resource ownership, synchronization, failure handling,
testing, and performance.

Do not write, fix, or execute code.
```

## Review my code

```text
Review my Phase 4 systems code using only questions.

Focus on:
- Resource ownership
- Error handling
- Memory safety
- Concurrency correctness
- Deadlocks and races
- System-call behavior
- Performance
- Edge cases

Do not identify exact fixes or provide replacement code.
Ask one guiding question at a time.
```

## Debugging assessment

```text
Give me a Linux systems debugging assessment.

Present symptoms, relevant logs, and observations.
Make me determine what to inspect next.

Do not reveal the root cause or provide the fix.
Ask questions that guide my investigation.
```

## GPU systems oral examination

```text
Examine my understanding of how Linux interacts with GPUs.

Ask me to explain user-space drivers, kernel drivers,
memory mappings, command queues, DMA, synchronization,
and asynchronous execution.

Ask one question at a time.
Do not provide explanations or answers.
```

---

# 20. Mastery Checklist

## Operating-system fundamentals

- [ ] I can explain the responsibilities of an operating system.
- [ ] I understand user mode and kernel mode.
- [ ] I understand system calls.
- [ ] I understand the difference between mechanism and policy.

## Processes and threads

- [ ] I understand processes and threads.
- [ ] I understand process creation.
- [ ] I understand copy-on-write.
- [ ] I understand scheduling and preemption.
- [ ] I understand context switching.
- [ ] I understand CPU affinity and oversubscription.

## Memory

- [ ] I understand virtual memory.
- [ ] I understand demand paging.
- [ ] I understand page faults.
- [ ] I understand memory mapping.
- [ ] I understand memory protection.
- [ ] I understand shared memory.

## Concurrency

- [ ] I understand data races.
- [ ] I understand mutexes.
- [ ] I understand atomic operations.
- [ ] I understand memory ordering conceptually.
- [ ] I understand condition variables.
- [ ] I can reason about deadlock, livelock, and starvation.

## I/O and devices

- [ ] I understand file descriptors.
- [ ] I understand buffered and nonblocking I/O.
- [ ] I understand pipes and sockets.
- [ ] I understand interrupts.
- [ ] I understand DMA.
- [ ] I understand the role of device drivers.

## GPU systems

- [ ] I understand the basic Linux GPU software stack.
- [ ] I understand user-space and kernel drivers.
- [ ] I understand GPU command submission conceptually.
- [ ] I understand asynchronous execution.
- [ ] I understand why GPU synchronization is required.
- [ ] I can identify the GPU driver stack on my Fedora system.

## Practical engineering

- [ ] I can inspect processes using Linux tools.
- [ ] I can trace system calls.
- [ ] I can inspect memory mappings.
- [ ] I can write a correct multithreaded application.
- [ ] I can handle resource and system-call failures.
- [ ] I can investigate performance using measurements.
- [ ] I can explain the operating system's role in GPU execution.

---

# 21. Recommended Resources

## Primary textbook

**Operating Systems: Three Easy Pieces — Remzi H. Arpaci-Dusseau and Andrea C. Arpaci-Dusseau**

https://pages.cs.wisc.edu/~remzi/OSTEP/

This is the primary textbook for Phase 4.

Focus on:

- Virtualization.
- Processes.
- Scheduling.
- Virtual memory.
- Concurrency.
- Persistence.

Work through the exercises rather than only reading the chapters.

## Linux systems programming

**The Linux Programming Interface — Michael Kerrisk**

A comprehensive reference for Linux and Unix systems programming.

**Linux man-pages**

https://man7.org/linux/man-pages/

Use the manual pages to understand actual API contracts.

Examples:

```bash
man 2 mmap
man 2 clone
man 2 read
man 2 write
man 7 futex
```

## Concurrency

**C++ Concurrency in Action — Anthony Williams**

Useful for understanding concurrency correctness and memory ordering.

**The Rustonomicon**

https://doc.rust-lang.org/nomicon/

Advanced Rust memory-safety concepts. Read selectively.

## Linux architecture

**Linux Kernel Documentation**

https://docs.kernel.org/

Useful for:

- Memory management.
- Scheduling.
- Driver interfaces.
- Synchronization.
- GPU subsystems.

## GPU-specific Linux interfaces

**Linux DRM GPU Driver Documentation**

https://docs.kernel.org/gpu/

Use this to begin understanding the Linux Direct Rendering Manager subsystem and GPU driver architecture.

**Mesa Documentation**

https://docs.mesa3d.org/

Useful for understanding user-space graphics drivers on Linux.

---

# 22. Completion Standard

You have completed Phase 4 when you can independently:

1. Explain how a Linux process executes.
2. Trace a system call from user space into the kernel conceptually.
3. Explain process and thread scheduling.
4. Describe how virtual memory and page faults work.
5. Write correct multithreaded code.
6. Reason about synchronization and memory ordering.
7. Build robust systems programs that handle failures.
8. Investigate CPU and memory behavior using Linux tools.
9. Explain how applications communicate with GPU drivers.
10. Defend your practical assessments without relying on an AI to solve them.

You do not need to become a Linux kernel developer at this stage.

You do need enough operating-system knowledge to understand the software and hardware boundary that GPU applications cross.

---

# 23. What Comes Next

**Phase 5: Mathematics and Numerical Computing**

The next phase develops the mathematical foundations required for high-performance GPU programming:

- Linear algebra.
- Vector and matrix operations.
- Numerical representation.
- Floating-point error.
- Numerical stability.
- Parallel reductions.
- Numerical algorithms.
- Matrix multiplication.
- Computational complexity.
- Numerical performance analysis.

This will prepare you to understand not only how GPU kernels execute, but also how to design mathematically correct and efficient computations for them.
