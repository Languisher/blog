---
title: "[CS:APP Chapter 6] Memory Hierarchy"
description: "Notes according to Computer System: A Programmer's Perspective Third Edition Chapter 6"
date: 2024-09-01
category: ["Computer System"]
tags: ["CS:APP"]
mathjax: true
---

## Overview of Memory Hierarchies

Reference: Computer System: A Programmer's Perspective Third Edition Chapter 6

A typical memory hierarchy is shown below:

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408062158551.png)

Storage devices with a faster transmission rate are often smaller and more expensive. Our goal in designing a memory system is to achieve a high transmission rate and ample storage spaces. A modern memory system fully exploits the potential of the above storage devices. It is based on two key concepts: **locality** of computer programs and **caching** between two consecutive memory hierarchies.

## Locality

First, let's introduce the concept of **locality**: Computer programs tend to reference data items that are either _close_ to other recently referenced data items or the _same_ data items that were recently referenced. In other words, after we access certain data, we're more likely to access the same data again or data that is nearby in the future.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/%E6%97%B6%E9%97%B4%E5%B1%80%E9%83%A8%E6%80%A7%E5%92%8C%E7%A9%BA%E9%97%B4%E5%B1%80%E9%83%A8%E6%80%A7.drawio.png)

These two patterns correspond to two types of locality: **spatial locality** and **temporal locality**. As shown in the diagram and according to their definitions, these concepts are straightforward to understand:
- **Temporal locality** means that a memory location that has been referenced once is likely to be referenced multiple times in the near future, such as repetitive writes to the `sum` variable in a for-loop.
- **Spatial locality** means that if a memory location is referenced once, the program is likely to reference a nearby memory location in the near future, such as accessing multiple elements in an array with a *stride-1 reference pattern*.

An example of pseudocode that demonstrates these two traits:

```
sum = 0
for i in 1...M
	for j in 1...N
		sum += matrix[i][j]
```

Here, we visit the two-dimensional array in *row-major order*, which exhibits spatial locality, and we repetitively visit the variable `sum`, which exhibits temporal locality. However, if we invert the sequence of `j` and `i`, this would result in a *stride-N reference pattern*, causing the program to lose spatial locality.

### Example of matrix multiplication (matmul)

Multiplication between large matrices can be very time-consuming, especially since matrices are stored in multi-dimensional arrays. Leveraging the principles of locality, both spatial and temporal, can significantly improve performance.

The mathematical formula for matrix multiplication is as follows: $$ \forall i \in [1,I], \quad \forall j \in [1,J], \quad c_{i,j} = \sum_{k=1}^K a_{i, k} b_{k, j} \quad \text{where } A=(A_{i,k}), B=(B_{k,j}) $$

**Spatial locality** suggests that accessing consecutive elements in memory is beneficial for performance. To optimize matrix multiplication by taking advantage of spatial locality, we can reorganize the computation: $$ (i,:) = (i,k)\times(k,:) $$
The corresponding pseudocode is:

```
Input: A, B; Output: C
for i in 1...I
	for k in 1...K
		r = A[i][k]
		for j in 1...J
			C[i][k] += B[k][j] * r
```

## Cache

In a memory hierarchy, each level holds data that is retrieved from the next level. **Cache** is a memory space in the upper level; however, it stores data in the lower level, allowing us to visit larger memory spaces with higher transmission speeds. In other words, a cache that is in level k has the following traits:
- It resides in the level k memory devices, thus enjoying the speed of level k
- The storage inside it is a subset of the contents in level k+1 memory devices, thus enjoying the cost of level k+1

Datas are stored in **blocks**. A block contains multiple data with a single address.
### How to transfer data between different layers using cache

Data transmission between two consecutive layers of memory devices (i.e., a level-(k+1) memory device and a level-k cache) is completed using a **transmission unit**, which contains one or multiple blocks.



The process begins when a request comes from a higher level to a lower level, retrieving lower-level data to pass from bottom to top. When we need data `d` from level-(k+1), two possibilities arise:
- **Cache hit**: If `d` is already in the level-k cache, we directly fetch `d` from the level-k cache.
- **Cache miss**: If `d` is not in the level-k cache, the following steps occur:
  1. Identify where the block that contains `d` (denoted as `b`) should be placed based on the **placement strategy** (e.g., `b mod 4`).
  2. If the cache is already full (which is usually the case), we need to **evict** or **replace** a block in the cache according to a certain **replacement strategy** (e.g., LRU). The evicted block is called a **victim block**.
  3. Fetch the block from level-(k+1) and place it in the level-k cache.

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Drawing%202024-09-01%2011.52.03.excalidraw.png)

**Note on Placement Strategy and Replacement Strategy**: These policies work together in a coordinated manner. The placement policy defines the potential locations for a block in the cache, and when the cache is full, the replacement policy selects which block among the possible locations should be evicted to make space for the new block.

### Types of cache misses

The cache will only be empty when the memory devices are first started. This situation, where the cache is initially empty, is known as a **cold cache**, and a cache miss due to this reason is called a **cold miss** or **compulsory miss**.

