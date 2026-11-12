# Phase 0: Mathematical and Computational Foundations

**Course:** GPU Engineering  
**Level:** Foundational  
**Prerequisites:** Basic algebra and general programming experience  
**Estimated duration:** 4–8 weeks  
**Recommended study:** 8–12 hours per week  
**Primary implementation language:** Rust  
**Hardware required:** None

---

# 1. Purpose of This Phase

Before learning how a CPU executes instructions or how a GPU runs thousands of parallel threads, you need to understand how a computer represents and manipulates information.

A computer does not fundamentally understand integers, characters, images, matrices, or programs.

At its lowest conceptual level, a digital computer stores and manipulates states represented using physical signals. We abstract those states into bits.

From bits, we construct:

- Numbers.
- Boolean values.
- Arithmetic operations.
- Logic circuits.
- Registers.
- Memory.
- Instructions.
- Programs.

This phase builds that foundation.

By the end, you should understand how a simple digital computer can represent information, perform arithmetic, and store state.

You should be able to explain these ideas without relying on language-specific abstractions.

## Learning objectives

You should be able to:

1. Explain what a bit represents.
2. Understand positional number systems.
3. Convert between binary, decimal, and hexadecimal.
4. Perform binary arithmetic.
5. Explain signed and unsigned integer representation.
6. Derive two's complement.
7. Understand overflow and fixed-width arithmetic.
8. Perform bitwise operations.
9. Understand Boolean algebra.
10. Build logic gates from simpler gates.
11. Construct combinational circuits.
12. Explain sequential circuits and stored state.
13. Understand how an ALU performs arithmetic and logic.
14. Explain how memory stores and retrieves values.
15. Understand bytes, addresses, and endianness.
16. Understand the basic representation of floating-point numbers.
17. Connect these concepts to actual computer programs.

---

# 2. Lesson 0.1 — What Is Information?

## 2.1 The central idea

A computer processes information.

But what is information?

Consider a light switch.

It can be:
- On.
- Off.

We can represent these states using two symbols:

- 1 = on.
- 0 = off.

The symbols are arbitrary. We could reverse them. What matters is that we have two distinguishable states.

A bit is a unit of information that can represent one of two possible states.

A bit does not have to physically be a light switch. In real hardware, digital states are represented using electrical properties of circuits.

The important abstraction is that we can reliably distinguish between two states.

## 2.2 Why binary?

Why use two states instead of ten?

A digital circuit must distinguish between physical signal levels.

Using two well-separated states makes it easier to distinguish valid values despite noise and variation.

For example, a circuit might interpret:
- A voltage in a lower range as 0.
- A voltage in an upper range as 1.

The exact voltage thresholds depend on the hardware technology.

This is a physical engineering decision, not a mathematical requirement.

A computer could theoretically use other number systems. Binary is especially practical because two-state digital circuits are relatively straightforward to implement reliably.

## 2.3 Bits and combinations

One bit has two possible values:

```text
0
1
```

Two bits have four possible combinations:

```text
00
01
10
11
```

Three bits have eight:

```text
000
001
010
011
100
101
110
111
```

Each additional bit doubles the number of possible combinations.

Therefore, with N bits, the number of possible combinations is:

\[
2^N
\]

This is one of the most important relationships in computer science.

### Worked examples

One bit:

\[
2^1 = 2
\]

Four bits:

\[
2^4 = 16
\]

Eight bits:

\[
2^8 = 256
\]

Thirty-two bits:

\[
2^{32} = 4,294,967,296
\]

A 32-bit pattern can represent over four billion distinct combinations.

It does not automatically mean the computer is representing four billion different integers. It means there are that many possible bit patterns. Their interpretation depends on the representation.

## 2.4 A fundamental distinction

A bit pattern and its meaning are different things.

For example:

```text
00000101
```

Could represent:
- The unsigned integer 5.
- A signed integer 5.
- A character code.
- A collection of flags.
- Part of an instruction.
- A pixel component.

The bits alone do not tell you what the data means.

The interpretation is determined by the system using them.

This distinction becomes essential when learning memory, programming languages, GPU buffers, and binary formats.

### Exercises

1. How many unique combinations can 5 bits represent?
2. How many bits are needed to represent at least 1,000 unique values?
3. Why does an 8-bit pattern not necessarily represent an integer?
4. What is the difference between a physical state and its logical interpretation?

---

# 3. Lesson 0.2 — Positional Number Systems

## 3.1 Decimal numbers

You already understand decimal numbers.

Consider:

\[
472
\]

This number is actually a shorthand for:

\[
4(10^2)+7(10^1)+2(10^0)
\]

Which gives:

\[
400+70+2=472
\]

Each position has a weight determined by a power of the base.

Decimal uses base 10.

The available digits are:

```text
0 1 2 3 4 5 6 7 8 9
```

