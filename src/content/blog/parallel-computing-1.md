---
title: "Parallel Computing 1 (CS149, CMU15-418)"
description: "Notes according to CS149/CMU15-418 Lecture 1-2."
date: 2024-08-17
category: ["Computer System"]
tags: ["Parallel Computing"]

---

These notes are based on CS149/CMU15-418 Lecture 1-2, according to the video[^2] and lecture slides[^1][^3]. The main purpose of these notes is to summarize and expand on the content of the courses.

These notes will address the following questions:

- Why do we need parallelism? **Multi-thread**
- What are the different forms of parallel execution? **Superscalar, SIMD, and Multi-core**
- What is memory latency and memory bandwidth?
- How can we reduce memory stalls?

## Why Parallelism?

Today, *improvements in single-threaded performance are progressing slowly*[^4] for two main reasons:

1. **Power Draw as a Function of Frequency:** Increasing clock frequency to enhance performance results in a significant rise in dynamic power consumption, leading to potential heat issues and reduced overall efficiency.

2. **Diminishing Returns from Instruction-Level Parallelism (ILP):** Techniques like ILP and superscalar execution, which were effective in the past for improving performance, now offer limited additional benefits.

A **parallel computer** is a _collection of processing elements_ that work together to solve problems _quickly_. Therefore, we focus on both performance and efficiency, utilizing multiple processors to achieve these goals. The objective is to maximize **performance per area** and **performance per Watt**.

For software engineers, they need to improve the parallelism of their programs, which is the aim of this course: How to properly utilize computer resources?

## Different Forms of Parallel Execution

### Computer Program and Processor

A **computer program** is a list of processor instructions. (e.g., compiling a C program would generate a list of instructions, and we could use various optimization methods to improve **Instruction Level Parallelism (ILP)**)

A **processor** that executes instructions should contain three basic components as shown in the picture below: **Fetch/Decode Unit**, **Arithmetic Logic Unit (ALU)**, and **Execution Context** (made up of registers): ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813202319.png)

A processor processes the instruction-level program and goes through these steps:

1. The processor retrieves the next program instruction from memory and determines what the instruction requires the processor to do next.
2. It gets the required operation inputs from registers.
3. The processor performs the operation using the execution unit.
4. It stores the output result back into the register.

### Parallel Execution Advancements

Here we introduce three types of parallel execution:

1. **Superscalar execution**: _Exploits Instruction Level Parallelism by executing multiple different instructions (of a single instruction stream) simultaneously inside one core._ In the past, we exploited ILP within an instruction stream, meaning we identified two or more instructions that could be executed simultaneously. (This is done automatically by the processor) The term **superscalar execution** refers to the processor automatically finding independent instructions in an instruction sequence and executing them in parallel on multiple execution units. Below is an example: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813201925.png) Accordingly, the ALU should match the number of F/D units, or else we have explored the dependencies between the instructions but lack the ability to execute them: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408171733350.png)

2. **Multi-core processors**: *Allow multiple cores (threads) to run in parallel (Thread-level parallelism), with each core executing a completely different instruction stream.* Early processors adopted complex structures such as data caches, out-of-order control logic, and sophisticated branch predictors. However, in multi-core designs, these structures were replaced by additional cores using the same number of transistors. Although a single core in a multi-core design may underperform compared to previous designs (by about 25%), fully exploiting parallelism can potentially lead to a speedup of 2x0.75=1.5 times. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813202726.png) Note that thread-level parallelism also applies to the same instructions but with different inputs, corresponding to the proposed `foreach` or `vector_mul` concept.[^5]

3. **Single Instruction Multiple Data (SIMD): effectively operating one atomic instruction on multiple (or an array of) inputs.** _Processing adds execution units (ALUs) to a core, enabling the same instruction to be executed across multiple data elements simultaneously._ This approach amortizes the cost and complexity of managing an instruction stream across many ALUs. The instruction is broadcast to all ALUs, while the input to the ALUs is different. The number of ALUs determines how many elements can be operated on in parallel. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408171745672.png)

