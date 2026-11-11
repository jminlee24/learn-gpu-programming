# Phase 14: GPU Compilers — From Source Code to Executable GPU Programs

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 14 of 16  
**Estimated duration:** 20–28 weeks  
**Study commitment:** 10–12 hours per week  
**Primary implementation language:** Rust  
**Supporting languages:** WGSL, SPIR-V, LLVM IR, C++20  
**Development platform:** Fedora Linux  
**GPU:** AMD Radeon RX 9070, 16 GB  
**Prerequisites:** Phases 0–13

---

# 1. Mission

A GPU compiler transforms a program written in a source language into a representation that can execute efficiently on a GPU.

This sounds straightforward:

\[
\text{Source Code}\rightarrow\text{GPU Instructions}
\]

But the actual process is much more involved.

A compiler must determine:

- Whether the source program is syntactically valid.
- Whether its operations are type-correct.
- Which variables refer to which declarations.
- How control flow is structured.
- Which values depend on other values.
- Which memory operations may alias.
- Which transformations preserve program behavior.
- Which operations can execute in parallel.
- How source-level memory maps onto GPU address spaces.
- How to represent synchronization.
- How to produce valid target code.
- Whether optimizations actually improve performance.

For GPU programs, the compiler must also account for:

- Thousands of concurrent invocations.
- Workgroups and subgroups.
- Divergent control flow.
- Memory coalescing.
- Register pressure.
- Workgroup-local memory.
- Barrier semantics.
- Hardware instruction capabilities.
- Numerical precision.

The result is an unusually rich combination of programming languages, mathematics, computer architecture, and systems engineering.

## 1.1 The compiler you will build

Your capstone compiler will process a restricted GPU compute language.

Its architecture will resemble:

```text
            Source Program
                   |
                   v
                 Lexer
                   |
                   v
                 Parser
                   |
                   v
          Abstract Syntax Tree
                   |
                   v
          Semantic Analysis
                   |
                   v
           Typed High-Level IR
                   |
                   v
          Control-Flow Analysis
                   |
                   v
                 SSA IR
                   |
                   v
          Optimization Passes
                   |
                   v
          GPU-Specific Lowering
                   |
                   v
              Target Backend
                /       \
               v         v
           WGSL        SPIR-V
               \         /
                v       v
          Validation / Driver
                   |
                   v
             AMD Radeon GPU
```

**Recommended implementation strategy:** Build the compiler front end, IR, and optimization passes yourself in Rust. Start with WGSL output, which provides a practical route to execution through wgpu. Add direct SPIR-V generation as an advanced backend only after the compiler's semantics and tests are stable.

WGSL output is a legitimate compiler backend, even though a separate compiler still translates WGSL into lower-level code.

## 1.2 Learning objectives

By completion, you should be able to:

1. Explain the major stages of a compiler.
2. Design a restricted programming language.
3. Implement lexical analysis.
4. Implement a parser.
5. Construct an abstract syntax tree.
6. Implement name resolution and type checking.
7. Design a typed intermediate representation.
8. Build control-flow graphs.
9. Compute dominators.
10. Convert suitable IR into static single assignment form.
11. Implement data-flow analyses.
12. Explain alias analysis.
13. Implement constant folding and propagation.
14. Implement dead-code elimination.
15. Understand common subexpression elimination.
16. Analyze loop transformations.
17. Explain GPU divergence and uniformity analysis.
18. Model GPU memory address spaces.
19. Preserve synchronization semantics.
20. Understand SPIR-V's execution model.
21. Inspect LLVM IR and AMD compiler output.
22. Explain register allocation and instruction scheduling.
23. Validate compiler transformations.
24. Build differential and property-based tests.
25. Benchmark compiler-generated GPU kernels.
26. Design and defend a small GPU compiler.

---

# 2. Scope and Engineering Constraints

A full production GPU compiler is a multi-year engineering project.

This phase deliberately targets a smaller, well-defined compiler.

## 2.1 Supported language subset

Your language should initially support:

- Scalar integers and floating-point values.
- Arithmetic expressions.
- Local variables.
- Immutable values.
- Conditional expressions or statements.
- Bounded loops.
- Buffer reads and writes.
- Kernel parameters.
- Built-in invocation identifiers.
- Simple functions.

Later extensions may include:

- Workgroup-local memory.
- Barriers.
- Atomic operations.
- Vector types.
- Multidimensional indexing.
- Specialization constants.

## 2.2 Explicitly excluded initially

Do not begin by supporting:

- Arbitrary recursion.
- Heap allocation.
- Dynamic dispatch.
- Exceptions.
- General-purpose pointers.
- Full Rust syntax.
- A complete standard library.
- Arbitrary function pointers.
- Every Vulkan shader capability.

These features introduce complexity that would obscure the core compiler concepts.

## 2.3 Correctness before optimization

A compiler that emits slow but correct GPU programs is a useful foundation.

A compiler that emits fast but incorrect programs is not.

For every optimization, establish:

1. The transformation's preconditions.
2. The behavior it is intended to preserve.
3. The cases in which it is invalid.
4. The tests that exercise those cases.
5. The measured performance effect.

---

# 3. Study Schedule

| Weeks | Topic | Deliverable |
|---|---|---|
| 1–2 | Compiler architecture and language design | Language specification |
| 3–4 | Lexing and parsing | Lexer, parser, AST |
| 5–6 | Semantic analysis and typing | Typed AST |
| 7–8 | Intermediate representations and CFGs | Typed IR and CFG |
| 9–10 | Dominance, SSA, and data flow | SSA construction and analyses |
| 11–13 | General optimization passes | Optimization pipeline |
| 14–16 | GPU execution and memory semantics | GPU-aware IR |
| 17–19 | WGSL and SPIR-V backends | Executable kernels |
| 20–22 | Compiler testing and performance | Validation framework |
| 23–28 | Capstone and technical defense | Complete GPU compiler |

The later weeks are flexible. If direct SPIR-V generation proves too costly, prioritize compiler correctness, optimization, and a robust WGSL backend.

---

# 4. Module One: What a Compiler Actually Does

## 4.1 Compiler stages

A traditional compiler can be divided into three broad components.

### Front end

The front end understands the source language.

Responsibilities include:

- Tokenization.
- Parsing.
- Name resolution.
- Type checking.
- Semantic validation.

### Middle end

The middle end analyzes and transforms an intermediate representation.

Responsibilities include:

- Control-flow analysis.
- Data-flow analysis.
- Constant propagation.
- Dead-code elimination.
- Loop optimization.
- Other semantics-preserving transformations.

### Back end

The back end translates an IR into a target representation.

Responsibilities may include:

- Instruction selection.
- Register allocation.
- Instruction scheduling.
- Target-specific lowering.
- Machine-code emission.

For your compiler, a WGSL backend can delegate some of these responsibilities to a downstream compiler.

## 4.2 Why compilers use intermediate representations

Imagine optimizing two source languages that both support addition, multiplication, and loops.

Without an IR, each language requires its own optimization logic.

With a common IR:

\[
\text{Language A}\rightarrow IR
\]

\[
\text{Language B}\rightarrow IR
\]

Both can share the same optimization passes.

The IR separates source-language details from optimization and target-specific code generation.

## 4.3 High-level versus low-level IR

A high-level IR may preserve concepts such as:

- Loops.
- Arrays.
- Structured conditionals.
- Vector operations.

A lower-level IR may represent:

- Basic blocks.
- Branches.
- Loads.
- Stores.
- Arithmetic instructions.
- Explicit control-flow edges.

Lowering converts higher-level operations into more explicit lower-level forms.

## 4.4 Compilation is not interpretation

A compiler translates a program into another representation.

An interpreter executes the program's operations directly or through an execution engine.

The distinction can blur when systems use:

- Bytecode.
- Just-in-time compilation.
- Virtual machines.
- Multi-stage compilation.

## 4.5 Ahead-of-time and just-in-time compilation

**Ahead-of-time compilation** performs translation before execution.

**Just-in-time compilation** performs some compilation during execution.

GPU systems may compile shader source or intermediate representations at pipeline creation time.

This means application startup and pipeline creation can involve substantial compilation work.

## 4.6 Worked example: constant folding

Consider:

```text
x = 4 * 8
y = x + 3
```

The compiler can determine:

\[
x=32
\]

and:

\[
y=35
\]

without executing the program.

It may replace the calculations with constants.

