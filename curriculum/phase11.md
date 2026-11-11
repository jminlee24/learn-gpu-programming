# Phase 11: Advanced Parallel Algorithms and GPU Data Structures

**Curriculum:** GPU Engineering — Complete Self-Study Program  
**Phase:** 11 of 16  
**Estimated duration:** 16–22 weeks  
**Study commitment:** 10–12 hours per week  
**Primary languages:** Rust and WGSL  
**Supporting languages:** C++20 and CUDA C++  
**Primary GPU:** AMD Radeon RX 9070, 16 GB  
**Platform:** Fedora Linux  
**Prerequisites:** Phases 0–10

---

# 1. Mission and Learning Objectives

A GPU is especially effective when thousands of independent operations perform similar work on predictable data.

Advanced parallel algorithms are difficult because real workloads frequently violate those assumptions.

Consider graph traversal. Some vertices have two neighbors, while others have thousands. One thread may finish almost immediately while another performs substantially more work. Neighbor addresses may be scattered throughout memory, and multiple threads may attempt to discover the same vertex.

The algorithm must handle all of these conditions without losing correctness.

This phase develops four core capabilities:

1. **Algorithmic decomposition:** Identify the work, dependencies, and synchronization boundaries.
2. **Parallel data movement:** Use scans, sorting, partitioning, and compaction to reorganize data efficiently.
3. **Irregular computation:** Manage variable work, sparse data, contention, and load imbalance.
4. **Performance engineering:** Select algorithms and data structures based on measured hardware behavior.

## Learning outcomes

By the end of Phase 11, you should be able to:

- Derive work and span bounds for advanced parallel algorithms.
- Implement work-efficient scans and segmented scans.
- Build stable stream-compaction pipelines.
- Implement GPU histograms and radix sorting.
- Explain parallel merge and comparison-based sorting.
- Work with sparse matrix representations.
- Implement sparse matrix-vector multiplication.
- Understand sparse matrix-matrix multiplication.
- Design parallel breadth-first search.
- Explain frontier-based and direction-optimizing graph traversal.
- Understand GPU work queues.
- Design parallel hash tables.
- Analyze contention and atomic progress.
- Apply load-balancing strategies to irregular workloads.
- Understand parallel dynamic programming and stencil computations.
- Use multi-dispatch algorithms correctly.
- Validate nondeterministic parallel implementations.
- Benchmark advanced GPU algorithms against CPU references.
- Build and defend a GPU algorithm library.

## Required engineering standard

Every major algorithm must have:

- A clearly defined input/output contract.
- A correct sequential reference implementation.
- A parallel correctness argument.
- Complexity analysis.
- Edge-case tests.
- Performance measurements.
- Documented limitations.

A GPU implementation is not complete merely because it produces the expected result on one input.

---

# 2. Study Schedule

| Module | Subject | Duration |
|---|---|---|
| 1 | Advanced parallel algorithm models | 1–2 weeks |
| 2 | Scans, segmented scans, and compaction | 2–3 weeks |
| 3 | Histograms, radix sort, and partitioning | 2–3 weeks |
| 4 | Sparse data structures and sparse linear algebra | 2–3 weeks |
| 5 | Parallel graph algorithms | 2–3 weeks |
| 6 | Work queues, hashing, and load balancing | 2–3 weeks |
| 7 | Stencils and parallel dynamic programming | 1–2 weeks |
| 8 | Integration, capstone, and examination | 2–3 weeks |

## Weekly workflow

Allocate approximately:

- 3 hours to textbook study and derivations.
- 4 hours to independent implementation.
- 2 hours to testing and benchmarking.
- 1–3 hours to exercises, documentation, and technical defense.

Maintain separate CPU reference, GPU implementation, benchmark, and experimental-result modules.

---

# 3. Module One: Advanced Parallel Algorithm Design

## 3.1 Work and span revisited

Recall the two fundamental measures:

\[
W(n)=\text{total computational work}
\]

\[
S(n)=\text{longest dependency chain}
\]

For \(P\) processors, a lower bound on execution time is:

\[
T_P\geq\max\left(\frac{W}{P},S\right)
\]

Under suitable scheduling assumptions, a useful upper bound is:

\[
T_P=O\left(\frac{W}{P}+S\right)
\]

These expressions describe algorithmic potential, not guaranteed GPU performance.

Actual GPU execution introduces memory, scheduling, synchronization, and launch costs.

## 3.2 Work efficiency

A parallel algorithm is work-efficient when its total work is asymptotically comparable to a good sequential algorithm.

For example, an algorithm with:

\[
W(n)=O(n)
\]

is generally preferable to an otherwise comparable algorithm with:

\[
W(n)=O(n\log n)
\]

for sufficiently large inputs.

However, a theoretically work-inefficient algorithm may outperform a work-efficient algorithm for particular hardware and input sizes because of lower constants, better locality, or greater parallelism.

## 3.3 Parallel depth

Consider two algorithms:

\[
A: W(n)=O(n),\quad S(n)=O(n)
\]

\[
B: W(n)=O(n),\quad S(n)=O(\log n)
\]

Algorithm B exposes substantially more parallelism.

But the benefit depends on whether enough work exists to amortize GPU launch and coordination overhead.

## 3.4 Regular versus irregular parallelism

Regular algorithms have predictable work and memory access patterns.

Examples:

- Vector addition.
- Dense matrix multiplication.
- Fixed-radius stencils.

Irregular algorithms have data-dependent work or access patterns.

Examples:

- Graph traversal.
- Sparse matrix operations.
- Hash table insertion.
- Dynamic work queues.

## 3.5 Static versus dynamic work assignment

Static assignment distributes work in advance.

Dynamic assignment allocates work as execution proceeds.

Static assignment is often simpler and has lower scheduling overhead.

Dynamic assignment may handle variable work more effectively.

## 3.6 GPU synchronization boundaries

A GPU workgroup can coordinate internally through supported workgroup synchronization.

However, independent workgroups in a normal dispatch cannot assume they execute simultaneously or in a particular order.

Many global algorithms therefore require multiple dispatches.

A multi-dispatch algorithm can use the completion and ordering guarantees of the GPU command system to separate stages.

## 3.7 Data-parallel decomposition

Before implementing an algorithm, answer:

1. What is the smallest independent unit of work?
2. Which operations depend on other operations?
3. Which data is read by multiple threads?
4. Which data is written by multiple threads?
5. Which synchronization scopes are necessary?
6. Which stages can execute independently?
7. What is the expected memory traffic?
8. What is the limiting resource?

## Worked example: filtering an array

Suppose we want to retain only positive values.

Input:

\[
[-3,5,0,8,-1,4]
\]

Desired output:

\[
[5,8,4]
\]

The predicate evaluations are independent.

However, output positions depend on how many earlier elements satisfy the predicate.

Thus the computation contains:

- Independent predicate evaluation.
- A global prefix dependency.
- Independent writes once output positions are known.

Recognizing this structure leads naturally to scan-based compaction.

## Exercises

