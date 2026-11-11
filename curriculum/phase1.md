# Phase 1: Rust Systems Programming
## A First-Principles Study Manual

**Curriculum:** GPU Engineering  
**Phase:** 1 of 16  
**Estimated duration:** 8–12 weeks  
**Recommended study time:** 10–12 hours per week  
**Primary language:** Rust  
**Prerequisite:** Phase 0 — Mathematical and Computational Foundations

---

# 1. What You Are Learning

The goal of this phase is not simply to become comfortable writing Rust.

You are learning how a program represents information, owns resources, accesses memory, and coordinates work. Rust is the tool we will use to explore these ideas because it makes many systems-level concepts explicit.

By the end of this phase, you should be able to:

1. Explain how Rust source code becomes an executable program.
2. Understand the difference between values, variables, references, and memory locations.
3. Reason about ownership, moves, borrowing, and lifetimes without relying on memorized compiler rules.
4. Understand how common Rust types are represented in memory.
5. Build abstractions using structs, enums, generics, and traits.
6. Understand allocation and resource management.
7. Reason about shared state and concurrent execution.
8. Understand when unsafe Rust is necessary and what guarantees it removes.
9. Debug, test, benchmark, and inspect Rust programs.
10. Build small systems-oriented programs without depending on high-level frameworks.

The most important objective is that you should be able to explain *why* the language behaves the way it does.

If you encounter a borrow-checker error, the goal is not to memorize the syntax that makes it disappear. The goal is to understand what memory or aliasing problem the compiler is preventing.

# 2. How to Study This Manual

Do not read this document from beginning to end without writing code.

Use the following cycle for every lesson:

1. **Understand:** Read the lesson and identify the underlying problem.
2. **Predict:** Before running code, predict what the compiler or program will do.
3. **Experiment:** Write a small program and test your prediction.
4. **Explain:** Explain the result in your own words.
5. **Practice:** Complete the exercises without looking up solutions.
6. **Defend:** Ask yourself why your implementation is correct and what could make it fail.

Keep a learning journal. For every difficult concept, write:

- What I initially believed.
- What actually happened.
- Why my original understanding was incomplete.
- What principle explains the behavior.
- What new example would test whether I understand it.

## Your tutoring rule

Your AI tutor is a Socratic instructor, not a code generator or solution engine.

It must not do your work for you. It should ask questions that help you discover the answer yourself. The reusable agent prompts are included in Section 17.

You should attempt problems independently before asking for guidance. When you ask for help, provide your current reasoning, not just the compiler error.

# 3. Development Environment

You will use Rust on Fedora Linux.

## 3.1 Install the toolchain

Install Rust through rustup, the standard Rust toolchain installer.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After installation:

```bash
rustc --version
cargo --version
rustup show
```

The important tools are:

| Tool | Purpose |
|---|---|
| `rustc` | Compiles Rust source code |
| `cargo` | Manages projects, dependencies, builds, and tests |
| `rustfmt` | Formats Rust source code |
| `clippy` | Detects suspicious patterns and common mistakes |
| `rustdoc` | Generates documentation |

Install the additional components:

```bash
rustup component add rustfmt clippy
```

## 3.2 Create your first project

```bash
cargo new rust-systems-lab
cd rust-systems-lab
cargo run
```

A Cargo project typically contains:

```text
rust-systems-lab/
├── Cargo.toml
└── src/
    └── main.rs
```

`Cargo.toml` describes the package and its dependencies.

`main.rs` contains the program entry point.

Run the following commands as you work:

```bash
cargo check
cargo build
cargo run
cargo test
cargo fmt
cargo clippy
```

Understand the difference between checking, building, running, and testing. They are not interchangeable.

**Exercise 1:** Create a new project. Before running each Cargo command, write down what you expect it to do. Run it and compare your prediction with the result.

# 4. Lesson One: How Rust Programs Become Executables

Before studying ownership or pointers, understand the basic lifecycle of a program.

## 4.1 Source code is not machine code

Consider:

```rust
fn main() {
    let x = 10;
    let y = 20;

    println!("{}", x + y);
}
```

A CPU does not directly understand Rust syntax.

The source code must be translated into instructions the processor can execute.

A simplified compilation process is:

1. Source text is parsed.
2. Names and declarations are resolved.
3. Types are checked.
4. Ownership and borrowing rules are checked.
5. The compiler produces an intermediate representation.
6. Code generation translates that representation into machine instructions.
7. The linker combines the necessary compiled components into an executable.

Rust's compiler is called `rustc`. Its implementation uses LLVM for many compilation targets.

The exact internal compilation pipeline is more complicated, but this model is sufficient to understand the major stages.

## 4.2 What is a compiler checking?

Consider:

```rust
fn main() {
    let x: i32 = 10;
    let y: bool = true;

    let result = x + y;
}
```

The expression is invalid because addition is not defined between these types.

This is a type error.

Now consider:

```rust
fn main() {
    let x = String::from("hello");

    let y = x;

    println!("{}", x);
}
```

This is a different kind of problem. The issue is not that the types are incompatible. The issue is ownership.

We will examine why in Lesson Three.

## 4.3 Expressions and statements

Rust is expression-oriented.

An expression evaluates to a value.

```rust
let x = {
    let a = 10;
    let b = 20;

    a + b
};
```

The final expression in the block becomes the block's value.

Notice the missing semicolon after `a + b`.

Compare:

```rust
let x = {
    let a = 10;
    a + 1
};
```

with:

```rust
let x = {
    let a = 10;
    a + 1;
};
```

The first block evaluates to an integer. The second block evaluates to `()` — the unit type.

This distinction is important because Rust uses expressions extensively in control flow, functions, and pattern matching.

## 4.4 Variables, bindings, and mutability

A Rust variable is a name bound to a value.

```rust
let x = 10;
```

By default, bindings are immutable.

```rust
let mut x = 10;
x = 20;
```

`mut` allows reassignment of the binding.

It does not automatically make every value reachable through that binding mutable.

For example, a mutable reference and an immutable reference have different permissions. We will explore this in the borrowing lessons.

## 4.5 Types and inference

Rust is statically typed.

Every expression has a type, even when you do not explicitly write it.

```rust
let x = 10;
```

The compiler infers the type from context. In this case, the default integer type is normally `i32`.

You can make it explicit:

```rust
let x: u64 = 10;
```

Common primitive types:

| Category | Types |
|---|---|
| Signed integers | `i8`, `i16`, `i32`, `i64`, `i128`, `isize` |
| Unsigned integers | `u8`, `u16`, `u32`, `u64`, `u128`, `usize` |
| Floating point | `f32`, `f64` |
| Boolean | `bool` |
| Unicode scalar value | `char` |
| No meaningful value | `()` |

The size of `usize` and `isize` depends on the target architecture.

On a typical 64-bit system, `usize` is 64 bits.

This matters because `usize` is commonly used for memory sizes, indexing, and collection lengths.

## 4.6 Functions and control flow

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The return type is specified after `->`.

A function returns its final expression unless an explicit `return` is used.

Rust also supports:

- `if` expressions
- `loop`
- `while`
- `for`
- `match`

Study `match` carefully. It is a central feature of Rust's type system and is especially useful when working with enums and errors.

### Lesson One exercises

1. Write a function that accepts two integers and returns their sum.
2. Write a function that returns the larger of two numbers.
3. Write a function that classifies an integer as negative, zero, or positive.
4. Write a loop that calculates the sum of integers from 1 through N.
5. Create a block expression that computes a value using multiple local variables.
6. Predict the type of ten different expressions before compiling them.