However, floating-point transformations require more care because rounding, NaNs, signed zero, and evaluation order can affect results.

## Lab 1: Compiler Pipeline Explorer

Investigate how the same simple computation is represented in:

- Rust source.
- LLVM IR.
- WGSL.
- SPIR-V.
- GPU machine code, if available through supported tooling.

Identify where:

- Types are represented.
- Control flow appears.
- Memory operations appear.
- Optimization occurs.
- Target-specific instructions are introduced.

## Knowledge checks

1. What is a compiler front end?
2. What is an IR?
3. Why are multiple IR levels useful?
4. What is lowering?
5. How does a compiler differ from an interpreter?
6. What is JIT compilation?
7. Why might pipeline creation be expensive?
8. Why can constant folding change floating-point behavior?

---

# 5. Module Two: Lexical Analysis

A compiler's first job is to turn source text into meaningful units.

## 5.1 Tokens

Consider:

```text
let x = 12 + 7;
```

A lexer may produce:

```text
LET
IDENTIFIER("x")
EQUALS
INTEGER(12)
PLUS
INTEGER(7)
SEMICOLON
```

Each token identifies a syntactic category.

## 5.2 Lexing versus parsing

Lexing recognizes individual tokens.

Parsing determines how tokens form valid language constructs.

For example, a lexer recognizes `+` as an operator.

A parser determines which expressions are its operands.

## 5.3 Token metadata

Useful token metadata includes:

- Token kind.
- Source offset.
- Line number.
- Column number.
- Source length.

This information supports precise diagnostics.

## 5.4 Lexer design

A simple lexer can scan source text from left to right.

At each position, it identifies the next token based on the current character and subsequent characters.

It must handle:

- Whitespace.
- Identifiers.
- Numbers.
- Operators.
- Punctuation.
- Comments.
- Invalid characters.

## 5.5 Identifiers

An identifier is a name such as:

```text
position
velocity
global_id
```

The language specification must define valid identifier characters.

## 5.6 Keywords

Keywords are reserved words with special grammatical meaning.

Examples:

```text
fn
let
if
else
return
```

## 5.7 Numeric literals

Numeric literals introduce questions about:

- Integer bases.
- Signedness.
- Decimal points.
- Exponents.
- Overflow.
- Type suffixes.

A lexer can preserve literal text while later semantic analysis determines its type and validity.

## 5.8 Error reporting

Consider:

```text
let x = 12 @ 7;
```

If `@` is not a valid token, the lexer should report its source position.

A useful diagnostic identifies the offending character and location rather than merely reporting that compilation failed.

## Lab 2: Rust Lexer

Implement a lexer for your restricted GPU language.

Required tokens:

- Identifiers.
- Integer literals.
- Floating-point literals.
- Arithmetic operators.
- Comparison operators.
- Assignment.
- Parentheses.
- Braces.
- Semicolons.
- Keywords.

Required tests:

- Empty input.
- Whitespace.
- Comments.
- Long identifiers.
- Invalid characters.
- Malformed numbers.
- Source-location tracking.

## Exercises

1. Define tokenization.
2. Why is source-location tracking important?
3. How do keywords differ from identifiers?
4. What is the relationship between lexing and parsing?
5. Why might literal type checking occur after lexing?
6. How should a lexer handle invalid input?

---

# 6. Module Three: Parsing and Abstract Syntax Trees

Parsing transforms a token sequence into a structured representation.

## 6.1 Grammars

A grammar defines valid language constructs.

A simplified expression grammar might be:

```text
expression     := addition
addition       := multiplication (("+" | "-") multiplication)*
multiplication := primary (("*" | "/") primary)*
primary        := NUMBER | IDENTIFIER | "(" expression ")"
```

This grammar establishes multiplication precedence over addition.

## 6.2 Operator precedence

Consider:

\[
2+3\times4
\]

The intended interpretation is:

\[
2+(3\times4)=14
\]

not:

\[
(2+3)\times4=20
\]

A parser must represent precedence correctly.

## 6.3 Associativity

Consider:

\[
10-4-2
\]

Subtraction is normally left-associative:

\[
(10-4)-2=4
\]

A parser must distinguish precedence from associativity.

## 6.4 Abstract syntax trees

An AST represents program structure without preserving every syntactic detail.

For:

```text
2 + 3 * 4
```

the AST might be:

```text
       Add
      /   \
     2    Multiply
          /      \
         3        4
```

## 6.5 AST representation in Rust

A simplified expression representation:

```rust
enum Expr {
    Integer(i64),
    Variable(String),
    Binary {
        op: BinaryOp,
        left: Box<Expr>,
        right: Box<Expr>,
    },
}

enum BinaryOp {
    Add,
    Subtract,
    Multiply,
    Divide,
}
```

This example illustrates recursive data structures.

`Box` provides indirection so the recursive enum has a known size.

A production parser would also attach source spans and use types appropriate to the language's numeric semantics.

## 6.6 Recursive-descent parsing

A recursive-descent parser uses functions corresponding to grammar rules.

It is suitable for a small language with a carefully designed grammar.

## 6.7 Pratt parsing

Pratt parsing is another useful technique for expression parsing.

It handles operator precedence through binding powers.

This can simplify adding operators.

## 6.8 Error recovery

A parser should not necessarily stop at the first syntax error.

It may recover at boundaries such as semicolons or closing braces.

However, error recovery must avoid generating misleading cascades of diagnostics.

## Lab 3: Parser and AST

Implement:

- Expression parsing.
- Operator precedence.
- Associativity.
- Variable declarations.
- Assignments.
- Conditional statements.
- Bounded loops.
- Function declarations.

Add AST printing for debugging.

Test malformed input and diagnostic quality.

## Exercises

1. What is a grammar?
2. What is operator precedence?
3. What is associativity?
4. What is an AST?
5. Why are recursive enums boxed?
6. Compare recursive-descent and Pratt parsing.
7. What is parser error recovery?
8. Why should AST nodes retain source spans?

---

# 7. Module Four: Semantic Analysis and Type Systems

A syntactically valid program may still be meaningless.

Consider:

```text
let x = 4;
let y = unknown_variable + x;
```

The parser may accept this program.

Semantic analysis should reject the unresolved variable.

## 7.1 Name resolution

Name resolution determines which declaration each variable reference denotes.

## 7.2 Symbol tables

A symbol table maps names to declarations and associated information.

For example:

| Name | Type | Scope |
|---|---|---|
| `x` | `i32` | Function |
| `y` | `f32` | Function |
| `input` | Buffer of `f32` | Kernel |

## 7.3 Lexical scope

A declaration's scope determines where its name is visible.

Nested scopes may allow shadowing.

Your language specification must define whether shadowing is permitted.

## 7.4 Type checking

Type checking determines whether operations are applied to compatible values.

For example:

\[
\texttt{i32}+\texttt{i32}\rightarrow\texttt{i32}
\]

may be valid.

But:

\[
\texttt{i32}+\texttt{buffer<f32>}
\]

should generally be invalid without a defined conversion or indexing operation.

## 7.5 Explicit versus implicit conversions

Implicit conversions make programs concise but can introduce ambiguity.

For an educational compiler, prefer a small number of explicit conversion rules.

## 7.6 Typed AST

A typed AST associates each expression with its resolved type.

For example:

```text
BinaryAdd: i32
├── IntegerLiteral(4): i32
└── Variable(x): i32
```

## 7.7 GPU-specific type constraints

GPU kernels introduce additional restrictions.

For example:

- Buffer indices may require specific integer types.
- Atomic operations require supported atomic types.
- Workgroup-local values have particular storage rules.
- Some scalar widths require optional device features.

## 7.8 Diagnostics

A good compiler error should identify:

- The invalid construct.
- The source location.
- The expected type.
- The actual type.

## Lab 4: Semantic Analyzer

Implement:

- Scoped symbol tables.
- Name resolution.
- Type checking.
- Function signature validation.
- Assignment validation.
- Return-type validation.
- GPU buffer-access validation.

Create tests for both valid and invalid programs.

## Exercises

1. Why is parsing insufficient to validate a program?
2. What is name resolution?
3. What is a symbol table?
4. What is lexical scope?
5. What is type checking?
6. Why are implicit conversions risky?
7. What is a typed AST?
8. Why are GPU target capabilities relevant to typing?

---

# 8. Module Five: Intermediate Representation Design

Your compiler's IR is its central engineering interface.