## 3.2 Binary numbers

Binary uses base 2.

Its digits are:

```text
0 1
```

Consider:

\[
101101_2
\]

Expand it:

\[
1(2^5)+0(2^4)+1(2^3)+1(2^2)+0(2^1)+1(2^0)
\]

Therefore:

\[
32+0+8+4+0+1=45
\]

So:

\[
101101_2=45_{10}
\]

The subscript indicates the base.

## 3.3 General positional representation

For a number in base \(b\):

\[
d_nd_{n-1}\ldots d_1d_0
\]

Its value is:

\[
\sum_{i=0}^{n}d_i b^i
\]

Each digit must satisfy:

\[
0\leq d_i < b
\]

This is not specific to binary or decimal.

It is a general mathematical system.

## 3.4 Binary to decimal conversion

Convert:

\[
110101_2
\]

Write the positional weights:

| Bit | 1 | 1 | 0 | 1 | 0 | 1 |
|---|---:|---:|---:|---:|---:|---:|
| Weight | 32 | 16 | 8 | 4 | 2 | 1 |

Multiply and add:

\[
32+16+0+4+0+1=53
\]

Therefore:

\[
110101_2=53_{10}
\]

## 3.5 Decimal to binary conversion

To convert decimal to binary, repeatedly divide by two and record the remainders.

Example: convert 45.

```text
45 / 2 = 22 remainder 1
22 / 2 = 11 remainder 0
11 / 2 = 5  remainder 1
5  / 2 = 2  remainder 1
2  / 2 = 1  remainder 0
1  / 2 = 0  remainder 1
```

Read the remainders from bottom to top:

```text
101101
```

Therefore:

\[
45_{10}=101101_2
\]

### Why does this work?

At every division, we separate the least significant binary digit from the remaining number.

The remainder is either 0 or 1.

The quotient contains the remaining higher-order digits.

Repeating this process extracts every binary digit.

## 3.6 Hexadecimal

Binary numbers become long very quickly.

Hexadecimal uses base 16 and provides a compact way to represent binary data.

Its digits are:

```text
0 1 2 3 4 5 6 7 8 9 A B C D E F
```

The letters represent:

| Hex | Decimal |
|---|---:|
| A | 10 |
| B | 11 |
| C | 12 |
| D | 13 |
| E | 14 |
| F | 15 |

One hexadecimal digit represents exactly four binary bits because:

\[
2^4=16
\]

For example:

```text
Binary:      1010 1101
Hexadecimal:    A    D
```

Therefore:

\[
10101101_2=AD_{16}
\]

And:

\[
AD_{16}=10(16)+13=173
\]

## 3.7 Why programmers use hexadecimal

Consider a 32-bit value:

```text
11010100101100101101001010101100
```

This is difficult to inspect.

In hexadecimal:

```text
D4B2D2AC
```

Both representations contain exactly the same information.

Hexadecimal is useful for:
- Memory addresses.
- Binary data.
- Machine instructions.
- Bit masks.
- Debugging.
- Register values.
- GPU resource inspection.

### Exercises

Convert the following:

1. \(1010_2\) to decimal.
2. \(11111111_2\) to decimal.
3. \(100000000_2\) to decimal.
4. \(255_{10}\) to binary.
5. \(173_{10}\) to hexadecimal.
6. \(C7_{16}\) to decimal.
7. \(101101111010_2\) to hexadecimal.

---

# 4. Lesson 0.3 — Binary Arithmetic

Computers need to perform arithmetic.

The good news is that binary arithmetic follows the same basic principles as decimal arithmetic.

## 4.1 Binary addition

The basic rules are:

| A | B | Sum | Carry |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

The most important rule is:

\[
1+1=10_2
\]

The result is zero in the current position and one carried to the next position.

### Worked example

Add:

```text
  1011
+ 0110
------
 10001
```

Check in decimal:

\[
11+6=17
\]

And:

\[
10001_2=17
\]

## 4.2 Binary subtraction

Binary subtraction follows similar rules.

For example:

```text
  1010
- 0011
------
  0111
```

Because:

\[
10-3=7
\]

Subtraction requires borrowing when the current bit is too small.

## 4.3 Binary multiplication

Binary multiplication is especially simple because each multiplier digit is either zero or one.

Example:

```text
     101
   x 011
   -----
     101
    1010
   -----
    1111
```

Check:

\[
5 \times 3 = 15
\]

## 4.4 Shifting and multiplication

Shifting a binary number left by one position multiplies its unsigned value by two, provided the result does not overflow the chosen width.

Example:

```text
00000101
```

Shift left once:

```text
00001010
```

The values are:

\[
5 \rightarrow 10
\]

Shift right once:

```text
00000101
```

becomes:

```text
00000010
```

