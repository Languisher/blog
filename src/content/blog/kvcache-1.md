---
title: "KVCache 1: Intrinsic Limitations"
description: "KVCache 分析系列一"
date: 2024-08-30
category: ["LLM"]
tags: ["Transformer", "KVCace"]
mathjax: true
---

An overview of Transformer-Based Models:

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/Drawing%202024-08-27%2013.23.31.excalidraw.png)

## Intrinsic Limitations of KVCache

**KVCache** is a method used to avoid repeated calculations of K and V values in Transformer blocks, but it comes with significant memory consumption.

How large could KVCache potentially be when generating the next token during the auto-regressive stage? The next section will address this question.

### KVCache Memory Grows Linearly with Multiple Aspects

In the *Prefilling stage*, the entire input sequence of tokens is fed into the Transformer blocks. The dimension of the embedded input token (denoted as $h_n$, corresponding to the `hidden_states` variable in the GPT-2 implementation) is `[seq_len, embedding_dimension]`. Within a single Transformer block, this embedded token first passes through a Conv1D layer to generate the query, key, and value vectors using the following formula: 

$$ q_n = W^Q.h_n, \quad k_n = W^K.h_n, \quad v_n = W^V.h_n $$

Each of these vectors retains the same dimension as the embedded token. After the convolutional layer, the *Multi-head Attention (MHA)* mechanism splits the three vectors into different heads. Consequently, we obtain `num_heads` key and value vectors, each with the dimension `[seq_len, head_dimension]`.

The total memory consumed by the key and value vectors is calculated as follows:

$$ 2q_a \times n_\text{layers} \times n_\text{heads} \times (l_\text{sequence} \times d_\text{head}) $$

Here, q_a represents the precision (e.g., `float16` or `float32`, which differ in memory consumption per unit).

These key and value vectors are stored because, in the _Auto-Regressive Stage_, they are crucial for generating subsequent tokens in the sequence. During this stage, the model repeatedly accesses the stored key and value vectors to compute attention scores for each new token, ensuring that the generated text maintains coherence and context. 

**Consequence**: The memory consumption of the KVCache grows linearly with the sequence length, number of layers, and number of attention heads.

### KVCache Memory Consumption Becomes a Bottleneck of Transformer-Based Models

Consider the example of deploying [LLaMa-2-7B](https://huggingface.co/meta-llama/Llama-2-7b), a relatively lightweight model, on an [A10 GPU](https://www.nvidia.com/en-us/data-center/products/a10-gpu/), which has 24 GB of memory. The model weights alone require approximately 10 GB of storage.

With the configuration `n_layers=32`, `n_heads=32`, `d_head=128`, and `d_model=4096`, a single GPU can generate a maximum of only 20k tokens before running out of memory, as the KVCache consumes about 0.5 MB per token, which is astonishing. [^1]

Addressing the KVCache memory consumption issue becomes crucial, especially as model sizes and sequence lengths continue to grow.

[^1]: [P. Plienhar, “LLM Inference Series 4: KV Caching - A Deeper Look.”](https://medium.com/@plienhar/llm-inference-series-4-kv-caching-a-deeper-look-4ba9a77746c8)