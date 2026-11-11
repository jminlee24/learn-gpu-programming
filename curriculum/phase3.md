# Phase 3: Computer Architecture — From Logic Gates to Modern Processors

**Curriculum:** GPU Engineering  
**Phase:** 3 of 16  
**Estimated duration:** 10–14 weeks  
**Recommended study time:** 10–12 hours per week  
**Prerequisites:** Phase 0 — Mathematical Foundations; Phase 1 — Rust Systems Programming; Phase 2 — Modern C++

---

# 1. What You Are Learning

A programmer writes instructions, but a processor executes machine instructions. Between those two things is an enormous amount of machinery:

- Instruction sets.
- Registers.
- Arithmetic and logic units.
- Instruction pipelines.
- Branch predictors.
- Cache hierarchies.
- Memory controllers.
- Virtual memory systems.
- Multiple execution units.
- Hardware-level parallelism.

Understanding these components changes how you approach programming.

For example, consider two programs that perform exactly the same mathematical operation.

One accesses memory sequentially. The other accesses memory randomly.

Their algorithmic complexity might be identical, but their execution times can differ dramatically because of cache behavior.

This phase teaches you to explain why.

## Learning objectives

By the end of this phase, you should be able to:

1. Explain the fundamental components of a computer.
2. Distinguish an instruction set architecture from a microarchitecture.
3. Explain how a processor fetches, decodes, and executes instructions.
4. Read basic x86-64 assembly.
5. Understand registers, the stack, and calling conventions.
6. Explain how arithmetic and memory instructions execute.
7. Understand instruction pipelines and hazards.
8. Explain superscalar execution and out-of-order execution.
9. Understand branch prediction and speculative execution.
10. Explain cache hierarchies and cache coherence.
11. Understand virtual memory, address translation, and TLBs.
12. Explain SIMD and CPU vector instructions.
13. Analyze memory access patterns and their performance implications.
14. Use hardware counters and benchmarks to investigate performance.
15. Explain how CPU architecture differs from GPU architecture.

## The central mental model

Think of a computer as a hierarchy of increasingly concrete abstractions:

| Level | What it describes |
|---|---|
| Application | The program and its algorithm |
| Programming language | C++, Rust, and their semantics |
| Compiler | Converts source code into lower-level representations |
| Instruction set architecture | Defines the machine instructions and programmer-visible CPU behavior |
| Microarchitecture | Defines how a particular processor implements those instructions |
| Digital logic | Gates, registers, multiplexers, and arithmetic circuits |
| Physical hardware | Transistors, wires, voltage, and electrical behavior |

The purpose of this phase is to understand the levels between your source code and the physical processor.

---

# 2. Set Up Your Architecture Laboratory

You will investigate actual compiled programs rather than relying entirely on diagrams.

## 2.1 Install development tools

On Fedora:

```bash
sudo dnf install gcc-c++ clang binutils gdb cmake make
```

Useful tools include:

| Tool | Purpose |
|---|---|
| GCC | Compile C and C++ |
| Clang | Alternative compiler |
| GDB | Debug programs |
| objdump | Inspect machine code |
| readelf | Inspect executable metadata |
| nm | Inspect symbols |
| lscpu | Inspect CPU information |
| Compiler Explorer | Inspect generated assembly |
| perf | Investigate hardware performance counters, where supported |

You can inspect your CPU with:

```bash
lscpu
```

Useful information includes:

- Architecture.
- Number of CPUs and cores.
- Threads per core.
- CPU model.
- Cache sizes.
- Instruction-set extensions.
- NUMA configuration.

Do not assume every CPU has the same capabilities. The architecture of the processor you are using is part of your experimental environment.

## 2.2 Inspect a compiled program