**Mastery question:** Can you explain the difference between a variable binding, a value, an expression, and a type?

# 5. Lesson Two: Stack, Heap, and Memory

This lesson introduces one of the most important concepts in systems programming: memory.

## 5.1 A program needs somewhere to store information

A running program uses memory to store:

- Local variables.
- Function parameters.
- Return values.
- Dynamically allocated data.
- Objects and data structures.
- Runtime bookkeeping.

Two important memory regions are commonly discussed:

- The stack.
- The heap.

These are useful conceptual models, although actual program memory layout is more complicated.

## 5.2 The stack

The stack is commonly used for function call frames and local data.

Consider:

```rust
fn calculate() {
    let x = 10;
    let y = 20;

    let result = x + y;
}
```

When the function executes, it has local state associated with its invocation.

A conceptual stack frame may contain information needed to execute the function, including local values.

When the function returns, its call frame is no longer active.

The stack is efficient because allocation and deallocation can often be managed by adjusting a stack pointer.

However, do not assume every local variable physically resides on the stack. The compiler can optimize values into registers or eliminate them entirely.

## 5.3 The heap

The heap provides dynamically allocated memory.

Consider:

```rust
let data = Box::new(42);
```

`Box<T>` owns a value stored in a heap allocation.

The variable `data` contains the owning pointer-like handle to that allocation.

When the `Box` is dropped, its allocation is released.

The important distinction:

- The stack and heap describe memory-management strategies.
- Ownership describes who is responsible for a value and its resources.

These concepts are related, but they are not the same thing.

## 5.4 Why dynamic allocation exists

Imagine you want to store a collection of values, but you do not know its size until the program runs.

A fixed-size array requires its size to be known in its type:

```rust
let values: [i32; 4] = [1, 2, 3, 4];
```

A vector can grow dynamically:

```rust
let mut values: Vec<i32> = Vec::new();

values.push(10);
values.push(20);
values.push(30);
```

A `Vec<T>` conceptually manages:

- A pointer to its allocation.
- Its current length.
- Its capacity.

The length describes how many elements are initialized and part of the collection.

The capacity describes how many elements can fit in the current allocation before it needs to grow.

Do not assume the exact internal representation is a stable public guarantee. The conceptual model is what matters at this stage.

## 5.5 Memory allocation is not free

Dynamic allocation has costs:

- Finding or obtaining memory.
- Updating allocator metadata.
- Potentially moving data when a collection grows.
- Releasing memory later.
- Cache and memory-access consequences.

These costs become important when writing high-performance GPU applications.

For example, repeatedly allocating small temporary buffers inside a tight loop may be much slower than reusing an existing allocation.

### Lesson Two exercises

1. Create an array containing ten integers.
2. Create a vector containing ten integers.
3. Grow the vector and inspect its length and capacity.
4. Use `Box<T>` to allocate a value.
5. Write a function that accepts a vector and returns its length.
6. Explain why a vector is useful when the number of elements is unknown.
7. Explain why stack allocation and heap allocation are not ownership models.

# 6. Lesson Three: Ownership and Move Semantics

Ownership is the central concept of Rust.

To understand it, begin with the problem it solves.

## 6.1 The resource-management problem

Suppose two parts of a program both believe they own the same memory allocation.

When the first part finishes, it releases the allocation.

The second part still believes the allocation exists.

It may attempt to access memory that has already been released.

This is a use-after-free bug.

Another problem occurs when two parts both believe they are responsible for releasing the same allocation.

Both may release it, resulting in a double-free bug.

Traditional systems languages often allow programmers to create these situations accidentally.

Rust introduces ownership rules to prevent many such errors at compile time.

## 6.2 The ownership model

Rust follows three core rules:

1. Every value has an owner.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value is dropped.

Consider:

```rust
fn main() {
    let x = String::from("hello");

    let y = x;
}
```

What happened to `x`?

For a `String`, assigning it to `y` moves ownership.

After the move, `x` cannot be used.

Why?

Because both variables cannot independently own the same allocation under ordinary ownership rules.

This prevents the allocation from being freed twice.

## 6.3 Copy versus Move

Some types implement the `Copy` trait.

For these types, assignment copies the value instead of moving ownership.

```rust
let x = 10;
let y = x;

println!("{}", x);
println!("{}", y);
```

Both bindings remain usable.

A typical integer such as `i32` implements `Copy`.

A `String` does not.

The distinction is not simply whether the type is small or large. It is determined by the type's semantics and trait implementations.

## 6.4 Why String is different

A `String` owns a dynamically allocated UTF-8 buffer.

Conceptually, it stores:

- A pointer to the buffer.
- A length.
- A capacity.

Copying the ownership handle without managing the allocation would create multiple independent owners of the same resource.

Rust therefore moves a `String` by default.

If you explicitly want a separate owned copy, use:

```rust
let x = String::from("hello");
let y = x.clone();
```

This allocates and copies the string contents.

`clone()` can be expensive. It should represent a deliberate choice.

## 6.5 Scope and Drop

Consider:

```rust
{
    let message = String::from("hello");
}
```

When the block ends, `message` goes out of scope.

Rust runs its destructor through the `Drop` mechanism.

For a `String`, this releases its owned allocation.

This deterministic resource-management model is useful beyond memory.

The same general pattern can manage:

- File handles.
- Network sockets.
- Locks.
- Operating-system resources.
- GPU resources.

This is one reason ownership is particularly relevant to systems and GPU programming.

## 6.6 Function arguments and ownership

Consider:

```rust
fn consume(value: String) {
    println!("{}", value);
}

fn main() {
    let message = String::from("hello");

    consume(message);
}
```

The function receives ownership of the string.

After the call, the caller cannot use the original binding.

This is useful when a function is supposed to take responsibility for a resource.

But what if the function only needs to inspect the string?

That is where borrowing becomes important.

### Lesson Three exercises

1. Write a program that moves a `String` between three variables.
2. Write a function that consumes a `String`.
3. Write a function that consumes a vector.
4. Predict which variables remain usable after each move.
5. Compare copying an integer with cloning a string.
6. Explain what happens when a value goes out of scope.
7. Explain why Rust does not simply copy every value during assignment.

**Mastery question:** Can you explain ownership without mentioning the borrow checker?

# 7. Lesson Four: Borrowing and References

Ownership prevents multiple independent owners of the same resource.

But forcing every function to take ownership would make ordinary programming inconvenient.

Consider a function that only needs to print a string.

It should not need to take ownership of the string.

Rust solves this using references.

## 7.1 Immutable references

A reference lets you access a value without taking ownership.

```rust
fn print_message(message: &String) {
    println!("{}", message);
}

fn main() {
    let message = String::from("hello");

    print_message(&message);

    println!("{}", message);
}
```

The function borrows the string.

The original owner retains ownership.

The `&` operator creates a reference.

The type `&String` means an immutable reference to a `String`.

The reference does not independently own the string's allocation.

## 7.2 Why immutable borrowing matters

Rust allows multiple immutable references to the same value at the same time.

Conceptually:

```rust
let value = String::from("hello");

let a = &value;
let b = &value;
let c = &value;
```

All three references can read the value.

This is safe because none of them can modify the underlying value through an immutable reference.

## 7.3 Mutable references

Sometimes a function needs to modify a value without taking ownership.

Rust supports mutable references.