A poorly designed IR makes optimization and code generation difficult.

## 8.1 What an IR must represent

Your IR should represent:

- Values.
- Types.
- Arithmetic.
- Memory operations.
- Control flow.
- Function calls.
- Kernel entry points.
- GPU built-ins.
- Address spaces.
- Synchronization operations.

## 8.2 Three-address code

A simple IR may express each instruction using a small number of operands.

For example:

```text
t0 = a + b
t1 = t0 * c
t2 = t1 - d
```

This makes data dependencies explicit.

## 8.3 Basic blocks

A basic block is a sequence of instructions with a single entry and a terminator that determines control-flow successors.

For example:

```text
block0:
    t0 = a > 0
    branch t0, block1, block2

block1:
    t1 = a * 2
    jump block3

block2:
    t2 = a + 2
    jump block3
```

## 8.4 Control-flow graphs

A CFG represents basic blocks as vertices and control-flow transfers as edges.

For the example:

```text
        block0
        /    \
       v      v
    block1  block2
       \      /
        v    v
        block3
```

## 8.5 Why CFGs matter

Many compiler analyses require understanding which blocks may execute before others.

Examples include:

- Liveness.
- Reaching definitions.
- Dominance.
- Loop detection.
- Dead-code elimination.

## 8.6 IR design trade-offs

A high-level IR preserves useful structure but may be harder to lower directly.

A low-level IR is explicit but may lose information useful for optimization.

You may benefit from two IR levels:

1. Typed high-level IR.
2. Lower-level CFG-based IR.

## 8.7 Value identifiers

Rather than storing variable names throughout the IR, assign unique identifiers to values and instructions.

This reduces ambiguity and simplifies analysis.

## 8.8 Verification

Implement an IR verifier that checks:

- Valid block identifiers.
- Valid instruction operands.
- Correct types.
- Valid terminators.
- Defined values.
- Valid control-flow edges.

## Lab 5: Typed IR and CFG

Lower your typed AST into a CFG-based IR.

Support:

- Arithmetic.
- Comparisons.
- Variables.
- Branches.
- Loops.
- Buffer operations.

Implement:

- IR printing.
- CFG visualization.
- Structural verification.

## Exercises

1. What is three-address code?
2. What is a basic block?
3. What is a CFG?
4. Why are terminators important?
5. Why use value identifiers?
6. What is lowering?
7. Why might multiple IR levels be useful?
8. What invariants should an IR verifier enforce?

---

# 9. Module Six: Dominance and Static Single Assignment

SSA is one of the most important concepts in modern compiler design.

## 9.1 Dominance

A block \(A\) dominates block \(B\) if every path from the function entry to \(B\) passes through \(A\).

The entry block dominates every reachable block.

## 9.2 Why dominance matters

Dominance helps determine whether a value definition is guaranteed to be available at a use.

It is central to:

- SSA construction.
- Loop analysis.
- Code motion.
- Optimization legality.

## 9.3 Static single assignment

In SSA form, each value is assigned exactly once.

Consider:

```text
x = 4
x = x + 1
```

SSA might represent this as:

```text
x0 = 4
x1 = x0 + 1
```

Each name has one definition.

## 9.4 Branch merging

Consider:

```text
if condition:
    x = 10
else:
    x = 20

y = x + 1
```

The value of `x` depends on the path taken.

In conventional SSA notation:

```text
then:
    x1 = 10

else:
    x2 = 20

merge:
    x3 = phi(x1, x2)
    y0 = x3 + 1
```

The phi operation selects the value associated with the incoming control-flow edge.

It is not an ordinary runtime function call.

## 9.5 Loops and SSA

Loops require values to flow from a previous iteration back to the loop header.

A loop-carried variable may use a phi node combining:

- The initial value.
- The value produced by the previous iteration.

## 9.6 Dominance frontiers

Dominance frontiers help determine where phi nodes may be needed.

A block lies in another block's dominance frontier when control-flow paths merge in a way that affects dominance relationships.

## 9.7 SSA benefits

SSA makes it easier to reason about:

- Definitions and uses.
- Constant propagation.
- Dead values.
- Value equivalence.
- Data dependencies.

## 9.8 SSA limitations

SSA simplifies value definitions but does not automatically solve memory aliasing.

Two memory loads may read different values even when their address operands are unchanged, because another operation may have modified the memory.

## Lab 6: SSA Construction

Implement:

- Dominator calculation.
- Dominance-frontier analysis.
- SSA value renaming.
- Phi insertion.
- SSA verification.

Test:

- Straight-line code.
- Diamond CFGs.
- Nested conditionals.
- Simple loops.
- Nested loops.

## Exercises

1. Define dominance.
2. Why is dominance useful?
3. What is SSA?
4. What is a phi node?
5. Why do branches require value merging?
6. How are loop-carried values represented?
7. What is a dominance frontier?
8. Why does SSA not eliminate memory aliasing problems?

---

# 10. Module Seven: Data-Flow Analysis

Data-flow analysis determines properties of programs by propagating information through control flow.

## 10.1 Forward and backward analysis

A forward analysis propagates information from predecessors to successors.

A backward analysis propagates information from successors to predecessors.

## 10.2 Reaching definitions

Reaching-definitions analysis determines which assignments may reach a program point without being overwritten.

## 10.3 Liveness

A value is live at a program point if its current value may be used along a future execution path before being redefined.

Liveness is important for:

- Register allocation.
- Dead-code elimination.
- Temporary storage analysis.

## 10.4 Liveness equations

For a basic block \(B\), a standard backward liveness formulation is:

\[
LiveOut(B)=\bigcup_{S\in Succ(B)}LiveIn(S)
\]

\[
LiveIn(B)=Use(B)\cup(LiveOut(B)-Def(B))
\]

where:

- \(Use(B)\) contains values used before being defined in the block.
- \(Def(B)\) contains values defined in the block.
- \(Succ(B)\) contains successor blocks.

## 10.5 Fixed-point iteration

Data-flow equations may depend cyclically on one another.

An iterative algorithm repeatedly updates the analysis state until no values change.

This is a fixed point.

## 10.6 Worked example

Consider:

```text
a = input
b = a + 1
c = b * 2
output = c
```

Immediately before `c = b * 2`, the value `b` is live because it is about to be used.

The value `a` is not live at that point if it has no future uses.

This distinction matters when deciding whether values need to remain in registers.

## 10.7 Data-flow lattices

Compiler analyses often operate over mathematical lattices.

A lattice provides a structured way to combine information from multiple control-flow paths.

For constant propagation, a value might be classified as:

- Not yet determined.
- A particular constant.
- Not a single known constant.

The merge operation combines facts from predecessor paths.

## 10.8 Worklist algorithms

A worklist algorithm processes blocks whose analysis state may have changed.

This can avoid revisiting unaffected blocks.

## Lab 7: Data-Flow Framework

Implement a reusable analysis framework.

Required analyses:

- Liveness.
- Reaching definitions.
- Constant propagation.

Use both hand-written and randomly generated CFGs.

Verify convergence and expected analysis results.

## Exercises

1. What is data-flow analysis?
2. Distinguish forward and backward analyses.
3. What is liveness?
4. Why is liveness useful for register allocation?
5. What is a fixed point?
6. What is a lattice?
7. Why are merge operations necessary?
8. What is a worklist algorithm?

---

# 11. Module Eight: Compiler Optimizations

Compiler optimizations transform programs while preserving their required behavior.

## 11.1 Constant folding

Evaluate constant expressions during compilation.

Example:

```text
x = 6 * 7
```

becomes:

```text
x = 42
```

## 11.2 Constant propagation

Replace uses of variables whose values are known constants.

Example:

```text
a = 5
b = a + 2
```

may become:

```text
a = 5
b = 7
```

## 11.3 Dead-code elimination

Remove instructions whose results cannot affect observable program behavior.

Example:

```text
a = 10
b = a + 3
c = 7
output = c
```

If `a` and `b` have no observable effects, their calculations may be removed.

## 11.4 Side effects

Not every unused result makes an instruction removable.

A store to a buffer may have an observable effect.

An atomic operation may affect other invocations.

A barrier has synchronization semantics even though it does not produce an ordinary arithmetic result.

## 11.5 Common subexpression elimination

Consider:

```text
a = x * y
b = x * y
```

If the operands and operation have compatible semantics, the second multiplication may be redundant.

