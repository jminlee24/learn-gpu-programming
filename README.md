# GPU Engineering: Complete Curriculum and Study System

**Version:** 4.0  
**Starting level:** Junior software engineer with college-level CS and mathematics  
**Target:** Senior GPU software engineer  
**Primary systems language:** Rust  
**Additional languages:** C++, CUDA C++, WGSL  
**Estimated core study:** 24–36 months at 10–12 hours per week  
**Long-term professional development:** 3–5+ years

---

# Part I — What This Course Is

This is a complete, long-term course for developing the knowledge and engineering ability required to work professionally on GPU software.

It is not intended to be a checklist of technologies to learn.

The course is designed to teach the underlying ideas, demonstrate how they work, require you to implement them, and assess whether you genuinely understand them.

The goal is to develop an engineer who can:

- Understand computer architecture from the transistor and logic level through modern GPU execution.
- Write efficient systems software in Rust and C++.
- Understand operating systems, memory, concurrency, and resource management.
- Derive parallel algorithms and reason about their correctness.
- Write GPU kernels and understand their execution.
- Diagnose performance bottlenecks using measurements.
- Build GPU libraries, runtimes, and abstractions.
- Understand compiler design and GPU code generation.
- Read technical papers and evaluate unfamiliar implementations.
- Make sound engineering decisions involving correctness, performance, portability, and maintainability.

## The fundamental learning loop

Every major concept should be learned through six stages:

1. **Mental model:** Understand the concept intuitively.
2. **First principles:** Derive why it works.
3. **Formal model:** Understand the mathematical or technical rules.
4. **Implementation:** Build it yourself.
5. **Experimentation:** Test edge cases and measure behavior.
6. **Evaluation:** Explain and defend your implementation.

You should not advance because you have watched a lecture or finished a chapter. You advance when you can independently apply the knowledge.

---

# Part II — The Course Volumes

The course consists of 16 standalone study manuals.

Each manual is a separate document that can be studied independently once its prerequisites are satisfied.

| Volume | Subject | Main outcome |
|---|---|---|
| 0 | Mathematical and computational foundations | Understand bits, logic, numbers, and digital systems |
| 1 | Rust systems programming | Understand ownership, memory, and systems software |
| 2 | Modern C++ | Understand C++ object lifetime, memory, and low-level programming |
| 3 | Computer architecture | Understand how processors execute programs |
| 4 | Operating systems | Understand processes, memory, concurrency, and Linux |
| 5 | Mathematics and numerical computing | Understand the mathematics and numerical behavior of computation |
| 6 | Concurrency and parallel algorithms | Design and reason about parallel programs |
| 7 | GPU architecture | Understand GPU hardware and execution |
| 8 | Portable GPU programming | Write GPU compute software using Rust, wgpu, and WGSL |
| 9 | CUDA programming | Develop NVIDIA GPU kernels |
| 10 | GPU optimization | Analyze and improve GPU performance |
| 11 | Advanced parallel algorithms | Implement fundamental high-performance GPU algorithms |
| 12 | Native Rust GPU programming | Explore Rust GPU programming ecosystems |
| 13 | GPU runtime systems | Build systems that manage GPU execution and resources |
| 14 | GPU compilers | Understand and implement GPU compilation |
| 15 | Specialization and capstone | Demonstrate advanced engineering ability |

## What every volume must contain

Every individual phase manual should be a complete study resource, not a topic list.

Each volume must include:

1. Prerequisite knowledge and diagnostic test.
2. Detailed explanation of the subject.
3. A logical sequence of lessons.
4. First-principles derivations.
5. Worked examples.
6. Important definitions and formal models.
7. Code examples with line-by-line explanations where appropriate.
8. Diagrams and execution walkthroughs where useful.
9. Common misconceptions and failure modes.
10. Guided exercises.
11. Independent problem sets.
12. Practical programming labs.
13. A substantial project.
14. Conceptual quizzes.
15. Derivation and reasoning quizzes.
16. Code comprehension and debugging quizzes.
17. A cumulative examination.
18. Project grading criteria.
19. A technical defense.
20. A reference section and additional study material.

A phase manual should contain enough explanation to make meaningful progress without constantly having to find another tutorial.

External resources are supplemental. They should provide additional perspectives, exercises, or authoritative technical references rather than replace the teaching material.

---

# Part III — Language and Hardware Strategy

## Rust

Rust is the primary systems programming language throughout the course.

It will be used to learn:

- Ownership and borrowing.
- Memory management.
- Resource lifetimes.
- Concurrency.
- Data structures.
- Systems programming.
- GPU abstractions.
- Runtime development.
- Compiler and tooling development.

## C++

C++ is essential for understanding and working with the broader GPU ecosystem, especially CUDA.

The course will cover:

