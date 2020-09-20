---
title: Effective Approaches to Attention-based Neural Machine Translation
categories: Paper Notes
tags:
  - deep learning
date: 2019-08-16 13:40:22
toc: true
---

> EMNLP 2015
> @stanford.edu

## Contents

[pdf](https://aclweb.org/anthology/D15-1166)

### Introduction

- Attentional mechanism jointly translate and align words
- 2 types of attention-based models
    - global
        - all source words
    - local
        - subset of source words

<!-- more -->

### Neural Machine Translation

![](https://i.imgur.com/69SFqyy.png)

#### Probabilistic model

$p(y|x)$

#### input

$x_1, ..., x_n$

#### output
$y_1, ..., y_m$

#### 2 components

- encoder: to compute a representation $s$ of source sentences
- decoder: to generate one target word at a time

$log\ p(y|x) = \Sigma_{j=1}^m log\ p(y_j|y_{<j}, s)$

#### using LSTM

#### the probability of decoding each word $y_j$

$p(y_j|y{<j}, s) = softmax(g(h_j))$

$g$ is the transformation fuction that outputs a vocabulary-sized vector

#### training objective

$J_t = \Sigma_{(x,y)\in\mathbb{D}}-log\ p(y|x)$

### Attention-based Models

- context vector $c_t$
    - captures relevant source-side information to help predict the current target word $y_t$
    - weighted average over all the source hidden states
- align weights $a_t$

$\tilde{h}_t = tanh(W_c[c_t;h_t])$

$p(y_t|y_{<t},x) = softmax(W_s\tilde{h}_t)$

#### Global Attention

![](https://i.imgur.com/n8SilT4.png)

$a_t(s) = align(h_t, \bar{h}_s) = \frac{exp(score(h_t, \bar{h}_s))}{\Sigma_{s'}exp(score(h_t, \bar{h}_{s'}))}$

$$
score(h_t,\bar{h}_s) =
\begin{cases}
h_t^\top\bar{h}_s \\
h_t^\top W_a\bar{h}_s \\
v_a^\top tanh(W_a[h_t^\top;\bar{h}_s])
\end{cases}
$$

> dot, general, concat

#### Local Attention

![](https://i.imgur.com/psJB2I3.png)

- window $D$
    - $c_t$ is derived as a weighted average within the window $[p_t-D,p_t+D]$
    - empirically selected
- $a_t$
    - fixed dimension $2D+1$
    - Gaussian distribution centered around $p_t$
    - $a_t(s) = align(h_t, \tilde{h}_s)exp(- \frac{(s - p_t)^2}{2\sigma^2})$
 
##### Monotonic alignment (local-m)

$p_t = t$

##### Predictive alignment (local-p)

$p_t = S \cdot sigmoid(v_p^\top tanh(W_p h_t))$

- source sentence length $S$
- model parameters (to be learnt): $v_p^\top$, $W_p$

#### Input-feeding Approach

![](https://i.imgur.com/ZWHl65J.png)

- to keep tracking of which source words have been translated
- alignment decisions should be made jointly taking into account past alignment information

#### Experiments

##### Training Details

- WMT’14 training data
    - 4.5M sentences pairs
    - 116M English words, 110M German words
    - limit vocabularies to 50K with token <unk\>
- LSTM models
    - 4 layers
    - 1000 cells
- parameters: uniformly initialized in [-0.1, 0.1]
- 10 epochs using SGD

##### Results

![](https://i.imgur.com/sCL1Knu.png)

![](https://i.imgur.com/6BwRjj8.png)

![](https://i.imgur.com/UtZYje2.png)

![](https://i.imgur.com/oXAktzA.png)

![](https://i.imgur.com/7xhm4Uc.png)