```rust
fn append_world(message: &mut String) {
    message.push_str(" world");
}

fn main() {
    let mut message = String::from("hello");

    append_world(&mut message);
}
```

The function receives permission to mutate the borrowed value.

Notice that the original binding must be mutable.

## 7.4 The aliasing rule

Rust enforces a fundamental rule:

At a given time, a value may have either:

- Any number of immutable references.
- One mutable reference.

It cannot have both an active mutable reference and other active references that violate this exclusivity.

Why?

Imagine:

1. One reference reads an array.
2. Another reference changes the array.
3. The first reference continues assuming the array has not changed.

This can create inconsistent observations and, in lower-level contexts, memory-safety problems.

Rust's borrowing rules prevent many such situations.

## 7.5 Non-lexical lifetimes

Rust does not always treat a borrow as lasting until the end of the surrounding block.

It can determine when a reference is no longer used.

For example:

```rust
let mut value = String::from("hello");

let reference = &value;

println!("{}", reference);

let mutable_reference = &mut value;

mutable_reference.push_str(" world");
```

The immutable borrow can end after its last use, allowing the mutable borrow afterward.

This is part of Rust's non-lexical lifetime analysis.

The important principle is that borrowing is about when references are used, not merely where their variables are declared.

## 7.6 Reborrowing

A mutable reference can be temporarily borrowed again.

This is called reborrowing.

```rust
fn modify(value: &mut i32) {
    *value += 1;
}

fn main() {
    let mut number = 10;

    modify(&mut number);
}
```

The function temporarily receives access through a mutable reference.

The caller retains ownership of the underlying value.

## 7.7 Dereferencing

A reference provides access to a value.

The dereference operator `*` accesses the referenced value.

```rust
let mut value = 10;

let reference = &mut value;

*reference += 5;
```

After this operation, the original value is 15.

Understand the distinction between:

- A value.
- A reference to a value.
- The operation of dereferencing a reference.

### Lesson Four exercises

1. Write a function that accepts an immutable reference to a string.
2. Write a function that accepts a mutable reference to a string.
3. Write a function that calculates the sum of a borrowed slice.
4. Create two immutable references to one value.
5. Experiment with an immutable reference followed by a mutable reference.
6. Create a program that produces a borrow-checker error. Explain the conflict.
7. Modify the program so the references do not conflict.
8. Explain why borrowing allows functions to inspect data without taking ownership.

# 8. Lesson Five: Slices, Strings, and Collections

You will work with buffers constantly in GPU programming.

A GPU kernel often processes a region of memory containing many values.

Rust slices provide a useful abstraction for this.

## 8.1 What is a slice?

A slice represents a contiguous region of elements.

For example:

```rust
let values = [10, 20, 30, 40, 50];

let subset = &values[1..4];
```

The slice contains references to the elements:

```text
20, 30, 40
```

A slice does not own the underlying data.

Conceptually, a slice consists of:

- A pointer to the first element.
- A length.

A slice is a dynamically sized type, so it is normally used behind a reference such as `&[i32]`.

## 8.2 Why slices are useful

Consider this function:

```rust
fn sum(values: &[i32]) -> i32 {
    values.iter().sum()
}
```

It can accept a borrowed region from an array or a vector.

This separates the function's requirements from the container that owns the data.

The function only needs a contiguous sequence of integers.

It does not need to know whether the caller used an array or a vector.

## 8.3 String and str

Rust has two important string types:

- `String`: an owned, growable UTF-8 string.
- `str`: a string slice, normally accessed through `&str`.

For example:

```rust
let owned = String::from("hello");

let borrowed: &str = &owned;
```

A `String` owns its buffer.

A `&str` borrows a region of UTF-8 data.

## 8.4 UTF-8 and indexing

A common mistake is assuming strings are arrays of characters.

Rust strings are UTF-8 encoded.

A character may occupy multiple bytes.

For example, the ASCII character `A` occupies one byte, while many other Unicode characters occupy multiple bytes.

This is why Rust does not allow ordinary integer indexing into a string.

You cannot assume that the fifth byte is the fifth character.

Understand these three concepts:

- Bytes.
- Unicode scalar values, represented by `char`.
- Grapheme clusters, which are closer to user-perceived characters.

They are not interchangeable.

## 8.5 Vectors

A vector owns a dynamically allocated contiguous collection.

```rust
let mut values = Vec::new();

values.push(10);
values.push(20);
values.push(30);
```

Important operations:

```rust
values.len();
values.capacity();
values.push(40);
values.pop();
values.clear();
```

Study the difference between length and capacity.

Also investigate what happens when the vector runs out of capacity.

### Lesson Five exercises

1. Write a function accepting `&[u32]`.
2. Calculate the average of a slice.
3. Find the maximum value in a slice.
4. Convert a vector into a borrowed slice.
5. Write a function accepting `&str`.
6. Inspect the UTF-8 bytes of a string.
7. Compare a string's byte length with its character count.
8. Implement a function that processes a buffer without taking ownership.

# 9. Lesson Six: Structs, Enums, and Pattern Matching

Rust's type system lets you describe the structure of your data.

This is important because systems programming frequently involves representing states and resources precisely.

## 9.1 Structs

A struct groups related fields.

```rust
struct Point {
    x: f64,
    y: f64,
}
```

Create an instance:

```rust
let point = Point {
    x: 1.0,
    y: 2.0,
};
```

Structs can have methods through `impl` blocks.

```rust
impl Point {
    fn magnitude_squared(&self) -> f64 {
        self.x * self.x + self.y * self.y
    }
}
```

Notice the `&self`.

The method borrows the instance immutably.

You will also encounter:

- `&mut self`
- `self`

These represent different ownership and access requirements.

## 9.2 Enums

An enum represents one of several possible variants.

```rust
enum DeviceState {
    Initializing,
    Ready,
    Running,
    Failed,
}
```

A variable of type `DeviceState` can hold one of these variants.

Enums are useful for representing states that should not be confused with one another.

For example, a GPU buffer might be:

- Unallocated.
- Allocated.
- Mapped.
- Released.

Representing these states explicitly can make invalid operations harder to express.

## 9.3 Pattern matching

Rust uses `match` to inspect values.

```rust
match state {
    DeviceState::Initializing => {
        println!("Initializing");
    }
    DeviceState::Ready => {
        println!("Ready");
    }
    DeviceState::Running => {
        println!("Running");
    }
    DeviceState::Failed => {
        println!("Failed");
    }
}
```

The compiler checks that all possible variants are handled.

This exhaustiveness checking is extremely useful when building systems with complex state transitions.

## 9.4 Option

Sometimes a value may or may not exist.

Rust represents this using:

```rust
Option<T>
```

It has two variants:

- `Some(T)`
- `None`

For example:

```rust
let value: Option<i32> = Some(10);
```

This avoids using arbitrary sentinel values such as `-1` to represent the absence of a result.

## 9.5 Result

Operations can fail.

Rust represents success or failure using:

```rust
Result<T, E>
```

It has two variants:

- `Ok(T)`
- `Err(E)`

For example, parsing an integer from a string can fail.

```rust
let result = "123".parse::<i32>();
```

You should understand how to handle both outcomes rather than ignoring errors.

### Lesson Six exercises

1. Create a struct representing a GPU buffer.
2. Add methods to inspect its length and capacity.
3. Create an enum representing the buffer's state.
4. Use `match` to handle every possible state.
5. Write a function returning `Option<i32>`.
6. Write a function returning `Result<i32, String>`.
7. Design an enum that represents a file's possible states.
8. Explain why explicit states are safer than loosely related Boolean variables.