Due to an ineffective placement strategy, blocks may be limited to certain cache blocks. For example, using `b mod 4` would limit blocks numbered 1, 5, 9, 13, etc., to the same cache block, preventing these blocks from being stored simultaneously in the cache, even if there is enough space for all of them. This is known as a **conflict miss**.

Additionally, if the number of requested blocks exceeds the cache’s maximum capacity, we must fetch the data multiple times. This is known as a **capacity miss**.

### Cache + Locality shows effectiveness

Cache-based memory hierarchy works also based on locality principles:
- Data are packaged into blocks that present spatial locality.
- Data loaded into cache are expected to be accessed in the future, presenting temporal locality.
- Stride-1 reference pattern makes it possible to predict and prefetch data.

## Cache Structure

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202409011349854.png)

The cache is organized into several components. Consider a cache with a total of **C** memory bits.

- The smallest unit is the **cache line**, where each line contains a single block of data with size **B = 2^b** bytes. In addition to the data block, each line has a **valid bit** to indicate whether the data block contains valid data or if it is filled with arbitrary data. The **tag bits** (consisting of **t** bits) are used to identify the specific line within a group. Each line within a group has a unique tag.

- Cache lines are grouped into **sets**. A set consists of **E = 2^e** lines (where **E** can be as small as 1).

- Finally, the cache is composed of **S = 2^s** sets.

We can deduce that:

$$ C = S \times E \times B = 2^s \times 2^e \times 2^b $$

The reason for introducing the concept of sets, rather than placing all lines into a single set, will be explained in a later chapter.

The address used to identify and fetch data consists of **m** bits, where **s** bits are used as the set index, **b** bits are used as the offset within a block, and the remaining **t = (m - s - b)** bits are used as the tag. If the tag in the address matches the tag in the set (and the valid bit is set to 1), then the data word is found within that set.

### How to Read Data Using the Address

The CPU first sends the address to the cache and requests the data stored at that address. The process involves three main steps: (1) Set Selection, (2) Line Matching, and (3) Bit Selection.[^1]

[^1]: For a detailed explanation, see Chapter 6.4.2.

1. **Set Selection**: The cache first selects the set that corresponds to the set index in the address.

2. **Line Matching**: For the data in the cache line to match the requested data, two conditions must be met:
   - The **valid bit** must be set to 1.
   - The **tag bits** in the address must match the tag bits of the line in the cache.

   If both conditions are met, the cache registers a hit. If either condition fails, it results in a cache miss, requiring the system to fetch the data from the main memory, update the cache line with the new data, and update the tag.

3. **Bit Selection**: The lower **b** bits of the address determine the offset within the block to select the specific data word.


![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202409011406202.png)
### Direct-mapped cache: E=1

A direct-mapped cache has only one line per set, making it the simplest type of cache to implement: there’s no need to distinguish between different lines within a set, and no searching algorithm is required.

However, if we need to read data that shares the same set index but has different tags (such as addresses 0 and 8 in the example), the cache will repeatedly eject and load the same set with these blocks. This phenomenon is known as **thrashing**. Thrashing leads to continuous cache misses, and the root cause is that different addresses are forced to map to the same location, resulting in conflicts.

### Fully associative cache: S=1

A fully associative cache is the opposite of a direct-mapped cache: all lines are contained in a single set, so no conflicts occur (since no two addresses are forced to map to the same location). However, the only way to distinguish between lines is by identifying the tags in parallel. This requires the tags to contain multiple bits, and necessitates the design of a very complex searching or matching algorithm.

### Set associative cache

A set associative cache strikes a balance between direct-mapped and fully associative caches. In a set associative cache, the cache is divided into several sets, and each set contains multiple lines. A memory address is mapped to a specific set based on the set index, but within that set, any line can be used to store the data.

This approach reduces the likelihood of thrashing, as multiple lines within a set can accommodate different data blocks that share the same set index. Therefore, it combines the simplicity of a direct-mapped cache with the flexibility of a fully associative cache.

### Write data, use cache or not?

There are two primary strategies for handling writes in a cache:

- **No-Write-Allocate**: Writing is performed directly to the main memory, bypassing the cache entirely.
- **Write-Allocate**: The data is first loaded into the cache, and then the cache line is updated.

The cache is utilized to leverage locality, which is beneficial in scenarios where we anticipate performing writes to nearby or consecutive lines (blocks).

Writing to a block in the cache can also be managed using two different approaches:

- **Write-Through**: Any changes made to the cache are immediately written through to the main memory. This ensures that the memory always reflects the most recent data, but it can lead to a higher frequency of writes to memory.

- **Write-Back**: The updated block is only written to the main memory when it is evicted from the cache (i.e., selected as the victim line). This approach reduces the number of memory write operations by delaying them until necessary. To implement write-back, a **dirty bit** is used to indicate whether the data in the cache block has been modified (is "dirty") and thus differs from the data in the main memory.
