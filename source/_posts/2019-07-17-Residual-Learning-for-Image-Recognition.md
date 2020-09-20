---
title: Residual Learning for Image Recognition
categories: Paper Notes
tags:
  - deep learning
date: 2019-07-17 23:48:35
toc: true
---

> CVPR 2016
> @microsoft.com

## Contents

[pdf](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf)

### Abstract
- residual networks
- VGG nets
- [COCO object detection dataset](https://blog.csdn.net/u012905422/article/details/52372755)

<!-- more -->

### 1. Introduction

#### Is learning better networks as easy as stacking more layers?
- Problem: [vanishing/exploding gradients](https://blog.csdn.net/cppjava_/article/details/68941436)
    - Hamper convergence from the beginning
- Solutions: normalized initialization and itermediate normalization layers
    - Tens of layers converge with SGD and back propagation

#### [Degradation problem](https://zhuanlan.zhihu.com/p/31852747)
- With the network depth increasing, accuracy gets saturated and then degrades rapidly
- Not caused by overfitting
- Adding more layers to a suitably deep model leads to highter training error
![](https://i.imgur.com/itqwaOA.png)

#### Deep residual learning framework
- desired underlying mapping $H(x)$
- another mapping $F(x):=H(x)-x$
- original mapping recasting $F(x)+x$
- If an identity mapping were optimal, it would be easier to push the residual to zero than to fit an identity mapping by a stack of nonlinear layers
- identity mapping: $F(x)=x$

#### Shortcut connections
![](https://i.imgur.com/R3fe3pU.png)
- Shortcut connections are those skipping one or more layers, simply perform identity mapping

#### ImageNet
- Residual nets: easy to optimize, plain nets: higher training error
- Accuracy
- 152-layer residual net (deepest)
- Lower complaxity than VGG nets
- Error 3.57%

### 2. Related Work

#### Residual Representations
- Powerful shallow representations for image retrieval and classification
    - VLAD: encodes by the residual vectors with respect to a dictionary
    - Fisher Vector: probabilistic version of VLAD
- Encoding residual vectors is shown to be more effective than encoding original vectors
- For solving Partial Differential Equations (PDEs)
    - Multigrid method
    - Hierarchical basis preconditioning

#### Shortcut Connections
- Learning residual functions -> identity shortcuts are never closed

### 3. Deep Residual Learning

#### Residual Learning
- Hypothesizes
    - multiple nonlinear layers can asymptotically approximate complicated functions
    - they can asymptotically approximate the residual functions
- If the added layers can be constructed as identity mappings, a deeper model should have training error no greater than its shallower counterpart
- Residual learning reformulation
    - If identity mappings are optimal, the solvers may simply drive the weights of the multiple nonlinear layers toward zero to approach identity mappings

#### Identity Mapping by Shortcuts

$y = F(x, {W_i}) + x$

$y = F(x, {W_i}) + W_s x$

$W_s$ for matching dimensions

#### Network Architectures
- Plain Network
    - Based on VGG-net
    - Design rules
        - same output feature map size
        - same number of filters
        - If the feature map size is halved, the number of filters is doubled
- Residual Network
    - based on plain network
    - add some shortcuts and turn it into residual counterpart version
    - identity mapping doesn’t increase the parameter in the network
    - use project matrix to make the dimension match between $F(x)$ and $x$

![](https://i.imgur.com/sa3H1o3.png)

#### Implementation
- [Batch normalization](http://violin-tao.blogspot.com/2018/02/ml-batch-normalization.html)
- SGD
- Learning rate: 0.1
- Weight decay of 0.0001 and a momentum of 0.9
- Don’t use dropout, because the percentage of parameters in the fully connected layer is low. (about 0.01%)

### 4. Experiments
![](https://i.imgur.com/L2GJ9SF.png)

##### Plain Networks
- Degradation problem

##### Residual Networks
- Three major observations
    - 34-layer ResNet is better than the 18-layer ResNet
    - Training error successfully reduced
    - The 18-layer ResNet converges faster (providing faster convergence at the early stage)

##### Identity vs. Projection Shortcuts
- (A) Zero-padding
- (B) Projection for increasing dimensions and other are identity
- (C\) All are projections
- C > B > A (slightly)

##### Deeper Bottleneck Architectures
![](https://i.imgur.com/nLlyX4i.png)

- 50-layer ResNets
- 101-layer and 152-layer ResNets

## Reference
- [ITREAD](https://www.itread01.com/content/1549918274.html)
- [Deep Residual Net 分析和实现](http://shuokay.com/2016/05/26/deep-residual-net/)
- [論文筆記](http://jermmy.xyz/2017/09/25/2017-9-25-paper-notes-deep-residual-learning/)
