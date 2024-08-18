---
title: "[PC-2] Parallel Programming Models and Basics"
description: "Notes according to CS149/CMU15-418 Lecture 3-4."
date: 2024-08-18
category: ["Computer System"]
tags: ["Parallel Computing"]
mermaid: true
mathjax: true

---

These notes are based on CS149/CMU15-418 Lecture 3-4, following the video[^2] and lecture slides[^1][^3]. The main purpose of these notes is to summarize and expand on the course content.

## SPMD Programming Model

### ISPC: Assignment of Array Elements

> What is **ISPC**: ISPC is an SPMD (to be introduced later) program compiler. Visit the GitHub link for more info.[^4]

We're going to use ISPC as an example to illustrate the concept of parallelism.

A _call to an ISPC function_ spawns a "gang" of ISPC "program instances" (similar to the concept of *pthreads*, but no, they are not pthreads themselves), all of which run ISPC code concurrently. Each instance has its own _copy_ of local variables, and upon return, all instances have completed. An example illustration can be found here: [^ISPC].

How are array elements assigned to program instances? Two methods can be considered:

- **Interleaved** assignment: All the ALUs within each processor operate on contiguous areas. This provides significant benefits for cache hits, as subsequent instances can read directly from subsequent cache lines. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408172310625.png)
- **Blocked** assignment: Each instance is assigned a contiguous block of memory. At a certain moment, the instances touch eight non-contiguous values in memory. Thus, a "gather" instruction is needed. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408172313051.png)

### SPMD Idea

**Abstraction: Single process, multiple data (SPMD):** _Running a gang involves spawning multiple logical instruction streams._ ISPC uses `foreach` to declare parallel loop iterations, indicating that this is the iteration through the entire gang. This is an example of the SPMD programming model.

**Implementation: SIMD.** The ISPC compiler handles mapping conditional control flow to vector instructions, using commands such as `foreach` to declare independencies.

SPMD allows one function to run _multiple instances_ of that function in parallel on _different input arguments_.

```mermaid
graph LR
A["Single thread of control"]-->B["Call SPMD function"]-->C["Spawn multiple instances of functions"]-->D["Function returns, resume single thread"]
```

## Parallel Programming Models

### Shared Address Space Model

**Abstraction:**

- Threads communicate by reading/writing to locations in a shared address space (shared variables). A metaphor is like a bulletin board: Mike sticks a note onto the board, and later Tom picks it up; everyone can read/write.
- **Synchronization problems:** Since simultaneous read/write by two or more threads can cause synchronization problems, we need locks, atomic operations[^7], etc., to manipulate synchronization primitives.

**Implementation:**

- Idea: Any processor can directly reference any memory location.
- All threads can read/write to all shared variables.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408182203744.png)

### Message Passing Model

**Abstraction:**

- Each thread operates within its own private address space.
- Threads _send_ by specifying the recipient, buffer to be transmitted, and optional message identifier.
- Threads _receive_ by specifying the sender, buffer to store the message, and optional message identifier.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408182202097.png)

### Data Parallel Model

**Abstraction:**

- **Self-limitation:** _Imposing a rigid program structure_ to facilitate simpler and faster programming.
- **Independent function (instances) invocation:** Organize computation as (identical) operations on sequences of elements, similar to NumPy vector operations or `map` onto a large collection of data.[^5][^8]
- **Functions without side effects:** Repeated access to a variable may cause load/store violations; check if the operation is for each instance or for each "gang" of instances (`uniform`, see discussion[^6]). Similarly, `foreach` must double-check the dependencies inside its loop.
- Multiple operations on uniform variables should be avoided. For example, to `sum` some operation results, instead of defining a global variable `sum` for each instance/pthread to load/write, a better solution would be to use a local variable, such as `tmp_result`. When all threads terminate and return to the main function, the main function serially sums up these returned `tmp_result`. (ISPC uses `reduce_add()`[^9])

**Implementation:**

- **Stream programming model:** A huge amount of input numbers are referred to as **streams**, while element-wise functions that are homogeneous to all inputs are called **kernels** (meanwhile without side effects).

    For example, for the function `foo(bar(x))` where `x` is a vector, the output result of the `bar` function for a certain scalar `x[index]` (i.e., `bar(x[index])`) is directly sent to the function `foo` without saving it to memory or waiting for other elements in the vector to finish executing `bar`.

- Stream programming offers enormous benefits such as bandwidth saving since data doesn't need to be written to memory, but it requires a complex library of operators to describe intricate data flows.

## Creating a Parallel Program

### Workflow Overview

To create a parallel program, first you need to:

- Identify the tasks that can be done in parallel.
- Partition the tasks in a reasonable way.
- Manage data access, transfer, and communication.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408182337412.png)

### Decomposition

**Decomposition** is the process of separating a complex task into sub-tasks that can be done *in parallel*. The term *in parallel* indicates that tasks should be independent of each other, so first, we need to analyze the dependencies. Moreover, tasks should be divided such that no computational resources are wasted. All execution units on a machine should be busy.

**Amdahl's Law** explains that maximum speedup through program parallelism is constrained by the portion of execution that is inherently sequential. Suppose that portion is s, and the amount of input is N. The maximum speedup is 
$$
\text{speedup} =\frac{\text{op time by 1 processor}}{\text{op time by } p \text{ processors}}= \frac{N/1+(1-s)N/1}{sN/1 + (1-s)N/p}= \frac{N}{sN+\frac{(1-s)N}{p}} = \frac{1}{s + \frac{1-s}{p}} \leq \frac{1}{s}
$$
Our main goal is to reduce the portion of sequential execution instructions in the program. Sometimes, the algorithm should be changed so that more tasks can be executed simultaneously or in parallel (see example [^11]).

### Assignment

**Assignment** is the process of assigning the (decomposed) (sub-)tasks to workers (threads). It aims to achieve a balance between workers to reduce communication loss.

It is not often the case that you should assign as many threads as the number of tasks, which would often cause communication and context-switching overhead.

### Orchestration

**Orchestration** involves maintaining dependencies, organizing memory, structuring communication, and scheduling tasks. The goal is to preserve locality, reduce the cost of communication, and minimize overhead and synchronization.

Locks and barriers are classic examples, where locks ensure exclusive access to uniform variables, and barriers divide computation into phases: the program cannot proceed to the next phase unless all threads have finished processing their tasks.[^12]

### Mapping

**Mapping** involves assigning threads or workers to hardware units, which is the responsibility of the OS.

[^ISPC]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/slide_47
[^1]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/
[^2]: https://www.bilibili.com/video/BV16k4y1z7z9?p=1&vd_source=5ade9da381cec8d2c191f450ccd0cf57
[^3]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progbasics/
[^4]: https://github.com/ispc/ispc
[^5]: https://numpy.org/doc/stable/reference/generated/numpy.vectorize.html
[^6]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/slide_90
[^7]: https://stackoverflow.com/questions/52196678/what-are-atomic-operations-for-newbies
[^8]: https://stackoverflow.com/questions/35215161/most-efficient-way-to-map-function-over-numpy-array
[^9]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progbasics/slide_38
[^10]: https://en.wikipedia.org/wiki/Amdahl%27s_law
[^11]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progbasics/slide_67
[^12]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progbasics/slide_79