1. Define work efficiency.
2. Explain parallel depth.
3. Distinguish regular from irregular parallelism.
4. Explain static and dynamic work assignment.
5. Why are global synchronization boundaries important?
6. Identify the dependencies in array filtering.
7. Explain why algorithmic work efficiency does not guarantee best measured performance.
8. Describe a GPU algorithm whose performance depends strongly on input distribution.

---

# 4. Module Two: Advanced Prefix Scans

Prefix scans are among the most important building blocks in parallel computing.

## 4.1 Scan as a general operation

Let \(\oplus\) be an associative binary operation with identity \(e\).

For input:

\[
[x_0,x_1,\ldots,x_{n-1}]
\]

the exclusive scan is:

\[
y_i=e\oplus x_0\oplus\cdots\oplus x_{i-1}
\]

The inclusive scan is:

\[
z_i=x_0\oplus x_1\oplus\cdots\oplus x_i
\]

Examples of scan operations include:

- Addition.
- Multiplication.
- Minimum.
- Maximum.
- Logical operations.
- Certain matrix-composition operations.

Associativity is essential for regrouping operations without changing their mathematical result.

Floating-point addition is not exactly associative, so different scan trees can produce different rounded results.

## 4.2 Hillis–Steele scan

A classic parallel scan algorithm combines elements at exponentially increasing offsets.

At iteration \(k\), an element may incorporate information from a position \(2^k\) elements earlier.

The algorithm requires:

\[
O(\log n)
\]

parallel rounds.

However, each round can process \(O(n)\) elements.

Therefore:

\[
W(n)=O(n\log n)
\]

\[
S(n)=O(\log n)
\]

This algorithm is easy to understand and exposes substantial parallelism, but it is not work-efficient.

## 4.3 Blelloch scan

A work-efficient scan uses two phases:

1. Upsweep.
2. Downsweep.

For a power-of-two input length, the upsweep combines values into a reduction tree.

The downsweep propagates prefix information through the tree.

## 4.4 Work derivation

For \(n=2^k\), the upsweep performs:

\[
\frac n2+\frac n4+\cdots+1=n-1
\]

combination operations.

The downsweep performs \(O(n)\) additional operations.

Thus:

\[
W(n)=O(n)
\]

There are \(O(\log n)\) levels in each phase.

Therefore:

\[
S(n)=O(\log n)
\]

## 4.5 Hierarchical GPU scan

A large GPU scan may require several dispatches.

A common structure is:

- Scan each block's input segment.
- Record each block's total.
- Scan the block totals.
- Add the appropriate offset to each block's output.

This structure separates local synchronization from global coordination.

## 4.6 Non-power-of-two inputs

Real workloads rarely have ideal lengths.

A correct scan must handle:

- Empty inputs.
- One-element inputs.
- Partial workgroups.
- Non-power-of-two lengths.
- Large arrays.
- Integer overflow.
- Floating-point rounding.

Padding can simplify an implementation, but the padded values must use the correct identity element.

## 4.7 Segmented scan

A segmented scan performs independent scans over multiple logical segments stored in one array.

For example:

Values:

\[
[2,3,4,1,5,2]
\]

Segment boundaries:

\[
[1,0,0,1,0,1]
\]

The inclusive segmented sum is:

\[
[2,5,9,1,6,2]
\]

Each segment begins a new accumulation.

## 4.8 Why segmented scans matter

Segmented scans are useful for:

- Sparse matrix operations.
- Grouped reductions.
- Variable-length records.
- Graph adjacency processing.
- Batched irregular computations.

They allow many small logical computations to share a large parallel dispatch.

## 4.9 Segment representation

Segments may be represented using:

- Boundary flags.
- Start offsets.
- Segment identifiers.

Each representation changes how the algorithm discovers boundaries and propagates prefix information.

## Worked example: segment boundaries

Consider:

\[
x=[4,2,7,3,1]
\]

with segment starts at indices 0 and 3.

The segments are:

\[
[4,2,7]\quad [3,1]
\]

Their inclusive scans are:

\[
[4,6,13]\quad[3,4]
\]

Combined:

\[
[4,6,13,3,4]
\]

The important property is that information from the first segment must not cross into the second.

## Exercises

1. Define a scan over an associative operation.
2. Derive Hillis–Steele work and span.
3. Derive Blelloch work and span.
4. Explain upsweep.
5. Explain downsweep.
6. Explain hierarchical scan.
7. Explain why global scans may require multiple dispatches.
8. Define segmented scan.
9. Explain segment-boundary propagation.
10. Explain why segmented scans are useful for sparse algorithms.

## Lab 1: Advanced Scan Library

Implement in Rust and WGSL:

- Inclusive sum scan.
- Exclusive sum scan.
- Inclusive minimum scan.
- Segmented inclusive sum scan.
- Arbitrary-length hierarchical scan.

Requirements:

- Sequential reference implementations.
- Automated tests.
- Multiple workgroup sizes.
- Correct boundary handling.
- Numerical validation.
- Performance measurements.

---

# 5. Module Three: Stream Compaction

Stream compaction removes elements that do not satisfy a predicate.

It is a fundamental data-reorganization primitive.

## 5.1 Stable compaction

Stable compaction preserves the relative order of retained elements.

Input:

\[
[8,3,7,2,9]
\]

Retain odd values:

\[
[3,7,9]
\]

The original order is preserved.

## 5.2 Predicate flags

Define:

\[
f_i=
\begin{cases}
1 & \text{if }x_i\text{ is retained}\\
0 & \text{otherwise}
\end{cases}
\]

For the example:

\[
f=[0,1,1,0,1]
\]

## 5.3 Output positions

An exclusive prefix sum of the flags gives:

\[
p=[0,0,1,2,2]
\]

Each retained element writes to:

\[
y_{p_i}=x_i
\]

for positions where \(f_i=1\).

The final output length is:

\[
m=\sum_i f_i
\]

## 5.4 Why the algorithm is parallel

Predicate evaluation is independent.

The scan computes global offsets.

The scatter phase writes each retained element to a distinct output position.

The correctness of the scatter depends on the scan producing unique destination positions for retained elements.

## 5.5 Stable versus unstable compaction

Stable compaction preserves order.

Unstable compaction may allow retained elements to appear in a different order.

Unstable approaches can sometimes use atomic counters or other allocation mechanisms.

The appropriate choice depends on whether order is part of the algorithm's contract.

## 5.6 Applications

Compaction appears in:

- Particle simulations.
- Graph frontier construction.
- Sparse processing.
- Filtering.
- Ray tracing.
- GPU memory management.
- Active-set algorithms.

## Exercises

1. Define stable compaction.
2. Define unstable compaction.
3. Explain the role of predicate flags.
4. Explain why a prefix scan produces output positions.
5. Explain why scatter writes do not collide in a correct stable compaction.
6. Explain how output length is determined.
7. Compare scan-based compaction with atomic allocation.
8. Explain why compaction can improve later kernel performance.

## Lab 2: GPU Stream Compaction

