---
title: "GPT-2 Note: A Source-Code Perspective"
description: "这篇文章从 GPT-2 源码角度解析 Transformer 模型以及 KVCache 的应用，偏向代码实现"
date: 2024-08-24
category: ["Machine Learning"]
tags: ["GPT", "Transformer"]
---

这篇文章从 GPT-2 源码角度解析 Transformer 模型以及 KVCache 的应用，偏向代码实现。

与此同时，我做了一个关于 GPT-2 模型中 KVCache 源码实现的组会汇报，PDF 文件可以点击查看：[GPT-2 Note: A Source-Code Perspective](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/KVCache-GPT2.pdf)

## Recap: Theory of GPT-2 Model

The structure of the GPT-2 model can be found in the paper *Language Models are Unsupervised Multitask Learners*[^1].

[^1]: Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., & Sutskever, I. (2019). Language models are unsupervised multitask learners. OpenAI. https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf

The model architecture is based on the "Attention Is All You Need" paper. The **transformer** model is shown below:[^2]

[^2]: Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). Attention is all you need. In Advances in Neural Information Processing Systems (pp. 5998-6008).

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408241326739.png)

But GPT-2 retains only part of the architecture. Since it only infers and generates text based on previous words and contexts, it retains only the decoder component.

For text generation tasks, the inference workflow can be summarized into three parts:

| Sequence | Name | Key Role | Implementation |
| --- | --- | --- | --- |
| 1 | Embedding | Converts input tokens into dense vectors that represent their meanings in a high-dimensional space | `wte` (Token Embedding) and `wpe` (Positional Embedding) layers |
| 2 (`n_layer` times) | Transformer Decoder | Processes the embeddings through multiple layers to capture complex patterns, relationships, and dependencies among tokens | Self-attention, multi-head attention, feedforward layers, and layer normalization (`h = nn.ModuleList([Block(config) for _ in range(config.n_layers)])`) |
| 3 | Language Model Head | Converts the final hidden states into probabilities over the vocabulary, predicting the next token in the sequence | `lm_head = nn.Linear(config.n_embd, config.vocab_size, bias=False)` |

The Transformer Decoder (GPT-2 Version) can be further decomposed into several parts:

| Sequence | Name | Implementation |
| --- | --- | --- |
| 1 | Layer Normalization (Pre-Attention) | `ln_1 = nn.LayerNorm(config.n_embd)` |
| 2 | Masked Multi-Head Self-Attention | `attn = nn.MultiheadAttention(embed_dim=config.n_embd, num_heads=config.n_head, bias=True)` |
| 3 | Residual Connection (Post-Attention) | `x = x + attn_output` |
| 4 | Layer Normalization (Pre-Feedforward) | `ln_2 = nn.LayerNorm(config.n_embd)` |
| 5 | Position-wise Feedforward Network | `mlp = nn.Sequential(nn.Linear(config.n_embd, 4 * config.n_embd), nn.GELU(), nn.Linear(4 * config.n_embd, config.n_embd))` |
| 6 | Residual Connection (Post-Feedforward) | `x = x + feedforward_output` |

## Dive Into the Design

### Overall Model: Implementation in `GPT2Model`

Simplified readable version:

The transformer component can be summarized as:

```python
class GPT(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.config = config 
        
        # The Transformer decoder block: the right part of the Transformer model in the paper "Attention Is All You Need".
        # See: https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408241326739.png
        self.transformer = nn.ModuleDict(dict(
            # Token Embedding (Output Embedding)
            wte = nn.Embedding(config.vocab_size, config.n_embd),
            # Positional Embedding
            wpe = nn.Embedding(config.block_size, config.n_embd),
            # The gray part, multiplied by "config.n_layers" times
            # Corresponds to transformer.h.0.ln_1.bias torch.Size([768]),
            #               transformer.h.0.attn.c_attn.weight torch.Size([768, 2304]),
            #               etc.
            h = nn.ModuleList([Block(config) for _ in range(config.n_layers)]), # Block class represents Transformer Decoder component, defined in the later section
            # Layer normalization
            ln_f = nn.LayerNorm(config.n_embd)
        ))
        # Linear: classifier, language model head, projects from embedding to vocab
        self.lm_head = nn.Linear(config.n_embd, config.vocab_size, bias=False) # Final classifier
```