Create a simple program:

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int main() {
    std::cout << add(10, 20) << '\n';
}
```

Compile it:

```bash
g++ -std=c++20 -O0 -g main.cpp -o program
```

Inspect its assembly:

```bash
objdump -d -C program
```

Now compile with optimization:

```bash
g++ -std=c++20 -O2 -g main.cpp -o program_optimized
```

Inspect the optimized version:

```bash
objdump -d -C program_optimized
```

Compare the two.

The output may be dramatically different. The compiler can inline functions, eliminate unnecessary operations, and transform expressions while preserving the observable behavior required by the language.

This leads to a fundamental principle:

**Your source code describes behavior. It does not prescribe the exact machine instructions that must execute.**

---

# 3. Lesson One: What Is a Computer?

Before learning how a processor works, define what a computer needs to do.

At the most basic level, a computer must:

1. Represent information.
2. Store information.
3. Perform operations on information.
4. Make decisions.
5. Move information between components.
6. Coordinate the execution of operations.

These requirements lead to the major components of a computer.

## 3.1 The processor

The central processing unit, or CPU, executes instructions.

A simplified CPU contains:

- Control logic.
- Registers.
- Arithmetic and logic units.
- Instruction decoding logic.
- Execution units.
- Interfaces to the memory hierarchy.

The CPU is not simply a calculator. It is a machine that repeatedly interprets and executes encoded instructions.

## 3.2 Memory

Memory stores information that a program needs.

The processor must retrieve instructions and data from memory to execute a program.

Different forms of memory have different properties:

- Capacity.
- Latency.
- Bandwidth.
- Cost.
- Physical location.
- Persistence.

You will study the memory hierarchy later.

## 3.3 Input and output

Computers interact with external devices through input/output systems.

Examples include:

- Storage devices.
- Network interfaces.
- Keyboards.
- Displays.
- GPUs.
- USB devices.

These devices may communicate with the processor through different buses, controllers, and protocols.

## 3.4 The stored-program concept

A fundamental idea in modern computing is that program instructions can be represented in memory.

The processor fetches instructions from memory and executes them.

This is commonly associated with the von Neumann architecture.

The important conceptual distinction is between:

- Instructions: describe operations.
- Data: values on which operations are performed.

Both can be represented as bits.

## 3.5 The basic execution cycle

A simplified processor repeatedly performs:

1. Fetch an instruction.
2. Decode the instruction.
3. Obtain required operands.
4. Execute the operation.
5. Store or communicate the result.
6. Continue with the next instruction.

Modern processors overlap and reorganize many of these activities, but this model is a useful starting point.

### Questions to answer

1. What is the difference between a CPU and memory?
2. Why does a processor need registers if it already has access to memory?
3. Why must a processor fetch instructions?
4. What determines which instruction executes next?
5. Why is a computer more than a CPU?
6. What does the stored-program concept allow a computer to do?

---

# 4. Lesson Two: Instruction Set Architecture

The instruction set architecture, or ISA, defines the interface between software and the processor.

It is one of the most important abstractions in computer architecture.

## 4.1 What an ISA defines

An ISA describes things such as:

- Available instructions.
- Register names and behavior.
- Data types supported by instructions.
- Addressing modes.
- Memory access rules.
- Exception behavior.
- Privilege levels.
- Instruction encoding.

For example, an ISA may define an instruction that adds two integer registers and writes the result into another register.

It specifies what that instruction means, not necessarily how the hardware performs it internally.

## 4.2 ISA versus microarchitecture

This distinction is essential.

An ISA is a specification.

A microarchitecture is a particular hardware implementation of that specification.

For example, two processors can implement the same ISA while having different:

- Pipeline depths.
- Cache sizes.
- Execution units.
- Branch predictors.
- Instruction scheduling mechanisms.
- Clock frequencies.

They can run the same compiled executable but have very different performance.

## 4.3 Common ISAs

You should recognize the major instruction set families:

| ISA | General characteristics |
|---|---|
| x86-64 | Widely used in desktop and server processors |
| ARM64 / AArch64 | Widely used in mobile, embedded, and server systems |
| RISC-V | Open ISA with a modular extension model |

The distinction between an ISA and a processor brand matters.

A CPU is not an ISA. Multiple manufacturers can implement the same ISA.

## 4.4 CISC and RISC

Historically, instruction sets have often been categorized as:

- CISC: Complex Instruction Set Computer.
- RISC: Reduced Instruction Set Computer.

x86 is traditionally classified as CISC, while ARM and RISC-V are commonly classified as RISC.

This is useful historical context, but modern implementations blur the distinction.

For example, a modern x86 processor may translate complex architectural instructions into simpler internal operations called micro-operations.

The ISA is not necessarily a direct description of the processor's internal execution machinery.

## 4.5 Machine instructions

A machine instruction is encoded as bits.

An instruction may specify:

- An operation.
- Source registers.
- Destination registers.
- Immediate values.
- Memory addressing information.

Consider a conceptual instruction:

```text
ADD destination, source
```

It represents an addition operation.

The exact syntax and behavior depend on the ISA.

### Exercises

1. Explain the difference between ISA and microarchitecture.
2. Research which ISA your CPU implements.
3. Identify at least five instructions supported by that ISA.
4. Explain why two processors implementing the same ISA can have different performance.
5. Explain why a compiler must know the target ISA.
6. Explain why an executable compiled for one ISA cannot necessarily run on another.

---

# 5. Lesson Three: Registers and the CPU Datapath

Registers are small, fast storage locations inside the processor.

They are central to instruction execution.

## 5.1 Why registers exist

Suppose a processor needs to calculate:

\[
a+b
\]

It needs the operands and a destination for the result.

A simplified processor might:

1. Load `a` into a register.
2. Load `b` into another register.
3. Add the registers.
4. Store the result.

Registers provide a convenient location for values used by execution units.

## 5.2 General-purpose registers

An x86-64 processor exposes general-purpose registers such as:

- RAX.
- RBX.
- RCX.
- RDX.
- RSI.
- RDI.
- RSP.
- RBP.
- R8 through R15.

These registers have specific roles in different instructions and calling conventions.

Other architectures use different register naming systems.

## 5.3 Special-purpose registers

Processors also have registers with specialized roles.

Examples include:

- Program counter or instruction pointer.
- Stack pointer.
- Flags or status register.
- Control registers.

The instruction pointer identifies the location of the next instruction according to the architectural execution model.

The stack pointer tracks the current stack position.

## 5.4 The arithmetic logic unit

The arithmetic logic unit, or ALU, performs operations such as:

- Addition.
- Subtraction.
- Bitwise AND.
- Bitwise OR.
- Bitwise XOR.
- Comparisons.
- Shifts.

A processor may contain multiple arithmetic and logic execution units.

## 5.5 The datapath

The datapath is the hardware through which values move and operations are performed.

A simplified datapath includes:

- Registers.
- ALUs.
- Multiplexers.
- Data paths.
- Control signals.

Control logic determines which values move where and which operation occurs.

This connects directly to the digital logic you studied in Phase 0.

## 5.6 Flags

Some instruction sets maintain status flags after arithmetic operations.

On x86, examples include:

- Zero flag.
- Carry flag.
- Sign flag.
- Overflow flag.

These flags can influence conditional instructions.

Do not confuse carry and overflow.

Carry generally describes unsigned arithmetic behavior, while signed overflow describes a result outside the representable signed range.

### Exercises

1. Explain why registers are useful.
2. What is the difference between a general-purpose register and a special-purpose register?
3. What does the instruction pointer represent?
4. What is the role of the ALU?
5. What is the difference between carry and signed overflow?
6. Trace a simple addition using registers, an ALU, and a destination register.

---

# 6. Lesson Four: Reading Assembly

Assembly language provides a human-readable representation of machine instructions.

It is one of the best ways to connect source code to processor behavior.

## 6.1 What assembly tells you

Assembly can reveal:

- Which instructions the compiler generated.
- Which registers are used.
- How values are moved.
- How functions are called.
- How branches are implemented.
- Whether vector instructions are used.
- Whether memory is accessed.

Assembly does not necessarily reveal every internal hardware operation.

The processor may decode an instruction into several internal micro-operations.

## 6.2 A basic x86-64 example

Consider:

```cpp
int add(int a, int b) {
    return a + b;
}
```

A simplified assembly representation might look like:

```asm
add:
    mov eax, edi
    add eax, esi
    ret
