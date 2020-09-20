---
title: Attention is All You Need
categories: Paper Notes
tags:
  - deep learning
date: 2019-08-16 14:00:13
toc: true
---

> NIPS 2017
> @google.com

## Contents

[pdf](https://arxiv.org/pdf/1706.03762.pdf)

### Introduction

- Recurrent Models
    - sequence operations
    - hard to do parallelization
- the Transformer
    - no recurrence
    - relying entirely on an attention mechanism
    - global dependencies
    - parallelization with 8 GPUs

<!-- more -->

### Self-attention

an attention mechanism relating different positions of a single sequence in order to compute a representation of the sequence

### Model Architecture

![](https://i.imgur.com/F8ftD1A.png)

$d_{model}=512$

#### Encoder

- $N$ = 6

#### Decoder

- $N$ = 6
- masking: self-attention layer is only allowed to attend to earlier positions in the output sequence

### Attention

#### Scaled Dot-Product Attention

![](https://i.imgur.com/XOR110Q.png)

$Attention(Q, K, V) = softmax(\frac{QK^\top}{\sqrt{d_k}})V$

divided by $\sqrt{d_k}$ to prevent extremely large

#### Multi-Head Attention

![](https://i.imgur.com/6a1jQO6.png)

$MultiHead(Q, K, V) = Concat(head_1, ..., head_h)W^O$

where $head_i = Attention(QW_i^Q, KW_i^K, VW_i^V)$

in the work
- $h = 8$
- $d_k = d_v = d_{model}/h = 64$
- $h$ projections for doing parallelization

###### Applications

- encoder-decoder attention
    - queries: from previous decoder layer
    - keys and values: from encoder
    - allows every position in the decoder to attend over all positions in the input sequence
- encoder self-attention
- decoder self-attention
    - to prevent leftward information flow: masking out (setting to -inf)

### Positional Encoding

$PE_{(pos,2i)} = sin(pos/10000^{2i}/d_{model})$

$PE_{(pos,2i+1)} = cos(pos/10000^{2i}/d_{model})$

- $pos$: position
- $i$: dimension

### Why Self-Attention

![](https://i.imgur.com/FVZjWmh.png)

### Training

dataset
- WMT 2014 English-German dataset
    - 4.5 million sentence pairs
    - vocabulary: about 37000 tokens
- WMT 2014 English-French dataset
    - 36M sentences
    - vocabulary: 32000 tokens

hardware and schedule
- 8 GPUs
- 100000 steps (each took about 0.4 sec) 12 hours
- big models
    - step time 1.0 sec
    - 300000 steps (3.5 days)

optimizer
- Adam optimizer

### Results

![](https://i.imgur.com/8qiS23M.png)

![](https://i.imgur.com/H307jHe.png)

(A) vary the number of attention heads and the attention key and value dimensions
(B) reducing the attention key size $d_k$ hurts model quality
(C\) and (D) show that bigger models are better and dropout is helpful in avoiding over-fitting
(E) replacing sinusoidal positional encoding with learned positional embeddings (early identical)

## Reference
[The Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
