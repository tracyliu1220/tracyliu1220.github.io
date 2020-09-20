---
title: Conditional Image Generation with PixelCNN Decoders
categories: Paper Notes
tags:
  - deep learning
date: 2019-08-30 14:00:13
toc: true
---

> NIPS 2016
> @google.com

## Contents

[pdf](https://papers.nips.cc/paper/6527-conditional-image-generation-with-pixelcnn-decoders.pdf)

### Introduction

- Generate pictures pixel by pixel
- Related Works
    - PixelRNN: better performance
    - PixelCNN: faster to train (easier to parallelize)
- Gated PixelCNN
- Conditional variant of the Gated PixelCNN

<!-- more -->

### PixelRNN and PixelCNN

#### The Distribution of PixelRNNs

$p(x) = \Pi_{i=1}^{n^2}p(x_i|x\_1, ...,x_{i-1})$

- $x$: input picture
- $x_i$: a single pixel

![](https://i.imgur.com/su71yZj.png)

#### Masking

to make sure the CNN can only use information about pixels above and to the left of the current pixel

![](https://i.imgur.com/SAattZD.png)

#### 3 Color Channels

![](http://vsooda.github.io/assets/pixelrnn/pixelrnn_masks_highlevel.png)

![](http://vsooda.github.io/assets/pixelrnn/pixelrnn_masks_A.png)

![](http://vsooda.github.io/assets/pixelrnn/pixelrnn_masks_B.png)

- B conditioned on (R, G)
- G conditioned on R
- first layer: mask A, otherwise: mask B


### Gated PixelCNN

#### Gated Convolutioal Layers

Gated Activation Unit

$y = tanh(W\_{k,f} * x) \odot \sigma(W_{k,g} * x)$

- $k$: the number of the layer
- $\odot$: element-wise product
- $*$: convolution operator

#### Blind spot

![](https://i.imgur.com/CwT4tBa.png)

#### A single layer block of a Gated PixelCNN

![](https://i.imgur.com/PofGftb.png)

- Notations
    - green: convolution operations
    - red: element-wise multiplications and additions
    - blue: splites feature maps
- Left part: vertical stack
- Right part: horizontal stack

### Conditional PixelCNN

$p(x|h) = \Pi_{i=1}^{n^2} p(x_i|x_1, ..., x\_{i-1},h)$

- $h$: a latent vector, image description

$y = tanh(W_{k,f} * x + V_{k,f}^T h) \odot \sigma(W_{k,g} * x + V_{k,g}^T h)$

- Applications of $h$
    - class dependent bias
        - what should be in the image
    - location dependent bias
        - where

#### Location Dependent

mapping $h$ to a spatial representation $s = m(h)$
where $s$ has the same width and height as the image

$y = tanh(W\_{k,f} * x + V\_{k,f} * s) \odot \sigma(W_{k,g} * x + V_{k,g} * s)$

### PixelCNN Auto-Encoders

- Replacing the deconvolutional decoder with a conditional PixelCNN

### Experiments

#### Unconditional Modeling with Gated PixelCNN

##### Performance of different models on CIFAR-10

![](https://i.imgur.com/3tGWVbY.png)

##### Performance of different models on ImageNet

![](https://i.imgur.com/eqvYbns.png)

#### Conditioning on ImageNet Classes

![](https://i.imgur.com/n4dln7t.png)

#### Conditioning on Portrait Embeddings

![](https://i.imgur.com/sIaap6D.png)

![](https://i.imgur.com/0IfsSXl.png)

#### PixelCNN Auto-Encoder

![](https://i.imgur.com/xa4fIJQ.png)


## Reference

- [Gated CNN](https://zhuanlan.zhihu.com/p/51212511)
- [Pixel RNN](http://vsooda.github.io/2016/10/30/pixelrnn-pixelcnn/) ([paper](https://arxiv.org/pdf/1601.06759.pdf))
- [GTU vs. GLU](https://zhuanlan.zhihu.com/p/27084042)
- [Language Modeling with Gated Convolutional Networks](https://arxiv.org/pdf/1612.08083v2.pdf)