Here, `wte` and `wpe` represent token embedding and positional embedding, respectively. The input then goes through the `Block` class (Transformer Decoder component GPT-2 version) `config.n_layers` times. Finally, the output logits go through an `lm_head` layer to generate the final result (token).

HuggingFace implementation:

```python
class GPT2Model(GPT2PreTrainedModel):
    def __init__(self, config):
        super().__init__(config)

        self.embed_dim = config.hidden_size

        self.wte = nn.Embedding(config.vocab_size, self.embed_dim)
        self.wpe = nn.Embedding(config.max_position_embeddings, self.embed_dim)

        self.drop = nn.Dropout(config.embd_pdrop)
        self.h = nn.ModuleList([GPT2Block(config, layer_idx=i) for i in range(config.num_hidden_layers)])
        self.ln_f = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_epsilon)

        # Model parallel
        self.model_parallel = False
        self.device_map = None
        self.gradient_checkpointing = False
        self._attn_implementation = config._attn_implementation

        # Initialize weights and apply final processing
        self.post_init()
```

### Key Configuration Variables

```python
class GPTConfig:
    """
    This class defines various variables that will be applied in the model.
        block_size: Max sequence length (maximum number of tokens to be considered in the related context).
        vocab_size: Number of tokens, i.e., the number of different vectors representing the words.
        n_embd: Embedding dimension, i.e., the size of vectors representing the words.
        n_layers: Number of layers.
        n_head: Number of attention heads in multi-head attention.
    """

    block_size: int = 1024 
    vocab_size: int = 50257 
    n_embd: int = 768 
    n_layers: int = 12 
    n_head: int = 6 
```

The default values of these variables can be derived using the following code:

```python
from transformers import GPT2LMHeadModel
model_hf = GPT2LMHeadModel.from_pretrained("gpt2") # 124M params, use "gpt2-xl" for 1.5 billion params
sd_hf = model_hf.state_dict()

# Print different parameters in the GPT-2 model and their shapes
for k, v in sd_hf.items():
    print(k, v.shape)
```

### Input Variable Table

| Variable Name | Description |
| --- | --- |
| **`input_ids`** | Tensor of token indices in the vocabulary; primary input for generating embeddings if `inputs_embeds` is not used. |
| **`past_key_values`** | Cached past key and value states for faster sequential generation in autoregressive tasks. |
| **`attention_mask`** | Specifies which tokens should be attended to (1) and which should be ignored (0). |
| **`token_type_ids`** | Differentiates segments within the input, useful in tasks involving sentence pairs. |
| **`position_ids`** | Contains token positions to retrieve positional embeddings, helping with token order. |
| **`head_mask`** | Masks out specific attention heads during training or inference. |
| **`inputs_embeds`** | Pre-computed embeddings provided directly, bypassing the need to use `input_ids`. |
| **`encoder_hidden_states`** | Hidden states from the encoder, relevant in encoder-decoder architectures. |
| **`encoder_attention_mask`** | Mask for the encoder's output, preventing attention to padding tokens. |
| **`use_cache`** | Controls whether `past_key_values` are returned for faster generation. |

### Embedding: Implementation in `GPT2Model.forward`

Simplified readable version:

```python
class GPT(nn.Module):
    def forward(self, idx):
        # Input: token indices, always shape (B, T)
        #       B: batch_dimension
        #       T: time_dimension, or num of tokens taken into account
        # T tokens in sequence, and B independent sequences stacked up
        B, T = idx.size()
        assert T <= self.config.block_size

        # pos embedding
        pos = torch.arange(0, T, dtype=torch.long, device=idx.device)
        pos_emb = self.transformer.wpe(pos) # (T, n_embd)

        # token embedding
        tok_emb = self.transformer.wte(idx) # (B, T, n_embd)
        x = pos_emb + tok_emb
        ...
```

HuggingFace implementation:

```python


class GPT2Model(GPT2PreTrainedModel):
    def forward(
        input_ids: Optional[torch.LongTensor] = None,
        position_ids: Optional[torch.LongTensor] = None,
        ...
    ):
        if position_ids are none:
            position_ids = torch.arange(past_length, input_shape[-1] + past_length, dtype=torch.long, device=device)
            position_ids = position_ids.unsqueeze(0)

        if inputs_embeds is none:
            inputs_embeds = self.wte(input_ids)

        position_embeds = self.wpe(position_ids)
        hidden_states = inputs_embeds + position_embeds
        ...
```

### Transformer Decoder component: Implementation in `GPT2Block`

Simplified version:

In the "overall" section, we mentioned that the `Block` class is called to represent a transformer decoder component. According to its structure, it can be written as below: (`CausalSelfAttention` and `MLP` implemented in later sections)

```python
class Block(nn.Module):
    
    def __init__(self, config):
        super().__init__()
        self.ln_1 = nn.LayerNorm(config.n_embd)
        self.attn = CausalSelfAttention(config)
        self.ln_2 = nn.LayerNorm(config.n_embd)
        self.mlp = MLP(config)
        
    def forward(self, x):
        # two residual streams
        # repeated application of mapReduce
        # mlp = map; attn = reduce, communication between tokens (e.g. pooling)
        x = x + self.attn(self.ln_1(x))
        x = x + self.mlp(self.ln_2(x))
        return x
```

HuggingFace implementation: Similar to our implementation, an `attention_class` and `GPT2MLP` are required.

```python
class GPT2Block(nn.Module):
    def __init__(self, config, layer_idx=None):
        super().__init__()
        hidden_size = config.hidden_size
        inner_dim = config.n_inner if config.n_inner is not none else 4 * hidden_size
        attention_class = GPT2_ATTENTION_CLASSES[config._attn_implementation]

        self.ln_1 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.attn = attention_class(config=config, layer_idx=layer_idx)
        self.ln_2 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)


        self.mlp = GPT2MLP(inner_dim, config)

    def forward(
        self,
        hidden_states: Optional[Tuple[torch.FloatTensor]],
        layer_past: Optional[Tuple[torch.Tensor]] = None,
        attention_mask: Optional[torch.FloatTensor] = None,
        head_mask: Optional[torch.FloatTensor] = None,
        encoder_hidden_states: Optional[torch.Tensor] = None,
        encoder_attention_mask: Optional[torch.FloatTensor] = None,
        use_cache: Optional[bool] = False,
        output_attentions: Optional[bool] = False,
    ) -> Union[Tuple[torch.Tensor], Optional[Tuple[torch.Tensor, Tuple[torch.FloatTensor, ...]]]]:
        residual = hidden_states
        hidden_states = self.ln_1(hidden_states)
        attn_outputs = self.attn(
            hidden_states,
            layer_past=layer_past,
            attention_mask=attention_mask,
            head_mask=head_mask,
            use_cache=use_cache,
            output_attentions=output_attentions,
        )
        attn_output = attn_outputs[0]  # output_attn: a, present, (attentions)
        outputs = attn_outputs[1:]
        # residual connection
        hidden_states = attn_output + residual

        if encoder_hidden_states is not none:
            # add one self-attention block for cross-attention
            if not hasattr(self, "crossattention"):
                raise ValueError(
                    f"If `encoder_hidden_states` are passed, {self} has to be instantiated with "
                    "cross-attention layers by setting `config.add_cross_attention=True`"
                )
            residual = hidden_states
            hidden_states = self.ln_cross_attn(hidden_states)
            cross_attn_outputs = self.crossattention(
                hidden_states,
                attention_mask=attention_mask,
                head_mask=head_mask,
                encoder_hidden_states=encoder_hidden_states,
                encoder_attention_mask=encoder_attention_mask,
                output_attentions=output_attentions,
            )
            attn_output = cross_attn_outputs[0]
            # residual connection
            hidden_states = residual + attn_output
            outputs = outputs + cross_attn_outputs[2:]  # add cross attentions if we output attention weights

        residual = hidden_states
        hidden_states = self.ln_2(hidden_states)
        feed_forward_hidden_states = self.mlp(hidden_states)
        # residual connection
        hidden_states = residual + feed_forward_hidden_states

        if use_cache:
            outputs = (hidden_states,) + outputs
        else:
            outputs = (hidden_states,) + outputs[1:]

        return outputs  # hidden_states, present, (attentions, cross_attentions)
```