```

The exact output depends on the compiler, optimization settings, target architecture, and calling convention.

Conceptually:

- The input arguments arrive in registers.
- The first argument is copied into a register used for the return value.
- The second argument is added.
- The result is returned.

On common x86-64 System V systems, integer arguments are often passed in registers such as RDI and RSI, with the integer return value in RAX.

This is an ABI convention, not a universal rule for every x86-64 environment.

## 6.3 Common instructions

Learn to recognize:

| Instruction | General purpose |
|---|---|
| `mov` | Move or copy data |
| `add` | Addition |
| `sub` | Subtraction |
| `imul` | Integer multiplication |
| `and` | Bitwise AND |
| `or` | Bitwise OR |
| `xor` | Bitwise XOR |
| `cmp` | Compare operands |
| `test` | Perform a bitwise test |
| `jmp` | Unconditional branch |
| `je` | Branch if equal |
| `jne` | Branch if not equal |
| `call` | Call a function |
| `ret` | Return from a function |
| `push` | Push onto the stack |
| `pop` | Pop from the stack |

The exact semantics depend on operand sizes and addressing modes.

## 6.4 Addressing modes

An instruction may operate on:

- An immediate constant.
- A register.
- A memory location.
- A computed memory address.

For example, an instruction might use an address formed from:

\[
\text{base}+\text{index}\times\text{scale}+\text{displacement}
\]

This is important for understanding array indexing and pointer arithmetic.

## 6.5 Branches

High-level conditionals often become conditional branches or conditional data movement.

For example:

```cpp
if (x > 10) {
    y = 1;
} else {
    y = 0;
}
```

The compiler may use a comparison followed by a conditional branch, or it may produce a branchless sequence.

You should learn to inspect both possibilities.

## 6.6 Assembly exercises

Create small functions for:

1. Integer addition.
2. Integer multiplication.
3. Array indexing.
4. A loop.
5. An `if` statement.
6. A function call.
7. A struct access.

Compile each with `-O0` and `-O2`.

For every example, answer:

- Which registers contain the inputs?
- Which instructions perform the computation?
- Which memory locations are accessed?
- Does the compiler use a branch?
- Did optimization change the instruction sequence?
- Can you explain the behavior without referring to the original source code?

---

# 7. Lesson Five: The Stack, Calling Conventions, and Function Calls

A function call is not magic. It requires a protocol between the caller and the callee.

That protocol is commonly described by a calling convention.

## 7.1 What happens during a function call?

A simplified function call may involve:

1. Placing arguments in registers or memory.
2. Saving information needed by the caller.
3. Transferring control to the function.
4. Creating or using a stack frame.
5. Executing the function.
6. Placing the return value in the required location.
7. Restoring the caller's state.
8. Returning to the caller.

Real implementations may omit some of these operations.

## 7.2 The stack

The stack is a region of memory used to manage function calls and automatic storage.

It commonly contains:

- Return addresses.
- Saved registers.
- Local variables.
- Temporary data.
- Other call-related information.

The precise layout depends on the ABI and compiler.

## 7.3 Stack pointer and frame pointer

The stack pointer tracks the current stack position.

A frame pointer, when used, can provide a stable reference point for a function's stack frame.

Optimized code may omit a dedicated frame pointer.

## 7.4 Stack frames

Consider:

```cpp
int calculate(int x) {
    int result = x * 2;
    return result;
}
```

A compiler may place some or all of this function's state in registers rather than allocating explicit stack slots.

The source-level existence of a local variable does not guarantee that it exists as a separate memory location in the optimized executable.

## 7.5 Calling conventions

A calling convention specifies rules such as:

- How arguments are passed.
- Where return values are placed.
- Which registers a function must preserve.
- Which registers a function may overwrite.
- How the stack is aligned.
- How control is transferred.

You should recognize the System V AMD64 ABI because it is common on Linux x86-64 systems.

## 7.6 Recursion

Recursive functions create nested function calls.

Each active invocation needs its own execution state.

This is one reason recursive algorithms can consume stack space.

A recursion depth that becomes too large may cause stack exhaustion.

## 7.7 Inlining

A compiler may inline a function by replacing a call with its body or an equivalent optimized representation.

Inlining can eliminate call overhead and expose additional optimization opportunities.

However, inlining can also increase code size and instruction-cache pressure.

### Exercises

1. Write a function with several arguments.
2. Inspect where the arguments are passed.
3. Write a function that calls another function.
4. Inspect the call and return instructions.
5. Compare a recursive function with an iterative equivalent.
6. Explain what a stack frame represents.
7. Explain why a local variable may not appear in memory.
8. Explain why inlining can both help and hurt performance.

---

# 8. Lesson Six: How a Processor Executes Instructions

You now know what instructions and registers are. Next, examine how processors execute instructions efficiently.

## 8.1 The simple sequential model

Imagine a processor executing one instruction at a time:

1. Fetch.
2. Decode.
3. Execute.
4. Access memory if needed.
5. Write back the result.

This is easy to understand, but it does not use the hardware efficiently.

## 8.2 Instruction pipelining

A pipeline divides instruction execution into stages.

Consider a simplified five-stage pipeline:

1. Instruction fetch.
2. Instruction decode.
3. Execute.
4. Memory access.
5. Write back.

Different instructions can occupy different stages simultaneously.

For example:

| Cycle | Instruction A | Instruction B | Instruction C |
|---|---|---|---|
| 1 | Fetch | | |
| 2 | Decode | Fetch | |
| 3 | Execute | Decode | Fetch |
| 4 | Memory | Execute | Decode |
| 5 | Write back | Memory | Execute |

The pipeline increases throughput by overlapping work.

It does not necessarily reduce the latency of an individual instruction.

This distinction between latency and throughput is fundamental.

## 8.3 Pipeline hazards

A hazard is a condition that prevents the pipeline from proceeding as originally planned.

Three major categories are:

### Data hazards

An instruction depends on a result that has not yet become available.

Example:

```text
A: R1 = R2 + R3
B: R4 = R1 + R5
```

Instruction B depends on instruction A.

### Control hazards

The processor does not yet know which instruction should execute next because of a branch.

### Structural hazards

Two operations require the same hardware resource at the same time.

## 8.4 Forwarding

A processor may forward a result directly from one pipeline stage to another instead of waiting for it to be written back to the register file.

This can reduce stalls caused by dependencies.

## 8.5 Stalls

A stall occurs when an instruction or pipeline stage must wait.

Stalls reduce throughput.

Understanding what causes them is essential for performance analysis.

## 8.6 Latency versus throughput

Latency measures how long an operation takes.

Throughput measures how many operations can be completed per unit of time.

A processor may have relatively high latency for a particular operation but still sustain high throughput by executing multiple independent operations concurrently.

### Exercises

1. Explain the purpose of pipelining.
2. Explain why pipelining increases throughput.
3. Distinguish latency from throughput.
4. Identify a data dependency between two instructions.
5. Explain a control hazard.
6. Explain what forwarding accomplishes.
7. Predict how a long dependency chain affects execution.

---

# 9. Lesson Seven: Superscalar and Out-of-Order Execution

Modern CPUs execute instructions much more aggressively than the simple pipeline model suggests.

## 9.1 Superscalar execution

A superscalar processor can issue multiple instructions in a cycle when enough independent execution resources are available.

For example, a processor may have:

- Multiple integer execution units.
- Floating-point units.
- Vector units.
- Load and store units.
- Branch execution resources.

The processor attempts to keep these units busy.

## 9.2 Instruction-level parallelism

Instruction-level parallelism, or ILP, is the ability to execute multiple independent instructions at the same time.

Consider:

```text
A = B + C
D = E + F
G = H + I
```

These calculations are independent.

A processor may execute them concurrently.

Now consider:

```text
A = B + C
D = A + E
G = D + F
```

These operations form a dependency chain.

The processor must respect the dependencies.

## 9.3 Out-of-order execution

A processor may execute instructions in a different order from the program's original instruction order, provided that the required architectural behavior is preserved.

This is called out-of-order execution.

It allows the processor to execute independent instructions while another instruction is waiting for data.

## 9.4 Reorder buffers

Many modern processors use structures such as reorder buffers to track instructions and preserve the required architectural retirement order.

The processor may execute instructions out of order but commit their results in an order that maintains the required program semantics.

This helps support precise exceptions and predictable architectural behavior.

## 9.5 Register renaming

Architectural registers are limited.

Processors can use a larger set of physical registers internally.

Register renaming helps eliminate false dependencies caused by multiple instructions reusing the same architectural register name.

Three important dependency categories are:

- Read after write: RAW.
- Write after read: WAR.
- Write after write: WAW.

RAW is a true data dependency. WAR and WAW can often be eliminated through register renaming.

## 9.6 Why this matters to programmers

The processor can exploit independent work, but it cannot freely violate true dependencies.

A program with long dependency chains may execute more slowly than one with enough independent operations.

This becomes important when designing CPU algorithms and eventually GPU kernels.

### Exercises

1. What is instruction-level parallelism?
2. What makes two instructions independent?
3. Why can out-of-order execution improve performance?
4. What is a reorder buffer?
5. What problem does register renaming solve?
6. Explain RAW, WAR, and WAW dependencies.
7. Why can't the processor simply execute every instruction in parallel?

---

# 10. Lesson Eight: Branch Prediction and Speculative Execution

Conditional branches create uncertainty about which instruction executes next.

Waiting for every branch to resolve can reduce pipeline efficiency.

Processors use branch prediction to make educated guesses.

## 10.1 Conditional branches

Consider:

```cpp
if (value > threshold) {
    process_a();
} else {
    process_b();
}
```

The processor may not know the branch outcome immediately.

It may need to wait for the comparison result.

## 10.2 Branch prediction

A branch predictor estimates which direction a branch will take.

Modern processors use sophisticated mechanisms based on prior branch behavior and other information.

The predictor may correctly predict a branch and allow execution to continue without waiting.

## 10.3 Speculative execution

A processor may execute instructions along a predicted path before the branch is fully resolved.

If the prediction is correct, useful work has already been completed.

If it is incorrect, the processor discards speculative results and redirects execution.

This is called a branch misprediction recovery or pipeline flush, depending on the implementation.

## 10.4 Branch predictability

Consider two cases:

- A condition that is almost always true.
- A condition whose outcome appears random.

The first may be easier for a branch predictor to predict.

The second may result in more mispredictions.

However, source-level appearance alone does not prove the performance outcome. You must measure the compiled program on the target hardware.

## 10.5 Branchless programming

Some algorithms can be expressed without conditional branches.

However, branchless code is not automatically faster.

It may perform unnecessary operations, increase register pressure, or prevent other optimizations.

The right approach depends on the workload and target architecture.

## 10.6 Security implications

Speculative execution has security implications because speculative work can affect microarchitectural state, such as caches.

Some attacks exploit observable timing differences to infer information.

You should recognize the concepts:

- Spectre.
- Meltdown.
- Cache side channels.
- Speculation barriers.

You do not need to implement an attack. You should understand why architectural correctness alone does not describe every observable microarchitectural effect.

### Exercises

1. Explain why branches can stall a pipeline.
2. What is a branch predictor?
3. What is speculative execution?
4. What happens when a branch prediction is incorrect?
5. Why is branchless code not always faster?
6. Explain the difference between architectural state and microarchitectural state.
7. Explain why caches can become relevant to security.

---

# 11. Lesson Nine: The Memory Hierarchy

This is one of the most important lessons in the entire curriculum.

A processor can perform arithmetic extremely quickly, but retrieving data from memory may be comparatively expensive.

To reduce this gap, computers use a hierarchy of storage.

## 11.1 The memory hierarchy

A simplified hierarchy looks like this:

| Level | Typical role | General characteristics |
|---|---|---|
| Registers | Immediate operands | Very small and very fast |
| L1 cache | Frequently accessed data | Small and very fast |
| L2 cache | Larger working set | Larger, generally slower |
| L3 cache | Shared or last-level cache | Larger, generally slower |
| Main memory | Program data | Much larger, higher latency |
| Storage | Persistent data | Much larger, much slower |

Exact sizes, organization, and latency vary by processor.

## 11.2 Why caches exist

The CPU is much faster than main memory.

If every memory access required waiting for main memory, many programs would spend much of their time waiting.

Caches keep copies of recently or frequently used memory data closer to the processor.

## 11.3 Locality

Caches exploit locality.

Two major forms matter:

### Temporal locality

If a program accesses a location, it may access that same location again soon.

Example:

```cpp
int total = 0;