Implement:

- Stable compaction.
- Unstable compaction.
- Predicate-based filtering.
- Output count generation.

Test:

- All elements retained.
- No elements retained.
- Alternating elements.
- Random predicates.
- Highly skewed predicates.
- Non-power-of-two input sizes.

Compare performance and output-order guarantees.

---

# 6. Module Four: Parallel Histograms and Radix Sorting

## 6.1 Histograms

A histogram counts occurrences of values or categories.

For input:

\[
[1,2,1,3,2,1]
\]

the counts are:

\[
h_1=3,\quad h_2=2,\quad h_3=1
\]

## 6.2 Global atomic histogram

A simple parallel histogram lets each thread increment the counter corresponding to its input value.

The algorithm is straightforward but can suffer from atomic contention.

## 6.3 Privatized histogram

An alternative creates partial histograms for workgroups or other partitions.

The partial histograms are later combined.

This trades additional storage and merging work for reduced contention.

## 6.4 Input distribution matters

Suppose all values map to the same bucket.

Many threads may contend for one counter.

Now suppose values are distributed evenly across thousands of buckets.

Contention may be much lower.

Thus histogram performance can depend strongly on the input distribution, not just its length.

## 6.5 Radix sorting

Radix sort processes keys according to digits or groups of bits.

For fixed-width integer keys, radix sorting can avoid comparison-based sorting.

A common parallel radix pass uses:

- Digit extraction.
- Histograms.
- Prefix sums.
- Stable redistribution.

## 6.6 Stable redistribution

For a stable least-significant-digit radix sort, each pass must preserve the order of elements sharing the same digit.

This property allows successive passes to build a globally sorted order.

## 6.7 Complexity

Suppose each key contains \(b\) bits and each pass processes \(r\) bits.

The number of passes is approximately:

\[
d=\left\lceil\frac br\right\rceil
\]

If each pass performs \(O(n+2^r)\) work, total work is:

\[
O\left(d(n+2^r)\right)
\]

The best radix width balances:

- Number of passes.
- Histogram size.
- Local storage.
- Synchronization.
- Memory traffic.

## 6.8 Radix width trade-off

Larger digits reduce the number of passes.

However, they require more histogram buckets.

For example:

- 4-bit digits have 16 buckets.
- 8-bit digits have 256 buckets.
- 11-bit digits have 2048 buckets.

Larger histograms may increase memory consumption and merging cost.

## 6.9 Signed integers and floating-point keys

Sorting signed integers or floating-point values requires careful treatment of their representations.

Raw bit patterns do not always have the same ordering as numerical values.

Special floating-point cases such as NaNs and signed zero require an explicitly defined sorting contract.

## 6.10 Comparison-based sorting

Parallel merge sort and sorting networks remain important alternatives.

Radix sorting is particularly attractive for suitable fixed-width keys, but comparison sorting supports more general orderings.

## Exercises

1. Explain histogram contention.
2. Explain privatization.
3. Why does input distribution matter?
4. Define radix sort.
5. Why must certain radix passes be stable?
6. Derive the number of radix passes.
7. Explain radix-width trade-offs.
8. Why is floating-point radix sorting nontrivial?
9. Compare radix sort and merge sort.
10. Explain how scans support radix sorting.

## Lab 3: Histogram and Radix Sort

Implement:

- Global atomic histogram.
- Privatized histogram.
- GPU radix sort for unsigned 32-bit integers.

Required tests:

- Already sorted.
- Reverse sorted.
- Uniform random.
- All equal.
- Repeated keys.
- Small inputs.
- Large inputs.

Verify sortedness and permutation preservation.

For stable variants, verify stability independently.

---

# 7. Module Five: Sparse Data Structures

Dense matrices store every element.

Sparse matrices store only selected nonzero elements.

This can reduce memory consumption and arithmetic work when most matrix entries are zero.

## 7.1 Dense storage

A dense matrix with dimensions:

\[
M\times N
\]

stores:

\[
MN
\]

elements.

For FP32 values, this requires approximately:

\[
4MN\text{ bytes}
\]

excluding metadata.

## 7.2 Coordinate format

COO stores nonzero entries as coordinate-value triples:

\[
(\text{row},\text{column},\text{value})
\]

For example:

\[
A=
\begin{bmatrix}
2&0&5\\
0&3&0\\
7&0&0
\end{bmatrix}
\]

can be represented by:

\[
(0,0,2),\ (0,2,5),\ (1,1,3),\ (2,0,7)
\]

## 7.3 Compressed Sparse Row

CSR commonly uses three arrays:

- Row offsets.
- Column indices.
- Values.

For the example matrix:

\[
\text{values}=[2,5,3,7]
\]

\[
\text{columns}=[0,2,1,0]
\]

\[
\text{row\_offsets}=[0,2,3,4]
\]

The entries belonging to row \(i\) occupy the half-open range:

\[
[\text{row\_offsets}[i],\text{row\_offsets}[i+1])
\]

## 7.4 Why CSR is useful

CSR stores rows compactly and supports efficient traversal of each row's nonzero entries.

It is widely used for sparse matrix-vector multiplication and graph adjacency lists.

## 7.5 Sparse matrix-vector multiplication

For:

\[
y=Ax
\]

each output row is:

\[
y_i=\sum_{j\in\text{nonzeros}(i)}A_{ij}x_j
\]

The arithmetic work is proportional to the number of nonzero entries, under the usual sparse representation.

## 7.6 Irregular row lengths

Some rows may contain no nonzero entries.

Others may contain thousands.

Assigning one GPU thread per row can create severe load imbalance.

## 7.7 Parallelization strategies

Possible mappings include:

- One thread per row.
- One subgroup per row.
- One workgroup per row.
- Nonzero-based partitioning.
- Hybrid strategies.

Each approach trades simplicity, utilization, memory access, and synchronization differently.

## 7.8 Sparse matrix-matrix multiplication

Sparse matrix-matrix multiplication is more difficult because the output sparsity pattern may not be known in advance.

It often requires:

- Discovering output structure.
- Accumulating partial products.
- Handling duplicate output coordinates.
- Allocating output storage.
- Managing irregular work.

## 7.9 Sparse formats beyond CSR

Other formats include:

- CSC.
- ELLPACK.
- Block sparse formats.
- Hybrid formats.

Different formats favor different sparsity patterns and operations.

## Exercises

1. Compare dense and sparse storage.
2. Explain COO.
3. Explain CSR.
4. Construct CSR arrays for a small matrix.
5. Explain sparse matrix-vector multiplication.
6. Why do irregular row lengths create load imbalance?
7. Compare thread-per-row and subgroup-per-row mappings.
8. Explain why sparse matrix-matrix multiplication is difficult.
9. Why does sparse format selection affect performance?
10. Explain how sparse matrices connect to graph algorithms.

## Lab 4: Sparse Linear Algebra

Implement:

- Dense-to-COO conversion.
- COO-to-CSR conversion.
- CPU CSR SpMV.
- GPU CSR SpMV.
- At least two GPU work-distribution strategies.

