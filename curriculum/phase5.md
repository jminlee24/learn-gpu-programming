# Phase 5: Mathematics and Numerical Computing

**Curriculum:** GPU Engineering  
**Phase:** 5 of 16  
**Estimated duration:** 12–16 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** Rust and C++20  
**Prerequisites:** Phases 0–4  
**Hardware:** CPU for foundational experiments; AMD Radeon RX 9070 for later GPU comparisons

---

# 1. Purpose and Learning Outcomes

A GPU is fundamentally a machine for performing large amounts of numerical computation.

To become an effective GPU engineer, you need to understand both:

1. The mathematics being computed.
2. How finite-precision hardware changes the computation.

A mathematically correct algorithm may produce inaccurate results when implemented with floating-point arithmetic.

An algorithm that performs fewer arithmetic operations may run slower because of memory access patterns.

An algorithm that works sequentially may require substantial restructuring to execute efficiently in parallel.

This phase develops the mathematical reasoning needed to understand these problems.

## Learning objectives

By the end of Phase 5, you should be able to:

1. Work confidently with vectors, matrices, and tensors.
2. Understand linear transformations and matrix multiplication.
3. Analyze numerical algorithms using computational complexity.
4. Explain floating-point representation and rounding.
5. Understand absolute and relative error.
6. Analyze numerical stability and conditioning.
7. Explain catastrophic cancellation.
8. Implement and analyze numerical reductions.
9. Understand parallel reduction trees.
10. Understand numerical behavior in matrix multiplication.
11. Understand numerical differentiation and integration.
12. Explain iterative numerical methods.
13. Analyze arithmetic intensity and memory traffic.
14. Design numerical experiments.
15. Validate numerical implementations using appropriate tolerances.
16. Understand the mathematical foundations of common GPU workloads.

## Central principle

**Mathematical equivalence does not guarantee numerical equivalence or equal performance.**

For example:

\[
(a+b)+c=a+(b+c)
\]

is true over real numbers.

But for floating-point arithmetic:

\[
\operatorname{fl}(\operatorname{fl}(a+b)+c)
\]

may differ from:

\[
\operatorname{fl}(a+\operatorname{fl}(b+c))
\]

because rounding occurs after operations.

This matters because parallel algorithms frequently change the order of computation.

---

# 2. Recommended Study Structure

| Module | Topic | Estimated time |
|---|---|---|
| 1 | Vectors and linear algebra | 2 weeks |
| 2 | Matrices and linear transformations | 2 weeks |
| 3 | Floating-point arithmetic | 2 weeks |
| 4 | Numerical error and stability | 2 weeks |
| 5 | Reductions and parallel numerical algorithms | 2 weeks |
| 6 | Matrix multiplication and performance | 2 weeks |
| 7 | Numerical methods and iterative algorithms | 2 weeks |
| 8 | Capstone and final examination | 2 weeks |

These estimates are flexible. Do not advance merely because the scheduled time has elapsed.

---

# 3. Module One: Vectors and Linear Algebra

## 3.1 What is a vector?

A vector can be represented as an ordered collection of numbers:

\[
\mathbf{v}=
\begin{bmatrix}
v_1\\
v_2\\
\vdots\\
v_n
\end{bmatrix}
\]

In computational applications, vectors commonly represent:

- Positions.
- Velocities.
- Forces.
- Features.
- Numerical samples.
- Model parameters.
- Intermediate results.

The dimension of the vector is the number of components.

## 3.2 Vector addition

Given:

\[
\mathbf{a}=
\begin{bmatrix}
a_1\\a_2\\a_3
\end{bmatrix}
,\quad
\mathbf{b}=
\begin{bmatrix}
b_1\\b_2\\b_3
\end{bmatrix}
\]

their sum is:

\[
\mathbf{a}+\mathbf{b}=
\begin{bmatrix}
a_1+b_1\\
a_2+b_2\\
a_3+b_3
\end{bmatrix}
\]

Vector addition is element-wise.

### Worked example

Let:

\[
\mathbf{a}=(2,4,6)
\]

\[
\mathbf{b}=(1,3,5)
\]

Then:

\[
\mathbf{a}+\mathbf{b}=(3,7,11)
\]

Computationally, the elements can be added independently.

This makes vector addition highly parallelizable.

## 3.3 Scalar multiplication

Multiplying a vector by a scalar multiplies each component:

\[
c\mathbf{v}=
\begin{bmatrix}
cv_1\\
cv_2\\
\vdots\\
cv_n
\end{bmatrix}
\]

This is also element-wise and naturally parallel.

## 3.4 Dot product

The dot product of two vectors is:

\[
\mathbf{a}\cdot\mathbf{b}
=
\sum_{i=1}^{n}a_ib_i
\]

### Worked example

\[
\mathbf{a}=(1,2,3)
\]

\[
\mathbf{b}=(4,5,6)
\]

Then:

\[
\mathbf{a}\cdot\mathbf{b}
=
1(4)+2(5)+3(6)=32
\]

The dot product combines two computational patterns:

1. Independent element-wise multiplication.
2. Reduction of intermediate results into one scalar.

This structure appears throughout GPU programming.

## 3.5 Vector norms

A norm measures vector magnitude.

The Euclidean norm is:

\[
\|\mathbf{v}\|_2=
\sqrt{\sum_{i=1}^{n}v_i^2}
\]

For:

\[
\mathbf{v}=(3,4)
\]

\[
\|\mathbf{v}\|_2=\sqrt{9+16}=5
\]

Other useful norms include:

\[
\|\mathbf{v}\|_1=\sum_i|v_i|
\]

and:

\[
\|\mathbf{v}\|_\infty=\max_i|v_i|
\]

Different norms are useful for different numerical analyses.

## 3.6 Orthogonality

Two vectors are orthogonal when their dot product is zero:

\[
\mathbf{a}\cdot\mathbf{b}=0
\]

Orthogonality is important in:

- Geometry.
- Linear systems.
- Optimization.
- Numerical methods.
- Machine learning.

## 3.7 Vector projection

The projection of \(\mathbf{a}\) onto a nonzero vector \(\mathbf{b}\) is:

\[
\operatorname{proj}_{\mathbf{b}}(\mathbf{a})
=
\frac{\mathbf{a}\cdot\mathbf{b}}
{\mathbf{b}\cdot\mathbf{b}}\mathbf{b}
\]

This operation uses dot products, scalar arithmetic, and vector scaling.

## Exercises

1. Compute vector addition for two four-dimensional vectors.
2. Compute a dot product by hand.
3. Compute the Euclidean norm of a vector.
4. Determine whether two vectors are orthogonal.
5. Derive the formula for vector projection.
6. Explain why vector addition is easy to parallelize.
7. Explain why a dot product requires coordination between partial results.
8. Estimate the arithmetic operations and memory accesses required for a dot product of length \(N\).

## Coding Lab 1: Vector Mathematics Library

Implement a small vector mathematics library in Rust.

Required operations:

- Vector addition.
- Vector subtraction.
- Scalar multiplication.
- Dot product.
- Euclidean norm.
- Normalization.
- Element-wise multiplication.

Requirements:

- Support configurable vector sizes.
- Validate dimension compatibility.
- Handle invalid input.
- Include unit tests.
- Compare results against independently calculated reference values.

Do not use an existing linear algebra library for the implementation.

---

# 4. Module Two: Matrices and Linear Transformations

## 4.1 What is a matrix?

A matrix is a rectangular arrangement of numbers.

\[
A=
\begin{bmatrix}
a_{11}&a_{12}&\cdots&a_{1n}\\
a_{21}&a_{22}&\cdots&a_{2n}\\
\vdots&\vdots&\ddots&\vdots\\
a_{m1}&a_{m2}&\cdots&a_{mn}
\end{bmatrix}
\]

An \(m\times n\) matrix contains \(m\) rows and \(n\) columns.

Matrices are widely used for:

- Linear transformations.
- Solving linear systems.
- Computer graphics.
- Machine learning.
- Scientific simulation.
- Signal processing.

## 4.2 Matrix-vector multiplication

For an \(m\times n\) matrix \(A\) and a vector \(\mathbf{x}\) of length \(n\):

\[
\mathbf{y}=A\mathbf{x}
\]

where:

\[
y_i=\sum_{j=1}^{n}a_{ij}x_j
\]

Each output element is a dot product.

### Worked example

\[
A=
\begin{bmatrix}
1&2\\
3&4
\end{bmatrix}
\]

\[
\mathbf{x}=
\begin{bmatrix}
5\\6
\end{bmatrix}
\]

Then:

\[
A\mathbf{x}=
\begin{bmatrix}
1(5)+2(6)\\
3(5)+4(6)
\end{bmatrix}
=
\begin{bmatrix}
17\\39
\end{bmatrix}
\]

Each row produces one output value.

This suggests a natural parallel decomposition.

## 4.3 Matrix multiplication

Given:

\[
A\in\mathbb{R}^{m\times k}
\]

and:

\[
B\in\mathbb{R}^{k\times n}
\]

their product is:

\[
C=AB
\]

where:

\[
C_{ij}=\sum_{p=1}^{k}A_{ip}B_{pj}
\]

### Worked example

\[
A=
\begin{bmatrix}
1&2\\
3&4
\end{bmatrix}
\]

\[
B=
\begin{bmatrix}
5&6\\
7&8
\end{bmatrix}
\]

Then:

\[
AB=
\begin{bmatrix}
19&22\\
43&50
\end{bmatrix}
\]

For example:

\[
C_{11}=1(5)+2(7)=19
\]

## 4.4 Matrix multiplication is not commutative

In general:

\[
AB\neq BA
\]

Matrix multiplication order matters.

This is important when composing transformations.

## 4.5 Matrix transpose

The transpose exchanges rows and columns:

\[
(A^T)_{ij}=A_{ji}
\]

For:

\[
A=
\begin{bmatrix}
1&2&3\\
4&5&6
\end{bmatrix}
\]

\[
A^T=
\begin{bmatrix}
1&4\\
2&5\\
3&6
\end{bmatrix}
\]

## 4.6 Identity matrix

The identity matrix \(I\) satisfies:

\[
AI=A
\]

and:

\[
IA=A
\]

for compatible dimensions.

## 4.7 Linear transformations

A linear transformation preserves vector addition and scalar multiplication.

For a linear map \(T\):

\[
T(\mathbf{a}+\mathbf{b})=T(\mathbf{a})+T(\mathbf{b})
\]

and:

\[
T(c\mathbf{a})=cT(\mathbf{a})
\]

Matrices can represent linear transformations.

## 4.8 Eigenvalues and eigenvectors

An eigenvector \(\mathbf{v}\neq0\) of a matrix \(A\) satisfies:

\[
A\mathbf{v}=\lambda\mathbf{v}
\]

where \(\lambda\) is an eigenvalue.

Eigenvalues and eigenvectors are useful in:

- Numerical linear algebra.
- Principal component analysis.
- Dynamical systems.
- Scientific computing.

You should understand the concept before implementing advanced eigenvalue algorithms.

## 4.9 Memory layout

Matrices can be stored in different layouts.

Two common layouts are:

- Row-major.
- Column-major.

For a row-major matrix with \(n\) columns:

\[
\operatorname{index}(i,j)=in+j
\]

For a column-major matrix with \(m\) rows:

\[
\operatorname{index}(i,j)=jm+i
\]

The mathematical matrix is the same.

The memory access pattern is different.

## Exercises

1. Multiply a matrix by a vector.
2. Multiply two small matrices.
3. Explain why matrix multiplication is not commutative.
4. Compute a matrix transpose.
5. Explain the identity matrix.
6. Derive the row-major indexing formula.
7. Derive the column-major indexing formula.
8. Explain why matrix layout affects cache locality.
9. Explain why each output element of matrix multiplication can be computed independently.
10. Identify the reductions involved in matrix multiplication.

