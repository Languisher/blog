---
title: "[PC-3] GPU architecture and CUDA Programming"
description: "Notes according to CS149/CMU15-418 Lecture 7."
date: 2024-08-19
category: ["Computer System"]
tags: ["Parallel Computing"]
mermaid: true

---

These notes are based on CS149/CMU15-418 Lecture 7, according to the video[^2] and lecture slides[^1]. The main purpose of these notes is to summarize and expand on the content of the course.

**Revision:** GPU is a chip consists of multiple core, where each core carries out *SIMD* and *multi-thread* execution.

## Historic Issues

**Shader program:** GPUs render high-quality pictures, in order to achieve this goal, each pixel has to be rendered by a user-defined *shader function* in order to define the colors of each pixel and the surface of the fragments. This intrinsic demand requires parallel computation since this function operates for each pixel on the screen. (Comparatively, they are not good at doing single non-SIMD instructions, such as branch prediction)

> A very interesting hack btw. [^3]

**Pipeline execution:** CPU executes instructions based on interruptions and frequent process switch, while before 2007, the user only needs to define the shader function and some basic attributes, then the GPU executes in pipeline operations.

## Architecture

I found @Gohan2021 comment very useful to address these issues[^4]:

- Is CUDA a data parallel programming model?
- Is CUDA a message passing model?
- CUDA analogy to ISPC instances and tasks?

> CUDA is a data parallel programming model and uses shared address space (e.g. input can be shared across multiple CUDA threads). It is not a message passing model because we do not need to send explicit messages among threads. I believe CUDA threads can communicate with each other via load/store to shared/global memory on the GPU.
>
> CUDA threads are like a ISPC instances because multiple threads can be gathered together for SIMT execution that runs in lock step for each instruction among all threads that are not masked off. The analogy to pthreads is less obvious because pthreads execute instructions at independent points among its hardware threads.

Discussion below are all based on CUDA.

### Threads

**Concurrent threads have hierarchy levels.** Threads ID are 3-D. The picture below explains the hierarchy, where `Nx` times `Ny` is the dimension of the input and the output matrices. The outmost layer is a grid (here dimension is 12x6) consists of multiple 2-D thread blocks  (here the totel amount of blocks is 6). ThreadsPerBlock is a 2-D vector, and its value should be defined by the user.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408190157600.png)

### Host and Device

A very easy understand of **Host** and **Device** is to consider **Host** as a CPU to execute sequential instructions, while **Device** is a GPU to carry out SPMD execution.

A **kernel** is a function written in CUDA C/C++ that runs on the GPU (Graphics Processing Unit). When you launch a kernel, it is executed by many parallel threads on the GPU. Kernel is an implementation of SPMD abstraction.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408190209878.png)

Host and device have distinct address spaces. To move data between spaces, use `cudaMemcpy()` demand.

### Shared Memory Spaces

There are two shared memory spaces in a GPU:

- A **device global (i.e. per-program) memory**, which is r/w-able by all threads
- A **per-block shared memory**, which is r/w-able by the threads in block

which reflects locality inside a GPU. Of course each thread have its own private memory spaces.



### Synchronization

CUDA use `__syncthreads()` as a *barrier* (See No.2 Note of this series) to explicitly compel the program to wait for all threads in the block to arrive at this point. CUDA has an implicit barrier when doing Host/Device synchronization.

The idea of atomic operations is also implemented in CUDA on the two shared memory spaces mentioned above.

[^1]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/gpuarch/
[^2]: https://www.bilibili.com/video/BV16k4y1z7z9?p=1&vd_source=5ade9da381cec8d2c191f450ccd0cf57
[^3]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/gpuarch/slide_18
[^4]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/gpuarch/slide_26