# 10. Lesson Seven: Generics, Traits, and Abstraction

Rust allows code to work with multiple types without duplicating its implementation.

This is particularly important when building reusable data structures and GPU abstractions.

## 10.1 Generics

Consider:

```rust
struct Container<T> {
    value: T,
}
```

`T` is a generic type parameter.

The same struct can hold different types:

```rust
let integer = Container { value: 10 };
let text = Container {
    value: String::from("hello"),
};
```

Generics let you describe reusable structures while preserving type safety.

## 10.2 Traits

A trait describes behavior that a type can implement.

```rust
trait Describe {
    fn describe(&self) -> String;
}
```

A type can implement the trait:

```rust
struct Buffer {
    size: usize,
}

impl Describe for Buffer {
    fn describe(&self) -> String {
        format!("Buffer size: {}", self.size)
    }
}
```

A trait is not simply inheritance.

It defines a set of behaviors that types agree to provide.

## 10.3 Trait bounds

You can constrain generic types using traits.

```rust
fn print_description<T: Describe>(value: &T) {
    println!("{}", value.describe());
}
```

This function accepts any type that implements `Describe`.

The compiler verifies that the required behavior exists.

## 10.4 Static dispatch and monomorphization

When a generic function is compiled, Rust commonly generates specialized versions for the concrete types used.

This process is called monomorphization.

It often allows generic code to be optimized efficiently.

However, generating many specialized versions can increase executable size.

## 10.5 Dynamic dispatch

Traits can also be used through trait objects.

```rust
fn print_description(value: &dyn Describe) {
    println!("{}", value.describe());
}
```

This uses dynamic dispatch.

The concrete implementation is selected at runtime through a virtual method table.

The trade-off is that dynamic dispatch introduces indirection and may restrict certain optimizations.

Understand when to use:

- Generic type parameters.
- `impl Trait`.
- `dyn Trait`.

### Lesson Seven exercises

1. Create a generic container.
2. Create a generic function that accepts a slice.
3. Define a trait for objects that report their memory size.
4. Implement that trait for two different structs.
5. Write a function with a trait bound.
6. Compare a generic function with a function accepting `&dyn Trait`.
7. Explain monomorphization.
8. Explain the difference between static and dynamic dispatch.

# 11. Lesson Eight: Lifetimes and the Borrow Checker

Lifetimes are often one of the most confusing parts of Rust.

The most important thing to understand is that a lifetime describes how long a reference is valid.

It does not extend the lifetime of the underlying data.

## 11.1 The dangling reference problem

Consider the following conceptual situation:

1. A function creates a local string.
2. The function returns a reference to that string.
3. The local string is destroyed when the function returns.
4. The caller attempts to use the reference.

The reference would point to data that no longer exists.

This is a dangling reference.

Rust prevents this.

## 11.2 Lifetime annotations

Consider:

```rust
fn longer<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

The lifetime parameter `'a` describes the relationship between the input references and returned reference.

The function promises that the returned reference is valid for the lifetime represented by `'a`.

It does not promise that the inputs live longer than they actually do.

## 11.3 Why lifetime annotations exist

A compiler needs to establish that a returned reference cannot outlive the data it refers to.

Lifetime annotations help express relationships that cannot always be inferred from the function signature alone.

They are not arbitrary syntax required whenever a reference appears.

Rust has lifetime elision rules that allow many common signatures to omit explicit annotations.

## 11.4 Lifetimes in structs

A struct can contain references.

```rust
struct TextView<'a> {
    text: &'a str,
}
```

The struct's lifetime parameter describes how long its borrowed text must remain valid.

This matters when creating views into buffers or other data structures.

## 11.5 The borrow checker as a reasoning tool

When you receive a lifetime error, ask:

1. Which value owns the underlying data?
2. Which reference points to it?
3. When does the owner become invalid?
4. How long is the reference being used?
5. Can the design avoid returning a reference?
6. Should the function return owned data instead?

Do not immediately add lifetime annotations until you understand the relationship being expressed.

### Lesson Eight exercises

1. Create a function that returns a borrowed string.
2. Create a struct containing a string slice.
3. Experiment with returning a reference to a local variable.
4. Explain why the compiler rejects it.
5. Implement a function that chooses between two string slices.
6. Compare returning `String` with returning `&str`.
7. Explain why a lifetime annotation cannot make a destroyed value valid.
8. Draw a lifetime diagram for a function with two borrowed inputs and one borrowed output.

# 12. Lesson Nine: Memory Layout and Pointers

This is one of the most important lessons for GPU engineering.

GPU programming involves passing data through buffers and memory regions. You need to understand what data looks like in memory before you can reason about data transfer or kernel access.

## 12.1 Values have a representation

Consider:

```rust
struct Example {
    a: u8,
    b: u64,
    c: u8,
}
```

It is tempting to assume the fields occupy exactly ten bytes.

But the compiler may insert padding to satisfy alignment requirements.

Memory layout is not necessarily identical to the sum of field sizes.

## 12.2 Alignment

A type may require its address to satisfy a particular alignment.

For example, a 64-bit integer commonly has an alignment requirement greater than that of an 8-bit integer.

Padding may be inserted so that fields begin at suitably aligned addresses.

Alignment matters because hardware and compilers may rely on it for efficient or valid memory access.

## 12.3 Inspecting memory layout

Rust provides:

```rust
std::mem::size_of::<T>()
std::mem::align_of::<T>()
```

Use these to inspect types.

Example:

```rust
use std::mem::{align_of, size_of};

fn main() {
    println!("size: {}", size_of::<u64>());
    println!("alignment: {}", align_of::<u64>());
}
```

Experiment with structs containing fields of different sizes.

Do not assume a particular field order or layout for ordinary Rust structs.

## 12.4 repr(C)

Rust normally uses a layout chosen by the compiler.

For interoperability with C, you can use:

```rust
#[repr(C)]
struct Vertex {
    x: f32,
    y: f32,
    z: f32,
}
```

`repr(C)` requests a C-compatible layout for the type.

This is useful for FFI and certain binary interfaces.

It does not automatically guarantee compatibility with every GPU buffer layout. GPU APIs and shader languages may impose additional alignment and layout rules.

## 12.5 Pointers

A pointer represents an address.

Rust has several pointer-related types:

| Type | Purpose |
|---|---|
| `&T` | Shared reference |
| `&mut T` | Mutable reference |
| `*const T` | Raw immutable pointer |
| `*mut T` | Raw mutable pointer |
| `Box<T>` | Owned heap allocation |
| `NonNull<T>` | Non-null raw pointer abstraction |

References have safety guarantees that raw pointers do not.

Raw pointers can be null, dangling, or misaligned.

Dereferencing raw pointers requires an unsafe context.

## 12.6 Pointer arithmetic

Systems programs sometimes need to operate directly on memory addresses.

This requires careful reasoning about:

- Allocation boundaries.
- Element sizes.
- Alignment.
- Valid initialization.
- Pointer provenance.
- Aliasing.
- Lifetimes.

Do not begin writing unsafe pointer arithmetic until you understand each of these concepts.

## 12.7 Byte representation

The following concepts are essential:

- A byte is eight bits.
- Integer values have a binary representation.
- Multi-byte values have an endianness.
- Structs may contain padding.
- Padding bytes should not automatically be treated as meaningful initialized data.

When working with binary formats, use explicit serialization and parsing rather than assuming the in-memory representation is the file representation.

### Lesson Nine exercises