## Coding Lab 2: Matrix Library

Extend your Rust library with:

- Matrix creation.
- Matrix addition.
- Matrix transpose.
- Matrix-vector multiplication.
- Matrix-matrix multiplication.
- Row-major storage.
- Dimension validation.

Use a contiguous memory representation.

Document your indexing formulas.

---

# 5. Module Three: Floating-Point Arithmetic

Floating-point arithmetic is one of the most important subjects in numerical GPU programming.

## 5.1 Why floating-point numbers exist

Computers have finite storage.

Real numbers have infinitely many possible values.

Therefore, a computer cannot represent every real number exactly.

Floating-point formats approximate a large range of values using a finite number of bits.

## 5.2 Scientific notation

A real number can be expressed as:

\[
x=m\times b^e
\]

where:

- \(m\) is the significand.
- \(b\) is the base.
- \(e\) is the exponent.

Binary floating-point arithmetic typically uses base 2.

## 5.3 IEEE 754 formats

Common binary formats include:

| Format | Total bits | Exponent bits | Fraction bits |
|---|---:|---:|---:|
| binary16 | 16 | 5 | 10 |
| binary32 | 32 | 8 | 23 |
| binary64 | 64 | 11 | 52 |

For normal binary floating-point numbers, the significand has an implicit leading bit.

This provides an additional bit of precision.

## 5.4 Representable numbers

Not every decimal fraction has an exact binary representation.

For example, \(0.1\) cannot be represented exactly in ordinary binary floating-point formats.

The stored value is a nearby approximation.

This is why:

\[
0.1+0.2
\]

may not compare exactly equal to:

\[
0.3
\]

in floating-point arithmetic.

## 5.5 Rounding

Floating-point operations generally produce a representable approximation of the exact mathematical result.

Common rounding modes include:

- Round to nearest, ties to even.
- Round toward zero.
- Round toward positive infinity.
- Round toward negative infinity.

The exact behavior depends on the operation, format, and execution environment.

## 5.6 Machine epsilon

Machine epsilon is related to the spacing between representable floating-point numbers near 1.

For binary32, the gap between 1 and the next larger representable number is:

\[
2^{-23}
\]

For binary64, it is:

\[
2^{-52}
\]

This is one reason double precision can represent nearby values more accurately than single precision.

## 5.7 Subnormal numbers

Subnormal numbers allow representation of values closer to zero than the smallest positive normal number.

They provide gradual underflow.

Some hardware or execution modes may handle subnormal values differently for performance reasons.

## 5.8 Infinity and NaN

IEEE 754 defines special values such as:

- Positive infinity.
- Negative infinity.
- NaN: Not a Number.

NaN can arise from invalid arithmetic operations.

NaN has unusual comparison behavior.

For example, a NaN does not compare equal to itself.

## 5.9 Signed zero

Floating-point formats can distinguish positive zero and negative zero.

Although they compare equal numerically, certain operations can produce different results depending on the sign.

## 5.10 Fused multiply-add

Fused multiply-add computes:

\[
a\times b+c
\]

with a single final rounding in the ideal IEEE fused-operation model.

This can improve accuracy compared with separately rounded multiplication and addition.

It can also change results relative to an implementation that performs the operations separately.

## Exercises

1. Explain why floating-point numbers are approximate.
2. Explain the exponent and significand.
3. Explain why \(0.1\) cannot be represented exactly in binary.
4. Define machine epsilon.
5. Explain the difference between binary32 and binary64.
6. Explain subnormal numbers.
7. Explain infinity and NaN.
8. Explain signed zero.
9. Explain why fused multiply-add can produce a different result from separate multiplication and addition.
10. Explain why floating-point behavior matters for parallel algorithms.

## Coding Lab 3: Floating-Point Explorer

Build a program that investigates:

- Representable values.
- Machine epsilon.
- Rounding behavior.
- Underflow.
- Overflow.
- NaN.
- Infinity.
- Signed zero.
- Floating-point addition order.
- Fused multiply-add.

The program should produce reproducible experiments and written observations.

---

# 6. Module Four: Numerical Error and Stability

A numerical algorithm must be evaluated by more than whether it terminates.

It must also produce sufficiently accurate results.

## 6.1 Absolute error

Absolute error is:

\[
E_{\text{abs}}=|x-\hat{x}|
\]

where:

- \(x\) is the exact or reference value.
- \(\hat{x}\) is the computed approximation.

### Worked example

If:

\[
x=10
\]

and:

\[
\hat{x}=9.9
\]

then:

\[
E_{\text{abs}}=0.1
\]

## 6.2 Relative error

Relative error is:

\[
E_{\text{rel}}=
\frac{|x-\hat{x}|}{|x|}
\]

for \(x\neq0\).

Relative error measures error relative to the magnitude of the reference value.

Near zero, relative error can become misleading or undefined.

## 6.3 Forward error

Forward error measures the difference between the computed output and the exact output.

For a function \(f\):

\[
E_{\text{forward}}=
|\hat{y}-f(x)|
\]

where \(\hat{y}\) is the computed result.

## 6.4 Backward error

Backward error asks:

How much would the input need to change for the computed result to be exact?

This is useful because a numerical algorithm may produce an exact solution to a nearby problem.

## 6.5 Conditioning

Conditioning describes how sensitive a mathematical problem is to small changes in its input.

A well-conditioned problem changes only slightly when its inputs change slightly.

An ill-conditioned problem may change substantially.

Conditioning is a property of the problem.

## 6.6 Numerical stability

Numerical stability describes how an algorithm behaves in the presence of rounding and other numerical errors.

Stability is a property of the algorithm and its implementation.

A well-conditioned problem can still be solved poorly by an unstable algorithm.

An ill-conditioned problem may be difficult even for a stable algorithm.

## 6.7 Catastrophic cancellation