For unsigned values, a logical right shift by one is equivalent to integer division by two, rounding down.

Be careful: signed shifts have additional rules and language-specific behavior.

## 4.5 Fixed-width arithmetic

A computer does not have infinitely many bits available for an integer.

An 8-bit value has exactly eight bits.

Suppose we add:

```text
11111111
00000001
--------
00000000
```

The mathematical result is 256.

But 256 requires nine bits:

```text
1 00000000
```

If the result is stored in only eight bits, the high carry is discarded.

The stored result becomes zero.

This is an example of overflow in fixed-width unsigned arithmetic.

### Key concept

For an N-bit unsigned integer, arithmetic that discards bits beyond the width behaves modulo:

\[
2^N
\]

For eight bits:

\[
256 \bmod 256=0
\]

This modular arithmetic is fundamental to understanding machine integers.

### Exercises

1. Add \(10101_2+1101_2\).
2. Subtract \(11000_2-1011_2\).
3. Multiply \(101_2\) by \(110_2\).
4. What happens when 255 is incremented in an 8-bit unsigned representation?
5. Explain why a left shift can lose information.

---

# 5. Lesson 0.4 — Unsigned Integers

An unsigned integer interprets every bit as a nonnegative contribution.

For an N-bit unsigned integer:

\[
x=\sum_{i=0}^{N-1}b_i2^i
\]

Each bit is either zero or one.

The minimum value is:

\[
0
\]

The maximum value occurs when every bit is one:

\[
2^{N-1}+2^{N-2}+\ldots+2^1+2^0
\]

This is a geometric series:

\[
2^N
\]

Therefore, the range is:

\[
0 \leq x \leq 2^N-1
\]

## Examples

| Width | Minimum | Maximum |
|---|---:|---:|
| 8-bit | 0 | 255 |
| 16-bit | 0 | 65,535 |
| 32-bit | 0 | 4,294,967,295 |
| 64-bit | 0 | 18,446,744,073,709,551,615 |

## Why integer width matters

Consider an image with 1920 × 1080 pixels.

The total pixel count is:

\[
1920 \times 1080=2,073,600
\]

A 16-bit unsigned integer can represent this number because its maximum is 65,535?

No.

The value exceeds that limit.

A 32-bit unsigned integer is needed.

This is why selecting the correct integer width matters in real software.

GPU programs frequently calculate:
- Pixel indices.
- Array offsets.
- Buffer sizes.
- Thread indices.
- Matrix dimensions.
- Memory addresses.

Incorrect integer widths can cause overflow and invalid memory access.

### Exercises

1. Derive the maximum value of an N-bit unsigned integer.
2. What is the smallest integer width that can represent 1,000,000?
3. Can a 16-bit unsigned integer represent 65,536?
4. Why can integer overflow cause memory safety problems?

---

# 6. Lesson 0.5 — Signed Integers and Two's Complement

Unsigned integers cannot represent negative numbers.

Computers need signed integers for values such as:

- Temperature changes.
- Coordinates.
- Differences.
- Negative offsets.
- Mathematical calculations.

There are several possible ways to represent signed integers.

Modern general-purpose processors commonly use two's complement.

## 6.1 The idea

For an N-bit two's-complement integer, the most significant bit has a negative weight.

The value is:

\[
x=-b_{N-1}2^{N-1}+
\sum_{i=0}^{N-2}b_i2^i
\]

The highest bit contributes a negative value.

All remaining bits contribute positive values.

## 6.2 Example with four bits

Consider:

```text
0000 =  0
0001 =  1
0010 =  2
0011 =  3
0100 =  4
0101 =  5
0110 =  6
0111 =  7
1000 = -8
1001 = -7
1010 = -6
1011 = -5
1100 = -4
1101 = -3
1110 = -2
1111 = -1
```

Notice that four bits represent:

\[
-8 \text{ through } 7
\]

For N bits, the range is:

\[
-2^{N-1}
\leq x \leq
2^{N-1}-1
\]

## 6.3 Deriving negative numbers

To find the two's-complement representation of a negative number:

1. Write the positive number in binary using the required width.
2. Invert every bit.
3. Add one.

Example: represent -5 using eight bits.

First write positive five:

```text
00000101
```

Invert:

```text
11111010
```

Add one:

```text
11111011
```

Therefore:

```text
-5 = 11111011
```

## 6.4 Why invert and add one?

Two's complement is designed so that addition can work for signed and unsigned values using the same underlying binary adder.

Consider:

\[
5+(-5)=0
\]

Using eight bits:

```text
  00000101
+ 11111011
----------
1 00000000
```

Discard the carry beyond the eight-bit result:

```text
00000000
```

This makes hardware arithmetic much simpler.

The same adder can perform signed and unsigned addition. The interpretation of the bits determines the meaning of the result.

## 6.5 Signed overflow