for (int i = 0; i < 1000; ++i) {
    total += values[i];
}
```

The variable `total` is repeatedly used.

### Spatial locality

If a program accesses one memory location, it may soon access nearby locations.

Sequential array traversal demonstrates spatial locality.

## 11.4 Cache lines

Caches transfer data in blocks called cache lines.

A cache line is a contiguous region of memory.

Many modern CPUs use cache lines of 64 bytes, but you should verify the actual hardware rather than assuming a universal size.

Suppose a cache line contains 64 bytes.

If your program accesses one integer from that line, the hardware may bring the entire line into the cache.

This makes nearby accesses potentially inexpensive.

## 11.5 Cache hits and misses

A cache hit occurs when the requested data is present in the relevant cache.

A cache miss occurs when it is not.

A miss may require retrieving the data from a lower level of the memory hierarchy.

The cost depends on which level contains the data.

## 11.6 Cache miss categories

A useful conceptual classification includes:

- Compulsory misses: first access to data.
- Capacity misses: the working set exceeds available cache capacity.
- Conflict misses: multiple memory locations compete for the same cache locations.

These categories help explain why cache behavior differs across algorithms.

## 11.7 Associativity

A cache must determine where a memory block can be stored.

Common organization types include:

- Direct-mapped.
- Set-associative.
- Fully associative.

Associativity affects placement flexibility and conflict behavior.

You should understand the basic trade-off between placement flexibility, lookup complexity, and hardware cost.

## 11.8 Cache replacement

When a cache set is full, the hardware may need to evict an existing line.

Replacement policies determine which line is selected.

The exact policy depends on the implementation.

Do not assume every cache uses a simple least-recently-used policy.

## 11.9 Write policies

Caches also need policies for writes.

Two important concepts are:

- Write-through: writes are propagated to a lower memory level.
- Write-back: modified data may remain in the cache until eviction or another coherence-related event.

You should understand the conceptual distinction.

## 11.10 A practical example: row-major versus column-major access

Consider a two-dimensional array stored contiguously in row-major order.

```cpp
constexpr int N = 4096;