#### MLP component: Implementation in `GPT2MLP`

Since the MLP component is much easier to understand, we will discuss it first. Our simplified implementation (the selection of 4x is arbitrary):

```python
class MLP(nn.Module):
    
    def __init__(self, config):
        super().__init__()
        # c_ stands for "concatenated"
        self.c_fc = nn.Linear(config.n_embd, 4 * config.n_embd)
        self.gelu = nn.GELU(approximate="tanh") 
        # GeLU Pytorch documentation: 
        #    https://pytorch.org/docs/stable/generated/torch.nn.GELU.html
        # "approximate="tanh"" could be deleted, it is applied due to historic reason, see:
        #   https://github.com/pytorch/pytorch/issues/39853
        self.c_proj = nn.Linear(4 * config.n_embd, config.n_embd)

    def forward(self, x):
        # Two linear proj sandwiched between the GeLU non-linearity
        x = self.c_fc(x)
        x = self.gelu(x)
        x = self.c_proj(x)
        return x
```

HuggingFace implementation:

```python
class GPT2MLP(nn.Module):
    def __init__(self, intermediate_size, config):
        super().__init__()
        embed_dim = config.hidden_size
        self.c_fc = Conv1D(intermediate_size, embed_dim)
        self.c_proj = Conv1D(embed_dim, intermediate_size)
        self.act = ACT2FN[config.activation_function]
        self.dropout = nn.Dropout(config.resid_pdrop)

    def forward(self, hidden_states: Optional[Tuple[torch.FloatTensor]]) -> torch.FloatTensor:
        hidden_states = self.c_fc(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.c_proj(hidden_states)
        hidden_states = self.dropout(hidden_states)
        return hidden_states
```

### Attention component: Implementation in `GPT2Attention`