## 11.6 Algebraic simplification

Transformations such as:

\[
x+0=x
\]

are generally straightforward for integer arithmetic under an appropriate semantic model.

Floating-point algebra is more subtle.

For example, replacing:

\[
x\times0
\]

with zero may not preserve behavior for NaNs or signed zero under strict floating-point semantics.

## 11.7 Loop-invariant code motion

An expression that produces the same value on every iteration may be moved outside the loop when doing so preserves behavior.

Example:

```text
for i in 0..n:
    scale = width * height
    output[i] = input[i] * scale
```

The multiplication computing `scale` may be hoisted if it is independent of the loop and moving it is semantically valid.

## 11.8 Loop unrolling

Loop unrolling duplicates the loop body to reduce control overhead and expose instruction-level parallelism.

However, it can increase:

- Code size.
- Register pressure.
- Instruction-cache pressure.

## 11.9 Strength reduction

Strength reduction replaces expensive operations with cheaper equivalent ones where legal.

For example, an address calculation inside a loop may use incremental pointer or index updates rather than repeated multiplication.

## 11.10 Optimization pass ordering

Optimizations interact.

For example:

1. Constant propagation exposes dead branches.
2. Dead-branch elimination removes unreachable code.
3. Dead-code elimination removes unused calculations.
4. Simplification exposes additional constant expressions.

Compilers often run passes repeatedly or organize them into pipelines.

## 11.11 Correctness constraints

Optimization legality depends on:

- Integer overflow semantics.
- Floating-point semantics.
- Memory aliasing.
- Volatile operations.
- Atomic operations.
- Synchronization.
- Observable side effects.

## Lab 8: Optimization Passes

Implement at least:

1. Constant folding.
2. Constant propagation.
3. Dead-code elimination.
4. Common subexpression elimination.
5. Branch simplification.
6. Loop-invariant code motion.

Each pass must have:

- Defined preconditions.
- IR verification.
- Unit tests.
- Differential tests.
- Before/after IR output.

## Exercises

1. Distinguish folding from propagation.
2. What is dead code?
3. Why are stores not automatically removable?
4. What is common subexpression elimination?
5. Why is floating-point reassociation dangerous?
6. What is loop-invariant code motion?
7. Why can loop unrolling reduce performance?
8. What is strength reduction?
9. Why does pass ordering matter?
10. How do you test optimization correctness?

---

# 12. Module Nine: Alias Analysis and Memory Dependence

GPU compilers must reason about memory operations.

## 12.1 The aliasing problem

Consider:

```text
a = load buffer_x[i]
store buffer_y[j], 10
b = load buffer_x[i]
```

Can the second load reuse the first result?

Only if the intervening store cannot modify the memory read by the load, or if another valid semantic argument establishes equivalence.

## 12.2 May-alias and no-alias

Alias analysis may classify memory accesses as:

- Definitely nonaliasing.
- Possibly aliasing.
- Definitely referring to the same location, where such a conclusion is supported.

Conservative analysis assumes aliasing when it cannot prove otherwise.

## 12.3 Memory dependence

Memory dependence exists when the ordering of memory operations may affect program behavior.

This includes:

- Read-after-write.
- Write-after-read.
- Write-after-write.

## 12.4 Address-space information

GPU address spaces can help analysis.

For example, a workgroup-local allocation and a storage-buffer allocation occupy different logical storage classes.

However, the compiler must respect the exact memory model and pointer rules of the target.

## 12.5 Range analysis

If the compiler can prove that two accesses target nonoverlapping ranges, it may be able to reorder them.

For example:

\[
A[0..99]
\]

and:

\[
A[100..199]
\]

are nonoverlapping under the stated indexing interpretation.

## 12.6 Conservative optimization

When alias analysis is uncertain, preserving operation order may be necessary.

This can reduce performance but protect correctness.

## Lab 9: Basic Alias Analysis

Implement a restricted alias analysis for:

- Distinct buffers.
- Constant indices.
- Simple affine indices.
- Nonoverlapping known ranges.

Document cases where the analysis must conservatively return “may alias.”

## Exercises

1. What is aliasing?
2. What is memory dependence?
3. Why can aliasing prevent load elimination?
4. What is conservative alias analysis?
5. How can address spaces help?
6. What is range analysis?
7. Why is general alias analysis difficult?
8. How does alias analysis affect GPU performance?

---

# 13. Module Ten: GPU Execution Semantics

A general-purpose compiler is not automatically a correct GPU compiler.

GPU execution introduces special semantics.

## 13.1 Invocations

A compute dispatch creates logical invocations.

Each invocation may access:

- Its global invocation identifier.
- Its local invocation identifier.
- Its workgroup identifier.
- Shared resources.

## 13.2 Workgroups

Invocations are organized into workgroups.

Workgroups provide a defined scope for certain shared-memory and synchronization operations.

## 13.3 Subgroups

Subgroups are smaller execution groups with specialized operations.

Their supported sizes and capabilities depend on the API and hardware.

## 13.4 Divergence

Different invocations may follow different control-flow paths.

Consider:

```text
if global_id % 2 == 0:
    output[global_id] = 1
else:
    output[global_id] = 2
```

Neighboring invocations may take different branches.

This can affect execution efficiency.

## 13.5 Uniformity

An expression is uniform over a set of invocations if it has the same value for every invocation in that set.

Uniformity matters for operations with collective execution requirements.

## 13.6 Barrier correctness

A workgroup barrier is not an ordinary function call.

It establishes synchronization according to the target execution model.

Compilers must not transform control flow in ways that violate barrier participation requirements.

## 13.7 Example of unsafe barrier placement

Conceptually:

```text
if local_id == 0:
    workgroup_barrier()
```

This is not a valid general pattern for workgroup synchronization because only one invocation reaches the barrier.

The compiler must respect the target language's uniform-control-flow requirements.

## 13.8 Address spaces

GPU programs may distinguish:

- Function-local storage.
- Private storage.
- Workgroup storage.
- Storage buffers.
- Uniform buffers.

These have different lifetime, accessibility, and synchronization properties.

## 13.9 Atomics

Atomic operations provide specified indivisible memory operations and ordering semantics within the supported GPU memory model.

The compiler must preserve these semantics.

## 13.10 Memory ordering

A compiler cannot arbitrarily reorder memory operations across synchronization boundaries.

The legality of transformations depends on the source and target memory models.

## 13.11 Uniformity analysis

A compiler may classify expressions based on whether they depend on varying invocation identifiers or other varying data.

For example:

```text
x = 5
```

is uniform.

But:

```text
x = global_id
```

varies across invocations.

This analysis can become more complicated through branches, loops, memory reads, and function calls.

## Lab 10: GPU Semantic Analyzer

Extend your compiler with:

- Invocation built-ins.
- Workgroup-local memory.
- Barrier operations.
- Atomic operations.
- Address-space annotations.
- Basic uniformity analysis.

Reject unsupported or invalid constructs.

Document which restrictions are conservative approximations and which are required by the target language.

## Exercises

1. What is a GPU invocation?
2. What is a workgroup?
3. What is a subgroup?
4. What is divergence?
5. What is uniformity?
6. Why are barriers semantically special?
7. Why are GPU address spaces important?
8. Why must atomics be modeled explicitly?
9. How can compiler transformations violate synchronization?
10. Why is uniformity analysis useful?

---

# 14. Module Eleven: GPU-Specific Optimization

General compiler optimizations are only part of GPU performance engineering.

## 14.1 Memory coalescing

GPU performance depends heavily on how adjacent invocations access memory.

A compiler may improve memory access patterns through transformations such as:

- Loop interchange.
- Tiling.
- Data-layout transformations.
- Index simplification.

However, many such transformations require knowledge of data layout and algorithm semantics.

## 14.2 Tiling

Tiling divides a computation into smaller blocks.

For matrix multiplication:

\[
C_{ij}=\sum_k A_{ik}B_{kj}
\]

a tiled implementation computes blocks of \(C\) using blocks of \(A\) and \(B\).

The purpose is to improve data reuse and reduce redundant memory traffic.

## 14.3 Arithmetic intensity

Arithmetic intensity is:

\[
I=\frac{\text{FLOPs}}{\text{Bytes transferred}}
\]

A compiler transformation that improves reuse may increase arithmetic intensity.

## 14.4 Register pressure

Each invocation requires registers for live values.

If an optimization increases the number of simultaneously live values, it may increase register demand.