std::vector<int> data(N * N);
```

A row-major traversal:

```cpp
for (int row = 0; row < N; ++row) {
    for (int col = 0; col < N; ++col) {
        data[row * N + col] += 1;
    }
}
```

A column-major traversal:

```cpp
for (int col = 0; col < N; ++col) {
    for (int row = 0; row < N; ++row) {
        data[row * N + col] += 1;
    }
}
```

Both traverse every element.

Their algorithmic complexity is the same:

\[
O(N^2)
\]

But their memory access patterns differ.

The first visits nearby elements consecutively.

The second jumps between distant memory regions.

The first generally makes better use of spatial locality.

This is an essential foundation for understanding GPU memory coalescing later.

### Exercises

1. Explain why caches exist.
2. Explain temporal locality.
3. Explain spatial locality.
4. What is a cache line?
5. What is a cache hit?
6. What is a cache miss?
7. Explain compulsory, capacity, and conflict misses.
8. Compare direct-mapped and set-associative caches.
9. Explain why row-major traversal generally benefits from spatial locality.
10. Explain why two algorithms with the same Big-O complexity can have very different runtimes.

---

# 12. Lesson Ten: Cache Coherence and False Sharing

When a CPU has multiple cores, each core may have its own private caches.

This introduces a new problem.

What happens when multiple cores access the same memory?

## 12.1 The coherence problem

Imagine two cores.

Core A reads a memory location.

Core B reads the same location.

Core A then writes a new value.

Core B must not continue treating its old cached value as the current coherent value.

The system needs a mechanism to coordinate cached copies.

This is the cache coherence problem.

## 12.2 Cache coherence protocols

Processors use cache coherence protocols to coordinate ownership and validity of cache lines.

One family of protocols is based on states such as:

- Modified.
- Exclusive.
- Shared.
- Invalid.

This is commonly called MESI.

You do not need to assume every processor implements exactly the same protocol. Understand the purpose of the state transitions.

## 12.3 Cache-line ownership

A write to a cache line may require obtaining exclusive ownership of that line.

If another core has a copy, the coherence system may need to invalidate or update its state.

This communication has a performance cost.

## 12.4 False sharing

Suppose two threads update different variables.

```cpp
struct Counters {
    int first;
    int second;
};
```

Thread A updates `first`.

Thread B updates `second`.

There may be no data race because the threads access different objects.

However, if both variables occupy the same cache line, their writes may cause cache-line ownership to move between cores.

This is false sharing.

False sharing is a performance problem, not necessarily a correctness problem.

## 12.5 Padding and alignment

Programmers sometimes use padding or alignment to separate frequently written variables.

C++ provides `alignas` for specifying alignment requirements.

C++ also provides `std::hardware_destructive_interference_size` on implementations that support it, as a hint for separating data that should avoid sharing a cache line.

Treat these values as implementation-dependent guidance, not universal physical truths.

## 12.6 Why this matters for GPUs

GPU architectures also have memory hierarchies and coordination between processing units.

The exact coherence model and caching behavior differ by architecture.

Do not directly assume that CPU cache coherence rules apply to GPU memory.

The transferable lesson is that memory access and ownership can have costs beyond the arithmetic performed by the program.

### Exercises

1. Explain why multiple CPU cores create a cache coherence problem.
2. What does a cache invalidation accomplish?
3. What does the MESI acronym represent?
4. Explain false sharing.
5. Why can false sharing happen without a data race?
6. How can data layout influence false sharing?
7. Why should you measure before adding padding?

---

# 13. Lesson Eleven: Virtual Memory and Address Translation

Programs use virtual addresses.

Physical memory uses physical addresses.

The operating system and processor cooperate to translate between them.

## 13.1 Why virtual memory exists

Virtual memory provides each process with an address space that is isolated from other processes.

It allows programs to use memory without directly managing physical memory placement.

It also supports:

- Memory protection.
- Process isolation.
- Demand paging.
- Shared memory mappings.
- Memory-mapped files.
- Address-space organization.

## 13.2 Virtual addresses

A virtual address identifies a location in a process's address space.

Two different processes may use the same virtual address while referring to different physical memory.

The operating system manages the mappings.

## 13.3 Pages

Virtual memory is generally managed in fixed-size blocks called pages.

The exact page sizes supported depend on the architecture and operating system.

A virtual address can be divided conceptually into:

- A virtual page number.
- An offset within the page.

The page number is used for translation. The offset identifies the location inside the page.

## 13.4 Page tables

Page tables describe mappings between virtual and physical memory.

The memory-management unit, or MMU, performs address translation using these structures.

## 13.5 Translation lookaside buffer

Page-table walks can require additional memory accesses.

Processors use a translation lookaside buffer, or TLB, to cache recent address translations.

A TLB hit means the translation is already cached.

A TLB miss means the processor may need to perform additional translation work.

## 13.6 Page faults

A page fault occurs when a memory access requires operating-system handling because the current mapping or permissions do not immediately permit the access.

A page fault does not always mean an invalid memory access.

It can occur during legitimate demand paging.

However, accessing memory in violation of protection rules can cause a process-level fault or termination.

## 13.7 Memory-mapped files

Operating systems can map file contents into a process's virtual address space.

This allows programs to interact with file-backed data using memory access operations.

It is useful for large datasets and certain storage workloads.

## 13.8 NUMA

Non-uniform memory access, or NUMA, describes systems where memory access costs depend on which processor or socket accesses which memory region.

On multi-socket systems, local memory may be faster than remote memory.

This becomes important in high-performance computing and server systems.

### Exercises

1. Explain why processes use virtual addresses.
2. What is the difference between virtual and physical addresses?
3. What is a page?
4. What does the MMU do?
5. What is a page table?
6. What does a TLB cache?
7. Why is a TLB miss potentially expensive?
8. Why is a page fault not necessarily an invalid access?
9. Explain NUMA.
10. Why might the physical location of memory matter for performance?

---

# 14. Lesson Twelve: CPU SIMD and Vector Instructions

A CPU can perform operations on multiple data elements using vector instructions.

This is called SIMD: Single Instruction, Multiple Data.

SIMD is one of the closest conceptual connections between CPU programming and GPU programming.

## 14.1 Scalar execution

A scalar addition operates on one pair of values:

\[
c=a+b
\]

For a vector of values, a scalar implementation repeats the operation:

\[
c_i=a_i+b_i
\]

for each element \(i\).

## 14.2 SIMD execution

A vector instruction operates on multiple values packed into a register.

Conceptually:

\[
\begin{bmatrix}
c_0\\
c_1\\
c_2\\
c_3
\end{bmatrix}
=
\begin{bmatrix}
a_0\\
a_1\\
a_2\\
a_3
\end{bmatrix}
+
\begin{bmatrix}
b_0\\
b_1\\
b_2\\
b_3
\end{bmatrix}
\]

One instruction performs multiple element-wise operations.

## 14.3 Vector registers

Modern CPUs provide vector registers.

Examples of x86 vector instruction families include:

- SSE.
- AVX.
- AVX2.
- AVX-512, where supported.

Different CPU models support different subsets.

You must inspect the target CPU's capabilities rather than assuming a particular extension is available.

## 14.4 Auto-vectorization

Compilers can sometimes transform ordinary loops into vector instructions.

For example:

```cpp
for (int i = 0; i < count; ++i) {
    output[i] = input_a[i] + input_b[i];
}
```

A compiler may generate SIMD instructions if it can establish that vectorization is valid and beneficial.

## 14.5 Why vectorization can fail

Potential obstacles include:

- Data dependencies.
- Unknown pointer aliasing.
- Irregular memory access.
- Unsupported operations.
- Alignment constraints.
- Small loop sizes.
- Complicated control flow.

The compiler must preserve the required semantics.

## 14.6 SIMD versus SIMT

You will encounter both terms:

- SIMD: Single Instruction, Multiple Data.
- SIMT: Single Instruction, Multiple Threads.

GPU programming commonly uses a SIMT execution model.

SIMD and SIMT are related concepts, but they are not interchangeable descriptions of every CPU and GPU architecture.

A GPU's threads, warps or wavefronts, scheduling model, and memory hierarchy introduce additional behavior.

### Exercises

1. Explain scalar execution.
2. Explain SIMD execution.
3. What is a vector register?
4. What is auto-vectorization?
5. Why might a compiler fail to vectorize a loop?
6. Explain why alignment can matter for vector operations.
7. Compare SIMD and SIMT conceptually.
8. Explain why SIMD is a useful bridge toward GPU programming.

---

# 15. Lesson Thirteen: Memory Bandwidth, Latency, and the Roofline Model

Understanding memory behavior leads to a more useful way of analyzing program performance.

## 15.1 Memory latency

Latency is the time required for a memory operation to complete.

A program may spend significant time waiting for data even if the requested operation is simple.

## 15.2 Memory bandwidth

Bandwidth measures how much data can be transferred per unit of time.

For example:

\[
\text{Bandwidth}=
\frac{\text{Bytes transferred}}{\text{Time}}
\]

A system can have high bandwidth but still have significant latency.

These are different properties.

## 15.3 Compute throughput

Compute throughput measures how many operations a processor can perform per unit of time.

A program may be limited by:

- Compute throughput.
- Memory bandwidth.
- Memory latency.
- Instruction dependencies.
- Branch behavior.
- Synchronization.
- Other resource constraints.

## 15.4 Arithmetic intensity

Arithmetic intensity measures how many arithmetic operations are performed per byte transferred.

\[
I=\frac{\text{Operations}}{\text{Bytes transferred}}
\]

A low arithmetic intensity program performs relatively little computation for each byte moved.

A high arithmetic intensity program performs more computation per byte.

## 15.5 The Roofline model

The Roofline model relates attainable performance to arithmetic intensity.

A simplified upper bound is:

\[
P \leq \min(P_{\text{peak}}, B_{\text{memory}}\times I)
\]

Where:

- \(P\) is attainable performance.
- \(P_{\text{peak}}\) is peak compute throughput.
- \(B_{\text{memory}}\) is memory bandwidth.
- \(I\) is arithmetic intensity.

This model helps identify whether a workload is likely compute-bound or memory-bound.

The real machine may have additional limits, so the Roofline model is a diagnostic abstraction rather than a guarantee.

## 15.6 Example: vector addition

Suppose you calculate:

\[
C_i=A_i+B_i
\]

For each element, assuming 32-bit floats:

- Read 4 bytes from A.
- Read 4 bytes from B.
- Write 4 bytes to C.
- Perform approximately one addition.

Ignoring cache effects and other details:

\[
\text{Bytes per element}=12
\]

\[
I\approx\frac{1}{12}
\]

operations per byte.

This is a low arithmetic intensity workload.

It is likely to be constrained by memory movement rather than arithmetic throughput on a sufficiently large dataset.

## 15.7 Why this matters for GPUs

GPUs often provide enormous arithmetic throughput and high memory bandwidth.

But high theoretical compute performance does not guarantee that a particular kernel will run quickly.

If a kernel spends most of its time moving data, additional arithmetic units may not improve performance.

This is why GPU optimization requires understanding both computation and memory behavior.

### Exercises

1. Explain memory latency.
2. Explain memory bandwidth.
3. Why are they different?
4. Calculate the bytes transferred by a simple vector operation.
5. Calculate arithmetic intensity.
6. Explain what it means for a workload to be memory-bound.
7. Explain what it means for a workload to be compute-bound.
8. Use the Roofline model to reason about a vector addition workload.

---

# 16. Lesson Fourteen: Benchmarking and Performance Analysis

You now have enough architecture knowledge to begin measuring real programs.

The goal is not merely to determine which program is faster. It is to explain why.

## 16.1 Why benchmarking is difficult

Performance measurements can be affected by:

- CPU frequency scaling.
- Thermal throttling.
- Background processes.
- Cache state.
- Memory allocation.
- Compiler optimization.
- Input size.
- Thread scheduling.
- Operating-system activity.
- Measurement overhead.

A single timing result is rarely sufficient.

## 16.2 Benchmarking methodology

A reasonable benchmark process is:

1. Define the workload.
2. Define the input size.
3. Compile with known settings.
4. Run correctness tests.
5. Warm up if appropriate.
6. Perform repeated measurements.
7. Record the distribution of results.
8. Compare equivalent workloads.
9. Inspect generated assembly.
10. Investigate hardware counters when available.

## 16.3 Avoid dead-code elimination

The compiler may remove calculations whose results are never used.

A benchmark must ensure that the work being measured is observable.

Otherwise, you may be measuring almost nothing.

## 16.4 Compiler optimization

Compare:

- `-O0`
- `-O1`
- `-O2`
- `-O3`

Do not assume the highest optimization level always produces the fastest result.

Inspect the generated code and measure.

## 16.5 Hardware counters

Hardware performance counters can expose information such as:

- CPU cycles.
- Instructions retired.
- Branches.
- Branch misses.
- Cache references.
- Cache misses.
- Context switches.

On Linux, `perf` can expose many of these counters when supported and permitted by system configuration.

Example:

```bash
perf stat ./program
```

You may need to adjust system permissions or use a different event depending on your hardware.

## 16.6 Cycles per instruction

CPI measures average cycles per instruction:

\[
CPI=\frac{\text{CPU cycles}}{\text{Instructions retired}}
\]

IPC is its inverse in a simplified interpretation:

\[
IPC=\frac{\text{Instructions retired}}{\text{CPU cycles}}
\]

A higher IPC can indicate better instruction throughput, but IPC alone does not tell you whether a program is faster.

Different instructions have different costs, and programs perform different amounts of work.

## 16.7 Amdahl's Law

Amdahl's Law describes the limits of speeding up part of a program.

If a fraction \(p\) of a program can be accelerated by a factor \(s\), the overall speedup is:

\[
S=\frac{1}{(1-p)+\frac{p}{s}}
\]

For example, if 90% of a program is accelerated by a factor of 10:

\[
S=\frac{1}{0.1+\frac{0.9}{10}}
\]

\[
S\approx5.26
\]

Even though the accelerated portion became ten times faster, the whole program improved by only about 5.26 times.

This concept becomes critical when deciding which parts of a program are worth optimizing or moving to a GPU.

## 16.8 Performance is an empirical property

Architecture gives you hypotheses.

Benchmarking tests those hypotheses.

Do not optimize based only on intuition.

### Exercises

1. Design a repeatable benchmark.
2. Explain why repeated measurements matter.
3. Explain dead-code elimination.
4. Calculate CPI and IPC from sample measurements.
5. Explain why IPC alone is not a complete performance metric.
6. Apply Amdahl's Law to a hypothetical GPU acceleration.
7. Identify possible causes of benchmark noise.
8. Explain why assembly inspection and measurement should complement each other.

---

# 17. Practical Laboratory Exercises

These labs are designed to build intuition through direct experimentation.

Do not ask an AI to implement them. Use the Socratic tutor prompt from the master curriculum.

## Lab A: Assembly Explorer

**Goal:** Understand how C++ becomes machine instructions.

Create a program containing:

- Integer addition.
- Integer multiplication.
- Conditional branches.
- Loops.
- Array access.
- Function calls.
- Struct field access.

For each operation:

1. Compile without optimization.
2. Inspect the assembly.
3. Compile with optimization.
4. Inspect the assembly again.
5. Identify the changed instructions.
6. Explain why the compiler might have transformed the code.

**Deliverable:** A document containing source snippets, assembly excerpts, and your explanations.

## Lab B: Cache Locality Experiment

**Goal:** Observe how memory access patterns influence performance.

Create a large two-dimensional array.

Compare:

- Row-major traversal.
- Column-major traversal.

Control the following:

- Data type.
- Array dimensions.
- Compiler optimization.
- Number of iterations.
- Benchmark methodology.

Record:

- Runtime.
- Input size.
- CPU model.
- Compiler version.
- Compilation flags.

**Questions:**

- Which traversal is faster?
- How large is the difference?
- Does the difference change with input size?
- What role might cache capacity play?
- How does spatial locality explain the behavior?

**Deliverable:** A benchmark report with measurements and an architectural explanation.

## Lab C: Branch Prediction Experiment

**Goal:** Investigate how branch predictability affects performance.

Create a program that processes an array of values.

Compare conditions with:

- Predictable outcomes.
- Randomized outcomes.
- Different data distributions.

Measure runtime and, where possible, branch misses.

Do not assume that removing a branch will improve performance.

**Deliverable:** A report explaining the relationship between branch behavior and observed performance.

## Lab D: SIMD Investigation

**Goal:** Understand how compilers use vector instructions.

Write a program that performs element-wise arithmetic on arrays.

Compare:

- A straightforward scalar implementation.
- Compiler-generated optimized code.
- An explicitly vectorized implementation, if your CPU supports the relevant instruction set.

Inspect the assembly and measure the results.

**Questions:**

- Did the compiler auto-vectorize?
- Which instructions were generated?
- How many elements can one vector instruction process?
- What happens when the array is too small?
- What effect does memory bandwidth have?

**Deliverable:** A comparison report with assembly excerpts.

## Lab E: Memory and TLB Experiment

**Goal:** Investigate how memory layout and address translation affect access patterns.

Create a large memory region and compare access patterns that vary:

- Sequential access.
- Strided access.
- Random access.
- Working-set size.

Where possible, use hardware counters to investigate cache and TLB behavior.

**Deliverable:** A report relating access patterns to observed performance.

---

# 18. Major Coding Assessment: Build a CPU Performance Laboratory

This is the primary programming assessment for Phase 3.

You will build a small C++ application that lets you compare different implementations of computational workloads.

## Requirements

Your program should support at least three workloads:

1. Vector addition.
2. Matrix traversal.
3. Reduction or summation.

For each workload, support multiple implementations where appropriate.

Examples include:

- Different memory access orders.
- Different loop structures.
- Different data types.
- Different input sizes.
- Different execution strategies.

The application should:

- Accept workload configuration.
- Generate or load test data.
- Validate results.
- Execute repeated benchmarks.
- Report measurements.
- Record the compiler and CPU environment.
- Produce machine-readable output.

## Required engineering properties

Your implementation should be designed so that:

- Correctness can be checked independently of performance.
- Each implementation performs equivalent work.
- Benchmark setup is not accidentally included in the measured region.
- Results are not optimized away.
- Input sizes are large enough to expose relevant hardware behavior.
- Measurement noise is considered.
- The program can be reproduced on another machine.

## Technical defense

You must be able to explain:

1. Why the implementations perform equivalent work.
2. Which operations are likely compute-bound.
3. Which operations are likely memory-bound.
4. What memory access patterns each implementation produces.
5. How compiler optimization changes the generated code.
6. Whether the processor uses vector instructions.
7. Whether cache locality explains the results.
8. What additional measurements would help explain unexpected behavior.

## Deliverables

- Source code.
- Automated correctness tests.
- Benchmark output.
- Assembly excerpts.
- A written performance report.
- A short technical defense.

Do not optimize blindly. First establish correctness, then establish a baseline, then form a hypothesis, then test it.

---

# 19. Phase 3 Quiz Bank

Use these questions to check your understanding before asking the tutor to generate additional questions.

## A. Computer Fundamentals

1. What are the fundamental responsibilities of a computer?
2. What is the stored-program concept?
3. What is the difference between a CPU and a memory system?
4. Why does a CPU need registers?
5. What are the main responsibilities of control logic?
6. What is a datapath?

## B. ISA and Assembly

7. What is an instruction set architecture?
8. What is the difference between ISA and microarchitecture?
9. Why can two CPUs execute the same binary but have different performance?
10. What is a machine instruction?
11. What is an addressing mode?
12. What does the instruction pointer represent?
13. What is the difference between an architectural register and a physical register?
14. Why does a compiler need to know the target ISA?

## C. Function Execution

15. What is a calling convention?
16. What is a stack frame?
17. What is the role of the stack pointer?
18. Why might a compiler omit a frame pointer?
19. How are function arguments passed?
20. What is inlining?
21. Why can inlining increase code size?

## D. Processor Execution

22. What is an instruction pipeline?
23. Why does pipelining improve throughput?
24. What is a data hazard?
25. What is a control hazard?
26. What is a structural hazard?
27. What is instruction-level parallelism?
28. What is out-of-order execution?
29. What problem does register renaming solve?
30. What are RAW, WAR, and WAW dependencies?

## E. Prediction and Speculation

31. What is branch prediction?
32. What is speculative execution?
33. What happens after a branch misprediction?
34. Why can unpredictable branches hurt performance?
35. Why is branchless code not automatically faster?
36. What is the difference between architectural and microarchitectural state?

## F. Memory Hierarchy

37. Why does the memory hierarchy exist?
38. What is temporal locality?
39. What is spatial locality?
40. What is a cache line?
41. What is a cache miss?
42. What are compulsory, capacity, and conflict misses?
43. What is cache associativity?
44. What is a cache replacement policy?
45. What is the difference between write-through and write-back?
46. Why can two algorithms with identical complexity have different runtimes?

## G. Cache Coherence and Virtual Memory

47. Why is cache coherence necessary?
48. What does a cache invalidation accomplish?
49. What is false sharing?
50. Why can false sharing occur without a data race?
51. What is virtual memory?
52. What is the difference between virtual and physical addresses?
53. What is a page table?
54. What does the MMU do?
55. What is a TLB?
56. What is a page fault?
57. What is NUMA?

## H. Vectorization and Performance

58. What is SIMD?
59. What is auto-vectorization?
60. Why might a compiler not vectorize a loop?
61. What is the difference between SIMD and SIMT?
62. What is memory latency?
63. What is memory bandwidth?
64. What is arithmetic intensity?
65. What is the Roofline model?
66. What does it mean for a program to be memory-bound?
67. What does it mean for a program to be compute-bound?
68. What is CPI?
69. What is IPC?
70. Why is IPC alone insufficient to determine performance?
71. What is Amdahl's Law?
72. Why is benchmarking difficult?

---

# 20. AI Tutor Session Prompts

Use the master Socratic Tutor Agent Prompt from the central curriculum. It contains the permanent rules: no answers, no implementation, no code modifications, and one guiding question at a time.

The following prompts are phase-specific.

## Start a lesson

```text
Tutor me through Phase 3, Lesson [NUMBER].

