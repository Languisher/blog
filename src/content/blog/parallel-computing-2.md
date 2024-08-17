---
title: "Parallel Computing 2 (CS149, CMU15-418)"
description: "Notes according to CS149/CMU15-418 Lecture 3."
date: 2024-08-17
category: ["Computer System"]
tags: ["Parallel Computing"]
mermaid: true

---

These notes are based on CS149/CMU15-418 Lecture 3, according to the video[^2] and lecture slides[^1][^3]. The main purpose of these notes is to summarize and expand on the content of the course.

## SPMD Programming Model

### ISPC: Assignment of Array Elements

- What is **ISPC**: visit the GitHub link.[^4]

- A _Call to an ISPC function_ spawns a "gang" of ISPC "program instances," all of which run ISPC code concurrently.
- Each instance has its own _copy_ of local variables, and upon return, all instances have completed.

An example illustration can be found here: [^ISPC].

How to assign array elements to program instances? Two methods could be considered:

- **Interleaved** assignment: All the ALUs within each processor operate on contiguous areas. This provides significant benefits for cache hits since subsequent instances can read directly from subsequent cache lines. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408172310625.png)
- **Blocked** assignment: Each instance is assigned a contiguous block of memory. At a certain moment, the instances now touch eight non-contiguous values in memory. Thus, a "gather" instruction is needed. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408172313051.png)

**Abstraction: SPMD.** ISPC uses `foreach` to declare parallel loop iterations to state that this is the iteration through the entire gang. This is an example of the **Single Program, Multiple Data (SPMD) programming model**: running a gang is spawning multiple logical instruction streams.

**Implementation: SIMD.** The ISPC compiler handles mapping conditional control flow to vector instructions.

### SPMD

SPMD allows one function to run _multiple instances_ of that function in parallel on _different input arguments_.

```mermaid
graph LR
A["Single thread of control"]-->B["Call SPMD function"]-->C["Spawn multiple instances of functions"]-->D["Function returns, resume single thread"]
```

## Parallel Programming Models

### Shared Address Space Model

Abstraction:

- Threads communicate by reading/writing to locations in a shared address space (shared variables). A metaphor is like a bulletin board: Mike sticks a note onto the board, and later Tom picks it up; everyone can read/write.
- Since simultaneous read/write by two or more threads can cause synchronization problems, we need locks, atomic ops[^7], etc., to manipulate synchronization primitives.

Implementation:

- Any processor can directly reference any memory location.

### Message Passing Model

Abstraction:

- Each thread operates within its own private address space.
- Threads *send* specifying: recipient, buffer to be transmitted, and optional message identifier.
- Threads _receive_ specifying: sender, buffer to store the message, and optional message identifier.

## Data Parallel Model

Abstraction:

- _Imposing a rigid program structure_ to facilitate simpler and faster programming.
- Organize computation as (identical) operations on sequences of elements, similar to numpy vector operations or `map` onto a large collection of data.[^5][^8]
- Repeated access to a variable may cause load/store violations; check if the operation is for each instance or for each "gang" of instances (`uniform`, see discussion[^6]). Similarly, `foreach` must double-check the dependencies inside its loop.

[^ISPC]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/slide_47

[^1]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/
[^2]: https://www.bilibili.com/video/BV16k4y1z7z9?p=1&vd_source=5ade9da381cec8d2c191f450ccd0cf57
[^3]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/
[^4]: https://github.com/ispc/ispc
[^5]: https://numpy.org/doc/stable/reference/generated/numpy.vectorize.html
[^6]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/progmodels/slide_90
[^7]: https://stackoverflow.com/questions/52196678/what-are-atomic-operations-for-newbies
[^8]: https://stackoverflow.com/questions/35215161/most-efficient-way-to-map-function-over-numpy-array