Test:

- Empty rows.
- Single-element rows.
- Highly skewed row lengths.
- Uniform sparsity.
- Random sparsity.
- Structured sparsity.

Compare correctness, memory consumption, and throughput.

---

# 8. Module Six: Parallel Graph Algorithms

Graphs are a major example of irregular parallelism.

A graph consists of:

\[
G=(V,E)
\]

where \(V\) is the vertex set and \(E\) is the edge set.

## 8.1 Graph representation

Common representations include:

- Adjacency lists.
- Adjacency matrices.
- CSR-style adjacency arrays.

CSR is often attractive for GPU traversal because it stores neighbor lists compactly.

## 8.2 Breadth-first search

Breadth-first search explores vertices in increasing distance from a source vertex in an unweighted graph.

The algorithm maintains a frontier of vertices to process.

## 8.3 Frontier-based execution

A parallel BFS iteration typically processes all vertices in the current frontier.

Each vertex examines its neighbors.

Newly discovered vertices become candidates for the next frontier.

## 8.4 Duplicate discovery

Multiple frontier vertices may share a neighbor.

Without coordination, multiple threads may attempt to discover or enqueue that neighbor.

The algorithm must prevent incorrect duplicate processing or explicitly tolerate duplicates according to its design.

## 8.5 Atomic visited-state updates

Atomic operations can help coordinate discovery.

However, atomics may become contended for high-degree vertices.

The exact memory-ordering requirements depend on how discovery state and associated data are published.

## 8.6 Frontier compaction

A frontier may be represented as:

- A dense bitmap.
- A compact list.
- A queue.
- A combination of representations.

Compaction can transform candidate vertices into a compact next-frontier list.

## 8.7 Direction-optimizing BFS

Traditional top-down BFS explores outgoing edges from frontier vertices.

Bottom-up BFS examines undiscovered vertices to determine whether they have a neighbor in the current frontier.

Direction-optimizing BFS may switch strategies depending on frontier size and graph structure.

## 8.8 Graph structure matters

Graph performance depends on:

- Vertex count.
- Edge count.
- Degree distribution.
- Diameter.
- Locality.
- Frontier size.
- Duplicate discovery.
- Graph directionality.

## 8.9 Complexity

Sequential BFS has work:

\[
O(|V|+|E|)
\]

under a conventional adjacency-list representation.

Parallel BFS can expose substantial concurrency, but its span depends on the graph's structure and the synchronization between frontier levels.

A graph with many BFS levels can have a long sequential dependency chain even when each level is highly parallel.

## 8.10 Other graph algorithms

Important GPU graph algorithms include:

- Connected components.
- PageRank.
- Single-source shortest paths.
- Triangle counting.
- Graph coloring.

These algorithms differ significantly in synchronization and work-distribution requirements.

## Exercises

1. Define a graph.
2. Explain adjacency lists.
3. Explain CSR graph storage.
4. Explain BFS frontiers.
5. Why is duplicate discovery difficult?
6. Why might BFS require atomics?
7. Explain frontier compaction.
8. Compare top-down and bottom-up BFS.
9. Why does degree distribution affect load balancing?
10. Explain how graph diameter affects parallel depth.
11. Why are graph workloads often memory-bound?
12. Why are graph algorithms difficult to benchmark fairly?

## Lab 5: GPU Breadth-First Search

Implement:

- CPU BFS reference.
- GPU frontier-based BFS.
- Frontier compaction.
- Visited-state tracking.
- Distance validation.

Test:

- Paths.
- Cycles.
- Trees.
- Disconnected graphs.
- High-degree hub graphs.
- Random graphs.
- Empty graphs.

Record frontier sizes and iteration counts.

---

# 9. Module Seven: GPU Work Queues and Dynamic Scheduling

Many irregular algorithms generate work dynamically.

Examples include:

- Graph traversal.
- Adaptive refinement.
- Ray processing.
- Sparse computation.
- Recursive subdivision.

## 9.1 Work queues

A work queue stores tasks awaiting processing.

A GPU work queue may be implemented using:

- Atomic counters.
- Ring buffers.
- Per-workgroup queues.
- Hierarchical queues.
- Multi-stage compaction.

## 9.2 Queue correctness

A concurrent queue must define:

- Capacity.
- Ownership.
- Reservation.
- Publication.
- Consumption.
- Completion.
- Overflow behavior.

A correct queue must not lose tasks or expose partially published tasks to consumers.

## 9.3 Atomic reservation

An atomic counter can reserve distinct output positions.

However, reservation alone does not guarantee that the corresponding payload has been written and is visible to consumers.

Publication ordering must be handled separately where required.

## 9.4 Bounded queues

A bounded queue has fixed capacity.

The algorithm must define what happens when the queue is full.

Possible policies include:

- Reporting overflow.
- Deferring work.
- Splitting work into additional stages.
- Using a larger buffer.

## 9.5 Persistent kernels

A persistent kernel uses long-running GPU workgroups to process multiple tasks rather than launching one thread per task.

Persistent execution can reduce certain scheduling overheads.

However, it introduces difficult questions about:

- Work distribution.
- Progress.
- Termination detection.
- Fairness.
- Resource residency.
- Synchronization.

## 9.6 Termination detection

A dynamic parallel algorithm must distinguish:

- No tasks currently available.
- No tasks remaining anywhere.
- Tasks still being generated by active workers.

These are not equivalent conditions.

Incorrect termination detection can cause lost work or deadlock.

## 9.7 GPU scheduling limitations

Ordinary GPU workgroups do not necessarily execute concurrently.

A design that requires all workgroups to reach a global barrier can deadlock if some workgroups are unable to become resident.

Specialized cooperative execution mechanisms have additional constraints and are not universally available.

## Exercises

1. What is a GPU work queue?
2. What is atomic reservation?
3. Why does reservation differ from publication?
4. What is a bounded queue?
5. Why must overflow behavior be defined?
6. What is a persistent kernel?
7. Why is termination detection difficult?
8. Why can naive grid-wide synchronization deadlock?
9. Compare queues with scan-based compaction.
10. Explain when dynamic scheduling may outperform static scheduling.

## Lab 6: GPU Work Queue

Build a bounded GPU task queue or a multi-dispatch worklist system.

Requirements:

- Correct task ownership.
- No lost tasks.
- No duplicate execution unless explicitly allowed.
- Capacity checks.
- Overflow handling.
- Termination detection.
- Stress testing.

Begin with a multi-dispatch implementation before investigating persistent execution.

---

# 10. Module Eight: Parallel Hash Tables

Hash tables support fast key-based lookup and insertion.

Parallel hash tables are challenging because multiple threads may compete for the same storage locations.

## 10.1 Hashing

A hash function maps a key to a bucket or candidate location.

A good hash function distributes expected keys reasonably evenly.

## 10.2 Collisions

Different keys may map to the same location.

Collision-handling strategies include:

- Open addressing.
- Linear probing.
- Quadratic probing.
- Chaining.
- Cuckoo hashing.

## 10.3 Open addressing

Open addressing stores entries within the table itself.

When a collision occurs, the algorithm probes alternative locations.

## 10.4 Atomic insertion

Concurrent insertion may use atomic compare-and-exchange to claim an empty slot.

However, correctness also requires deciding when the associated value becomes visible.

A key-only atomic claim does not automatically make a multi-field entry safely readable.

## 10.5 Load factor

The load factor is:

\[
\alpha=\frac{\text{occupied slots}}{\text{total slots}}
\]

Higher load factors may increase probing costs.

The relationship depends on the collision strategy and key distribution.

## 10.6 Contention

Contention can arise from:

- Popular buckets.
- Similar hash values.
- High load factors.
- Repeated keys.
- Poor hash distribution.

## 10.7 Resizing

Concurrent resizing is difficult because threads may access old and new storage simultaneously.

A simpler GPU design may rebuild the table in a separate dispatch sequence.

## 10.8 Progress guarantees

A hash table using atomics is not automatically lock-free in the formal progress-guarantee sense.

Its progress properties depend on the complete algorithm, including collision resolution and resizing.

## Exercises

1. Define hashing.
2. Explain collisions.
3. Compare open addressing and chaining.
4. Explain atomic slot reservation.
5. Why is entry publication difficult?
6. Define load factor.
7. Why does load factor affect performance?
8. Why is concurrent resizing difficult?
9. Why do repeated keys cause contention?
10. Why does using atomics not automatically guarantee lock-free progress?

## Lab 7: GPU Hash Table

Implement a bounded GPU hash table for integer keys.

Requirements:

- Insertion.
- Lookup.
- Collision handling.
- Capacity handling.
- Duplicate-key policy.
- Correctness tests.
- Performance benchmarks.

Investigate performance as load factor changes.

---

# 11. Module Nine: Load Balancing for Irregular Workloads

Load balancing determines how evenly work is distributed across execution resources.

## 11.1 Why static partitioning can fail

Suppose four workers receive tasks requiring:

\[
[1,1,1,100]
\]

units of work.

Even though each worker receives one task, the workload is highly imbalanced.

Three workers finish quickly while one continues working.

## 11.2 Work-oriented partitioning

Instead of assigning one logical object per thread, an algorithm can partition by estimated work.

For sparse matrices, this may mean assigning groups of nonzero entries rather than groups of rows.

## 11.3 Degree-based graph partitioning

In graph algorithms, high-degree vertices can be assigned more execution resources than low-degree vertices.

Possible strategies include:

- Thread-per-vertex.
- Subgroup-per-vertex.
- Workgroup-per-vertex.
- Edge-centric processing.
- Hybrid scheduling.

## 11.4 Prefix sums for load balancing

If each task has a known work count, a prefix sum can map work indices to task ownership.

This transforms irregular task sizes into a more regular global work domain.

## 11.5 Dynamic load balancing

Dynamic queues allow workers to obtain additional work after finishing their current tasks.

This can improve utilization but introduces coordination overhead.

## 11.6 Performance metrics

Useful load-balancing measurements include:

- Work per thread.
- Work per workgroup.
- Active execution time.
- Idle time.
- Queue contention.
- Maximum task duration.
- Distribution of task sizes.

## Exercises

1. Define load imbalance.
2. Explain why equal task counts do not imply equal work.
3. Explain work-oriented partitioning.
4. Explain degree-based graph partitioning.
5. Why can prefix sums support load balancing?
6. Compare static and dynamic scheduling.
7. Why can dynamic scheduling be expensive?
8. Explain how load balancing interacts with memory locality.

## Lab 8: Irregular Workload Scheduler

Create synthetic workloads with configurable task sizes.

Compare:

- Static equal-task partitioning.
- Work-weighted partitioning.
- Dynamic queue scheduling.

Measure throughput and imbalance.

---

# 12. Module Ten: Stencils and Parallel Dynamic Programming

Not all advanced parallel algorithms are sparse or graph-based.

Some contain regular spatial dependencies or structured recurrence relations.

## 12.1 Stencil computation

A stencil computes each output from nearby input elements.

For a one-dimensional three-point stencil:

\[
y_i=ax_{i-1}+bx_i+cx_{i+1}
\]

Each output depends on neighboring values.

## 12.2 Parallelism within a time step

If all outputs read from an immutable input array and write to a separate output array, they can often be computed in parallel.

## 12.3 Dependencies across time steps

Suppose the computation repeats:

\[
x^{(t+1)}=F(x^{(t)})
\]

The next time step depends on the previous time step.

This introduces a global temporal dependency.

## 12.4 Double buffering

Two buffers can alternate roles:

- Current state.
- Next state.

This avoids overwriting values that other threads still need during the current step.

## 12.5 Halo regions

When data is partitioned into tiles, each tile may need neighboring values outside its own interior.

These neighboring values form a halo region.

## 12.6 Shared-memory tiling

A stencil may load a tile and its halo into workgroup memory.

Threads can reuse neighboring values.

The benefit depends on the ratio of useful interior work to halo traffic.

## 12.7 Parallel dynamic programming

Dynamic programming often involves dependencies between neighboring states.

For example:

\[
D_{i,j}=f(D_{i-1,j},D_{i,j-1},D_{i-1,j-1})
\]

Cells cannot all be computed independently.

However, cells on certain diagonals may be independent once earlier diagonals are complete.

This is wavefront parallelism.

## 12.8 Dependency analysis

Before parallelizing dynamic programming, identify:

- State dependencies.
- Independent antichains.
- Synchronization boundaries.
- Memory requirements.
- Opportunities for tiling.

## Exercises

1. Define a stencil.
2. Explain spatial dependencies.
3. Explain temporal dependencies.
4. Why is double buffering useful?
5. What is a halo region?
6. Why can tiling improve stencil performance?
7. Explain wavefront parallelism.
8. Why can dynamic programming have limited span?
9. Explain how dependency graphs guide parallelization.
10. Compare stencil parallelism with graph parallelism.

## Lab 9: Stencil and Wavefront Computation

Implement:

- A two-dimensional stencil.
- Multiple time steps.
- Double buffering.
- A tiled variant.
- A small wavefront dynamic-programming problem.

Validate each against a sequential reference.

---

# 13. Module Eleven: Correctness in Advanced GPU Algorithms

Advanced GPU algorithms often produce nondeterministic execution orders.

That does not necessarily mean their results are incorrect.

## 13.1 Determinism

A deterministic algorithm produces the same observable result for the same input under its defined execution environment.

Some parallel algorithms guarantee deterministic output.

Others permit multiple valid outputs.

## 13.2 Correctness contracts

Define what must remain invariant.

Examples:

- Sorting: output is ordered and preserves the input multiset.
- Compaction: retained elements satisfy the predicate.
- Stable compaction: retained order is preserved.
- BFS: distances are correct.
- Hash table: inserted keys are retrievable.
- Reduction: numerical result meets the stated error bound.