High register demand can reduce occupancy or cause spilling.

## 14.5 Loop unrolling trade-off

Unrolling can expose more independent arithmetic.

But it also increases the number of instructions and potentially live values.

A good compiler must balance instruction-level parallelism against resource pressure.

## 14.6 Fusion

Kernel fusion combines multiple operations into fewer GPU kernels.

For example:

\[
y_i=2x_i
\]

followed by:

\[
z_i=y_i+3
\]

could be fused into:

\[
z_i=2x_i+3
\]

Fusion may avoid writing and rereading an intermediate buffer.

But fusion can also increase register pressure and reduce scheduling flexibility.

## 14.7 Reduction optimization

Parallel reductions involve:

- Associativity assumptions.
- Workgroup synchronization.
- Shared memory.
- Multiple reduction stages.

Floating-point reductions are particularly sensitive to evaluation order.

## 14.8 Vectorization

Vectorization combines scalar operations into vector operations where supported and profitable.

On GPUs, source-level vector types do not necessarily map directly onto physical SIMD lanes.

The compiler must account for the target execution model.

## 14.9 Hardware-aware specialization

A compiler may generate different variants for different:

- Workgroup sizes.
- Tile sizes.
- Scalar widths.
- Feature sets.
- Memory configurations.

## 14.10 Autotuning

Autotuning measures multiple candidate configurations and selects one based on performance.

A simple tuning space might be:

\[
W\in\{32,64,128,256\}
\]

where \(W\) is workgroup size.

However, legal workgroup sizes and useful configurations depend on device limits, kernel resources, and execution semantics.

## 14.11 Cost models

A compiler may estimate performance using:

- Arithmetic operation counts.
- Memory traffic.
- Register demand.
- Workgroup occupancy.
- Launch overhead.
- Instruction throughput.

Cost models are approximations.

Actual benchmarking remains important.

## Lab 11: GPU Optimization Experiments

Implement or generate variants for:

- Workgroup size.
- Loop unrolling.
- Tiling.
- Kernel fusion.
- Memory access patterns.

Measure performance on the AMD GPU.

Record:

- Input size.
- Workgroup size.
- Kernel variant.
- Execution time.
- Correctness.
- Estimated memory traffic.
- Observed performance changes.

## Exercises

1. What is memory coalescing?
2. Why does tiling improve reuse?
3. Define arithmetic intensity.
4. What is register pressure?
5. Why can unrolling hurt occupancy?
6. What is kernel fusion?
7. Why can fusion reduce memory traffic?
8. Why are floating-point reductions difficult to reorder?
9. What is autotuning?
10. Why are compiler cost models imperfect?

---

# 15. Module Twelve: SPIR-V and GPU Backends

Your compiler must eventually emit code that another system can validate and execute.

## 15.1 WGSL backend

A WGSL backend translates your IR into valid WGSL source.

Advantages:

- Easier to inspect.
- Easier to debug.
- Integrates with your existing wgpu infrastructure.
- Delegates SPIR-V generation and lower-level compilation.

Disadvantages:

- Another compiler controls subsequent lowering.
- Some low-level capabilities may not be expressible.
- Backend behavior depends on WGSL and wgpu support.

## 15.2 SPIR-V backend

A direct SPIR-V backend emits SPIR-V instructions.

It must handle:

- Types.
- Constants.
- Functions.
- Entry points.
- Decorations.
- Storage classes.
- Capabilities.
- Control flow.
- Resource interfaces.
- Execution modes.
- Memory semantics.

## 15.3 Structured control flow

Vulkan-compatible SPIR-V shader modules have structured control-flow requirements.

A compiler cannot assume that every arbitrary CFG can be emitted directly without appropriate structuring.

## 15.4 SPIR-V types

SPIR-V uses explicit type declarations.

For example, integer and floating-point types must be represented according to their supported widths and capabilities.

## 15.5 Resource decorations

Shader resource bindings require appropriate interface metadata.

The backend must ensure the generated resource declarations match host-side pipeline bindings.

## 15.6 Entry points

A compute entry point must specify the appropriate execution model and required execution configuration.

## 15.7 Validation

Use SPIR-V validation tools with the appropriate target environment.

A module passing structural validation may still contain algorithmic or synchronization errors.

## 15.8 Disassembly

SPIR-V disassembly is useful for inspecting:

- Control flow.
- Memory operations.
- Types.
- Entry points.
- Decorations.
- Capabilities.

## 15.9 Driver compilation

The GPU driver further compiles or translates SPIR-V into architecture-specific executable instructions.

Your compiler does not directly control every optimization performed by the driver.

## 15.10 Backend strategy

Recommended progression:

1. Emit WGSL.
2. Validate generated WGSL.
3. Execute kernels through wgpu.
4. Inspect resulting SPIR-V when available.
5. Implement a limited direct SPIR-V backend.
6. Compare generated representations.

## Lab 12: Executable GPU Backend

Generate and execute:

- Vector addition.
- Elementwise multiplication.
- Conditional transform.
- Matrix-vector multiplication.
- Reduction.

For each kernel:

1. Compile source into IR.
2. Verify IR.
3. Apply optimizations.
4. Generate target code.
5. Validate target code.
6. Execute on the GPU.
7. Compare against a CPU reference.

## Exercises

1. What is a compiler backend?
2. Why begin with WGSL?
3. What are SPIR-V storage classes?
4. What are SPIR-V decorations?
5. What is structured control flow?
6. Why are entry-point declarations necessary?
7. What does SPIR-V validation establish?
8. Why does the driver perform additional compilation?

---

# 16. Module Thirteen: LLVM and AMD GPU Compilation

Your custom compiler is intentionally small.

Studying established compiler infrastructure provides a broader perspective.

## 16.1 LLVM

LLVM provides reusable compiler infrastructure including:

- Intermediate representations.
- Analysis frameworks.
- Optimization passes.
- Target backends.
- Code-generation infrastructure.

## 16.2 LLVM IR

LLVM IR is a typed, SSA-based intermediate representation.

It includes:

- Functions.
- Basic blocks.
- Instructions.
- Types.
- Memory operations.
- Control flow.
- Attributes.
- Metadata.

## 16.3 GPU-specific LLVM targets

LLVM includes GPU-related backend infrastructure, including AMDGPU and NVPTX targets.

These targets have different capabilities and execution environments.

## 16.4 AMDGPU

The AMDGPU backend supports code generation for AMD GPU architectures in supported toolchain configurations.

The actual compilation path for a Vulkan shader on Fedora may involve Mesa's shader compiler infrastructure rather than simply passing source through a standalone LLVM AMDGPU backend.

Do not assume all AMD GPU software stacks use the same compiler pipeline.

## 16.5 Mesa shader compilation

Mesa contains shader compilation infrastructure used by several open-source GPU drivers.

Its internal IRs and compilation stages are important examples of production GPU compiler design.

## 16.6 NIR

NIR is Mesa's intermediate representation for shaders.

It supports shader analysis and optimization before target-specific lowering.

Study its design as an example of a mature GPU-focused IR.

## 16.7 Instruction selection

Instruction selection maps IR operations to supported target instructions.

A high-level multiplication or memory operation may require multiple machine instructions depending on types and hardware capabilities.

## 16.8 Register allocation

Register allocation assigns program values to physical registers.

When there are insufficient registers, some values may be spilled to memory.

## 16.9 Instruction scheduling

Instruction scheduling arranges instructions to satisfy dependencies while attempting to improve execution efficiency.

## 16.10 Compiler inspection

Where tooling permits, compare:

- High-level source.
- Optimized IR.
- SPIR-V.
- Driver-generated instructions.
- Measured kernel performance.

Avoid attributing performance differences to one compiler stage without supporting evidence.

## Lab 13: Production Compiler Investigation

Study a simple GPU kernel through the available AMD compilation stack.

Document:

- Source language.
- Front-end compiler.
- Intermediate representations.
- Optimization stages.
- Target backend.
- Generated instructions, if inspectable.
- Hardware execution.

Compare your compiler's design with the production system.

## Exercises

1. What is LLVM?
2. What is LLVM IR?
3. What is the AMDGPU backend?
4. What is Mesa NIR?
5. What is instruction selection?
6. What is register allocation?
7. What is spilling?
8. What is instruction scheduling?
9. Why do GPU compiler pipelines differ by API?
10. Why is production compiler inspection valuable?

---