Consider an 8-bit signed integer.

The maximum is 127.

Adding one produces:

```text
01111111
00000001
--------
10000000
```

The result represents -128.

The mathematical result was 128, which is outside the representable signed range.

This is signed overflow.

In programming languages, overflow behavior depends on the language and operation. In Rust, ordinary integer addition can panic in debug builds and wraps in optimized builds unless using explicit wrapping or checked operations. C++ signed integer overflow is undefined behavior.

Never assume every language handles overflow identically.

### Exercises

1. Represent -12 using eight bits.
2. Convert `11110110` into a signed eight-bit integer.
3. Derive the signed range for 16 bits.
4. Explain why two's complement makes subtraction possible using an adder.
5. Explain the difference between signed and unsigned interpretation of the same bits.

---

# 7. Lesson 0.6 — Bitwise Operations

Bitwise operations manipulate individual bits.

They are essential in:
- Systems programming.
- Device registers.
- Memory flags.
- Binary protocols.
- GPU masks.
- SIMD operations.
- Low-level optimization.

## 7.1 AND

AND returns one only when both input bits are one.

| A | B | A AND B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

Example:

```text
  11001010
& 10101100
----------
  10001000
```

AND is useful for masking bits.

## 7.2 OR

OR returns one when at least one input is one.

```text
  11001010
| 10101100
----------
  11101110
```

OR is useful for setting selected bits.

## 7.3 XOR

XOR returns one when the inputs differ.

```text
  11001010
^ 10101100
----------
  01100110
```

XOR is useful for:
- Toggling bits.
- Comparing bit patterns.
- Simple parity calculations.

## 7.4 NOT

NOT flips every bit.

```text
NOT 00001111
    --------
    11110000
```

The exact result depends on the width of the value.

## 7.5 Bit masks

A mask selects specific bits.

Suppose an 8-bit value is:

```text
10110110
```

You want to inspect the lowest four bits.

Use:

```text
00001111
```

AND:

```text
  10110110
& 00001111
----------
  00000110
```

The lower four bits are now isolated.

## 7.6 Setting, clearing, and toggling bits

Suppose bit 2 represents a feature flag.

To set bit 2:

```rust
value |= 1 << 2;
```

To clear bit 2:

```rust
value &= !(1 << 2);
```

To toggle bit 2:

```rust
value ^= 1 << 2;
```

To test bit 2:

```rust
let enabled = (value & (1 << 2)) != 0;
```

These operations are common in systems code.

## 7.7 Shifts

Left shift:

```rust
value << n
```

Right shift:

```rust
value >> n
```

For unsigned integers, left shifting by one moves every bit one position left, discarding bits that leave the width.

A shift is not automatically safe multiplication. You must account for overflow and the language's shift rules.

### Exercises

1. Compute `10101100 AND 11110000`.
2. Compute `10101100 OR 00001111`.
3. Compute `10101100 XOR 11111111`.
4. Explain how to test whether a number is even using a bitwise operation.
5. Write code that sets, clears, toggles, and tests bit 5.
6. Explain why the width of a mask matters.

---

# 8. Lesson 0.7 — Boolean Algebra

Boolean algebra is the mathematics of values that are either true or false.

It is the foundation of digital logic.

Let:

\[
A,B \in \{0,1\}
\]

We can define operations on these values.

- AND: \(A \land B\)
- OR: \(A \lor B\)
- NOT: \(\neg A\)
- XOR: \(A \oplus B\)

## 8.1 Truth tables

A truth table enumerates every possible input combination and the corresponding output.

For two inputs, there are:

\[
2^2=4
\]

possible combinations.

For three inputs:

\[
2^3=8
\]

possible combinations.

Truth tables are useful for specifying and verifying logic.

## 8.2 De Morgan's laws

Two important identities are:

\[
\neg(A \land B)=\neg A \lor \neg B
\]

and:

\[
\neg(A \lor B)=\neg A \land \neg B
\]

These identities allow logic circuits to be transformed into equivalent forms.

You should be able to prove them by constructing truth tables.

## 8.3 Boolean simplification

Consider:

\[
A \land 1=A
\]

And:

\[
A \lor 0=A
\]

Other useful identities:

\[
A \land 0=0
\]

\[
A \lor 1=1
\]

\[
A \land A=A
\]

\[
A \lor A=A
\]

\[
A \oplus A=0
\]

\[
A \oplus 0=A
\]

These rules help simplify circuits.

## 8.4 Why this matters

Digital circuits are built from logic operations.

A processor's arithmetic unit is not magic. It is a collection of circuits implementing Boolean functions.

Understanding Boolean algebra allows you to reason about the behavior of those circuits.

### Exercises