Assume I have read the material but do not fully understand it.
Start from first principles and test my reasoning.

Ask one guiding question at a time.
Never explain the answer or solve the problem for me.
```

## Generate a quiz

```text
Administer a new Phase 3 quiz on [TOPIC].

Generate 20 original questions.
Include conceptual reasoning, assembly interpretation,
hardware scenarios, and performance analysis.

Ask one question at a time.
Do not reveal answers, hints that give away answers, or an answer key.
Use my responses to determine the next question.
```

## Start a coding assessment

```text
Administer the Phase 3 CPU Performance Laboratory assessment.

Present the requirements and constraints only.
Do not provide code, pseudocode, algorithms, or implementation steps.

Make me explain my design and measurement methodology.
Then guide me through the assessment exclusively using questions.

Never implement, edit, debug, or fix anything for me.
```

## Review benchmark results

```text
Help me reason about my Phase 3 benchmark results.

Do not interpret the results for me or tell me what they mean.
Ask questions that help me form hypotheses, identify confounding
variables, inspect the generated assembly, and reason about
cache behavior, instruction throughput, and memory bandwidth.

Do not give me conclusions.
```

## Assembly examination

```text
Give me an assembly-reading examination based on Phase 3.

Present small snippets of x86-64 assembly and ask me to explain
their behavior, register usage, memory accesses, and control flow.