- Modern C++.
- RAII.
- Move semantics.
- Templates.
- Generic programming.
- Memory models.
- Undefined behavior.
- Low-level optimization.
- CUDA C++.

The objective is proficiency in both languages, not choosing one and ignoring the other.

## GPU technologies

| Technology | Role |
|---|---|
| WGSL | Portable GPU shader and compute language |
| wgpu | Rust GPU API |
| Vulkan | Low-level GPU API |
| CUDA C++ | NVIDIA GPU programming |
| CubeCL | Rust GPU computing framework |
| cudarc | Rust CUDA ecosystem |
| LLVM | Compiler infrastructure |
| SPIR-V | GPU intermediate representation |
| PTX | NVIDIA intermediate representation |

These technologies will be introduced when their underlying concepts are understood.

## Your development hardware

Primary environment:

- Fedora Linux.
- AMD Radeon RX 9070, 16 GB VRAM.
- Rust toolchain.
- C++ toolchain.
- Vulkan.
- wgpu and WGSL.

Use the RX 9070 for portable GPU compute and Vulkan experimentation.

For CUDA, use an NVIDIA cloud GPU or remote machine. The RX 9070 cannot execute NVIDIA CUDA kernels.

Investigate ROCm compatibility for your exact GPU, driver, operating system, and software version before making it part of a project.

Do not make buying additional hardware a prerequisite for learning.

---

# Part IV — Assessment and Mastery

## 1. Types of assessment

Every volume uses multiple forms of assessment.

### Conceptual questions

Test whether you understand what something is and why it exists.

### Derivation questions

Test whether you can reconstruct a result from fundamental assumptions.

### Application questions

Test whether you can apply knowledge to a new problem.

### Code comprehension

Test whether you understand what an implementation actually does.

### Debugging

Test whether you can identify incorrect assumptions, bugs, and failure modes.

### Practical projects

Test whether you can build working software.

### Technical defense

Test whether you can explain and justify the design of your software.

## 2. Mastery levels

| Level | Meaning |
|---|---|
| 0 — Unfamiliar | Cannot explain the concept |
| 1 — Basic | Can explain it with help |
| 2 — Competent | Can solve standard problems independently |
| 3 — Proficient | Can apply it to unfamiliar problems |
| 4 — Advanced | Can derive, optimize, critique, and teach it |

The target is Level 3 for core concepts and Level 4 for the subjects in which you eventually specialize.

## 3. Passing standards

| Assessment | Requirement |
|---|---|
| Topic quizzes | 85% |
| Phase examinations | 85% |
| Required correctness tests | 100% |
| Project requirements | All mandatory requirements satisfied |
| Technical defense | Able to explain major decisions |
| Critical concepts | Level 3 mastery before progression |

A passing score does not override a serious misunderstanding of a prerequisite concept.

If you score 90% but cannot explain a fundamental concept, that concept needs further work.

## 4. Quiz progression

Use this sequence:

1. Diagnostic quiz.
2. Lesson quizzes.
3. First-principles quizzes.
4. Misconception checks.
5. Code comprehension assessments.
6. Practical assessments.
7. Cumulative examination.
8. Technical defense.

Questions should become more difficult as understanding improves.

Do not reveal answers before the user attempts the question.

When an answer is incorrect, identify the exact misconception, teach the missing concept, and provide a different problem to verify understanding.

---

# Part V — The Learning Workflow

## Weekly schedule

Suggested commitment: 10–12 hours.

| Activity | Time |
|---|---:|
| Detailed study | 3 hours |
| Derivations and reasoning | 2 hours |
| Programming exercises | 3 hours |
| Project development | 2 hours |
| Quizzes and review | 2 hours |

A useful weekly pattern:

- Monday: Learn new concepts.
- Tuesday: Derive and work through examples.
- Wednesday: Programming exercises.
- Thursday: Implementation and debugging.
- Friday: Quiz and review.
- Saturday: Project work.
- Sunday: Cumulative review and planning.

## Study notes

For every lesson, maintain:

- What I learned.
- What problem the concept solves.
- How it works.
- Why it works.
- What assumptions it makes.
- What can go wrong.
- What I implemented.
- What I still do not understand.

## Mistake tracking

Maintain a `MISTAKES.md` file.

For every significant mistake, record:

1. The original question.
2. Your original answer.
3. The correct explanation.
4. Why your reasoning failed.
5. A new example testing the same concept.

The objective is to identify recurring reasoning errors rather than simply accumulate correct answers.

---

# Part VI — Quiz Generation System

Use the prompts in the relevant phase manual. The following are course-wide prompts.

## Standard quiz

