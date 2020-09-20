---
title: Learning Deep Features for Discriminative Localization
categories: Paper Notes
tags:
  - deep learning
date: 2019-09-12 19:00:00
toc: true
---

> CVPR 2016
> @csail.mit.edu

## Contents

[pdf](http://cnnlocalization.csail.mit.edu/Zhou_Learning_Deep_Features_CVPR_2016_paper.pdf)

### Introduction

#### Problem

In some CNN models, the objects localization ability is lost when fully-connected layers are used for classification

#### Goal

To both classify the image and localize class-specific image regions in a single forward-pass

<!-- more -->

### Related Works

- Weakly-supervised object localization
    - Bergamo et al, Cinbis et al, Pinheiro et al, Oquab et al
    - They are not trained end-to-end
- Visualizing CNNs
    - visualize the internal representation learned by CNNs in an attempt to better understand their properties
    - Zeiler et al, Mahendran et al, Dosovitskiy et al

### Class Activation Mapping (CAM)

using global average pooling (GAP)

- $f_k(x, y)$: the activation of unit $k$ in the last convolutional layer at spatial location $(x, y)$
- $F^k = \sum_{x, y}f_k(x, y)$: the result of perfoming global average pooling for unit $k$
- $w_k^c$: the importance of $F_k$ for class $c$
- $S_c = \sum_kw_k^cF_k$: the input of the softmax
- $P_c = \frac{exp(S_c)}{\sum_c exp(S_c)}$: the output of the softmax for class $c$
- $M_c$: class activation map for class c

$F^k = \sum_{x, y}f_k(x, y)$

$S_c = \sum_k w_k^c \sum_{x,y} f_k(x, y) = \sum_{x,y}\sum_k w_k^cf_k(x,y)$

$M_c(x,y) = \sum_kw_k^cf_k(x,y)$

$S_c = \sum_{x,y} M_c(x,y)$

![](https://i.imgur.com/a8NzB3s.png)

### Global Average Pooling vs. Global Max Pooling

- GAP
    - loss encourages the network to identify the extent of the object
- GMP
    - loss encourages to identify just one discrimminative part

### Weakly-supervised Object Localization

#### Setup

- Originals
    - AlexNet
    - VGGnet
    - GoogLeNet
- Remove the fully-connected layers before the final output and replace them with GAP followed by a fully-conneted softmax layer
    - AlexNet-GAP
    - VGGnet-GAP
    - GoogLeNet-GAP
- AlexNet\*-GAP
    - AlexNet is the most affected by the removal of the fully-connected layers
    - add two convolutional layers just before GAP

#### Results

##### Classification

![](https://i.imgur.com/FF7Mq0Z.png)

In most cases there is a small performance drop of 1 − 2% when removing the additional layers from the various networks

##### Localization

![](https://i.imgur.com/vrf0K6N.png)

![](https://i.imgur.com/MCWz7zE.png)

![](https://i.imgur.com/4OAQC6E.png)

Generate bounding boxes: First segment the regions of which the value is above 20% of the max value of the CAM. Then take the bounding box that covers the largest connected component in the seg-mentation map

### Deep Features for Generic Localization

#### Fine-grained Recognition

![](https://i.imgur.com/vL1HVc7.png)

#### Pattern Discovery

##### Discovering informative objects in the scenes

![](https://i.imgur.com/3x0Ylm0.png)

##### Concept localization in weakly labeled images

![](https://i.imgur.com/uli2PRh.png)

##### Weakly supervised text detector

![](https://i.imgur.com/r4SPtRf.png)

##### Interpreting visual question answering

![](https://i.imgur.com/b4u54aR.png)

## Visualizing Class-Specific Units

![](https://i.imgur.com/IeQcQLy.png)


## Reference

- [浅谈Class Activation Mapping（CAM）](https://zhuanlan.zhihu.com/p/51631163)
- [CAM(Class Activation Mapping)](https://blog.csdn.net/qq_21097885/article/details/90039462)
- [ResNet, AlexNet, VGG, Inception: 理解各种各样的CNN架构](https://zhuanlan.zhihu.com/p/32116277)
- [《Self-produced Guidance for Weakly-supervised Object Localization》论文笔记](https://zhuanlan.zhihu.com/p/41508276)
- [Note from marmot](https://hackmd.io/@marmot0814/rJ3YkmSLr)
- [Note from darklenx](https://hackmd.io/qz5xCiQYTJyiCogr2oCKLA)