Increase difficulty as I improve.
Never explain or reveal the answer.
```

## First-principles remediation

```text
Test my understanding of computer architecture from first principles.

Identify gaps in my understanding of registers, execution,
pipelines, caches, virtual memory, and performance.

Use questions that expose incorrect assumptions.
Do not provide explanations or answers.
```

---

# 21. Phase 3 Final Examination

**Recommended duration:** 4–5 hours.

| Section | Weight |
|---|---:|
| Computer fundamentals and ISA | 10% |
| Registers and assembly | 10% |
| Function execution and calling conventions | 10% |
| Pipelines and instruction execution | 15% |
| Out-of-order execution and prediction | 10% |
| Cache hierarchy and coherence | 15% |
| Virtual memory | 10% |
| SIMD and performance models | 10% |
| Performance laboratory | 10% |

For each conceptual question, your answer must explain:

1. What problem the hardware component solves.
2. How it works conceptually.
3. What limitations it introduces.
4. How it affects software.
5. How you would measure its impact.

## Oral technical defense

You should be able to explain:

- How a C++ function becomes machine instructions.
- How the CPU executes those instructions.
- Why dependencies restrict parallel execution.
- How caches reduce memory access costs.
- Why memory access patterns matter.
- How virtual addresses become physical memory accesses.
- How vector instructions process multiple values.
- Why an apparently simple benchmark can produce misleading results.

---

# 22. Mastery Checklist

## Fundamentals

- [ ] I understand the major components of a computer.
- [ ] I understand the stored-program concept.
- [ ] I can explain the basic instruction execution cycle.
- [ ] I understand the difference between ISA and microarchitecture.

## Registers and execution

- [ ] I understand general-purpose and special-purpose registers.
- [ ] I can read basic x86-64 assembly.
- [ ] I understand function calling conventions.
- [ ] I understand stack frames.
- [ ] I understand pipelines and hazards.
- [ ] I understand instruction-level parallelism.
- [ ] I understand out-of-order execution.
- [ ] I understand branch prediction.

## Memory

- [ ] I understand the memory hierarchy.
- [ ] I understand temporal and spatial locality.
- [ ] I understand cache lines.
- [ ] I understand cache associativity.
- [ ] I understand cache coherence.
- [ ] I understand false sharing.
- [ ] I understand virtual memory.
- [ ] I understand page tables and TLBs.
- [ ] I understand NUMA conceptually.

## Performance

- [ ] I understand SIMD.
- [ ] I understand auto-vectorization.
- [ ] I understand memory bandwidth and latency.
- [ ] I can calculate arithmetic intensity.
- [ ] I understand the Roofline model.
- [ ] I understand CPI and IPC.
- [ ] I can design a repeatable benchmark.
- [ ] I can use assembly and measurements to investigate performance.

## Independent reasoning

- [ ] I can explain why an access pattern is cache-friendly or cache-unfriendly.
- [ ] I can explain why a program might be memory-bound.
- [ ] I can explain why a dependency chain restricts instruction-level parallelism.
- [ ] I can defend a performance hypothesis using evidence.
- [ ] I can distinguish an observed result from an explanation of that result.

---

# 23. How This Phase Connects to GPU Programming

This phase establishes the foundation for understanding GPUs.

| CPU architecture concept | GPU connection |
|---|---|
| Instruction set architecture | GPU instruction sets and intermediate representations |
| Registers | Per-thread GPU registers |
| Instruction pipeline | GPU execution pipelines |
| Instruction-level parallelism | Parallel instruction scheduling |
| SIMD | Vectorized execution concepts |
| Branch prediction | Divergence and execution-path management |
| Cache hierarchy | GPU cache hierarchy |
| Memory bandwidth | GPU global-memory bandwidth |
| Memory latency | GPU latency hiding |
| Cache coherence | GPU memory consistency and synchronization models |
| Virtual memory | GPU virtual addressing and unified memory concepts |
| False sharing | Related concerns in memory access and contention |
| Roofline model | GPU kernel performance analysis |

Some of these concepts transfer directly; others have different implementations on GPUs.

For example, CPU branch prediction and GPU thread divergence are not the same mechanism.

A CPU often attempts to predict a single control-flow path. A GPU commonly executes groups of threads together, and divergent control flow can require the hardware to handle multiple paths.

You will study that distinction in the GPU architecture phase.

---

# 24. Recommended Resources

## Core computer architecture

1. **Computer Organization and Design — Patterson and Hennessy**
   - A foundational treatment of processor architecture.
   - Particularly useful for instruction sets, datapaths, pipelines, and memory systems.

2. **Computer Architecture: A Quantitative Approach — Hennessy and Patterson**
   - A deeper, more quantitative treatment.
   - Use it as a reference when you want to understand architectural trade-offs.

3. **Nand2Tetris**
   - https://www.nand2tetris.org/
   - An excellent first-principles course that builds a computer from elementary logic concepts.

## Assembly and hardware

4. **Compiler Explorer**
   - https://godbolt.org/
   - Inspect compiler output and compare architectures.

5. **Intel Software Developer Manuals**
   - https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html
   - Reference documentation for x86 architecture and instruction behavior.

6. **AMD Architecture and Software Optimization Documentation**
   - https://www.amd.com/en/developer.html
   - Useful for understanding AMD processor architecture and optimization.

7. **Agner Fog's Optimization Resources**
   - https://www.agner.org/optimize/
   - Detailed resources on instruction timing, microarchitecture, and optimization.

## Performance analysis

8. **Linux perf documentation**
   - https://perf.wiki.kernel.org/
   - Learn to inspect hardware counters and system performance.

9. **Performance Ninja**
   - https://github.com/dendibakh/perf-ninja
   - Practical performance engineering exercises.

10. **The Roofline Model**
    - https://crd.lbl.gov/divisions/amcr/computer-science-amcr/par/research/roofline/
    - Background on the Roofline performance model.

---

# Final Objective

At the end of this phase, you should no longer think of a program as a sequence of source-code statements.

You should be able to reason about:

- Which instructions execute.
- Which registers hold the data.
- Which dependencies restrict execution.
- Which memory locations are accessed.
- Whether those accesses benefit from locality.
- Whether the processor is limited by computation or data movement.
- How to test your performance hypotheses.

This is the foundation of performance engineering.

**The next phase will be Operating Systems.** You will study processes, threads, scheduling, virtual memory, system calls, synchronization, and how software interacts with hardware through the operating system.