1. Prove De Morgan's first law with a truth table.
2. Simplify \(A \land (A \lor B)\).
3. Simplify \(A \lor (A \land B)\).
4. Explain why XOR is different from OR.
5. Construct a truth table for \((A \land B)\lor C\).

---

# 9. Lesson 0.8 — Logic Gates

A logic gate is a physical circuit implementing a Boolean function.

The basic gates are:

- AND.
- OR.
- NOT.
- XOR.
- NAND.
- NOR.

## 9.1 NAND

NAND is NOT AND:

\[
NAND(A,B)=\neg(A\land B)
\]

Truth table:

| A | B | Output |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

NAND is particularly important because it is functionally complete.

This means any Boolean function can be constructed using only NAND gates.

## 9.2 Building NOT from NAND

Connect both NAND inputs to the same signal:

\[
NAND(A,A)=\neg A
\]

## 9.3 Building AND from NAND

First calculate NAND:

\[
X=NAND(A,B)
\]

Then invert it:

\[
AND(A,B)=NAND(X,X)
\]

## 9.4 Building OR from NAND

Using De Morgan's law:

\[
A\lor B=\neg(\neg A\land\neg B)
\]

Therefore, we can construct OR using NAND gates and inverted inputs.

This demonstrates how a small set of basic components can implement more complex behavior.

## 9.5 Why NAND matters

A digital computer can be constructed from repeated combinations of simple logic components.

You do not need a separate physical component for every mathematical operation.

Instead, complex behavior emerges from combining simpler circuits.

### Lab

Implement these functions using only NAND gates:

1. NOT.
2. AND.
3. OR.
4. XOR.
5. Multiplexer.

Verify each implementation with a truth table.

---

# 10. Lesson 0.9 — Combinational Circuits

A combinational circuit produces outputs based only on its current inputs.

There is no stored state.

Examples:
- Adders.
- Multiplexers.
- Decoders.
- Comparators.
- Logic units.

## 10.1 Half adder

A half adder adds two one-bit values.

Inputs:
- A.
- B.

Outputs:
- Sum.
- Carry.

The sum is:

\[
S=A\oplus B
\]

The carry is:

\[
C=A\land B
\]

Truth table:

| A | B | Sum | Carry |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

## 10.2 Full adder

A full adder also receives a carry-in.

Inputs:
- A.
- B.
- Carry-in.

Outputs:
- Sum.
- Carry-out.

One implementation is:

\[
S=A\oplus B\oplus C_{in}
\]

The carry-out is:

\[
C_{out}=(A\land B)\lor(C_{in}\land(A\oplus B))
\]

You should be able to derive this from the truth table.

## 10.3 Building wider adders

A full adder can be connected to another full adder.

The carry-out of one stage becomes the carry-in of the next.

This produces a ripple-carry adder.

For a four-bit adder:

```text
Bit 0 -> Bit 1 -> Bit 2 -> Bit 3
  |        |        |        |
 Carry    Carry    Carry    Output
```

This allows the circuit to add multi-bit values.

The limitation is that carries must propagate through the stages.

This creates delay.

More advanced adders use different structures to reduce carry propagation time.

## 10.4 Multiplexers

A multiplexer selects one of multiple inputs.

A 2-to-1 multiplexer has:
- Input A.
- Input B.
- Select S.
- Output Y.

Its Boolean expression is:

\[
Y=(\neg S\land A)\lor(S\land B)
\]

When S is zero, output A.

When S is one, output B.

Multiplexers are important because they allow circuits to select between different values.

They appear throughout processor datapaths.

## 10.5 Arithmetic logic units

An ALU performs arithmetic and logical operations.

A simplified ALU might support:

- Addition.
- Subtraction.
- AND.
- OR.
- XOR.
- Comparison.

A control input selects the operation.

An ALU combines arithmetic circuits, logic circuits, and multiplexers.

### Exercises

1. Derive a full adder truth table.
2. Build a four-bit ripple-carry adder.
3. Derive the multiplexer expression.
4. Explain why a multiplexer is useful in a processor.
5. Design an ALU that supports addition, AND, OR, and XOR.

---

# 11. Lesson 0.10 — Sequential Logic and State

Combinational circuits depend only on current inputs.

But computers must remember information.

A processor must retain:
- Register values.
- Program counters.
- Memory contents.
- Execution state.

This requires sequential logic.

## 11.1 The idea of state

A system has state if its future behavior depends partly on information retained from the past.

Consider a simple counter.

At every clock event, it increments its stored value.

The next value depends on the previous value.

\[
State_{next}=State_{current}+1
\]

This is different from a purely combinational function.

## 11.2 Latches and flip-flops

A latch can store a bit.

A flip-flop is commonly used to store a bit with behavior controlled by a clock edge.

A register is built from multiple storage elements.

For example, an eight-bit register stores eight bits.

## 11.3 Clocked systems

A clock provides timing events that coordinate state changes.

