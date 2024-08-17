---
title: "Parallel Computing 2 (CS149, CMU15-418)"
description: "Notes according to CS149/CMU15-418 Lecture 1-2."
date: 2024-08-17
category: ["Computer System"]
tags: ["Parallel Computing"]
---

My notes according to CS149/CMU15-418 Lecture 1-2, according to the video[^2] and lecture slides[^1].  This main purpoe of these notes is to summarize and expand on the content of courses. 

This note will cope with these questions:

- Why do we need parallelism?
- What are different forms of parallel exectuion?
- Memory hierarchy
- Threads and workers.

## Why Parallelism ?

Today, *improvements in single-threaded performance are progressing slowly* for two main reasons:

1. **Power Draw as a Function of Frequency:** Increasing clock frequency to enhance performance results in a significant rise in dynamic power consumption, leading to potential heat issues and reduced overall efficiency.
   
2. **Diminishing Returns from Instruction-Level Parallelism (ILP):** Techniques like ILP and superscalar execution, which were effective in the past for improving performance, now offer limited additional benefits.

A **parallel computer** is a _collection of processing elements_ that work together to solve problems _quickly_. Therefore, we focus on both performance and efficiency, utilizing multiple processors to achieve these goals. The objective is to maximize **performance per area** and **performance per Watt**. 

For the software engineers, they have to improve parallelism of their programs, which is the aim of this course: How to properly utilize the computer resources?

## Different Forms of Parallel Execution

### Computer Program and Processor

A **computer program** is a list of processor instructions. (e.g. compile a C program would generate a list of instructions, and we could use various optimization methods to improve **Instruction Level Parallelism (ILP)**)

A **processor** that executes instructions should contain three basic components as shown in the picture below: **Fetch/Decode Unit**, **Arithmetic Logic Unit (ALU)** and **Execution Context** (made up of registers):![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813202319.png)

A processor processes the instruction-level program and goes through these steps:

1.	The processor retrieves the next program instruction from memory and determines what the instruction requires the processor to do next.
2.	It gets the required operation inputs from registers.
3.	The processor performs the operation using the execution unit.
4.	It stores the output result back into the register.

### Parallel Execution Advancements

Here we introduce three types of parallel execution:
1. _Exploit Instruction Level Parallelism, execute multiple different instructions simultaneously inside one core._ In the past, we exploited ILP within an instruction stream, meaning we identified two or more instructions that could be executed simultaneously. (This is done automatically by the processor) The term __superscalar execution__ refers to the processor automatically finding independent instructions in an instruction sequence and executing them in parallel on multiple execution units. Below is an example: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813201925.png) Accordingly, the ALU should match the number of F/D units, or else we have explored the dependencies between the instructions but lack the ability to execute them. 

1. **Multi-core processors allow multiple threads to run in parallel (Thread-level parallelism).** Early processors adopted complex structures such as data caches, out-of-order control logic, and sophisticated branch predictors. However, in multi-core designs, these structures were replaced by additional cores using the same number of transistors. Although a single core in a multi-core design may underperform compared to previous designs (by about 25%), fully exploiting parallelism can potentially lead to a speedup of 2x0.75=1.5 times. ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813202726.png)
2. **Single Instruction Multiple Data (SIMD) processing adds execution units (ALUs) to a core, enabling the same instruction to be executed across multiple data elements simultaneously.** This approach amortizes the cost and complexity of managing an instruction stream across many ALUs. The number of ALUs determines how many elements can be operated on in parallel. Below is an example: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813203015.png)



Below is a comparison between CPUs and GPUs: ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813203711.png)


## Accessing Memory 

Two main concepts:

1. **Memory latency** is the amount of time it takes for a memory request from a processor to be serviced by the memory system.
2. **Memory bandwidth** is the rate at which the memory system can provide data to a processor.
3. 
These can be metaphorically compared to a highway: the number of lanes determines how many cars can pass through in a given time, corresponding to memory bandwidth; the length or condition of the road determines the time it takes to reach the destination, corresponding to memory latency.

A processor **stalls** when it cannot execute the next instruction in a stream because the dependent instruction is not yet complete. This can be due to various reasons, including long I/O transmission times or cache misses.

In the context of _Computer Systems_, communication between different levels of the memory hierarchy is limited, and the concept of **caching** reduces stall lengths. However, if processors request data at too high a rate, the memory system cannot keep up. As observed below, a GPU can fetch memory at a bandwidth of at most 177 GB/sec, but matrix multiplication for very large matrices may require more than 5 TB/sec of bandwidth. Consequently, the GPU must wait until the data is fully read from memory.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813205310.png)



We now introduce the concept of **throughput-oriented systems**: Using multiple threads can potentially increase the time it takes for any single thread to complete its work (e.g., thread 1 might start when it is runnable but wait until thread 4 finishes before resuming). However, this approach can increase overall system throughput when running multiple threads.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Pasted%20image%2020240813204903.png)

Multi-threading, caches and prefetching are both used for reduces stalls, in consequence hides latency. However, the maximal latency hiding ability is decided by the number of threads per core times the number of cores and the number of SIMD ALUs per core, usually 512 for a small CPU. The overcomming bandwidth limits are a common challange for application developers, since no amount of latency hiding help this.

Moreover, GPU architectures use the same throughput computing ideas as CPUs but GPUs push these concepts to extreme scales.

[^1]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/whyparallelism/
[^2]: https://www.bilibili.com/video/BV16k4y1z7z9?p=1&vd_source=5ade9da381cec8d2c191f450ccd0cf57
[^3]: https://gfxcourses.stanford.edu/cs149/fall21/lecture/multicorearch/

