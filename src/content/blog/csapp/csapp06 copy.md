---
title: "[CS:APP Chapter 8] Exceptional Control Flow"
description: "Notes according to Computer System: A Programmer's Perspective Third Edition Chapter 8"
date: 2024-09-01
category: ["Computer System"]
tags: ["CS:APP"]
---


Reference: Computer System: A Programmer's Perspective Third Edition Chapter 8


## Exceptions

Exceptions could be divided into two categories: **Asynchronous exceptions** and **Synchronous exceptions**, their difference is whether the exception is toggled by instructions or not. Asynchronous exceptions are toggled by external I/O devices or a counter, which allows the kernel to take over control and switch to also processes regularly.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202409011644101.png)

**Interruption** is an asynchronous exception.

**Traps** is an intentional exception, usually used for breakpoints or system calls (`syscall`), such as opening files.

**Faults** is called by errors. Examples of faults:
- Page fault: Try to load data from memory, if it succeed, it would return to the instruction that causes page fault.
- Segmentation fault (e.g. Access array elements that are out of bound): First the page fault would be triggered, however the kernel founds out that it could not fix the error, thus abort.

**Abort** is abort, errors that could not be fixed ;( We know in advance that it is fatal (such as the memory has been externally damaged)

## Basic Concepts of Processes

**Process** is a core concept in computer systems. On modern systems, many programs run simultaneously, and a process is defined as an *instance* of a program in execution. Processes create the illusion that our program is the only one currently running on the system, thus monopolizing the processor and memory.

To achieve the illusion of “monopolization,” corresponding key abstractions are needed:
- **Logical control flow** gives the illusion that our program exclusively controls the processor.
- **Private address space** gives the illusion that our program exclusively controls the memory system.

### Concurrent and Parallel Flows: How Processes Monopolize the Processor

This corresponds to the first abstraction: the process monopolizes the processor.

In reality, *all processes in the system take turns using the processor.* As shown in the diagram below, each column corresponds to the **logical control flow** of a process, and each vertical bar represents a portion of the logical control flow.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408141228940.png)

Each process, or its logical control flow, runs in an interleaved fashion: first, a portion of Process A runs, then Process B, then Process C, and so on. After running a portion of its flow, each process is **preempted** by another process, and it enters a temporarily suspended state.

#### Concurrency

*When multiple processes run simultaneously*: For different logical control flows (of processes), if their execution times *overlap* (where overlap means that if Process X starts before Process Y, Process X must end after Process Y starts), they are referred to as **concurrent flows**. The execution of these concurrent flows is called **concurrency**. The term **multitasking** aptly describes the concept of concurrency. Additional terminology: Each time slice of a process executing its control flow (i.e., the time bars in the diagram) is referred to as a **time slice**, so multitasking is also known as **time slicing**.

Note that the term *multitasking* differs slightly from its everyday use: it doesn’t mean performing different tasks at the exact same time, like studying while playing a game (which is more of a parallel concept). Instead, it refers to *alternating between tasks* (like studying for a while, then playing a game, and repeating this process).

#### Parallelism

In computing terms, the relationship between concurrency and parallelism is as follows: *If concurrent flows run on different cores or computers* (which means they can run simultaneously), these flows can be called **parallel flows**, and their execution is known as **parallel execution**. It’s easy to see that parallel flows are a subset of concurrent flows—parallel flows are always concurrent, but not all concurrent flows are parallel.

If processes are not concurrent, they are **sequential**.

### Private Address Space: How Processes Monopolize the System’s Address Space

This corresponds to the second abstraction: the process monopolizes the system’s address space.

A process allocates a **private address space** for each program, meaning this space cannot be read or written by other processes.

### User Mode and Kernel Mode: Restrictions to the processes

With processes, we need a mechanism to restrict the execution of instructions and the range of private address spaces. This is achieved by setting a mode bit that divides processes into two modes: **user mode** and **kernel mode**. In kernel mode, a process can access all instructions and memory locations; in contrast, in user mode, the process has no privileges and can only access a limited part of its address space.

The only way to switch between user mode and kernel mode is through **exceptions**.

### Context and Context Switching

Every program in the system has a state. Therefore, a process's **context** consists of the states required to manage the program's normal operation, including the stack, general-purpose registers, program counter, environment variables, and the set of open file descriptors.

To determine when to preempt a process, the kernel makes decisions, and the corresponding decision-maker is called the **scheduler**. The kernel **schedules** processes, meaning it controls the transfer from the current process to a new one. The mechanism for this is known as a **context switch**:
1. Save the context of the current process.
2. Restore the context of the new (or previously preempted) process.
3. Transfer control to the new process.

Combining the above two sections, process switching can be summarized as shown in the diagram below:

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408141301953.png)