At a conceptual level:

1. Combinational logic calculates a result.
2. The result is presented to storage elements.
3. A clock event causes state to update.
4. The next cycle begins.

Real hardware has timing constraints, propagation delays, setup times, and hold times. Those details become more important in digital design.

## 11.4 Why this matters for processors

A processor repeatedly:

1. Reads its current state.
2. Computes a result.
3. Updates state.
4. Continues execution.

Registers and memory provide the state that makes program execution possible.

### Exercises

1. Explain the difference between combinational and sequential logic.
2. Why does a processor need registers?
3. What is the purpose of a clock?
4. Explain why a counter requires stored state.
5. Describe the relationship between combinational logic and a register.

---

# 12. Lesson 0.11 — Memory and Addressing

A computer needs to store large amounts of information.

Memory provides a collection of storage locations.

Conceptually, memory can be viewed as an indexed collection of bytes.

Each byte has an address.

For example:

```text
Address     Contents
0x1000      01001000
0x1001      01100101
0x1002      01101100
0x1003      01101100
```

An address identifies a location.

A read operation retrieves the value stored at an address.

A write operation changes the stored value.

## 12.1 Why addresses matter

Programs use addresses to locate data.

An array is conceptually a sequence of values occupying memory.

For an array with elements of equal size:

\[
Address(i)=BaseAddress+i\times ElementSize
\]

For example, if an array begins at address 1000 and each element occupies four bytes:

| Index | Address |
|---|---:|
| 0 | 1000 |
| 1 | 1004 |
| 2 | 1008 |
| 3 | 1012 |

This relationship is essential for understanding pointers and array indexing.

## 12.2 Bytes

A byte is conventionally eight bits.

A byte can represent:

\[
2^8=256
\]

different patterns.

Common data widths include:

- 8 bits = 1 byte.
- 16 bits = 2 bytes.
- 32 bits = 4 bytes.
- 64 bits = 8 bytes.

## 12.3 Memory layout

A value is stored as a sequence of bytes.

For example, a 32-bit value requires four bytes.

The order of those bytes in memory depends on the system's endianness.

## 12.4 Endianness

Consider the 32-bit hexadecimal value:

```text
0x12345678
```

Big-endian stores the most significant byte first:

```text
Address   Byte
1000      12
1001      34
1002      56
1003      78
```

Little-endian stores the least significant byte first:

```text
Address   Byte
1000      78
1001      56
1002      34
1003      12
```

The numerical value is the same.

Only the byte ordering differs.

Endianness matters when:
- Reading binary files.
- Communicating across machines.
- Inspecting memory.
- Working with hardware.
- Implementing serialization.

### Exercises

1. Explain the difference between an address and a value.
2. Derive the address of array element 7 when the base is 1000 and each element occupies four bytes.
3. Represent `0xAABBCCDD` in big-endian byte order.
4. Represent the same value in little-endian byte order.
5. Explain why endianness does not change the numerical value.

---

# 13. Lesson 0.12 — Floating-Point Representation

Integers are not sufficient for scientific computing.

Computers need to represent fractional values.

Examples include:

- 0.5.
- 3.14159.
- -0.00001.
- Large and small physical measurements.

Floating-point representation provides an approximation of real numbers.

## 13.1 Why not store every decimal digit?

Real numbers can have arbitrarily many digits.

A finite computer cannot store every possible real number exactly.

Instead, floating-point formats store a finite approximation.

The IEEE 754 standard defines widely used floating-point formats.

## 13.2 Basic structure

A normalized binary floating-point number is conceptually represented using:

- Sign.
- Exponent.
- Significand.

For a simplified binary floating-point representation:

\[
x=(-1)^s \times 1.f \times 2^e
\]

Here:
- \(s\) determines the sign.
- \(f\) represents the fractional significand bits.
- \(e\) is the exponent.

Actual IEEE 754 formats use biased exponents and have special encodings for zero, subnormal values, infinities, and NaNs.

## 13.3 Why floating-point values are approximate

Consider the decimal number 0.1.

Its binary expansion repeats indefinitely.

A finite binary floating-point format cannot represent it exactly.

The stored value is therefore an approximation.

This explains why:

```text
0.1 + 0.2
```

may not produce exactly the same representation as:

```text
0.3
```

## 13.4 Precision and range

A floating-point format has limited precision and a limited exponent range.

A larger exponent range does not necessarily mean more precision.

The number of significand bits determines how finely values can be distinguished at a given scale.

## 13.5 Why this matters for GPU programming

GPU workloads frequently use:
- FP32.
- FP16.
- BF16.
- Other reduced-precision formats.

Different formats trade precision, range, memory usage, and throughput.

This becomes especially important in machine learning and scientific computing.

You will study numerical stability in much greater depth in Phase 5.

### Exercises