1. Inspect the size and alignment of primitive types.
2. Create three structs with different field orders.
3. Compare their sizes and alignments.
4. Create a `repr(C)` struct.
5. Convert an integer into its little-endian byte representation.
6. Reconstruct the integer from those bytes.
7. Explain why a Rust struct's memory layout is not automatically a portable binary format.
8. Explain why GPU buffer alignment matters.

# 13. Lesson Ten: Smart Pointers and Interior Mutability

Rust's ownership model is strict, but it also provides abstractions for situations where ownership needs to be shared.

## 13.1 Box

`Box<T>` provides unique ownership of a heap allocation.

Use it when you need an owned value allocated on the heap.

Examples include:

- Recursive data structures.
- Large values that should not be stored inline.
- Explicit heap allocation.

## 13.2 Rc

`Rc<T>` provides reference-counted shared ownership within a single thread.

Multiple `Rc` handles can point to the same allocation.

When the final handle is dropped, the allocation can be released.

`Rc` is not designed for thread-safe shared ownership.

## 13.3 Arc

`Arc<T>` provides atomically reference-counted shared ownership.

It can be used across threads.

However, `Arc<T>` does not automatically make the contents mutable or thread-safe.

It only makes shared ownership management thread-safe.

## 13.4 RefCell

`RefCell<T>` provides interior mutability through runtime borrowing checks.

Instead of enforcing all borrowing rules at compile time, it checks borrowing at runtime.

If the borrowing rules are violated, the program can panic.

## 13.5 Mutex

`Mutex<T>` provides mutual exclusion.

It allows threads to coordinate access to shared mutable state.

Only one thread can hold the lock at a time.

## 13.6 Choosing the right abstraction

| Abstraction | Ownership model | Thread-safe? |
|---|---|---|
| `Box<T>` | Unique ownership | Depends on `T` |
| `Rc<T>` | Shared ownership | No |
| `Arc<T>` | Shared ownership | Reference counting is thread-safe |
| `RefCell<T>` | Interior mutability | No |
| `Mutex<T>` | Locked shared access | Yes, when its type requirements are met |
| `RwLock<T>` | Multiple readers or one writer | Yes, when its type requirements are met |

Do not choose a smart pointer because it makes the compiler error disappear.

Choose it because it accurately represents the ownership and synchronization requirements of your program.

### Lesson Ten exercises

1. Build a recursive linked structure using `Box`.
2. Create multiple `Rc` handles to one value.
3. Observe how reference counting changes as handles are cloned and dropped.
4. Use `RefCell` to mutate data through an immutable binding.
5. Trigger and understand a runtime borrow panic.
6. Create shared state using `Arc<Mutex<T>>`.
7. Explain why `Arc<T>` alone does not provide unrestricted mutable access.

# 14. Lesson Eleven: Modules, Errors, Testing, and Tooling

Systems programming requires more than writing functions.

You need to organize, test, and debug software reliably.

## 14.1 Modules

Rust modules organize code and control visibility.

You should understand:

- `mod`
- `pub`
- `use`
- Module paths.
- Crates.
- Packages.
- Library crates.
- Binary crates.

Practice splitting a project into multiple modules.

## 14.2 Error handling

Rust distinguishes between recoverable and unrecoverable errors.

A panic represents an unrecoverable failure in the current execution context.

`Result<T, E>` represents an operation that may fail and whose failure can be handled.

The `?` operator propagates errors to the caller.

You should understand what it does conceptually rather than treating it as syntax magic.

## 14.3 Testing

Rust includes a built-in testing framework.

```rust
#[test]
fn test_addition() {
    assert_eq!(2 + 2, 4);
}
```

You should write tests for:

- Normal inputs.
- Empty inputs.
- Boundary values.
- Invalid inputs.
- Overflow conditions.
- Unexpected state transitions.

Also learn to use integration tests and documentation tests.

## 14.4 Debugging

Become comfortable with:

- Compiler diagnostics.
- `dbg!`.
- `println!`.
- `cargo test`.
- `cargo clippy`.
- Debugger breakpoints.
- Inspecting variable state.
- Reading backtraces.

Learn to use a debugger such as GDB or LLDB with Rust programs.

Do not treat compiler messages as instructions to copy blindly. Interpret what invariant or type relationship the compiler is complaining about.

### Lesson Eleven exercises

1. Split an application into three modules.
2. Create a library crate and a binary that uses it.
3. Write a function that returns a `Result`.
4. Propagate an error using `?`.
5. Write unit tests for normal and boundary cases.
6. Create an integration test.
7. Introduce a bug and debug it using a debugger.

# 15. Lesson Twelve: Concurrency Fundamentals

GPU programming is fundamentally parallel.

Before studying GPU execution models, you need a solid understanding of concurrent execution on CPUs.

## 15.1 Concurrency versus parallelism

Concurrency means multiple tasks can make progress during overlapping periods.

Parallelism means multiple tasks are executing simultaneously.

A program can be concurrent without executing on multiple CPU cores.

This distinction matters when reasoning about threads, scheduling, and synchronization.

## 15.2 Threads

Rust provides threads through the standard library.

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        println!("Hello from another thread");
    });

    handle.join().unwrap();
}
```

The spawned thread executes independently of the calling thread.

`join()` waits for it to finish.

Study what happens when the thread captures data from its surrounding environment.

## 15.3 Send and Sync

Two important Rust traits are:

- `Send`: a value can be transferred between threads.
- `Sync`: a value can be safely shared between threads through references.

These traits help Rust enforce thread-safety guarantees.

Understand why some types implement these traits and others do not.

## 15.4 Shared state

Multiple threads sometimes need to access shared data.

A mutex provides exclusive access:

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
```

You should understand the purpose of each abstraction:

- `Arc` allows shared ownership.
- `Mutex` coordinates access.
- The protected value is only accessed while holding the lock.

## 15.5 Channels

Channels allow threads to communicate by sending messages.

This can reduce the need for shared mutable state.

Study:

- Sending.
- Receiving.
- Blocking.
- Ownership transfer through messages.
- Producer-consumer patterns.

## 15.6 Data races and deadlocks

A data race occurs when conflicting memory accesses happen without appropriate synchronization.

A deadlock occurs when tasks wait indefinitely for resources held by one another.

Both are important concurrency hazards.

Rust's type system prevents many data races in safe Rust, but it does not prevent every concurrency bug.

Logical races and deadlocks are still possible.

## 15.7 Atomics

Atomic operations provide synchronization primitives that can be used without ordinary mutex locking.

Start with:

- Atomic loads.
- Atomic stores.
- Atomic increments.
- Compare-and-swap.
- Memory ordering.

Do not attempt to memorize memory-ordering rules.

First understand why CPUs and compilers reorder operations and why synchronization is needed.

Detailed atomic algorithms belong in the later concurrency phase.

### Lesson Twelve exercises

1. Spawn two threads.
2. Make them calculate separate results.
3. Join both threads.
4. Share a counter using `Arc<Mutex<T>>`.
5. Implement a producer-consumer program using channels.
6. Explain the difference between concurrency and parallelism.
7. Explain `Send` and `Sync`.
8. Describe a deadlock scenario and how you might avoid it.

# 16. Lesson Thirteen: Unsafe Rust and FFI

Unsafe Rust is necessary for some low-level operations.

However, unsafe does not mean that Rust stops being a systems language or that the compiler stops checking everything.

It means certain guarantees are now the programmer's responsibility.

## 16.1 What unsafe permits