## 13.3 Invariants

An invariant is a property that must remain true throughout some portion of execution.

Examples include:

- Each reserved queue slot has a unique owner.
- No output position is written by two compaction threads.
- Every discovered BFS vertex has a valid predecessor path.
- Each hash-table slot has a defined state.

## 13.4 Race conditions versus data races

A race condition is a correctness problem caused by timing-dependent behavior.

A data race is a more specific memory-model concept involving conflicting unsynchronized accesses.

Atomic operations can eliminate certain data races without eliminating all race conditions.

## 13.5 Differential testing

Differential testing compares a parallel implementation against an independent reference implementation.

This is particularly useful for:

- Scans.
- Sorting.
- Graph traversal.
- Sparse operations.
- Hash tables.

## 13.6 Property-based testing

Property-based tests verify general properties rather than only individual expected outputs.

For example, a sorting test can check:

- Nondecreasing order.
- Input/output length equality.
- Multiset preservation.

## 13.7 Numerical validation

Floating-point algorithms require carefully chosen error criteria.

Absolute and relative tolerances may both be relevant.

For reductions, matrix operations, and sparse computations, expected error can depend on:

- Input magnitude.
- Number of accumulated terms.
- Conditioning.
- Accumulation order.
- Precision.

## 13.8 Stress testing

Concurrency tests should include:

- Empty workloads.
- Tiny workloads.
- Large workloads.
- High contention.
- Skewed distributions.
- Repeated execution.
- Boundary sizes.

## Exercises

1. Define determinism.
2. Explain correctness contracts.
3. Define invariants.
4. Distinguish race conditions from data races.
5. Explain differential testing.
6. Explain property-based testing.
7. Why are numerical tolerances necessary?
8. Why are skewed workloads important?
9. Why can a program pass many tests and still contain concurrency bugs?
10. Why must benchmark results be discarded when correctness fails?

---

# 14. Major Assessment A: Parallel Data Movement Library

Build a reusable GPU library implementing:

1. Inclusive scan.
2. Exclusive scan.
3. Segmented scan.
4. Stable compaction.
5. Unstable compaction.
6. Histogram.
7. Radix sort.

## Requirements

- CPU reference implementations.
- Arbitrary-length support.
- Correct boundary handling.
- Clear output contracts.
- Automated correctness tests.
- GPU timing.
- End-to-end timing.
- Performance reports.

## Technical defense

Explain:

- Work and span.
- Synchronization stages.
- Memory traffic.
- Atomic contention.
- Stability guarantees.
- Output-position correctness.
- Performance bottlenecks.

---

# 15. Major Assessment B: Sparse and Graph Computing Library

Build a GPU library implementing:

1. COO representation.
2. CSR representation.
3. COO-to-CSR conversion.
4. Sparse matrix-vector multiplication.
5. Frontier-based BFS.
6. Frontier compaction.

## Requirements

- Correct CPU references.
- Graph and sparse-matrix generators.
- Support for skewed distributions.
- Reproducible benchmarks.
- Memory usage analysis.
- Load-balancing experiments.

## Technical defense

Explain:

- Sparse storage.
- Irregular memory access.
- Load imbalance.
- Frontier construction.
- Duplicate discovery.
- Synchronization.
- Graph-dependent performance.

---

# 16. Major Assessment C: Irregular GPU Runtime

Build a small runtime for irregular GPU work.

## Required components

- Worklist representation.
- Task generation.
- Task consumption.
- Capacity management.
- Completion detection.
- Load-balancing statistics.
- Correctness validation.

## Required experiments

Compare:

- Static partitioning.
- Work-weighted partitioning.
- Queue-based scheduling.

Use workloads with:

- Uniform work.
- Skewed work.
- Heavy-tailed work.
- Dynamically generated work.

## Technical defense

Explain:

- Queue invariants.
- Task ownership.
- Synchronization scope.
- Overflow behavior.
- Termination detection.
- Scheduling overhead.
- Load-balancing effectiveness.

---

# 17. Final Capstone: Advanced GPU Algorithms Library

Build a reusable Rust GPU library using wgpu and WGSL.

## Required algorithm families

### Data movement

- Scan.
- Segmented scan.
- Compaction.
- Partitioning.

### Sorting

- Histogram.
- Radix sort.

### Sparse computation

- CSR representation.
- Sparse matrix-vector multiplication.

### Graph computation

- Frontier-based BFS.
- Frontier compaction.

### Irregular execution

- Worklist or bounded queue.
- Load-balancing experiments.

## Architecture requirements

The library should separate:

- Device management.
- Buffer management.
- Shader compilation.
- Pipeline construction.
- Algorithm dispatch.
- Validation.
- Benchmarking.

## Correctness requirements

Every algorithm must have:

- A CPU reference.
- Unit tests.
- Edge-case tests.
- Stress tests.
- Documented invariants.
- Defined error behavior.

## Performance requirements

For each algorithm:

1. Establish a baseline.
2. Measure GPU execution time.
3. Measure end-to-end time.
4. Estimate memory traffic.
5. Analyze work distribution.
6. Investigate relevant bottlenecks.
7. Test at least one alternative design.
8. Explain the result.

## Required reports

### Report 1: Algorithmic foundations

Explain the work, span, and dependency structure of each algorithm.

### Report 2: Correctness

Document invariants, synchronization, and testing methodology.

### Report 3: Performance

Present benchmarks and analyze bottlenecks.

### Report 4: Irregular workloads

Explain how input distribution affects performance.

### Report 5: Architectural mapping

Explain how your algorithms use:

- Workgroups.
- Wavefronts.
- Registers.
- Workgroup memory.
- Global memory.
- Atomics.
- Multi-dispatch synchronization.

### Report 6: Engineering trade-offs

Compare alternative implementations and explain why no single design is optimal for every workload.

---

# 18. Phase 11 Quiz Bank

## Algorithmic foundations

1. What is work efficiency?
2. What is span?
3. Why does span limit parallel speedup?
4. What is irregular parallelism?
5. Why does irregular work cause load imbalance?
6. What is a synchronization boundary?
7. Why do GPU algorithms often require multiple dispatches?
8. Why can a work-inefficient algorithm outperform a work-efficient one?

## Scans

9. What is an inclusive scan?
10. What is an exclusive scan?
11. What is Hillis–Steele scan?
12. What is Blelloch scan?
13. What is upsweep?
14. What is downsweep?
15. What is hierarchical scan?
16. What is segmented scan?
17. Why are associative operations important?
18. Why do non-power-of-two inputs matter?

## Compaction

19. What is stable compaction?
20. What is unstable compaction?
21. What are predicate flags?
22. Why is scan useful for compaction?
23. How are output positions determined?
24. Why must scatter writes be unique?
25. How is output length calculated?
26. Why can compaction improve subsequent kernels?

## Histograms and sorting