1. Why can some decimal fractions not be represented exactly in binary?
2. Explain the roles of sign, exponent, and significand.
3. Distinguish precision from range.
4. Why can two mathematically equivalent expressions produce different floating-point results?
5. Why is floating-point behavior important when parallelizing an algorithm?

---

# 14. Lesson 0.13 — Connecting the Pieces: A Simple Computer

You now have the basic components needed to understand a simple computer.

A simplified model contains:

- Memory.
- Registers.
- An arithmetic logic unit.
- A control unit.
- An instruction sequence.

A program is represented using instructions.

The processor repeatedly retrieves instructions, interprets them, performs operations, and updates state.

A simplified execution cycle looks like:

```text
          +----------------+
          |     Memory     |
          +----------------+
                  |
                  v
          +----------------+
          | Instruction    |
          | Fetch          |
          +----------------+
                  |
                  v
          +----------------+
          | Decode         |
          +----------------+
                  |
                  v
          +----------------+
          | Execute        |
          | ALU / Logic    |
          +----------------+
                  |
                  v
          +----------------+
          | Update State   |
          +----------------+
                  |
                  +-------> Next instruction
```

This is a simplified conceptual model. Real processors use pipelining, caches, speculative execution, and many other mechanisms.

The important connection is that everything you have studied contributes to program execution:

- Binary represents the information.
- Logic gates perform operations.
- Adders perform arithmetic.
- Registers retain state.
- Memory stores data and instructions.
- Control logic determines what happens next.

This is the foundation for Phase 3, Computer Architecture.

---

# 15. Practical Lab — Build a Memory Inspector in Rust

## Objective

Build a program that allows you to inspect how values are represented in memory.

This project connects mathematical representations to actual software.

## Requirements

Your program must accept integer inputs and display:

- Decimal.
- Binary.
- Hexadecimal.
- Byte representation.
- Signed interpretation where applicable.
- Unsigned interpretation.
- Overflow demonstrations.
- Endianness.

## Example behavior

Input:

```text
173
```

Possible output:

```text
Decimal:       173
Binary (8-bit): 10101101
Hexadecimal:   0xAD
Unsigned:      173
Signed i8:     -83
```

The signed interpretation differs because the same eight bits are being interpreted using two's complement.

## Suggested Rust concepts

Use:
- `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`.
- `format!`.
- Bitwise operators.
- `to_le_bytes()`.
- `to_be_bytes()`.
- `checked_add()`.
- `wrapping_add()`.
- `overflowing_add()`.

## Required experiments

1. Add 1 to the maximum value of each unsigned type.
2. Compare checked and wrapping arithmetic.
3. Display a signed value as raw bits.
4. Convert between big-endian and little-endian byte sequences.
5. Compare the binary representation of positive and negative values.
6. Inspect the bit patterns of several floating-point numbers.

## Project quality requirements

- Separate representation logic from input/output.
- Include unit tests.
- Handle invalid input.
- Document assumptions.
- Avoid silently truncating values.
- Explain what each output means.

---

# 16. Practical Lab — Build a Logic Simulator

## Objective

Implement digital logic as software.

The simulator should model Boolean components and combine them into larger circuits.

## Required components

- NOT.
- AND.
- OR.
- XOR.
- NAND.
- NOR.
- Multiplexer.
- Half adder.
- Full adder.
- Four-bit adder.
- Basic ALU.

## Testing

Every component must be tested against its truth table.

For a two-input gate, test all four possible input combinations.

For a three-input component, test all eight combinations.

The ALU must support at least:

- Addition.
- AND.
- OR.
- XOR.

## Extension

Build a simple sequential counter that retains its value between clock steps.

The purpose is to model the difference between combinational logic and stored state.

---

# 17. Guided Problem Set

Try these without looking up solutions.

## Section A — Number representation

1. How many distinct patterns can 12 bits represent?
2. How many bits are required to represent 10,000 distinct values?
3. Convert `11011010` to decimal.
4. Convert 349 to binary.
5. Convert `0xDEADBEEF` to binary.
6. Explain why hexadecimal is convenient for representing binary data.

## Section B — Arithmetic

7. Add `101101` and `11101`.
8. Subtract `110000` minus `10101`.
9. Multiply `1011` by `101`.
10. What is the result of adding 1 to an 8-bit unsigned value of 255?
11. Explain modular arithmetic in the context of fixed-width integers.

## Section C — Signed integers

12. Represent -37 using eight-bit two's complement.
13. Interpret `11101001` as a signed eight-bit integer.
14. Derive the range of a 32-bit signed integer.
15. Explain why the minimum signed value has no positive counterpart in the same width.
16. Explain how an adder can perform both signed and unsigned addition.

## Section D — Boolean logic