# 17. Module Fourteen: Compiler Correctness and Testing

Compiler testing is unusually important because a compiler can silently change program behavior.

## 17.1 Unit tests

Test individual components:

- Lexer.
- Parser.
- Type checker.
- IR verifier.
- Dominator analysis.
- SSA construction.
- Optimization passes.
- Backends.

## 17.2 Golden tests

A golden test compares compiler output against a known expected representation.

Golden tests are useful for:

- AST printing.
- Diagnostics.
- IR formatting.
- Generated shader text.

They should not be the only correctness mechanism.

## 17.3 Differential testing

Differential testing compares multiple implementations of the same semantics.

For example:

\[
\text{Reference Interpreter}
\]

versus:

\[
\text{Compiled GPU Program}
\]

Both execute equivalent inputs.

Differences may indicate compiler bugs, unsupported semantics, or numerical behavior that requires further analysis.

## 17.4 Reference interpreter

Build a simple interpreter for your source language or high-level IR.

It can serve as an independent correctness oracle.

For GPU-specific behavior, define which semantics the interpreter models and which it does not.

## 17.5 Property-based testing

Generate random valid programs and verify properties such as:

- Parsing succeeds for valid syntax.
- IR verification succeeds after lowering.
- Optimization preserves defined behavior.
- Generated shaders validate.
- Results agree with the reference implementation within the specified numerical contract.

## 17.6 Metamorphic testing

Metamorphic testing checks relationships between transformed inputs or programs.

For example, renaming a local variable should not change program results if name resolution remains valid.

## 17.7 Fuzzing

Fuzzers can generate malformed or unexpected inputs.

Targets include:

- Lexer.
- Parser.
- Type checker.
- IR verifier.
- Backend.

The compiler should reject invalid inputs without crashing or violating memory safety.

## 17.8 Floating-point testing

Floating-point programs require explicit numerical expectations.

Different evaluation orders may produce different rounded results.

A valid test strategy may include:

- Exact comparison where justified.
- Absolute error.
- Relative error.
- ULP-based comparisons.
- Special handling of NaNs and infinities.

## 17.9 Optimization differential testing

For each optimization pass:

1. Execute the unoptimized program.
2. Execute the optimized program.
3. Compare behavior under the defined semantics.
4. Investigate discrepancies.
5. Record the failing program as a regression test.

## 17.10 Compiler invariants

Every IR transformation should preserve required structural invariants.

Run the IR verifier after passes during development.

## Lab 14: Compiler Validation Framework

Build:

- Unit tests.
- Golden tests.
- Reference interpreter.
- Differential tests.
- Random program generator.
- Fuzz targets.
- Regression test suite.

Create a report documenting the kinds of bugs each testing method can and cannot detect.

## Exercises

1. Why are compiler bugs dangerous?
2. What is a golden test?
3. What is differential testing?
4. Why build a reference interpreter?
5. What is property-based testing?
6. What is metamorphic testing?
7. What is fuzzing?
8. Why are floating-point tests difficult?
9. Why verify IR after transformations?
10. What makes a compiler test suite trustworthy?

---

# 18. Major Assessment A: Compiler Front End

Build a complete front end for your restricted GPU language.

## Requirements

- Lexer.
- Parser.
- AST.
- Name resolution.
- Type checking.
- Source spans.
- Diagnostics.
- Typed AST.
- Error recovery where practical.

## Deliverables

- Language specification.
- Rust source code.
- Unit tests.
- Invalid-program test suite.
- Example programs.
- Architecture document.

## Evaluation

| Category | Weight |
|---|---:|
| Language specification | 15% |
| Lexer and parser | 25% |
| Semantic analysis | 25% |
| Diagnostics | 15% |
| Testing | 20% |

---

# 19. Major Assessment B: IR and Optimization Engine

Build a compiler middle end.

## Requirements

- Typed IR.
- CFG construction.
- Dominator analysis.
- SSA representation.
- Data-flow analysis.
- Constant propagation.
- Dead-code elimination.
- Common subexpression elimination.
- At least one loop optimization.
- IR verifier.

## Deliverables

- IR specification.
- Analysis implementation.
- Optimization passes.
- Before/after IR examples.
- Correctness tests.
- Performance investigation.

## Evaluation

| Category | Weight |
|---|---:|
| IR design | 20% |
| CFG and SSA | 20% |
| Data-flow analysis | 20% |
| Optimizations | 20% |
| Verification and testing | 20% |

---

# 20. Major Assessment C: GPU Backend

Build a backend capable of executing generated GPU programs.

## Requirements

- WGSL emission.
- Kernel entry points.
- Buffer bindings.
- Invocation identifiers.
- Arithmetic.
- Branching.
- Bounded loops.
- Shader validation.
- GPU execution.
- CPU reference comparison.

## Advanced requirements

Choose at least two:

- Direct SPIR-V emission.
- Workgroup-local memory.
- Barrier support.
- Atomic operations.
- Reduction lowering.
- Kernel fusion.
- Workgroup-size specialization.

## Evaluation

| Category | Weight |
|---|---:|
| Correct code generation | 35% |
| GPU execution | 20% |
| Validation | 20% |
| Advanced features | 15% |
| Documentation | 10% |

---

# 21. Major Assessment D: GPU Compiler Performance Study

Investigate how compiler transformations affect actual GPU performance.

## Required experiments

1. Constant propagation.
2. Dead-code elimination.
3. Loop unrolling.
4. Kernel fusion.
5. Workgroup-size specialization.
6. Memory access transformation.

## Required methodology

For each experiment:

- Establish correctness.
- Measure baseline performance.
- Apply one controlled transformation.
- Measure transformed performance.
- Record multiple trials.
- Inspect generated code where possible.
- Explain observed differences.
- Identify uncertainty.

## Deliverables

- Benchmark harness.
- Raw measurements.
- Summary tables.
- Performance plots.
- Analysis report.

---

# 22. Final Capstone: Rust GPU Compute Compiler

Your final project combines the entire phase.

## 22.1 Compiler architecture

Implement:

```text
Source
  |
  v
Lexer
  |
  v
Parser
  |
  v
Typed AST
  |
  v
High-Level IR
  |
  v
CFG / SSA
  |
  v
Optimization Passes
  |
  v
GPU Lowering
  |
  v
WGSL Backend
  |
  v
wgpu / Vulkan
  |
  v
AMD GPU
```

A direct SPIR-V backend is an optional advanced extension.

## 22.2 Required language features

- Scalar arithmetic.
- Local variables.
- Conditions.
- Bounded loops.
- Buffer indexing.
- Buffer writes.
- Kernel entry points.
- Invocation identifiers.
- Typed function parameters.

## 22.3 Required compiler analyses

- Type checking.
- CFG construction.
- Dominance.
- SSA.
- Liveness.
- Constant propagation.
- Basic alias analysis.
- GPU uniformity checks where relevant.

## 22.4 Required optimizations

- Constant folding.
- Constant propagation.
- Dead-code elimination.
- Common subexpression elimination.
- Branch simplification.
- Loop-invariant code motion.
- At least one GPU-specific optimization.

## 22.5 Required kernels

Compile and execute:

1. Vector addition.
2. Elementwise transform.
3. Conditional transform.
4. Matrix-vector multiplication.
5. Reduction.
6. One advanced kernel of your choice.

## 22.6 Required correctness infrastructure

- Reference interpreter.
- IR verifier.
- Shader validation.
- CPU/GPU differential testing.
- Randomized test programs.
- Floating-point correctness policy.
- Regression tests.

## 22.7 Required performance infrastructure

- Optimization enable/disable flags.
- Kernel timing.
- Workgroup-size experiments.
- Generated-code inspection.
- Benchmark result export.
- Reproducible toolchain configuration.

## 22.8 Required documentation

Produce:

### Language specification

Describe syntax, types, operations, memory model, and restrictions.

### Compiler architecture

Explain the front end, middle end, and back end.

### IR specification

Define instructions, types, control flow, and invariants.

### Optimization manual

Describe every optimization and its correctness preconditions.

### Backend documentation

Explain how IR operations map to WGSL or SPIR-V.

### Validation report

Describe tests, reference behavior, and known limitations.

### Performance report

Present measured results and explain uncertainty.

## 22.9 Technical defense

You must be able to answer:

1. Why did you choose your IR?
2. How does name resolution work?
3. How are types checked?
4. How is control flow represented?
5. Why is SSA useful?
6. How are phi nodes introduced?
7. How does liveness analysis work?
8. How does constant propagation work?
9. How do you prevent invalid dead-code elimination?
10. How do you handle memory aliasing?
11. How do you preserve GPU barrier semantics?
12. How do you represent GPU address spaces?
13. How do you emit valid shader code?
14. How do you test optimization correctness?
15. How do you interpret performance differences?
16. Which features would you add next?

---

# 23. Phase 14 Quiz Bank

## Compiler architecture

1. What is a compiler?
2. What is a front end?
3. What is a middle end?
4. What is a back end?
5. What is an IR?
6. Why are multiple IR levels useful?
7. What is lowering?
8. What is JIT compilation?
9. What is ahead-of-time compilation?
10. Why can GPU pipeline creation involve compilation?

## Lexing and parsing

11. What is a token?
12. What is a lexer?
13. What is a grammar?
14. What is a parser?
15. What is an AST?
16. What is operator precedence?
17. What is associativity?
18. What is recursive-descent parsing?
19. What is Pratt parsing?
20. Why is source-location tracking important?

## Semantic analysis

21. What is name resolution?
22. What is a symbol table?
23. What is lexical scope?
24. What is type checking?
25. What is a typed AST?
26. Why are implicit conversions risky?
27. What is a semantic error?
28. Why do GPU capabilities affect type checking?

## IR and CFG

29. What is three-address code?
30. What is a basic block?
31. What is a CFG?
32. What is a terminator?
33. What is a control-flow edge?
34. Why use value identifiers?
35. What is an IR verifier?
36. Why are CFGs useful for optimization?

## Dominance and SSA

37. What is dominance?
38. What is a dominator tree?
39. What is a dominance frontier?
40. What is SSA?
41. What is a phi node?
42. Why are phi nodes needed?
43. How are loops represented in SSA?
44. Why does SSA not solve memory aliasing?

## Data-flow analysis

45. What is data-flow analysis?
46. What is forward analysis?
47. What is backward analysis?
48. What is liveness?
49. What are reaching definitions?
50. What is a fixed point?
51. What is a lattice?
52. What is a worklist algorithm?

## Optimizations

53. What is constant folding?
54. What is constant propagation?
55. What is dead-code elimination?
56. What is common subexpression elimination?
57. What is branch simplification?
58. What is loop-invariant code motion?
59. What is loop unrolling?
60. What is strength reduction?
61. Why does pass ordering matter?
62. Why are floating-point optimizations difficult?

## Memory analysis

63. What is aliasing?
64. What is may-alias analysis?
65. What is no-alias analysis?
66. What is memory dependence?
67. What is range analysis?
68. Why can aliasing prevent load elimination?
69. Why must atomics be treated specially?
70. Why is conservative analysis useful?

## GPU semantics

71. What is a GPU invocation?
72. What is a workgroup?
73. What is a subgroup?
74. What is divergence?
75. What is uniformity?
76. Why do barriers constrain optimization?
77. What are GPU address spaces?
78. Why are atomics part of the memory model?
79. Why is workgroup-local memory special?
80. Why can control-flow transformations break GPU correctness?

## GPU optimization

81. What is memory coalescing?
82. What is tiling?
83. What is arithmetic intensity?
84. What is register pressure?
85. What is spilling?
86. What is kernel fusion?
87. Why can fusion reduce memory traffic?
88. Why can unrolling reduce occupancy?
89. What is autotuning?
90. Why are cost models imperfect?

## Backends and production compilers

91. What is WGSL?
92. What is SPIR-V?
93. What is a shader entry point?
94. What is structured control flow?
95. What is a SPIR-V capability?
96. What is LLVM IR?
97. What is Mesa NIR?
98. What is instruction selection?
99. What is register allocation?
100. What is instruction scheduling?

## Correctness and testing

101. What is differential testing?
102. What is a reference interpreter?
103. What is a golden test?
104. What is property-based testing?
105. What is metamorphic testing?
106. What is fuzzing?
107. Why verify IR after every pass?
108. Why do floating-point results require special comparison?
109. Why does valid SPIR-V not guarantee correct output?
110. How can compiler optimization performance be measured fairly?

---

# 24. Final Examination

**Recommended duration:** 10–14 hours.

| Section | Weight |
|---|---:|
| Front-end design | 15% |
| IR and control flow | 15% |
| SSA and data flow | 20% |
| Optimization correctness | 20% |
| GPU semantics and lowering | 15% |
| Backend, validation, and performance | 15% |

## Part A: Parsing

Design a grammar and AST for a restricted compute language.

Explain precedence, associativity, and error handling.

## Part B: Semantic analysis

Analyze a program containing variable scopes, types, and invalid operations.

## Part C: CFG and SSA

Construct a CFG and convert appropriate values to SSA form.

Explain dominance and phi placement.

## Part D: Data-flow analysis

Calculate liveness and constant-propagation information for a small program.

## Part E: Optimization

Analyze which transformations are legal and which would change behavior.

Include floating-point, aliasing, and side-effect cases.

## Part F: GPU semantics

Explain how divergence, workgroup memory, and barriers constrain compiler transformations.

## Part G: Backend

Describe how a kernel is lowered into a valid GPU shader.

## Part H: Technical defense

Defend your compiler's architecture, correctness, and performance.

## Passing standard

Recommended:

- At least 80% overall.
- At least 85% in optimization correctness.
- No known invalid IR transformations.
- No unresolved shader validation failures in required kernels.
- Correct CPU/GPU comparisons under documented numerical rules.
- Reproducible compiler and benchmark environment.

---

# 25. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

**The AI tutor must never provide answers, code, pseudocode, implementation instructions, debugging fixes, file edits, completed derivations, or assessment solutions.**

It must ask exactly one guiding question at a time.

## Master Socratic Tutor Prompt

```text
You are my Socratic GPU compiler engineering tutor.

I am studying Phase 14 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   derivations, projects, or examinations.

2. Never write code, pseudocode, algorithms, or implementations.

3. Never modify files, execute code, debug programs, or perform
   project work on my behalf.

4. Never reveal answer keys, completed calculations,
   hidden tests, or solutions.

5. Never provide hints that effectively disclose a solution.

6. Guide my reasoning exclusively through questions.

7. Ask exactly one guiding question per response.

8. If I am incorrect, ask a question that exposes the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to reason about parsing, typing, IR design,
    CFGs, SSA, data flow, optimization legality,
    GPU execution semantics, lowering, and correctness.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness,
    invariants, edge cases, semantics, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only one guiding question.
```

## Start a lesson

```text
Tutor me through Phase 14, Module [NUMBER].

Start from first principles.

Ask exactly one question at a time.

Require me to reason about compiler architecture,
language semantics, intermediate representations,
optimization correctness, and GPU execution.

Never provide answers or solutions.
```

## Generate a quiz

```text
Administer a 20-question Phase 14 quiz on [TOPIC].

Include conceptual reasoning, mathematical analysis,
compiler invariants, and GPU-specific constraints.

Ask exactly one question at a time.

Adapt difficulty based on my responses.

Never reveal answers.
```

## Front-end assessment

```text
Administer the Phase 14 compiler front-end assessment.

Present requirements and constraints only.

Ask me to justify my grammar, parser architecture,
type system, name resolution, and diagnostics.

Do not provide code, pseudocode, implementation guidance,
debugging fixes, or solutions.

Ask exactly one question at a time.
```

## SSA and data-flow examination

```text
Examine my understanding of SSA and data-flow analysis.

Ask questions about:
- Basic blocks
- CFGs
- Dominance
- Dominance frontiers
- Phi nodes
- Liveness
- Reaching definitions
- Fixed-point analysis

Never provide completed calculations or solutions.

Ask exactly one question at a time.
```

## Optimization correctness review

```text
Review my compiler optimization design using only questions.

Focus on:
- Semantic equivalence
- Integer overflow
- Floating-point behavior
- Aliasing
- Side effects
- Atomics
- Barriers
- Control flow
- IR invariants

Do not provide fixes, code, or solutions.

Ask exactly one guiding question at a time.
```

## GPU compiler examination

```text
Examine my understanding of GPU compiler semantics.

Ask questions about:
- Invocations
- Workgroups
- Subgroups
- Divergence
- Uniformity
- Address spaces
- Atomics
- Barriers
- Memory coalescing
- Register pressure
- Tiling
- Fusion

Never provide answers.

Ask exactly one question at a time.
```