To fully exploit the above three core concepts (**Superscalar (exploiting ILP within an instruction stream)**, **SIMD (multiple ALUs controlled by the same instruction)**, **Multi-core (multiple streams)**), below is a comparison: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408171801507.png)

An example of a comparison between CPUs and GPUs: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813203711.png)

To cope with the SIMD processing issue (of a single core), we propose the terminology of **instruction stream coherence**, which is the property of a program where the same instruction sequence applies to many data elements. However, it is not necessary for the instruction stream across different cores.

## Accessing Memory

Two main concepts:

1. **Memory latency** is the amount of time it takes for a memory request from a processor to be serviced by the memory system.
2. **Memory bandwidth** is the rate at which the memory system can provide data to a processor.

These can be metaphorically compared to a highway: the number of lanes determines how many cars can pass through in a given time, corresponding to memory bandwidth; the length or condition of the road determines the time it takes to reach the destination, corresponding to memory latency.

The *bandwidth* is the *critical* resource. Therefore, performant parallel programs will:

- *Fetch data from memory less often*: Reuse data previously loaded and share data across threads
- *Prefer performing additional arithmetic*: Compared to load/store, the math is "free"

## Reducing Memory Stalls

A processor **stalls** when it cannot execute the next instruction in a stream because the dependent instruction is not yet complete. This can be due to various reasons, including long I/O transmission times or cache misses.

In the context of _Computer Systems_, communication between different levels of the memory hierarchy is limited, and the concept of **caching** reduces stall lengths. However, if processors request data at too high a rate, the memory system cannot keep up. As observed below, a GPU can fetch memory at a bandwidth of at most 177 GB/sec, but matrix multiplication for very large matrices may require more than 5 TB/sec of bandwidth. Consequently, the GPU must wait until the data is fully read from memory.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813205310.png)

### Threads

We now introduce the concept of **throughput-oriented systems: Interleaving the threads.** Using multiple threads can potentially increase the time it takes for any single thread to complete its work (e.g., thread 1 might start when it is runnable but wait until thread 4 finishes before resuming). However, this approach can increase overall system throughput when running multiple threads.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813204903.png)

However, we still need to store the execution contexts, usually in context storage (L1 cache). This could be done by adding more blocks to the execution context block (the blue one). The number of execution context blocks determines how many threads a single core can handle.

It is essential to point out that the utilization rate does not exceed 100%, thus the utilization rate stops increasing when the number of threads reaches a certain amount (which could be calculated). For more detailed info: [^7].

### Other Methods

Multi-threading, caches, and prefetching are all used to reduce stalls and consequently hide latency. However, the maximal latency-hiding ability is decided by the number of threads per core times the number of cores and the number of SIMD ALUs per core, usually 512 for a small CPU. Overcoming bandwidth limits is a common challenge for application developers since no amount of latency hiding helps with this.

Moreover, GPU architectures use the same throughput computing ideas as CPUs, but GPUs push these concepts to extreme scales.

## Final Design

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408172235931.png)



## From Another Perspective

To fully exploit parallel processors efficiently, an application should:

- Have _sufficient parallel work_ to utilize all available units
- Ensure groups of parallel work items require the _same sequences of instructions_ (SIMD)
- Expose more parallel work



## Extraneous

- Conditioning instructions to SIMD instructions: some ALUs wait when the other branch is executing.[^6]

[^1]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/whyparallelism/
[^2]: https://www.bilibili.com/video/BV16k4y1z7z9?p=1&vd_source=5ade9da381cec8d2c191f450ccd0cf57
[^3]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/
[^4]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/whyparallelism/slide_51
[^5]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/slide_29
[^6]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/slide_38
[^7]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/slide_74

---

This revised version corrects the grammar issues and ensures consistent terminology and structure throughout the document.