An unsafe block can permit operations such as:

- Dereferencing raw pointers.
- Calling unsafe functions.
- Accessing or modifying mutable static state under applicable rules.
- Implementing unsafe traits.
- Accessing union fields.

The compiler still performs type checking and other ordinary checks.

## 16.2 Unsafe does not mean unchecked

Consider:

```rust
unsafe {
    // An unsafe operation may be performed here.
}
```

The block tells the compiler that the programmer is taking responsibility for certain safety conditions.

It does not prove those conditions are satisfied.

## 16.3 Safety invariants

An invariant is a condition that must remain true for an abstraction to be correct.

For a custom buffer, invariants might include:

- The pointer refers to a valid allocation.
- The allocation is sufficiently aligned.
- The length never exceeds capacity.
- Every initialized element is valid.
- No two mutable references violate aliasing requirements.
- The allocation is released exactly once.

Every unsafe abstraction needs clearly documented invariants.

## 16.4 Foreign Function Interfaces

An FFI allows Rust to interact with code written in other languages, commonly C.

You should understand:

- `extern "C"`.
- `#[repr(C)]`.
- Raw pointers.
- Ownership across language boundaries.
- Calling conventions.
- Error handling across FFI.
- Who allocates and who frees resources.

GPU libraries often expose native APIs that require interaction with C-compatible interfaces.

That makes FFI a foundational skill.

### Lesson Thirteen exercises

1. Read a simple unsafe Rust example and identify the unsafe operation.
2. Identify the safety conditions required by that operation.
3. Create a `repr(C)` struct.
4. Explain how Rust could call a C function.
5. Write a safety-invariant document for a hypothetical raw buffer.
6. Explain why an unsafe block does not automatically make a program correct.

**Important:** Do not build a custom allocator or complex raw-pointer collection until you can explain the invariants that make it safe.

# 17. AI Tutor and Assessment Agent Prompts

These prompts are intentionally strict.

The objective is to make the AI behave like an instructor who helps you develop your own reasoning instead of acting as a developer who completes tasks.

Use the relevant prompt as the agent's system or instruction prompt when creating a dedicated study agent.

## 17.1 Universal Socratic Tutor Rules

These rules apply to both quizzes and coding assessments.

**Copy this instruction into your agent:**

You are my Socratic systems-programming tutor. Your purpose is to teach me through guided reasoning, not to solve problems for me.

I am independently learning Rust systems programming and GPU engineering. I want to develop the ability to reason from first principles, debug my own programs, and understand why systems behave as they do.

### Absolute restrictions

1. NEVER give me the answer to a question.
2. NEVER write code for me.
3. NEVER complete an exercise, assessment, or project on my behalf.
4. NEVER modify, create, or edit files for me.
5. NEVER execute commands, run tests, compile programs, or operate tools on my behalf.
6. NEVER provide a complete solution, pseudocode solution, or implementation outline that effectively solves the task.
7. NEVER reveal hidden test cases or assessment answers.
8. NEVER give away the next step in a way that removes the reasoning I am supposed to perform.
9. NEVER turn a hint into a disguised answer.
10. NEVER abandon the Socratic approach simply because I ask you to give me the answer.

### Required teaching behavior

- Ask guiding questions that help me discover the answer.
- Ask only one primary question at a time.
- Wait for my response before asking the next question.
- Base each follow-up question on my previous answer.
- Start with the most fundamental concept involved.
- Help me identify assumptions, invariants, constraints, and edge cases.
- If my reasoning is incorrect, ask a question that exposes the contradiction.
- If I am stuck, make the question progressively more specific without revealing the solution.
- Encourage me to predict program behavior before running it.
- Ask me to explain why something works rather than merely whether it works.
- Require me to defend my design choices.
- Ask me to identify what evidence would confirm or disprove my hypothesis.

### Response format

Your responses must consist exclusively of guiding questions.

Do not include declarative explanations, answers, code, pseudocode, completed commands, or solution summaries.

You may briefly identify the topic using a question, but every response must remain question-based.

Ask one primary question per response. Avoid overwhelming me with multiple questions at once.

### When I request an answer

If I say "just give me the answer," "show me the code," or otherwise request that you solve the task, do not comply.

Instead, ask a question that helps me identify what I am struggling to understand.

### When I provide code

Do not rewrite, fix, or complete my code.

Ask questions that help me inspect:

- What I expected the code to do.
- What it actually does.
- Which assumption might be incorrect.
- Which ownership or borrowing rule might apply.
- Which invariant may be violated.
- Which test could distinguish between competing explanations.

### Teaching objective

Your success is measured by whether I can independently explain and solve the problem afterward, not by how quickly you can produce a solution.

You are a tutor, not an answer engine.

---

## 17.2 Coding Assessment Agent Prompt

Use this prompt when you want the agent to administer coding exercises, review your reasoning, or guide you through a project.

**Copy the following prompt:**

You are my Rust systems programming coding-assessment instructor.

Your job is to administer coding assessments and guide me through them using the Socratic method.

I am studying Rust from first principles and eventually intend to become a professional GPU software engineer.

### Assessment procedure

1. Present one coding assessment at a time.
2. Explain the objective, constraints, expected behavior, and evaluation criteria without providing an implementation.
3. Ask me to explain my initial approach before I start coding.
4. Ask me to identify the relevant types, ownership relationships, memory requirements, and invariants.
5. Let me write the implementation independently.
6. When I provide code, ask questions that help me evaluate it.
7. Ask me to predict its behavior on ordinary and edge-case inputs.
8. Ask me to design tests independently.
9. Ask me to explain compiler errors and runtime failures.
10. Continue questioning until I can independently justify the implementation.

### Strict restrictions

- NEVER write code for me.
- NEVER modify my code.
- NEVER run my code.
- NEVER execute terminal commands.
- NEVER create files.
- NEVER generate tests for me.
- NEVER provide a complete implementation.
- NEVER provide pseudocode that directly translates into the solution.
- NEVER provide the answer to a coding question.
- NEVER reveal hidden test cases.
- NEVER fix a bug for me.
- NEVER complete a project or assessment on my behalf.

You may ask me what tests I would write, what command I would run, or what part of the code I would inspect. I must perform those actions myself.

### Code review behavior

When I submit code:

- Do not rewrite it.
- Do not provide corrected code.
- Do not identify the exact replacement expression.
- Ask me questions about the relevant behavior.
- Ask me to trace execution.
- Ask me to inspect ownership and borrowing.
- Ask me to reason about allocation and memory layout.
- Ask me to consider overflow, invalid inputs, and resource cleanup.
- Ask me to justify any unsafe code and its safety invariants.

If my implementation is incorrect, guide me toward discovering the problem through questions.

### Difficulty adaptation

If I demonstrate strong understanding, increase the complexity of the next assessment.

If I struggle, reduce the problem into smaller conceptual questions without giving away the implementation.

Do not mistake a successful compilation for mastery.

### Assessment completion

An assessment is only complete when I can independently:

- Explain the design.
- Explain the relevant Rust concepts.
- Defend the ownership model.
- Identify important edge cases.
- Explain why the implementation is correct.
- Discuss its performance characteristics.
- Identify its limitations.

At the end, ask me to provide my own assessment of what I learned.

### Response restriction

Every response must consist exclusively of guiding questions.

Do not provide declarative explanations, code, pseudocode, answers, or completed work.

You are an instructor, not an implementation assistant.

---

## 17.3 Quiz Generation and Administration Agent Prompt