Catastrophic cancellation occurs when subtracting nearly equal values causes significant loss of relative precision.

Consider:

\[
a=1.000001
\]

\[
b=1.000000
\]

Their difference is:

\[
a-b=0.000001
\]

If the stored values have already been rounded, the small difference may be highly sensitive to those errors.

## 6.8 Stable algebraic reformulation

Sometimes an expression can be rewritten to reduce numerical error.

For example, expressions involving subtraction of nearly equal quantities may have mathematically equivalent formulations with better numerical behavior.

The correct formulation depends on the problem.

## 6.9 Error accumulation

Repeated arithmetic operations introduce rounding errors.

The way errors accumulate depends on:

- Operation order.
- Input magnitudes.
- Precision.
- Algorithm structure.
- Conditioning.
- Rounding behavior.

## 6.10 Testing numerical software

Numerical tests should consider:

- Absolute tolerance.
- Relative tolerance.
- Special values.
- Boundary cases.
- Input scaling.
- Reference precision.
- Reproducibility.

An approximate comparison may use a criterion such as:

\[
|a-b|
\leq
\epsilon_{\text{abs}}
+
\epsilon_{\text{rel}}\max(|a|,|b|)
\]

But tolerances should be chosen based on the numerical problem, not selected arbitrarily.

## Exercises

1. Calculate absolute error.
2. Calculate relative error.
3. Explain why relative error is problematic near zero.
4. Explain forward error.
5. Explain backward error.
6. Distinguish conditioning from stability.
7. Explain catastrophic cancellation.
8. Give an example of error accumulation.
9. Explain why exact equality is often unsuitable for floating-point tests.
10. Explain why numerical tolerances must depend on the problem.

## Coding Lab 4: Numerical Error Analyzer

Implement a tool that:

- Compares floating-point results.
- Computes absolute error.
- Computes relative error.
- Handles zero and special values.
- Supports multiple precisions.
- Reports error distributions.
- Tests numerical algorithms against reference implementations.

Document the limitations of each error metric.

---

# 7. Module Five: Reductions and Parallel Numerical Algorithms

A reduction combines many values into a smaller result, often a single scalar.

Examples include:

- Sum.
- Minimum.
- Maximum.
- Product.
- Logical AND.
- Logical OR.

Reductions are central to GPU programming.

## 7.1 Sequential reduction

Consider:

\[
S=\sum_{i=1}^{N}x_i
\]

A sequential algorithm processes the values one after another.

Its work complexity is:

\[
O(N)
\]

Its dependency chain is also approximately \(O(N)\).

## 7.2 Parallel reduction

A parallel reduction can combine independent pairs.

For eight values:

```text id="8wlyks"
Level 0: a b c d e f g h
Level 1:  ab  cd  ef  gh
Level 2:    abcd    efgh
Level 3:       total
```

At each level, independent operations can execute simultaneously.

The total number of additions remains:

\[
N-1
\]

for a full binary reduction of \(N\) values.

But the critical path can be reduced to approximately:

\[
O(\log N)
\]

assuming sufficient parallel resources and a balanced tree.

## 7.3 Work versus span

Parallel algorithms are often analyzed using:

- Work: Total operations performed.
- Span: Length of the longest dependency chain.

For a sequential sum:

\[
W=O(N)
\]

\[
S=O(N)
\]

For a balanced tree reduction:

\[
W=O(N)
\]

\[
S=O(\log N)
\]

This distinction is fundamental for parallel algorithm design.

## 7.4 Floating-point associativity

A parallel reduction changes the grouping of additions.

For exact arithmetic, this does not change the result.

For floating-point arithmetic, it can.

Therefore, a parallel reduction may produce a different result from a sequential reduction.

The difference does not automatically indicate a bug.

## 7.5 Pairwise summation

Pairwise summation combines values in a tree-like structure.

It can often produce better numerical accuracy than straightforward sequential accumulation, depending on the inputs and precision.

## 7.6 Compensated summation

Compensated summation algorithms, such as Kahan summation, track some of the rounding error introduced during accumulation.

They can improve accuracy for certain workloads.

However, they introduce additional arithmetic and dependencies.

This creates a trade-off between numerical accuracy and performance.

## 7.7 Reduction identities

Many reductions have an identity element.

For addition:

\[
x+0=x
\]

For multiplication:

\[
x\times1=x
\]

For maximum over a suitable domain, an identity can be a value smaller than every valid input.

Identity elements are important when dividing work into independent groups.

## 7.8 Associative operations

An operation is associative when:

\[
(a\circ b)\circ c=a\circ(b\circ c)
\]

Associativity allows flexible grouping.

Floating-point addition is not strictly associative, so parallel floating-point reductions require explicit numerical expectations.

## Exercises

1. Explain a reduction.
2. Derive the work complexity of a sequential sum.
3. Derive the span of a balanced tree reduction.
4. Explain why a tree reduction can execute in parallel.
5. Explain why reduction order affects floating-point results.
6. Explain pairwise summation.
7. Explain compensated summation.
8. Explain the role of an identity element.
9. Explain the difference between work and span.
10. Explain why parallelism does not automatically guarantee speedup.

## Coding Lab 5: Reduction Benchmark

Implement and compare:

- Sequential summation.
- Pairwise summation.
- Compensated summation.
- A multithreaded reduction.

Measure:

- Runtime.
- Throughput.
- Numerical error.
- Scaling with input size.
- Scaling with thread count.

Use a higher-precision reference where appropriate.

Explain both performance and numerical differences.

---

# 8. Module Six: Matrix Multiplication and Performance

Matrix multiplication is one of the most important GPU workloads.

It appears in:

- Machine learning.
- Scientific computing.
- Graphics.
- Physics simulation.
- Signal processing.

## 8.1 Naive matrix multiplication

For square matrices:

\[
C=AB
\]

where:

\[
C_{ij}=\sum_{k=1}^{N}A_{ik}B_{kj}
\]

A straightforward implementation uses three nested loops.