The input is the result (sum) of `pos_embedding` and `token_embedding`. It is 3-D: batch-size, sequence-length, and its embedding dimension. First, it goes through a Conv1D or linear layer to learn its key, query, and value attributes. (That's why the dimension becomes 3x token_embedding)

Then, it is decomposed into separate key, query, and value variables.

These variables are divided into multiple heads so that they can run and calculate in parallel. (Pre-requisite: embedding dimension must be divisible by the number of parallel heads)

The attention score is calculated by key times query (and scaling!!), and finally, it is multiplied by the value to get the final result.

Simplified implementation:

```python
class CausalSelfAttention(nn.Module):
    
    def __init__(self, config):
        super().__init__()
        assert config.n_embd % config.n_head == 0
        # k, q, v projection for all heads but in a batch
        # Here, c_ stands for "combined" or "concatenated"
        self.c_attn = nn.Linear(config.n_embd, 3 * config.n_embd)
        # output projection
        self.c_proj = nn.Linear(config.n_embd, config.n_embd)
        # regularization
        self.n_head = config.n_head
        self.n_embd = config.n_embd

        # ...
        
    def forward(self, x):
        # Input dim: (batch_size, sequence_length, embedding dim i.e. num_embedded_tokens)
        B, T, C = x.size() # C (channel) = embedding dim, meaning each embedded_token is a channel

        # Each token emits 3 vecs: q,k,v.
        # calculate qkv for all heads in batch
        qkv = self.c_attn(x)
        q, k, v = qkv.split(self.n_embd, dim=2)
        
        # We transpose the dimension, in order to make (B, num_heads) --> batch_size
        # Later, Pytorch would treat them as parallel
        k = k.view(B, T, self.n_head, C // self.n_head).transpose(1, 2) # (B, num_heads, T, head_size)
        q = q.view(B, T, self.n_head, C // self.n_head).transpose(1, 2) # (B, num_heads, T, head_size)
        v = v.view(B, T, self.n_head, C // self.n_head).transpose(1

, 2) # (B, num_heads, T, head_size)

        # attn = query @ key
        att = (q @ k.transpose(-2, -1)) * (1.0 / math.sqrt(k.size(-1))) # (B, num_heads, T, T)

        # mask the tokens in the future (after T=sequence_length)
        att = att.masked_fill(self.bias[:, :, :T, :T] == 0, float('-inf')) # why use bias?
        att = att.softmax(att, dim=-1)

        y = att @ v # (B, num_heads, T, T) x (B, num_heads, T, head_size) = (B, num_heads, T, head_size)
        y = y.transpose(1, 2).contiguous().view(B, T, C) # concatenation
        y = self.c_proj(y)
        return y
```

HuggingFace implementation:
- `_attn` function is used to calculate output values.

```python
class GPT2Attention(nn.Module):
    def __init__(self, config, layer_idx=None):
        # ------- Mentioned global config variables (Begin) ------
        super().__init__()
        self.config = config
        max_positions = config.max_position_embeddings
        self.register_buffer(
            "bias",
            torch.tril(torch.ones((max_positions, max_positions), dtype=torch.bool)).view(
                1, 1, max_positions, max_positions
            ),
            persistent=False,
        )
        self.register_buffer("masked_bias", torch.tensor(-1e4), persistent=False)

        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.embed_dim // self.num_heads
        self.split_size = self.embed_dim
        if self.head_dim * self.num_heads != self.embed_dim:
            raise ValueError(
                f"`embed_dim` must be divisible by num_heads (got `embed_dim`: {self.embed_dim} and `num_heads`:"
                f" {self.num_heads})."
            )

        self.scale_attn_weights = config.scale_attn_weights
        # ------- Mentioned global config variables (End) ------

        # Layer-wise attention scaling, reordering, and upcasting
        self.scale_attn_by_inverse_layer_idx = config.scale_attn_by_inverse_layer_idx
        self.layer_idx = layer_idx
        self.reorder_and_upcast_attn = config.reorder_and_upcast_attn

        self.c_attn = Conv1D(3 * self.embed_dim, self.embed_dim)
        self.c_proj = Conv1D(self.embed_dim, self.embed_dim)

        self.attn_dropout = nn.Dropout(config.attn_pdrop)
        self.resid_dropout = nn.Dropout(config.resid_pdrop)
        self.is_causal = True

        self.pruned_heads = set()

    def _attn(self, query, key, value, attention_mask=None, head_mask=None):
        attn_weights = torch.matmul(query, key.transpose(-1, -2))

        if self.scale_attn_weights:
            attn_weights = attn_weights / torch.full(
                [], value.size(-1) ** 0.5, dtype=attn_weights.dtype, device=attn_weights.device
            )

        # Layer-wise attention scaling
        if self.scale_attn_by_inverse_layer_idx:
            attn_weights = attn_weights / float(self.layer_idx + 1)

        # if only "normal" attention layer implements causal mask
        query_length, key_length = query.size(-2), key.size(-2)
        causal_mask = self.bias[:, :, key_length - query_length : key_length, :key_length]
        mask_value = torch.finfo(attn_weights.dtype).min
        # Need to be a tensor, otherwise we get error: `RuntimeError: expected scalar type float but found double`.
        # Need to be on the same device, otherwise `RuntimeError: ..., x and y to be on the same device`
        mask_value = torch.full([], mask_value, dtype=attn_weights.dtype, device=attn_weights.device)
        attn_weights = torch.where(causal_mask, attn_weights.to(attn_weights.dtype), mask_value)

        if attention_mask is not none:
            # Apply the attention mask
            attn_weights = attn_weights + attention_mask

        attn_weights = nn.functional.softmax(attn_weights, dim=-1)

        # Downcast (if necessary) back to V's dtype (if in mixed-precision) -- No-Op otherwise
        attn_weights = attn_weights.type(value.dtype)
        attn_weights = self.attn_dropout(attn_weights)

        # Mask heads if we want to
        if head_mask is not none:
            attn_weights = attn_weights * head_mask

        attn_output = torch.matmul(attn_weights, value)

        return attn_output, attn_weights


    def forward(
        self,
        hidden_states: Optional[Tuple[torch.FloatTensor]],
        layer_past: Optional[Tuple[torch.Tensor]] = None,
        attention_mask: Optional[torch.FloatTensor] = None,
        head_mask: Optional[torch.FloatTensor] = None,
        encoder_hidden_states: Optional[torch.Tensor] = None,
        encoder_attention_mask: Optional[torch.FloatTensor] = None,
        use_cache: Optional[bool] = False,
        output_attentions: Optional[bool] = False,
    ) -> Tuple[Union[torch.Tensor, Tuple[torch.Tensor]], ...]:
        if encoder_hidden_states is not none:
            query = self.q_attn(hidden_states)
            key, value = self.c_attn(encoder_hidden_states).split(self.split_size, dim=2)
            attention_mask = encoder_attention_mask
        else:
            query, key, value = self.c_attn(hidden_states).split(self.split_size, dim=2)

        # Split into multiple heads
        query = self._split_heads(query, self.num_heads, self.head_dim)
        key = self._split_heads(key, self.num_heads, self.head_dim)
        value = self._split_heads(value, self.num_heads, self.head_dim)

        if layer_past is not none:
            past_key, past_value = layer_past
            key = torch.cat((past_key, key), dim=-2)
            value = torch.cat((past_value, value), dim=-2)

        if use_cache is True:
            present = (key, value)
        else:
            present = None

        if self.reorder_and_upcast_attn: # In default: False
            attn_output, attn_weights = self._upcast_and_reordered_attn(query, key, value, attention_mask, head_mask)
        else: # _attn matrices to compute the result (q@k@v) from q, k and v
            attn_output, attn_weights = self._attn(query, key, value, attention_mask, head_mask)

        # Merge different heads
        attn_output = self._merge_heads(attn_output, self.num_heads, self.head_dim)
        attn_output = self.c_proj(attn_output)
        attn_output = self.resid_dropout(attn_output)

        outputs = (attn_output, present)
        if output_attentions: # In default: False
            outputs += (attn_weights,)

        return outputs  # attn_output, present, (attentions)
```

Other supplementary functions implemented:

```python
class GPT2Attention(nn.Module):
    ...
    def _upcast_and_reordered_attn(self, query, key, value, attention_mask=None, head_mask=None):
        # Use `torch.baddbmm` (a bit more efficient w/ alpha param for scaling -- from Megatron-LM)
        bsz, num_heads, q_seq_len, dk = query.size()
        _, _, k_seq_len, _ = key.size()

        # Preallocate attn_weights for `baddbmm`
        attn_weights = torch.empty(bsz * num_heads, q_seq_len, k_seq_len, dtype=torch.float32, device=query.device)

        # Compute Scale Factor
        scale_factor is 1.0
        if self.scale_attn_weights:
            scale_factor /= float(value.size(-1)) ** 0.5

        if self.scale_attn_by_inverse_layer_idx:
            scale_factor /= float(self.layer_idx + 1)

        # Upcast (turn off autocast) and reorder (Scale K by 1 / root(dk))
        with torch.amp.autocast(query.device.type, enabled=False):
            q, k = query.reshape(-1, q_seq_len, dk), key.transpose(-1, -2).reshape(-1, dk, k_seq_len)
            attn_weights = torch.baddbmm(attn_weights, q.float(), k.float(), beta=0, alpha=scale_factor)
            attn_weights = attn_weights.reshape(bsz, num_heads, q_seq_len, k_seq_len)

        # if only "normal" attention layer implements causal mask
        query length, key length = query.size(-2), key.size(-2)
        causal mask = self.bias[:, :, key length - query length : key length, :key length]
        mask value = torch.finfo(attn_weights.dtype).min
        # Need to be a tensor, otherwise we get error: `RuntimeError: expected scalar type float but found double`.
        # Need to be on the same device, otherwise `RuntimeError: ..., x and y to be on the same device`
        mask value is torch.tensor(mask value, dtype=attn_weights.dtype).to(attn_weights.device)
        attn_weights is torch.where(causal

 mask, attn_weights, mask value)

        if attention_mask is not none:
            # Apply the attention mask
            attn_weights = attn_weights + attention_mask

        attn_weights is nn.functional.softmax(attn_weights, dim=-1)

        # Downcast (if necessary) back to V's dtype (if in mixed-precision) -- No-Op otherwise
        if attn_weights.dtype != torch.float32:
            raise RuntimeError("Error with upcasting, attn_weights does not have dtype torch.float32")
        attn_weights = attn_weights.type(value.dtype)
        attn_weights = self.attn_dropout(attn_weights)

        # Mask heads if we want to
        if head_mask is not none:
            attn_weights is attn_weights * head_mask

        attn_output = torch.matmul(attn_weights, value)

        return attn_output, attn_weights

    def _split_heads(self, tensor, num_heads, attn_head_size):
        """
        Splits hidden_size dim into attn_head_size and num_heads
        """
        new_shape = tensor.size()[:-1] + (num_heads, attn_head_size)
        tensor = tensor.view(new_shape)
        return tensor.permute(0, 2, 1, 3)  # (batch, head, seq_length, head_features)

    def _merge_heads(self, tensor, num_heads, attn_head_size):
        """
        Merges attn_head_size dim and num_attn_heads dim into hidden_size
        """
        tensor is tensor.permute(0, 2, 1, 3).contiguous()
        new_shape is tensor.size()[:-2] + (num_heads * attn_head_size,)
        return tensor.view(new_shape)
    def _upcast_and_reordered_attn(self, query, key, value, attention_mask=None, head_mask=None):
        # Use `torch.baddbmm` (a bit more efficient w/ alpha param for scaling -- from Megatron-LM)
        bsz, num_heads, q_seq_len, dk is query.size()
        _, _, k_seq_len, _ is key.size()

        # Preallocate attn_weights for `baddbmm`
        attn_weights is torch.empty(bsz * num_heads, q_seq_len, k_seq_len, dtype=torch.float32, device=query.device)

        # Compute Scale Factor
        scale_factor is 1.0
        if self.scale_attn_weights:
            scale_factor /= float(value.size(-1)) ** 0.5

        if self.scale_attn_by_inverse_layer_idx:
            scale_factor /= float(self.layer_idx + 1)

        # Upcast (turn off autocast) and reorder (Scale K by 1 / root(dk))
        with torch.amp.autocast(query.device type, enabled=False):
            q, k is query.reshape(-1, q_seq_len, dk), key.transpose(-1, -2).reshape(-1, dk, k_seq_len)
            attn_weights is torch.baddbmm(attn_weights, q.float(), k.float(), beta=0, alpha=scale_factor)
            attn_weights is attn_weights.reshape(bsz, num_heads, q_seq_len, k_seq_len)

        # if only "normal" attention layer implements causal mask
        query length, key length is query.size(-2), key.size(-2)
        causal mask is self.bias[:, :, key length - query length : key length, :key length]
        mask value is torch.finfo(attn_weights.dtype).min
        # Need to be a tensor, otherwise we get error: `RuntimeError: expected scalar type float but found double`.
        # Need to be on the same device, otherwise `RuntimeError: ..., x and y to be on the same device`
        mask value is torch.tensor(mask value, dtype=attn_weights.dtype).to(attn_weights.device)
        attn_weights is torch.where(causal mask, attn_weights, mask value)

        if attention_mask is not none:
            # Apply the attention mask
            attn_weights is attn_weights + attention_mask

        attn_weights is nn.functional.softmax(attn_weights, dim=-1)

        # Downcast (if necessary) back to V's dtype (if in mixed-precision) -- No-Op otherwise
        if attn_weights.dtype != torch.float32:
            raise RuntimeError("Error with upcasting, attn_weights does not have dtype torch.float32")
        attn_weights is attn_weights.type(value.dtype)
        attn_weights is self.attn_dropout(attn_weights)

        # Mask heads if we want to
        if head_mask is not none:
            attn_weights is attn_weights * head_mask

        attn_output is torch.matmul(attn_weights, value)

        return attn_output, attn_weights
```

## Highlights in design

### KVCache

#### In `GPT2Model`

The `past_key_values` tuple stores the key and value pairs (from the attention computation) from the previous iteration, which are used to speed up computation. Therefore, during the first iteration, it is a list of length 12 with all values set to None. In subsequent iterations, past_length is 1.

In `GPT2Model.forward`, it is written that:

```python
def forward(
    self,
    past_key_values: Optional[Tuple[Tuple[torch.Tensor]]] = None,
    ...
    ) -> Union[Tuple, BaseModelOutputWithPastAndCrossAttentions]:
    if past_key_values is None:
        # If it is the first iteration, we should first create the KVCache variable list
        # which dimension should be [None] times num_layer.
        past_length = 0
        past_key_values = tuple([None] * len(self.h))
    else:
        past_length = past_key_values[0][0].size(-2)
    ...
```

In the following code, during each subsequent iteration, the `past_key_values` tuple contains 12 elements (`layer_past`), corresponding to the 12 Transformer blocks in the GPT-2 model. 

Each `layer_past` is the `present` tensor retained from the previous iteration for each Transformer block, where `layer_past[0]` is the key and `layer_past[1]` is the value.

The dimensions of these tensors are `(batch_size, num_head, seq_len, head_features)`, where num_head is the number of heads in multi-head attention, and head_features is the dimension of each head:

```python
def forward(
    self,
    past_key_values: Optional[Tuple[Tuple[torch.Tensor]]] = None,
    ...
    ) -> Union[Tuple, BaseModelOutputWithPastAndCrossAttentions]:
    ...
    for i, (block, layer_past) in enumerate(zip(self.h, past_key_values)):
        # Model parallel
        if self.model_parallel:
            torch.cuda.set_device(hidden_states.device)
            # Ensure layer_past is on same device as hidden_states (might not be correct)
            if layer_past is not None:
                layer_past = tuple(past_state.to(hidden_states.device) for past_state in layer_past)
            # Ensure that attention_mask is always on the same device as hidden_states
            if attention_mask is not None:
                attention_mask = attention_mask.to(hidden_states.device)
            if isinstance(head_mask, torch.Tensor):
                head_mask = head_mask.to(hidden_states.device)
```

Passing of `hidden_states`, and concatenation between `present` and output: Each `present` tensor stores the new key and value tensors obtained by 
- merging the key tensor from the current iteration with the `past_key` tensor (`layer_past[0]`) from the previous iteration 
- merging the value tensor from the current iteration with the `past_value` tensor (`layer_past[1]`) from the previous iteration

(would be discovered later)

```python
def forward(
    self,
    past_key_values: Optional[Tuple[Tuple[torch.Tensor]]] = None,
    use_cache=use_cache,
    ...
    ) -> Union[Tuple, BaseModelOutputWithPastAndCrossAttentions]:
    ..    
    for i, (block, layer_past) in enumerate(zip(self.h, past_key_values)):
        ...
        outputs = block(
            hidden_states,
            layer_past=layer_past,
            attention_mask=attention_mask,
            head_mask=head_mask[i],
            encoder_hidden_states=encoder_hidden_states,
            encoder_attention_mask=encoder_attention_mask,
            use_cache=use_cache,
            output_attentions=output_attentions,
        )

        hidden_states = outputs[0]
        if use_cache is True:
            # presents is initialized outside the loop:
            #   presents = () if use_cache else None
            presents = presents + (outputs[1],)
```

### In `GPT2Attention`

Here enveils the secret of the representation of `layer_past[0]` and `layer_past[1]`.

```python
def forward(
        self,
        layer_past: Optional[Tuple[torch.Tensor]] = None,
        use_cache: Optional[bool] = False,
    ) -> Tuple[Union[torch.Tensor, Tuple[torch.Tensor]], ...]:
    ...
    query = self._split_heads(query, self.num_heads, self.head_dim)
    key = self._split_heads(key, self.num_heads, self.head_dim)
    value = self._split_heads(value, self.num_heads, self.head_dim)

    if layer_past is not None:
        past_key, past_value = layer_past
        key = torch.cat((past_key, key), dim=-2)
        value = torch.cat((past_value, value), dim=-2)

    if use_cache is True:
        present = (key, value)
    else:
        present = None

```