Use this prompt whenever you want a new quiz, practice exam, or conceptual assessment.

**Copy the following prompt:**

You are my Rust systems programming examination instructor.

Your purpose is to test my understanding and teach me through guided questioning.

Generate original quizzes based on the study material I provide or the phase and topic I specify.

### Quiz generation rules

When generating a quiz:

1. Ask one question at a time.
2. Do not reveal the answer.
3. Do not include an answer key.
4. Do not show explanations before I attempt the question.
5. Do not reveal hints unless I request help.
6. Vary the question formats.
7. Include conceptual questions, code tracing, debugging scenarios, memory reasoning, and design questions.
8. Include questions that test first-principles understanding rather than memorization.
9. Gradually increase difficulty.
10. Include questions that connect multiple concepts.

### Question types

Generate questions involving:

- Conceptual reasoning.
- Predicting compiler behavior.
- Predicting program output.
- Ownership and move semantics.
- Borrowing and lifetimes.
- Memory layout and alignment.
- Type system reasoning.
- Error handling.
- Generics and traits.
- Concurrency.
- Unsafe Rust.
- Performance trade-offs.
- Debugging and code review.

### Administration rules

- Ask one question and wait for my response.
- Evaluate my reasoning internally, but do not reveal the correct answer.
- If my response is incorrect or incomplete, ask a guiding question.
- If I am close, ask a question that helps me refine my reasoning.
- If I am completely stuck, break the concept into smaller questions.
- Do not move to the next question until I demonstrate sufficient understanding of the current one.
- Do not reveal the answer even after several failed attempts.
- Track topics where my reasoning is weak and revisit them later.

### Difficulty levels

Use these levels:

**Level 1 — Recognition:** Identify concepts and terminology.

**Level 2 — Understanding:** Explain why a concept exists.

**Level 3 — Application:** Apply a concept to a new example.

**Level 4 — Analysis:** Predict behavior and identify subtle problems.

**Level 5 — Design:** Choose an appropriate design and defend the trade-offs.

**Level 6 — Engineering judgment:** Evaluate correctness, performance, safety, and maintainability together.

### Final restriction

Your responses must consist exclusively of guiding questions.

Never provide answers, explanations, code, or completed solutions.

Your objective is to make me capable of answering the questions independently.

---

## 17.4 Reusable Quiz Generation Requests

Once you have configured the agent, you can give it requests like these:

**Conceptual quiz**

Create a 20-question Level 2–4 quiz on Rust ownership, borrowing, and move semantics. Focus on first-principles reasoning and misconceptions. Administer one question at a time. Never reveal answers.

**Code-tracing quiz**

Create a code-tracing assessment involving ownership, scopes, references, and `Drop`. Ask me to predict compiler behavior and program output. Do not show the expected output.

**Memory quiz**

Create a difficult quiz about stack and heap memory, pointers, alignment, `Vec<T>`, `Box<T>`, and memory layout. Include scenarios where I must reason about actual memory representation.

**Borrow-checker quiz**

Create 15 progressively difficult ownership and borrowing scenarios. Ask me to determine which operations are valid and explain why. Do not identify the correct answer.

**Unsafe Rust quiz**

Create a Level 5 assessment about raw pointers, aliasing, alignment, initialization, pointer validity, and safety invariants. Require me to reason about what makes an unsafe operation sound.

**Cumulative examination**

Create a comprehensive Phase 1 examination that tests every major topic. Include conceptual questions, code tracing, design scenarios, and debugging problems. Ask one question at a time and do not reveal answers.

**Remedial quiz**

Identify the concepts I have struggled with in previous questions. Generate a new quiz that tests those concepts using different examples. Do not repeat the same questions.

# 18. Practical Coding Assessments

These assessments are intended to demonstrate that you can apply the concepts from the lessons.

Do not start an assessment until you have studied its prerequisite lessons.

## Assessment A: Memory Inspector

**Prerequisites:** Lessons 1–5 and 9.

Build a command-line program that inspects the representation of primitive values and collections.

Your program should eventually support:

- Inspecting integer sizes.
- Inspecting alignment.
- Displaying integer byte representations.
- Displaying endianness.
- Inspecting array and vector lengths.
- Inspecting vector capacity.
- Comparing the sizes of different structs.
- Demonstrating the difference between owned and borrowed data.

### Constraints

- Use safe Rust initially.
- Do not use unsafe pointer arithmetic.
- Organize the project into modules.
- Include automated tests.
- Handle invalid user input.

### Questions you must be able to answer

- What is the difference between a type's size and its alignment?
- Why can two structs with identical fields have different sizes?
- What does vector capacity represent?
- Why is an in-memory struct not automatically a portable file format?
- Which operations allocate memory?

**Deliverable:** A working memory-inspection CLI and a written explanation of its behavior.

## Assessment B: Byte Buffer and Parser

**Prerequisites:** Lessons 3–9 and 11.

Build a small byte-buffer abstraction that can parse structured binary data.

Your program should support:

- Owning a byte buffer.
- Reading fixed-width integers.
- Handling little-endian and big-endian values.
- Checking whether requested bytes exist.
- Tracking a cursor position.
- Reporting invalid input.
- Returning structured parsing errors.

### Constraints

- Prefer slices and safe indexing.
- Avoid unnecessary cloning.
- Avoid unsafe code.
- Handle truncated and malformed inputs.
- Test boundary conditions.

### Engineering questions

- Who owns the underlying buffer?
- Does the parser need ownership or a borrow?
- What happens if the cursor advances past the buffer?
- How should invalid input be represented?
- When would returning a borrowed slice be better than allocating a new vector?
- What invariants must the parser preserve?

**Deliverable:** A tested binary parser with documented invariants and error behavior.

## Assessment C: Concurrent Work Queue

**Prerequisites:** Lessons 10–12.

Build a small concurrent work queue.

The queue should eventually support:

- Multiple producers.
- Multiple consumers.
- Shared work distribution.
- Graceful shutdown.
- Error handling.
- Tests involving multiple threads.

Start with standard-library synchronization primitives.

Do not attempt lock-free programming at this stage.

### Engineering questions

- Which thread owns each task?
- What state is shared?
- Which operations require synchronization?
- What happens if a consumer exits unexpectedly?
- How do you prevent deadlock?
- How should shutdown be communicated?
- What does `Send` guarantee?
- What does `Sync` guarantee?

**Deliverable:** A concurrent queue with a written explanation of its ownership and synchronization model.

## Assessment D: Safe Rust Data Structure

**Prerequisites:** Lessons 3–10 and 13.

Build a simplified vector-like data structure using safe Rust.

It should support:

- Adding elements.
- Removing elements.
- Inspecting length.
- Inspecting capacity.
- Iterating over elements.
- Clearing its contents.

Use an ordinary `Vec<T>` internally at first.

The objective is to design a clean abstraction and understand the behavior of the underlying collection.

Do not implement a custom allocator or raw-pointer storage yet.

**Deliverable:** A reusable generic collection abstraction with tests and documentation.

# 19. Phase 1 Quiz Bank

These questions are intended for independent practice. They do not include answers.

Try answering each question in writing before discussing it with your Socratic tutor.

## Section A: Rust Fundamentals

1. What is the difference between a statement and an expression?
2. Why is Rust considered statically typed?
3. What does type inference accomplish?
4. What is the difference between a mutable variable binding and a mutable reference?
5. What happens during compilation?
6. Why does a program need a linker?
7. What is the difference between `cargo check` and `cargo build`?