27. What is atomic contention?
28. What is histogram privatization?
29. Why does input distribution affect histogram performance?
30. What is radix sort?
31. Why does radix sorting use histograms?
32. Why does radix sorting use scans?
33. Why is stability important in least-significant-digit radix sort?
34. How does radix width affect performance?
35. Why is floating-point radix sorting difficult?
36. When might comparison sorting be preferable?

## Sparse computation

37. What is a sparse matrix?
38. What is COO?
39. What is CSR?
40. What is CSC?
41. What is sparse matrix-vector multiplication?
42. Why do irregular row lengths matter?
43. What is thread-per-row processing?
44. What is subgroup-per-row processing?
45. Why is sparse matrix-matrix multiplication difficult?
46. Why does sparse format selection matter?

## Graph algorithms

47. What is BFS?
48. What is a frontier?
49. Why can multiple threads discover the same vertex?
50. Why are atomics useful in graph traversal?
51. What is frontier compaction?
52. What is top-down BFS?
53. What is bottom-up BFS?
54. What is direction-optimizing BFS?
55. Why does degree distribution matter?
56. Why does graph diameter affect parallel depth?

## Work queues and hashing

57. What is a GPU work queue?
58. What is atomic reservation?
59. Why does publication ordering matter?
60. What is a bounded queue?
61. What is termination detection?
62. What is a persistent kernel?
63. Why can grid-wide synchronization deadlock?
64. What is open addressing?
65. What is compare-and-exchange?
66. What is hash-table load factor?
67. Why does high load factor increase probing?
68. Why is concurrent resizing difficult?

## Load balancing and dependencies

69. What is work-weighted partitioning?
70. Why can equal task counts produce unequal workloads?
71. What is dynamic scheduling?
72. How can scans support work distribution?
73. What is a stencil?
74. What is a halo region?
75. What is double buffering?
76. What is wavefront dynamic programming?

## Correctness and performance

77. What is an invariant?
78. What is differential testing?
79. What is property-based testing?
80. Why can nondeterministic execution still be correct?
81. Why are numerical tolerances necessary?
82. Why must concurrency tests include skewed inputs?
83. Why should correctness be checked before performance?
84. Why is end-to-end timing important?

---

# 19. Final Examination

**Recommended duration:** 7–9 hours, including practical analysis.

| Section | Weight |
|---|---:|
| Work/span and advanced scans | 15% |
| Compaction and radix sorting | 15% |
| Sparse data structures | 15% |
| Graph algorithms | 20% |
| Work queues and hash tables | 15% |
| Load balancing and structured dependencies | 10% |
| Correctness and performance engineering | 10% |

## Part A: Algorithm derivations

Derive work and span for:

- A parallel scan.
- A compaction pipeline.
- A multi-pass radix sort.

## Part B: Data movement

Analyze stable compaction and segmented scan correctness.

## Part C: Sparse computation

Construct CSR representations and compare sparse work-distribution strategies.

## Part D: Graph algorithms

Design a frontier-based BFS and explain its synchronization requirements.

## Part E: Concurrent data structures

Analyze a bounded work queue and parallel hash table.

## Part F: Load balancing

Evaluate scheduling strategies for a skewed workload.

## Part G: Performance

Analyze memory traffic, contention, and GPU utilization.

## Part H: Technical defense

Defend your advanced GPU algorithms library.

### Passing criteria

A recommended standard is:

- At least 80% overall.
- At least 75% in correctness and synchronization questions.
- No unresolved correctness failures in the capstone.
- Ability to explain every major algorithm without AI assistance.

---

# 20. Copy-Ready Socratic AI Tutor Prompts

All tutor prompts are consolidated here.

**Tutor restriction:** The AI tutor must never supply answers, code, pseudocode, implementation instructions, debugging fixes, file edits, completed derivations, or assessment solutions.

The manuals themselves may contain worked examples. The tutor must not complete the student's work.

## Master Socratic Tutor Prompt

```text
You are my Socratic advanced parallel algorithms tutor.

I am studying Phase 11 of a rigorous GPU engineering curriculum.

NON-NEGOTIABLE RULES:

1. Never provide answers to quizzes, exercises, assessments,
   derivations, projects, or examinations.

2. Never write code, pseudocode, algorithms, or implementations.

3. Never modify files, execute code, debug programs, or perform
   any project work on my behalf.

4. Never reveal answer keys, hidden tests, completed calculations,
   or solutions.

5. Never provide hints that effectively disclose a solution.

6. Guide my reasoning exclusively through questions.

7. Ask exactly one guiding question per response.

8. If I am incorrect, ask a question that exposes the misconception.

9. If I am stuck, ask a simpler prerequisite question.

10. Require me to explain work, span, invariants, dependencies,
    synchronization, memory access, load balancing, and performance.

11. For coding assessments, present requirements only.
    I must design, implement, test, and debug everything myself.

12. For quizzes, ask one question at a time and adapt difficulty
    based on my responses.

13. For code review, ask questions about correctness, resource
    ownership, synchronization, and performance.
    Never provide exact fixes.

14. Never provide a final answer during an assessment.

15. Keep responses concise and focused.

Your responses must contain only one guiding question.
```

## Start a lesson

```text
Tutor me through Phase 11, Module [NUMBER].

Start from first principles.
Ask exactly one question at a time.
Require me to reason about dependencies, correctness,
work, span, and GPU execution.

Never provide answers or solutions.
```

## Generate a quiz

```text
Administer a 20-question Phase 11 quiz on [TOPIC].

Include conceptual reasoning, work/span analysis,
algorithm correctness, synchronization, and performance.

Ask exactly one question at a time.
Adapt difficulty based on my responses.
Never reveal answers.
```

## Start a coding assessment

```text
Administer Phase 11 Coding Assessment [A/B/C].

Present the requirements and constraints only.

Ask me to explain my algorithmic design,
correctness argument, synchronization strategy,
and expected performance.

Do not provide code, pseudocode, implementations,
debugging fixes, or solutions.

Ask one question at a time.
```

## Review GPU algorithm code

```text
Review my GPU algorithm code using only questions.

Focus on:
- Algorithmic invariants
- Work and span
- Memory safety
- Atomic operations
- Synchronization
- Output uniqueness
- Load balancing
- Numerical correctness
- Performance

Do not provide fixes or replacement code.
Ask exactly one guiding question at a time.
```

## Graph algorithm examination

```text
Examine my understanding of GPU graph algorithms.

Ask questions about:
- Graph representations
- BFS frontiers
- Duplicate discovery
- Atomic synchronization
- Frontier compaction
- Load balancing
- Direction-optimizing traversal
- Correctness invariants

Never provide answers or implementation guidance.
Ask one question at a time.
```

## Concurrent data structure examination

```text
Examine my understanding of GPU work queues
and parallel hash tables.

Ask questions about:
- Ownership
- Atomic reservation
- Publication ordering
- Queue capacity
- Termination detection
- Collision handling
- Progress guarantees
- Correctness invariants

Never provide solutions or debugging fixes.
Ask exactly one question at a time.
```

## Performance investigation