Its arithmetic complexity is:

\[
O(N^3)
\]

## 8.2 Floating-point operation count

For each output element:

- Approximately \(N\) multiplications.
- Approximately \(N-1\) additions.

For an \(N\times N\) output:

\[
\text{FLOPs}\approx2N^3
\]

This approximation is commonly used in performance analysis.

## 8.3 Why naive implementations can be slow

A naive implementation may repeatedly load the same matrix elements from memory.

Performance depends heavily on:

- Loop order.
- Cache locality.
- Data layout.
- Vectorization.
- Register usage.
- Memory bandwidth.

## 8.4 Loop ordering

Matrix multiplication can be written using different loop orders.

Examples:

- `i-j-k`.
- `i-k-j`.
- `j-i-k`.
- `k-i-j`.

Mathematically, these can represent the same operation.

But they may have different memory access patterns and floating-point accumulation orders.

## 8.5 Blocking and tiling

Blocking divides matrices into smaller regions called tiles.

Instead of processing entire rows or columns repeatedly, the algorithm works on blocks that can fit into faster memory.

The goal is to reuse loaded data.

## 8.6 Arithmetic intensity

Matrix multiplication performs many arithmetic operations relative to the amount of data that must be transferred when data reuse is effective.

This makes it a strong candidate for compute-intensive GPU acceleration.

However, a poorly implemented matrix multiplication may still be limited by memory movement.

## 8.7 Register blocking

Small portions of matrix data can sometimes be held in registers.

This reduces repeated memory accesses.

The trade-off is increased register usage.

On GPUs, register usage can influence occupancy and scheduling.

## 8.8 GPU connection

Later, you will implement tiled matrix multiplication using GPU workgroups.

You will need to understand:

- Thread indexing.
- Tile dimensions.
- Shared memory.
- Synchronization.
- Memory coalescing.
- Register pressure.
- Arithmetic intensity.

The mathematics in this module will make those implementations easier to reason about.

## Exercises

1. Derive the \(O(N^3)\) complexity of naive matrix multiplication.
2. Estimate the floating-point operation count.
3. Explain why loop order matters.
4. Explain blocking.
5. Explain tiling.
6. Explain arithmetic intensity.
7. Explain why matrix multiplication benefits from data reuse.
8. Explain why register usage matters.
9. Explain how numerical accumulation order can affect results.
10. Explain why theoretical FLOPs do not guarantee high performance.

## Coding Lab 6: Matrix Multiplication Benchmark

Implement:

- Naive matrix multiplication.
- Alternative loop orderings.
- Blocked matrix multiplication.
- Multithreaded matrix multiplication.

Compare:

- Correctness.
- Runtime.
- FLOPs per second.
- Cache behavior.
- Scaling with matrix dimensions.
- Numerical differences.

Do not use BLAS for the implementation.

You may use a trusted library as a reference for validation.

---

# 9. Module Seven: Numerical Methods

GPU programming frequently involves solving numerical problems that do not have simple closed-form solutions.

This module introduces several important categories.

## 9.1 Numerical differentiation

The derivative of a function is:

\[
f'(x)=
\lim_{h\to0}
\frac{f(x+h)-f(x)}{h}
\]

A forward-difference approximation is:

\[
f'(x)\approx
\frac{f(x+h)-f(x)}{h}
\]

A central-difference approximation is:

\[
f'(x)\approx
\frac{f(x+h)-f(x-h)}{2h}
\]

The choice of \(h\) involves a trade-off.

If \(h\) is too large, truncation error can dominate.

If \(h\) is too small, floating-point rounding and cancellation can dominate.

## 9.2 Numerical integration

Numerical integration approximates the area under a function.

For example, the trapezoidal rule approximates:

\[
\int_a^b f(x)\,dx
\]

using a collection of trapezoids.

For evenly spaced samples:

\[
h=\frac{b-a}{N}
\]

\[
\int_a^b f(x)\,dx
\approx
h\left[
\frac{f(a)+f(b)}{2}
+
\sum_{i=1}^{N-1}f(a+ih)
\right]
\]

The sum can be parallelized.

## 9.3 Root finding

Root-finding algorithms solve:

\[
f(x)=0
\]

Common methods include:

- Bisection.
- Newton's method.
- Secant method.

Newton's method uses:

\[
x_{n+1}
=
x_n-\frac{f(x_n)}{f'(x_n)}
\]

It can converge rapidly under suitable conditions but may fail or behave poorly when assumptions are violated.

## 9.4 Iterative linear solvers

Large linear systems often appear in scientific computing:

\[
A\mathbf{x}=\mathbf{b}
\]

For sufficiently large or sparse systems, direct methods may be expensive.

Iterative methods approximate the solution through repeated updates.

Examples include:

- Jacobi iteration.
- Gauss-Seidel iteration.
- Conjugate gradient for suitable systems.

## 9.5 Jacobi iteration

For a system:

\[
A\mathbf{x}=\mathbf{b}
\]

Jacobi iteration computes new values using the previous iteration's values.

For row \(i\):

\[
x_i^{(k+1)}
=
\frac{1}{a_{ii}}
\left(
b_i-\sum_{j\neq i}a_{ij}x_j^{(k)}
\right)
\]

Each new component depends on the previous iteration.

This makes many updates independently computable.

The method's convergence depends on properties of the matrix.

## 9.6 Convergence

An iterative algorithm converges when its approximations approach the desired solution under appropriate conditions.

A stopping criterion might involve:

- Residual norm.
- Change between iterations.
- Maximum iteration count.
- Problem-specific tolerance.

## 9.7 Residuals

For:

\[
A\mathbf{x}=\mathbf{b}
\]

the residual is:

\[
\mathbf{r}=\mathbf{b}-A\mathbf{x}
\]

The residual indicates how closely the computed solution satisfies the original equation.

A small residual does not always imply a small solution error if the problem is ill-conditioned.

## Exercises

1. Derive a forward-difference approximation.
2. Explain why decreasing \(h\) does not always improve accuracy.
3. Explain the trapezoidal rule.
4. Explain bisection.
5. Explain Newton's method.
6. Explain why Newton's method can fail.
7. Explain iterative linear solvers.
8. Explain Jacobi iteration.
9. Explain convergence.
10. Explain residuals.
11. Explain why a small residual does not always imply a small solution error.
12. Identify parallelizable operations in numerical integration and iterative solvers.

## Coding Lab 7: Numerical Methods Toolkit

Implement:

- Numerical differentiation.
- Trapezoidal integration.
- Bisection root finding.
- Newton's method.
- Jacobi iteration.

For each algorithm:

- Define valid inputs.
- Define failure conditions.
- Document stopping criteria.
- Measure numerical error.
- Test convergence.
- Compare against trusted reference results.

---

# 10. Module Eight: Probability, Statistics, and Numerical Experiments

Performance engineering relies on measurement.

Measurements are noisy.

You need enough statistics to interpret them correctly.

## 10.1 Mean

For observations:

\[
x_1,x_2,\ldots,x_n
\]

the arithmetic mean is:

\[
\bar{x}=\frac{1}{n}\sum_{i=1}^{n}x_i
\]

## 10.2 Variance

Sample variance is:

\[
s^2=
\frac{1}{n-1}
\sum_{i=1}^{n}(x_i-\bar{x})^2
\]

for \(n>1\).

Variance measures dispersion.

## 10.3 Standard deviation

Standard deviation is:

\[
s=\sqrt{s^2}
\]

It expresses variability in the same units as the observations.

## 10.4 Median

The median is the middle observation after sorting.

It is less sensitive to extreme outliers than the mean.

## 10.5 Percentiles

Percentiles describe positions in a distribution.

For example, the 95th percentile is a value at or below which approximately 95% of observations fall, depending on the percentile definition used.

Percentiles are useful for analyzing latency distributions.

## 10.6 Benchmark noise

Benchmark measurements may vary because of:

- CPU frequency changes.
- Thermal behavior.
- Cache state.
- Scheduling.
- Background activity.
- Memory allocation.
- Operating-system behavior.

Repeated measurements help characterize this variability.

## 10.7 Confidence intervals

A confidence interval describes a procedure that, under its statistical assumptions, produces intervals containing the true parameter at a specified long-run frequency.

A 95% confidence interval is not generally a 95% probability that the fixed parameter lies inside the particular interval after it has been computed.

## 10.8 Experimental design

Good numerical experiments should:

1. State a hypothesis.
2. Control relevant variables.
3. Use repeatable inputs.
4. Record the environment.
5. Collect enough measurements.
6. Analyze variability.
7. Avoid unsupported causal conclusions.

## Exercises

1. Calculate the mean of a dataset.
2. Calculate sample variance.
3. Calculate standard deviation.
4. Explain the median.
5. Explain percentiles.
6. Explain why benchmark distributions matter.
7. Explain the difference between variability and systematic bias.
8. Explain why a single benchmark run is insufficient.
9. Explain what a confidence interval does and does not mean.
10. Design a controlled performance experiment.

---

# 11. Major Coding Assessment A: Numerical Computing Library

Build a Rust numerical computing library.

## Required functionality

### Vector operations

- Addition.
- Subtraction.
- Scaling.
- Dot product.
- Norms.

### Matrix operations

- Addition.
- Transpose.
- Matrix-vector multiplication.
- Matrix-matrix multiplication.

### Numerical utilities

- Absolute error.
- Relative error.
- Approximate equality.
- Handling of NaN and infinity.

### Reductions

- Sequential sum.
- Pairwise sum.
- Compensated sum.

## Engineering requirements

- Clear APIs.
- Explicit dimension validation.
- No undefined behavior.
- Comprehensive tests.
- Meaningful error handling.
- Performance benchmarks.
- Documentation of numerical limitations.

## Technical defense

You must explain:

1. Why your data structures use their chosen memory layout.
2. How indexing works.
3. Which operations are parallelizable.
4. How floating-point errors arise.
5. Why reduction order matters.
6. How numerical correctness is validated.
7. How performance is measured.

---

# 12. Major Coding Assessment B: Matrix Multiplication Optimization

Build a CPU matrix multiplication benchmark suite.

## Implementations

Implement at least:

1. A baseline matrix multiplication.
2. Two alternative loop orderings.
3. A blocked implementation.
4. A multithreaded implementation.

## Measurements

Record:

- Matrix dimensions.
- Runtime.
- Floating-point throughput.
- Memory usage.
- Compiler settings.
- CPU characteristics.
- Numerical differences.

## Analysis requirements

Explain:

- Which implementation is fastest.
- Why memory layout matters.
- Why blocking improves data reuse.
- Whether vectorization occurs.
- How thread count affects performance.
- What limits further speedup.
- How the results relate to the Roofline model.

## Constraints

Do not copy an existing optimized matrix multiplication implementation.

Develop your own approach and defend your design decisions.

---

# 13. Major Coding Assessment C: Numerical Stability Investigation

Build an experimental program that compares numerical algorithms.

## Required experiments

Investigate:

1. Sequential versus pairwise summation.
2. Compensated summation.
3. Floating-point addition order.
4. Numerical differentiation step size.
5. Iterative solver convergence.
6. Single precision versus double precision.

## Required analysis

For each experiment:

- State a hypothesis.
- Define the reference result.
- Measure error.
- Record input characteristics.
- Explain observed behavior.
- Identify limitations.
- Explain whether the result generalizes.

The objective is not merely to show that floating-point errors exist.

It is to understand when they matter and how to reason about them.

---

# 14. Final Capstone: A Numerical Performance Laboratory

Combine your work into a reusable numerical computing and benchmarking project.

## Required capabilities

Your application must support:

- Vector operations.
- Matrix operations.
- Reductions.
- Numerical error analysis.
- Configurable input sizes.
- Multiple algorithm implementations.
- Benchmarking.
- Structured result output.

## Required reports

Produce three reports:

### Report 1: Numerical correctness

Explain how you validate numerical results and choose tolerances.

### Report 2: Performance

Explain how memory layout, arithmetic intensity, and parallelism affect runtime.

### Report 3: Accuracy versus performance

Compare algorithms that trade numerical accuracy for computational efficiency.

## Technical defense

Be prepared to explain:

- Every major algorithm.
- Every important numerical assumption.
- The source of rounding errors.
- The expected computational complexity.
- The memory access patterns.
- The parallel structure.
- The benchmark methodology.
- The limitations of your conclusions.

---

# 15. Phase 5 Quiz Bank

## Linear algebra

1. What is a vector?
2. What is a vector norm?
3. What is a dot product?
4. What is orthogonality?
5. What is vector projection?
6. What is a matrix?
7. What is matrix multiplication?
8. Why is matrix multiplication not commutative?
9. What is a matrix transpose?
10. What is a linear transformation?
11. What is an eigenvector?
12. What is an eigenvalue?

## Floating-point arithmetic

13. Why are floating-point numbers approximate?
14. What is a significand?
15. What is an exponent?
16. What is machine epsilon?
17. Why is \(0.1\) not exactly representable in binary floating point?
18. What is rounding?
19. What is underflow?
20. What is overflow?
21. What is a subnormal number?
22. What is NaN?
23. What is signed zero?
24. What is fused multiply-add?

## Numerical error

25. What is absolute error?
26. What is relative error?
27. What is forward error?
28. What is backward error?
29. What is conditioning?
30. What is numerical stability?
31. What is catastrophic cancellation?
32. Why can subtraction be numerically dangerous?
33. Why can floating-point equality tests be unreliable?
34. How should numerical tolerances be selected?

## Parallel numerical algorithms

35. What is a reduction?
36. What is the work complexity of sequential summation?
37. What is the span of a balanced tree reduction?
38. Why does parallel reduction change addition order?
39. What is pairwise summation?
40. What is compensated summation?
41. What is an identity element?
42. What is associativity?
43. What is work?
44. What is span?

## Matrix multiplication

45. What is the computational complexity of naive matrix multiplication?
46. How many floating-point operations does it perform approximately?
47. Why does loop ordering matter?
48. What is blocking?
49. What is tiling?
50. What is arithmetic intensity?
51. Why is data reuse important?
52. Why does matrix multiplication map well to GPUs?

## Numerical methods

53. What is numerical differentiation?
54. What is numerical integration?
55. What is truncation error?
56. Why can a smaller step size increase rounding error?
57. What is root finding?
58. What is Newton's method?
59. What is Jacobi iteration?
60. What is convergence?
61. What is a residual?
62. Why can a small residual coexist with a large solution error?

## Statistics and benchmarking

63. What is a mean?
64. What is variance?
65. What is standard deviation?
66. What is a median?
67. What is a percentile?
68. Why are repeated measurements necessary?
69. What is benchmark noise?
70. What is a confidence interval?
71. Why must experiments control variables?
72. Why is performance an empirical property?

---

# 16. Final Examination

**Recommended duration:** 5–6 hours.

| Section | Weight |
|---|---:|
| Linear algebra | 15% |
| Floating-point arithmetic | 15% |
| Numerical error and stability | 20% |
| Reductions and parallel algorithms | 15% |
| Matrix multiplication | 15% |
| Numerical methods | 10% |
| Statistics and benchmarking | 10% |

## Part A: Mathematical derivations

Derive:

- Dot product.
- Matrix multiplication.
- Vector norms.
- Parallel reduction complexity.
- Matrix multiplication operation count.
- Arithmetic intensity for a simple workload.

## Part B: Floating-point reasoning

Analyze:

- Rounding.
- Non-associativity.
- Cancellation.
- Overflow and underflow.
- Precision trade-offs.

## Part C: Numerical stability

Evaluate hypothetical algorithms for:

- Conditioning.
- Forward error.
- Backward error.
- Stability.
- Appropriate validation methods.

## Part D: Parallel algorithms

Analyze:

- Work.
- Span.
- Dependency structure.
- Reduction trees.
- Numerical reproducibility.

## Part E: Performance

Explain:

- Memory layout.
- Data reuse.
- Arithmetic intensity.
- Tiling.
- Vectorization.
- Parallel scaling.

## Part F: Coding defense

Defend your numerical performance laboratory.

---

# 17. Copy-Ready AI Tutor Prompts

All prompts are consolidated here.

The tutor must never solve your assessments, write code, provide pseudocode, reveal answers, or perform work for you.

## Master Socratic Tutor Prompt

```text id="vp4oag"
You are my Socratic mathematics and numerical computing tutor.

I am studying Phase 5 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   mathematical derivations, or examinations.

2. Never solve equations or calculations for me.

3. Never write code, pseudocode, algorithms, or implementations.

4. Never modify files, execute code, debug programs, or complete
   any assessment work on my behalf.

5. Never reveal answer keys or hidden tests.

6. Never give hints that effectively disclose solutions.

7. Guide my reasoning exclusively through questions.

8. Ask one primary question per response.

9. If I am incorrect, ask a question that exposes the misconception.

10. If I am stuck, ask a simpler prerequisite question.

11. Require me to explain assumptions, derivations,
    numerical errors, and performance trade-offs.

12. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

13. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

14. For code review, ask questions about numerical correctness,
    error handling, memory layout, complexity, and performance.
    Never provide exact fixes.

15. Never provide a final answer during an assessment.

16. Keep responses concise and focused.

Your responses must contain only guiding questions.
```

## Start a lesson

```text id="7dtct0"
Tutor me through Phase 5, Module [NUMBER].

Start from first principles.
Ask one question at a time.
Require me to derive formulas rather than memorize them.
Never provide answers or solutions.
```

## Generate a quiz

```text id="4apxhh"
Administer a 20-question Phase 5 quiz on [TOPIC].

Include conceptual reasoning, mathematical derivations,
numerical analysis, and performance questions.

Ask one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text id="p0b72g"
Administer Phase 5 Coding Assessment [A/B/C].

Present requirements and constraints only.
Ask me to explain my mathematical approach and design.

Guide me through questions about correctness, numerical stability,
error analysis, testing, memory layout, and performance.

Do not write code, pseudocode, or implementations.
```

## Review numerical results

```text id="14qx6a"
Help me investigate my numerical results using only questions.

Ask about:
- Reference values
- Absolute and relative error
- Conditioning
- Numerical stability
- Floating-point precision
- Operation order
- Convergence
- Input characteristics

Do not interpret the results for me or provide conclusions.
```

## Mathematical oral examination

```text id="mvn7xd"
Examine my understanding of Phase 5 mathematics.

Ask me to derive formulas and justify each step.
Challenge incorrect assumptions through questions.

Do not perform calculations or provide answers.
```

## Performance assessment

```text id="o6fbkv"
Examine my understanding of numerical performance.

Ask questions about:
- Arithmetic intensity
- Memory bandwidth
- Cache locality
- Matrix tiling
- Reductions
- Work and span
- Parallel scaling
- Benchmark methodology

Ask one question at a time.
Never provide answers or solutions.
```

---

# 18. Mastery Checklist

## Linear algebra

- [ ] I understand vectors and matrices.
- [ ] I can compute dot products.
- [ ] I understand vector norms.
- [ ] I understand matrix-vector multiplication.
- [ ] I understand matrix multiplication.
- [ ] I understand linear transformations.
- [ ] I understand eigenvalues and eigenvectors conceptually.

## Floating-point arithmetic

- [ ] I understand IEEE 754 binary floating-point formats.
- [ ] I understand rounding.
- [ ] I understand machine epsilon.
- [ ] I understand subnormal numbers.
- [ ] I understand NaN and infinity.
- [ ] I understand fused multiply-add.
- [ ] I understand floating-point non-associativity.

## Numerical analysis

- [ ] I can calculate absolute error.
- [ ] I can calculate relative error.
- [ ] I understand conditioning.
- [ ] I understand numerical stability.
- [ ] I understand catastrophic cancellation.
- [ ] I can design numerical correctness tests.

## Parallel algorithms

- [ ] I understand reductions.
- [ ] I understand work and span.
- [ ] I understand parallel reduction trees.
- [ ] I understand pairwise summation.
- [ ] I understand compensated summation.
- [ ] I understand numerical reproducibility issues.

## Matrix performance

- [ ] I understand naive matrix multiplication.
- [ ] I understand loop ordering.
- [ ] I understand blocking and tiling.
- [ ] I understand arithmetic intensity.
- [ ] I understand data reuse.
- [ ] I can benchmark matrix multiplication implementations.

## Numerical methods

- [ ] I understand numerical differentiation.
- [ ] I understand numerical integration.
- [ ] I understand root finding.
- [ ] I understand iterative solvers.
- [ ] I understand convergence.
- [ ] I understand residuals.

## Experimental methodology

- [ ] I understand means and variances.
- [ ] I understand benchmark noise.
- [ ] I can design controlled experiments.
- [ ] I can compare numerical algorithms.
- [ ] I can explain accuracy-versus-performance trade-offs.

---

# 19. Recommended Resources

## Linear algebra

**Introduction to Linear Algebra — Gilbert Strang**

A foundational treatment of vectors, matrices, and linear transformations.

**MIT OpenCourseWare: Linear Algebra**

https://ocw.mit.edu/

## Numerical analysis

**Numerical Analysis — Richard L. Burden and J. Douglas Faires**

Covers numerical methods, approximation, and error analysis.

**Accuracy and Stability of Numerical Algorithms — Nicholas J. Higham**

A deeper reference for floating-point error and numerical stability.

## Floating-point arithmetic

**What Every Computer Scientist Should Know About Floating-Point Arithmetic — David Goldberg**

A classic introduction to floating-point behavior.

**IEEE 754 Standard**

The primary standard defining floating-point arithmetic behavior.

## Parallel numerical computing

**Introduction to Parallel Computing — Ananth Grama and others**

Useful for work/span reasoning and parallel algorithms.

**The Art of Multiprocessor Programming — Maurice Herlihy and Nir Shavit**

Useful for broader parallel computation and synchronization concepts.

## Numerical libraries

Study the documentation and APIs of:

- BLAS.
- LAPACK.
- Eigen.
- nalgebra.
- ndarray.

Do not rely on these libraries to complete your foundational implementations.

Use them to understand mature numerical interfaces and validate your work.

---

# 20. Completion Standard

You have completed Phase 5 when you can independently:

1. Derive and implement fundamental vector and matrix operations.
2. Explain floating-point representation and rounding.
3. Analyze numerical errors.
4. Distinguish conditioning from stability.
5. Implement numerically aware reductions.
6. Explain parallel reduction work and span.
7. Implement and benchmark matrix multiplication.
8. Explain tiling and data reuse.
9. Implement basic numerical methods.
10. Design reliable numerical experiments.
11. Defend your accuracy and performance decisions.
12. Explain how these concepts transfer to GPU programming.

The goal is not to memorize formulas.

The goal is to understand how mathematical computation behaves under finite precision, limited memory bandwidth, and parallel execution.

---

# 21. What Comes Next

**Phase 6: Concurrency and Parallel Algorithms**

The next phase will focus on:

- Parallel computation models.
- Threads and task parallelism.
- Work and span analysis.
- Parallel reductions.
- Prefix sums.
- Parallel sorting.
- Synchronization.
- Lock-free programming.
- Memory models.
- Work stealing.
- Parallel algorithm design.
- Performance scaling.
- Amdahl's Law and Gustafson's Law.

Phase 6 will connect the mathematical foundations of Phase 5 with the execution models you will encounter in GPU programming.