## Section B: Memory and Ownership

8. What is the conceptual difference between stack and heap allocation?
9. Why does Rust use ownership?
10. What is a move?
11. Why does `String` behave differently from `i32` during assignment?
12. What happens when a value goes out of scope?
13. What is the purpose of `Drop`?
14. Why can cloning be expensive?
15. Why is ownership important for managing GPU resources?

## Section C: Borrowing

16. Why can Rust allow multiple immutable references?
17. Why does Rust restrict mutable references?
18. What is a dangling reference?
19. What does dereferencing accomplish?
20. What is a slice?
21. Why does a function accepting `&[T]` offer more flexibility than one accepting `Vec<T>`?
22. What is a non-lexical lifetime?
23. Why can an immutable borrow sometimes end before the end of its surrounding block?

## Section D: Types and Abstractions

24. What problem do enums solve?
25. Why is exhaustive pattern matching useful?
26. What is the difference between `Option<T>` and `Result<T, E>`?
27. What is a trait?
28. What is monomorphization?
29. What is dynamic dispatch?
30. When might a trait object be preferable to a generic parameter?

## Section E: Memory Representation

31. What is alignment?
32. Why does padding exist?
33. What does `repr(C)` guarantee?
34. Why is a reference different from a raw pointer?
35. What is endianness?
36. Why should you not assume a Rust struct is a portable binary format?
37. What are the length and capacity of a vector?

## Section F: Concurrency and Unsafe Rust

38. What is the difference between concurrency and parallelism?
39. What does `Send` represent?
40. What does `Sync` represent?
41. What problem does a mutex solve?
42. What is a data race?
43. What is a deadlock?
44. Why does `Arc<T>` not automatically make `T` mutable?
45. What does an unsafe block permit?
46. Why must unsafe abstractions document their invariants?
47. Why is a program that compiles not necessarily a correct program?

# 20. Phase 1 Final Examination

Complete this examination after finishing the lessons and coding assessments.

**Recommended duration:** 3–4 hours.

| Section | Weight |
|---|---:|
| Rust language fundamentals | 10% |
| Ownership and borrowing | 20% |
| Memory and data representation | 20% |
| Types and abstractions | 10% |
| Concurrency | 15% |
| Unsafe Rust and FFI | 10% |
| Practical coding assessment | 15% |

### Examination requirements

For each conceptual question:

- Explain the underlying problem.
- Explain how Rust addresses it.
- Give an example.
- Describe a situation where the concept matters.

For code-tracing questions:

- Predict compiler behavior.
- Explain ownership transitions.
- Identify the lifetime of relevant references.
- Explain any allocation or deallocation.
- Justify your conclusion.

For the practical portion:

- Build one of the assessments independently.
- Write tests.
- Document design decisions.
- Explain the memory and ownership model.
- Discuss limitations and performance trade-offs.

Do not use compilation success as the sole measure of correctness.

# 21. Mastery Criteria

Use this checklist to decide whether you are ready to move on.

## Language fundamentals

- [ ] I can explain how Rust code becomes an executable.
- [ ] I understand Rust's type system and type inference.
- [ ] I understand expressions, functions, and control flow.
- [ ] I can organize code using modules and crates.

## Ownership and memory

- [ ] I can explain ownership from first principles.
- [ ] I understand moves and copies.
- [ ] I understand borrowing and aliasing restrictions.
- [ ] I can reason about lifetimes.
- [ ] I understand stack and heap allocation.
- [ ] I can inspect size and alignment.
- [ ] I understand pointers and slices.

## Abstractions

- [ ] I can create structs and enums.
- [ ] I understand `Option` and `Result`.
- [ ] I can use generics and traits.
- [ ] I understand static and dynamic dispatch.
- [ ] I understand `Box`, `Rc`, `Arc`, and `Mutex`.

## Systems programming

- [ ] I can write a multithreaded program.
- [ ] I understand `Send` and `Sync`.
- [ ] I understand basic synchronization.
- [ ] I can explain unsafe Rust's responsibilities.
- [ ] I can describe FFI and C-compatible layouts.
- [ ] I can debug and test a Rust application.

## Independent reasoning

- [ ] I can explain compiler errors without relying on copied fixes.
- [ ] I can identify relevant invariants.
- [ ] I can predict program behavior before running it.
- [ ] I can defend my implementation choices.
- [ ] I can complete the practical assessments without having the AI write code or solutions for me.

You should be able to explain every checked item without reading directly from this document.

# 22. How This Phase Connects to GPU Programming

Everything in this phase prepares you for the GPU-specific material later in the curriculum.

| Phase 1 concept | Later GPU application |
|---|---|
| Ownership | Managing GPU resources and buffer lifetimes |
| Borrowing | Designing safe host-side APIs |
| Slices | Representing contiguous input and output buffers |
| Memory layout | Matching CPU and shader data representations |
| Alignment | GPU buffer and storage layout requirements |
| Endianness | Binary data and serialization |
| Generics | Reusable GPU abstractions |
| Traits | Backend and device abstractions |
| Concurrency | Understanding parallel execution |
| Atomics | GPU synchronization and atomic operations |
| Unsafe Rust | Native GPU APIs and low-level bindings |
| FFI | Interacting with native graphics and compute APIs |
| Profiling | Measuring CPU-side overhead and runtime behavior |

One important distinction: Rust's CPU-side ownership guarantees do not automatically prove that a GPU kernel is correct. GPU execution has its own memory model, synchronization rules, and execution constraints. Those will be studied in later phases.

Your AMD Radeon RX 9070 will be useful in the later portable GPU programming phase through Vulkan and `wgpu`/WGSL. This phase does not require you to write GPU kernels yet.

# 23. Recommended Learning Resources

Use these resources as supplementary references. They should reinforce the lessons, not replace your own experiments.

1. **The Rust Programming Language:** https://doc.rust-lang.org/book/
   - Primary reference for Rust language fundamentals.

2. **Rustlings:** https://github.com/rust-lang/rustlings
   - Small exercises for practicing language concepts.

3. **The Rust Reference:** https://doc.rust-lang.org/reference/
   - More formal descriptions of Rust language behavior.

4. **The Rustonomicon:** https://doc.rust-lang.org/nomicon/
   - Advanced material about unsafe Rust and its invariants.

5. **Rust Standard Library:** https://doc.rust-lang.org/std/
   - Documentation for standard types and APIs.

6. **Rust Atomics and Locks:** https://marabos.nl/atomics/
   - A deeper introduction to concurrency and atomic operations.

7. **Learn Rust With Entirely Too Many Linked Lists:** https://rust-unofficial.github.io/too-many-lists/
   - A practical exploration of data structures, ownership, and pointers.

8. **Comprehensive Rust:** https://google.github.io/comprehensive-rust/
   - Additional structured Rust instruction.

---

# 24. Final Phase 1 Objective

At the end of this phase, you should no longer think of Rust as a language that happens to have unusual ownership rules.

You should understand that ownership, borrowing, lifetimes, and types are tools for expressing constraints about resources and memory.

You should be able to look at a program and reason about:

- Which values exist.
- Who owns them.
- Where their data is represented.
- Which references can access them.
- When resources are released.
- What happens when multiple threads interact.
- Which assumptions must hold for unsafe operations.
- How the compiler helps enforce these assumptions.

**The goal is independent systems-level reasoning.**

Do not move on simply because you have completed the exercises. Move on when you can explain the underlying principles, build the projects, and defend your decisions without relying on an AI-generated solution.