```text
Help me investigate advanced GPU algorithm performance
using only questions.

Ask about:
- Work distribution
- Memory access patterns
- Atomic contention
- Synchronization
- Load imbalance
- Input distributions
- Arithmetic intensity
- Benchmark methodology
- CPU versus GPU performance

Do not interpret results for me.
Do not provide conclusions or optimizations.
Ask exactly one question at a time.
```

## Final oral examination

```text
Conduct a rigorous Phase 11 oral examination.

Challenge my understanding of:
- Advanced scans
- Compaction
- Radix sorting
- Sparse computation
- Graph algorithms
- Concurrent data structures
- Load balancing
- Correctness
- Performance engineering

Ask exactly one question at a time.
Never provide answers, solutions, code, or hints
that reveal the answer.
```

---

# 21. Mastery Checklist

## Algorithmic reasoning

- [ ] I can calculate work and span.
- [ ] I understand work efficiency.
- [ ] I can identify algorithmic dependencies.
- [ ] I understand irregular parallelism.
- [ ] I can reason about synchronization boundaries.

## Scans and compaction

- [ ] I can implement inclusive scan.
- [ ] I can implement exclusive scan.
- [ ] I understand Hillis–Steele scan.
- [ ] I understand Blelloch scan.
- [ ] I can implement segmented scan.
- [ ] I can implement stable compaction.
- [ ] I can explain output-position correctness.

## Sorting

- [ ] I can implement GPU histograms.
- [ ] I understand atomic contention.
- [ ] I understand histogram privatization.
- [ ] I can implement radix sort.
- [ ] I understand sorting stability.
- [ ] I can explain radix-width trade-offs.

## Sparse computation

- [ ] I understand COO.
- [ ] I understand CSR.
- [ ] I can convert COO to CSR.
- [ ] I can implement sparse matrix-vector multiplication.
- [ ] I understand irregular row-length problems.
- [ ] I can compare sparse work-distribution strategies.

## Graph algorithms

- [ ] I understand CSR graph storage.
- [ ] I can implement CPU BFS.
- [ ] I can implement GPU frontier-based BFS.
- [ ] I understand duplicate discovery.
- [ ] I understand frontier compaction.
- [ ] I understand direction-optimizing BFS.
- [ ] I can analyze graph-dependent performance.

## Concurrent data structures

- [ ] I understand GPU work queues.
- [ ] I understand atomic reservation.
- [ ] I understand publication ordering.
- [ ] I understand termination detection.
- [ ] I understand parallel hash tables.
- [ ] I understand collision handling.
- [ ] I understand load factor.
- [ ] I can explain progress guarantees.

## Load balancing

- [ ] I understand static scheduling.
- [ ] I understand dynamic scheduling.
- [ ] I understand work-weighted partitioning.
- [ ] I understand degree-based graph partitioning.
- [ ] I can measure load imbalance.

## Correctness

- [ ] I can define algorithmic invariants.
- [ ] I understand nondeterministic execution.
- [ ] I can perform differential testing.
- [ ] I can design property-based tests.
- [ ] I understand numerical validation.
- [ ] I can stress-test concurrent algorithms.

## Capstone

- [ ] My advanced GPU algorithms library works.
- [ ] My implementations are validated.
- [ ] My benchmarks are reproducible.
- [ ] My reports explain algorithmic trade-offs.
- [ ] I can defend correctness and performance independently.

---

# 22. Recommended Resources

## Parallel algorithms

**Introduction to Parallel Algorithms — Joseph JáJá**

A rigorous foundation for parallel algorithm analysis.

**Introduction to Parallel Computing — Grama, Gupta, Karypis, and Kumar**

Useful for parallel models, algorithm design, and performance analysis.

**The Art of Multiprocessor Programming — Herlihy and Shavit**

Useful for concurrent data structures and synchronization.

## GPU algorithms

**Programming Massively Parallel Processors — Kirk and Hwu**

An essential textbook for GPU algorithm design.

**GPU Gems 3 — Chapter 39: Parallel Prefix Sum (Scan) with CUDA**

https://developer.nvidia.com/gpugems/gpugems3/part-vi-gpu-computing/chapter-39-parallel-prefix-sum-scan-cuda

This is a foundational treatment of GPU scan algorithms. Some implementation details reflect older GPU architectures, but the algorithmic principles remain valuable.

**NVIDIA CUB**

https://nvidia.github.io/cccl/cub/

Study the documented interfaces and algorithmic capabilities of optimized parallel primitives.

## Sparse and graph computation

**Graph Algorithms in the Language of Linear Algebra — Jeremy Kepner and John Gilbert**

Useful for connecting graph algorithms with sparse matrix operations.

**SuiteSparse Matrix Collection**

https://sparse.tamu.edu/

A source of real-world sparse matrices for benchmarking.

**Graph500**

https://graph500.org/

Useful for understanding large-scale graph benchmarking.

## Rust and GPU programming

**wgpu**

https://docs.rs/wgpu/

**WGSL Specification**

https://www.w3.org/TR/WGSL/

**AMD GPUOpen**

https://gpuopen.com/

## Research directions

Study published research on:

- Parallel prefix scans.
- GPU radix sorting.
- Direction-optimizing BFS.
- GPU sparse matrix-vector multiplication.
- GPU hash tables.
- GPU load balancing.
- Persistent GPU kernels.

For each paper, identify the algorithm's assumptions, hardware generation, correctness argument, and evaluation methodology before attempting to reproduce its results.

---

# 23. Completion Standard

You have completed Phase 11 when you can independently:

1. Derive work and span for advanced parallel algorithms.
2. Implement hierarchical and segmented scans.
3. Implement stable GPU compaction.
4. Implement GPU radix sorting.
5. Explain sparse matrix formats.
6. Implement GPU sparse matrix-vector multiplication.
7. Implement frontier-based GPU BFS.
8. Explain atomic coordination and publication.
9. Design a bounded GPU worklist.
10. Analyze irregular load balancing.
11. Validate nondeterministic parallel algorithms.
12. Benchmark advanced GPU workloads.
13. Explain algorithmic and architectural trade-offs.
14. Defend your complete GPU algorithms library.

**Do not advance with unresolved correctness failures in graph traversal, concurrent queues, or parallel sorting.** These algorithms expose synchronization mistakes that can become much harder to diagnose in later runtime and compiler work.

---

# 24. What Comes Next

**Phase 12: Native Rust GPU Programming**

The next phase will explore GPU programming beyond portable WGSL.

Topics include:

- Rust-to-GPU compilation.
- GPU-targeted Rust subsets.
- SPIR-V.
- Vulkan compute.
- Shader compilation.
- Rust ownership across host/device boundaries.
- GPU-safe data representations.
- Native GPU bindings.
- Device memory management.
- Rust GPU ecosystem limitations.
- Compiler-generated GPU code.
- GPU-specific Rust abstractions.
- Low-level GPU execution.
- Performance comparisons with WGSL and CUDA.

You will investigate how Rust can be used to build lower-level GPU software and how language, compiler, and hardware constraints interact.