17. Construct the truth table for `(A AND B) OR C`.
18. Prove De Morgan's second law.
19. Explain why NAND is functionally complete.
20. Build XOR from NAND gates.
21. Derive the equations for a full adder.

## Section E — Memory

22. What is the difference between a byte and an address?
23. If an array begins at address 4096 and each element is eight bytes, what is the address of element 10?
24. Explain big-endian and little-endian storage.
25. Explain why a pointer is not the same thing as the value it points to.

## Section F — Floating point

26. Why is 0.1 not exactly representable in binary floating point?
27. Explain the difference between precision and range.
28. Why might two different summation orders produce different results?
29. Explain why floating-point arithmetic is important to GPU computing.

---

# 18. Phase 0 Examination

**Passing score: 85/100**

The examination should be taken after completing the lessons, labs, and guided problem set.

Suggested weighting:

| Category | Weight |
|---|---:|
| Number systems and conversion | 15 |
| Binary arithmetic | 15 |
| Signed and unsigned representation | 15 |
| Bitwise operations | 10 |
| Boolean algebra and logic gates | 15 |
| Combinational and sequential circuits | 10 |
| Memory and addressing | 10 |
| Floating-point fundamentals | 10 |

## Oral technical defense

You must be able to explain the following without relying on memorized definitions:

1. How can a collection of bits represent an integer?
2. Why does two's complement work?
3. How can an adder perform subtraction?
4. How can NAND gates construct arbitrary logic?
5. How does a simple ALU work?
6. What is the difference between a circuit that computes a value and a circuit that stores state?
7. How does a memory address identify data?
8. Why does floating-point arithmetic introduce approximation?

## Final project defense

Demonstrate your Memory Inspector and Logic Simulator.

Explain:
- The mathematical representation of the values.
- The implementation decisions.
- Edge cases.
- Tests.
- Limitations.

You should be able to explain the underlying concepts without relying on your code.

---

# 19. Phase 0 Quiz Prompts

## Prompt A — Phase 0 diagnostic

```text
I am beginning Phase 0 of my GPU engineering curriculum.

Test my existing understanding of:
- Binary and hexadecimal.
- Number representation.
- Signed and unsigned integers.
- Binary arithmetic.
- Boolean logic.
- Logic gates.
- Memory and addressing.
- Floating-point fundamentals.

Start with basic questions and increase difficulty.

Ask one question at a time.
Do not reveal answers before I respond.
Identify missing prerequisites and create a study plan
based on my results.
```

## Prompt B — Binary and integer mastery

```text
Act as a computer architecture instructor.

Teach and examine me on binary number representation,
fixed-width arithmetic, signed integers, and two's complement.

I want to derive the rules from first principles.

Include conversion problems, arithmetic, overflow,
edge cases, and explanations of why the representations work.

Ask one question at a time.
Do not reveal answers prematurely.
```

## Prompt C — Digital logic examination

```text
Create a difficult digital logic examination.

Test:
- Boolean algebra.
- Truth tables.
- De Morgan's laws.
- NAND and NOR.
- Adders.
- Multiplexers.
- ALU design.
- Sequential logic.

Require me to derive circuits and explain why they work.

Do not give me solutions before I attempt each problem.
```

## Prompt D — Memory representation examination

```text
Examine my understanding of how computers represent data
in memory.

Include:
- Bits and bytes.
- Addresses.
- Integer widths.
- Endianness.
- Signed and unsigned interpretation.
- Floating-point representation.
- Alignment fundamentals.

Include practical Rust examples.

Ask me to predict actual memory representations
before revealing the results.
```

## Prompt E — Misconception investigation

```text
Investigate my understanding of Phase 0.

I want difficult questions that reveal subtle misunderstandings.

Focus on:
- Confusing bit patterns with their interpretations.
- Confusing signed and unsigned arithmetic.
- Incorrect assumptions about overflow.
- Misunderstanding two's complement.
- Confusing logical and physical state.
- Misunderstanding addresses and stored values.
- Assuming floating-point arithmetic is exact.

Ask one question at a time and create remedial exercises
for every misconception you find.
```

---

# 20. Completion Criteria

Do not move to Phase 1 until you can:

- Convert confidently between binary, decimal, and hexadecimal.
- Derive the representable range of an N-bit integer.
- Explain and calculate two's-complement values.
- Explain fixed-width overflow.
- Use bitwise operations to manipulate individual bits.
- Derive basic Boolean identities.
- Construct basic gates from NAND.
- Derive a full adder.
- Explain a multiplexer.
- Explain the difference between combinational and sequential logic.
- Describe how memory addresses work.
- Explain endianness.
- Explain the basic structure of floating-point representation.
- Complete both practical projects.
- Pass the phase examination.

**The most important test:** Can you explain how a computer represents and manipulates information without treating the behavior as magic?

If yes, you are ready to move on to Rust systems programming.