```text
I am studying Phase [PHASE], Topic [TOPIC] in my GPU engineering
curriculum.

Generate a rigorous quiz that tests actual understanding.

Start with fundamentals and gradually increase difficulty.

Include:
- Conceptual questions.
- First-principles reasoning.
- Worked scenarios.
- Mathematical problems where relevant.
- Code comprehension where relevant.
- Common misconceptions.
- Unfamiliar applications.

Ask one question at a time.
Do not reveal the answer before I attempt it.
Require explanations.
Grade my reasoning, not just my final answer.

When I am wrong:
1. Identify the exact misconception.
2. Explain the missing concept.
3. Give me a simpler problem.
4. Test the concept again using a different example.

At the end, provide a score, weaknesses, and recommended practice.
```

## First-principles examination

```text
Teach and examine me on [TOPIC] from first principles.

Start from the simplest assumptions and build the concept
through a sequence of questions.

Require me to derive important results rather than memorize them.

Challenge unsupported assumptions.
Use counterexamples.
Introduce edge cases.
Ask me to explain why alternative approaches are incorrect.

Do not reveal the next step until I respond.

At the end, determine whether I could reconstruct the concept
independently.
```

## Misconception examination

```text
Identify what I misunderstand about [TOPIC].

Create questions that distinguish genuine understanding
from superficial familiarity.

Use:
- Tricky edge cases.
- Counterexamples.
- Misleading but plausible answers.
- Code tracing.
- Prediction problems.
- Questions about why alternative answers are wrong.

Ask one question at a time.

When I make a mistake, isolate the underlying misconception
and test it with a new example.
```

## Hard phase examination

```text
Create a difficult examination for Phase [PHASE].

Assume I have studied the material and completed the required labs.

Test:
- Conceptual knowledge.
- Derivations.
- Code comprehension.
- Debugging.
- Practical applications.
- Design decisions.
- Performance reasoning.
- Connections between different topics.

Do not reveal answers in advance.
Ask one question at a time.
Award partial credit.
Require explanations.

At the end, score me out of 100, identify weak areas,
and determine whether I have met the phase requirements.

Passing score: 85%.
```

## Targeted remedial practice

```text
Here are my latest quiz results:

[PASTE RESULTS]

Identify the underlying concepts I failed to understand.

Generate new problems that test those concepts in different ways.

Include:
- Easier problems to rebuild the fundamentals.
- Standard problems to test competence.
- Hard problems to test transfer.
- At least one problem connecting multiple concepts.

Do not repeat the original questions.

Ask one question at a time and verify that I have corrected
each misconception.
```

## Senior engineer interview

```text
Act as a senior GPU systems engineer interviewing me.

My current phase is [PHASE].

Ask realistic engineering questions appropriate for my current level.

Focus on:
- Architecture.
- Memory.
- Correctness.
- Parallel execution.
- Performance.
- Debugging.
- Numerical behavior.
- Design tradeoffs.

Challenge vague answers.
Ask follow-up questions.
Require evidence for performance claims.

Evaluate my technical depth, systems understanding,
problem-solving ability, and communication.
```

---

# Part VII — Repository and Progress Tracking

Suggested repository:

```text
gpu-engineering/
|
|-- curriculum/
|   |-- master-curriculum.md
|   |-- phase-00-foundations.md
|   |-- phase-01-rust.md
|   |-- phase-02-cpp.md
|   |-- ...
|
|-- implementations/
|-- exercises/
|-- quizzes/
|-- examinations/
|-- projects/
|-- benchmarks/
|-- notes/
|-- mistakes/
|-- papers/
|
|-- PROGRESS.md
```

## Progress tracker

```markdown
# GPU Engineering Progress

Current phase:
Current lesson:
Weekly study hours:

## Phase status

| Phase | Status | Exam | Project | Mastery |
|---|---|---:|---|---|
| 0 | Not started | - | - | 0 |
| 1 | Not started | - | - | 0 |
| 2 | Not started | - | - | 0 |
| ... | ... | ... | ... | ... |

## Current weaknesses

- None identified.

## Next lesson

- Not started.
```

After each lesson, update your mastery levels. After each phase, record the exam score, project evaluation, and remaining weaknesses.

---

# Part VIII — The Standard for Completion

Completing all 16 volumes does not automatically make someone a senior engineer.

The course develops the technical foundation. Senior-level judgment also requires experience with real codebases, production constraints, performance regressions, debugging, code review, and collaboration.

The final objective is to be able to independently:

- Investigate unfamiliar hardware and software.
- Form and test a correct mental model.
- Design and implement parallel algorithms.
- Explain the execution and memory behavior of GPU code.
- Diagnose bottlenecks with profiling evidence.
- Build reliable systems software.
- Understand the compiler and runtime beneath an API.
- Review and critique other implementations.
- Make defensible architectural decisions.
- Communicate technical findings clearly.

**The governing principle of this curriculum: understand the machine, derive the algorithm, implement the system, measure the result, and defend the engineering decisions.**