## Backend assessment

```text
Administer my Phase 14 GPU backend assessment.

Ask me to explain:
- IR lowering
- Shader entry points
- Resource bindings
- Structured control flow
- WGSL
- SPIR-V
- Validation
- Driver compilation

Do not provide code, pseudocode, implementation steps,
or debugging fixes.

Ask exactly one question at a time.
```

## Compiler performance investigation

```text
Help me investigate compiler-generated GPU performance
using only questions.

Ask about:
- Generated instructions
- Memory traffic
- Register pressure
- Workgroup sizes
- Loop unrolling
- Kernel fusion
- Compiler pass ordering
- Benchmark methodology
- Numerical correctness

Do not interpret results for me.
Do not provide conclusions or optimizations.

Ask exactly one question at a time.
```

## Final oral examination

```text
Conduct a rigorous Phase 14 oral examination.

Challenge my understanding of:
- Compiler architecture
- Parsing
- Semantic analysis
- IR design
- CFGs
- SSA
- Data-flow analysis
- Optimization correctness
- GPU semantics
- SPIR-V
- Code generation
- Testing
- Performance

Ask exactly one question at a time.

Never provide answers, solutions, code,
or hints that reveal the answer.
```

---

# 26. Mastery Checklist

## Compiler fundamentals

- [ ] I understand front ends, middle ends, and back ends.
- [ ] I understand intermediate representations.
- [ ] I understand lowering.
- [ ] I understand AOT and JIT compilation.
- [ ] I can explain the GPU compilation pipeline.

## Front end

- [ ] I can implement a lexer.
- [ ] I can define a grammar.
- [ ] I can implement a parser.
- [ ] I can build an AST.
- [ ] I understand precedence and associativity.
- [ ] I can implement name resolution.
- [ ] I can implement type checking.
- [ ] I can generate useful diagnostics.

## Intermediate representations

- [ ] I can design a typed IR.
- [ ] I can construct CFGs.
- [ ] I can implement an IR verifier.
- [ ] I understand dominance.
- [ ] I understand dominance frontiers.
- [ ] I understand SSA.
- [ ] I can explain phi nodes.
- [ ] I can analyze loop-carried values.

## Data-flow analysis

- [ ] I understand liveness.
- [ ] I understand reaching definitions.
- [ ] I understand fixed-point iteration.
- [ ] I understand data-flow lattices.
- [ ] I can implement a worklist analysis.

## Optimizations

- [ ] I can implement constant folding.
- [ ] I can implement constant propagation.
- [ ] I can implement dead-code elimination.
- [ ] I can implement common subexpression elimination.
- [ ] I understand loop-invariant code motion.
- [ ] I understand loop unrolling.
- [ ] I understand optimization pass ordering.
- [ ] I can justify transformation correctness.

## Memory analysis

- [ ] I understand alias analysis.
- [ ] I understand memory dependence.
- [ ] I understand range analysis.
- [ ] I understand atomic side effects.
- [ ] I understand conservative optimization.

## GPU semantics

- [ ] I understand GPU invocations.
- [ ] I understand workgroups and subgroups.
- [ ] I understand divergence.
- [ ] I understand uniformity.
- [ ] I understand GPU address spaces.
- [ ] I understand barrier semantics.
- [ ] I understand GPU memory ordering.
- [ ] I can explain GPU-specific optimization constraints.

## GPU optimization

- [ ] I understand memory coalescing.
- [ ] I understand tiling.
- [ ] I understand arithmetic intensity.
- [ ] I understand register pressure.
- [ ] I understand spilling.
- [ ] I understand kernel fusion.
- [ ] I understand autotuning.
- [ ] I can benchmark compiler transformations.

## Backends

- [ ] I can generate WGSL.
- [ ] I understand SPIR-V.
- [ ] I understand shader entry points.
- [ ] I understand structured control flow.
- [ ] I can validate generated shaders.
- [ ] I understand LLVM IR.
- [ ] I understand Mesa NIR.
- [ ] I understand instruction selection.
- [ ] I understand register allocation.

## Testing

- [ ] I can build a reference interpreter.
- [ ] I can implement differential tests.
- [ ] I can implement property-based tests.
- [ ] I understand metamorphic testing.
- [ ] I understand fuzzing.
- [ ] I can test floating-point behavior.
- [ ] I can maintain compiler regression tests.

## Capstone

- [ ] My compiler parses valid programs.
- [ ] My compiler rejects invalid programs.
- [ ] My typed IR is verified.
- [ ] My SSA and analyses are correct.
- [ ] My optimization passes preserve semantics.
- [ ] My backend emits valid shaders.
- [ ] My generated kernels execute on the AMD GPU.
- [ ] My results agree with reference implementations.
- [ ] My performance experiments are reproducible.
- [ ] I can defend my compiler architecture independently.

---

# 27. Recommended Resources

## Compiler textbooks

**Engineering a Compiler — Keith Cooper and Linda Torczon**

Excellent for compiler architecture, IRs, analyses, and optimization.

**Compilers: Principles, Techniques, and Tools — Aho, Lam, Sethi, and Ullman**

Foundational coverage of parsing, semantic analysis, optimization, and code generation.

**Modern Compiler Implementation — Andrew Appel**

Useful for compiler construction and intermediate representations.

**Advanced Compiler Design and Implementation — Steven Muchnick**

More advanced material on optimization and data-flow analysis.

## LLVM

**LLVM Documentation**

https://llvm.org/docs/

**LLVM Language Reference**

https://llvm.org/docs/LangRef.html

**LLVM Kaleidoscope Tutorial**

https://llvm.org/docs/tutorial/

The Kaleidoscope tutorial is a useful example of constructing a small compiler, although it does not directly teach GPU compilation.

## SPIR-V

**SPIR-V Registry**

https://registry.khronos.org/SPIR-V/

**SPIRV-Tools**

https://github.com/KhronosGroup/SPIRV-Tools

**SPIRV-Cross**

https://github.com/KhronosGroup/SPIRV-Cross

## Mesa and AMD

**Mesa**

https://www.mesa3d.org/

**Mesa Source Repository**

https://gitlab.freedesktop.org/mesa/mesa

**AMD GPUOpen**

https://gpuopen.com/

## Rust compiler engineering

**rustc Development Guide**

https://rustc-dev-guide.rust-lang.org/

**The Rust Reference**

https://doc.rust-lang.org/reference/

## GPU programming and performance

**Programming Massively Parallel Processors — Kirk and Hwu**

**Computer Architecture: A Quantitative Approach — Hennessy and Patterson**

## Suggested reading order

1. Engineering a Compiler.
2. LLVM Kaleidoscope.
3. LLVM IR documentation.
4. SPIR-V specification.
5. Mesa NIR documentation and source.
6. GPU architecture and optimization references.
7. Production compiler implementation studies.

---

# 28. Completion Standard

You have completed Phase 14 when you can independently:

1. Design a restricted GPU programming language.
2. Implement a lexer and parser.
3. Implement semantic analysis.
4. Build a typed IR.
5. Construct CFGs.
6. Compute dominance relationships.
7. Represent programs in SSA.
8. Implement data-flow analyses.
9. Implement semantics-preserving optimizations.
10. Analyze memory dependencies.
11. Model GPU-specific execution semantics.
12. Lower programs into executable shader code.
13. Validate generated code.
14. Execute generated kernels on your AMD GPU.
15. Build a reference interpreter.
16. Perform differential testing.
17. Measure optimization performance.
18. Explain the relationship between your compiler and production GPU compiler infrastructure.
19. Defend the correctness of your transformations.
20. Present a complete, reproducible Rust GPU compiler.

The defining competency is **the ability to explain and justify every transformation between source-language semantics and GPU execution**.

---

# 29. What Comes Next

**Phase 15: Specialization and Final Capstone**

The final phase integrates everything you have learned across the curriculum.

You will choose a specialization such as:

- GPU compiler engineering.
- GPU runtime and systems engineering.
- High-performance GPU algorithms.
- Numerical computing and scientific simulation.
- Machine learning systems.
- GPU programming-language infrastructure.

You will then design, implement, benchmark, document, and defend a substantial engineering project.

The final capstone should demonstrate not only implementation ability, but also the engineering judgment to choose appropriate abstractions, prove correctness, measure performance, identify limitations, and communicate technical decisions.
