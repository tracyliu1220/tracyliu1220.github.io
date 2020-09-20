---
title: Convolutional Neural Networks
categories: Notes
tags:
  - deep learning
date: 2019-09-08 18:37:17
comments: false
---

## Convolution Operation

Take 2-D image with a 2-D kernel for example

$S(i, j) = (I * K)(i, j) = \sum_m \sum_n I(m, n)K(i-m, j-n)$

![](http://brohrer.github.io/images/cnn6.png)

## Local Connectivity

![](https://i.imgur.com/Q6ypLUF.png)

## Growing Receptive Fields

![](https://i.imgur.com/kjfavKV.png)

## Weight Sharing

![](https://i.imgur.com/sBqdd7v.png)

## Components of CNN

- Input layer
- Convolutional Layer
    - Convolution stage: Affine trasform
    - Detector stage: Nonlinearity
    - Pooling stage
- Next Layer

## Pooling

Replaces the output of the net with a
summary statistic of the nearby outputs

![](http://brohrer.github.io/images/cnn8.png)

Examples of pooling functions
- Maximum within a rectangular neighborhood
- Average of a rectangular neighborhood
- L2 norm of a rectangular neighborhood
- Weighted average based on the distance from the central pixel
- LSE (Log-Sum-Exp) Pooling $y=log\sum_{i=1}^n e^{x_i}$

## Multiple Channels

### Multiple Input Channels
![](https://i.imgur.com/PtBv9s5.png)

### Multiple Output Maps
![](https://i.imgur.com/lXG2VwJ.png)

### Multi-channel Convolution

- Input $V$: $V_{l,j,k}$
    - channel $l$
    - row $j$
    - column $k$
- Output $Z$: $Z_{i,j,k}$
    - channel $i$
    - row $j$
    - column $k$
- Kernel $K$: $K_{i,l,m,n}$
    - output channel $i$
    - input channel $l$

$Z_{i,j,k} = \sum_{l,m,n} V_{l,j+m-1,k+n-1}K_{i,l,m,n}$

## Stride

![](https://i.imgur.com/rKoCFAt.png)

## Zero-padding

- Valid convolution: no zero-padding (shrinking size)
- Same convolution: just enough zero-padding (padding with $k-1$ zeros)
- Full convolution: padding with enough zeros (every pixel is visited k times)

## Reference

- [卷積神經網路的運作原理](https://brohrer.mcknote.com/zh-Hant/how_machine_learning_works/how_convolutional_neural_networks_work.html)
