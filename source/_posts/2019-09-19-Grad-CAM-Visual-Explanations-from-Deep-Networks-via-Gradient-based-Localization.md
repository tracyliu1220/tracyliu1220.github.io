---
title: "Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization"
categories: Paper Notes
tags:
  - deep learning
date: 2019-09-19 19:00:00
toc: true
---

> ICCV 2017
> @vt.edu @gatech.edu

## Contents

- [slide](https://docs.google.com/presentation/d/1X3DKRFiszOit3wCd2hyxW4_dVIyKg5-fJAMiO2PUFn4/edit?fbclid=IwAR1b7G4J8JE6pEpI5RCKJS5NcecSH3BOEmom-nemnA4OfcwnoDJmK-GqCx8&pli=1#slide=id.p)
- [pdf](https://arxiv.org/pdf/1610.02391.pdf)

### Introduction

- Input: Images
- Output: Visualization of discrimination regions
- Interpretability
- Without architectural changes or re-training
- To give explaination to the prediction of some models
- General version of CAM

<!-- more -->

### Grad-CAM

#### Neuron importance weights $\alpha_k^c$

- class: $c$
- the gradient of the score for $c$: $y^c$
- feature maps: $A^k$
- the number of pixels: $Z$

$$\alpha_k^c = \frac{1}{Z} \sum_i\sum_j \frac{\partial y^c}{\partial A_{ij}^k}$$

$$L_{Grad-CAM}^c = ReLU(\sum_k \alpha_k^cA^k)$$


### Compare with CAM

- the prediction: $S^c$

$$S^c = \sum_k w_k^c \frac{1}{Z}\sum_i\sum_j A_{ij}^k$$

$$S^c = \frac{1}{Z}\sum_i\sum_j\sum_k w_k^cA_{ij}^k$$

$$L_{CAM}^c = \sum_k w_k^cA_{ij}^k$$

#### GAP

- $A^k$: a feature map of a convolutional layer
- $A^k_{ij}$: a pixel value of $A^k$
- $F^k$: GAP of the feature map in the last layer
- $Z = i \times j$: the number of pixels

$$F^k=\frac{1}{Z}\sum_i\sum_jA^k_{ij}$$

#### Final Result

- $Y^c$: the prediction 
- $W^c_k$: weights


$$Y^c=\sum_k w^c_k \cdot F^k$$

**From GAP**

$$\frac{\partial Y^c}{\partial F^k}=\frac{\partial Y^c}{\partial A^k_{ij}}\frac{\partial A^k_{ij}}{\partial F^k}=\frac{\frac{\partial Y^c}{\partial A^k_{ij}}}{\frac{\partial F^k}{\partial A^k_{ij}}}
$$

$$
\Rightarrow \frac{\partial Y^c}{\partial F^k}=\frac{\partial Y^c}{\partial A^k_{ij}}\cdot Z
$$

**From Final Result**

$$
\frac{\partial Y^c}{\partial F^k}=\frac{\partial \sum_k w^c_k \cdot F^k}{\partial F^k}=w^c_k
$$


$$
\Rightarrow w^c_k=\frac{\partial Y^c}{\partial A^k_{ij}}\cdot Z
$$

$$
\sum_i\sum_jw^c_k=\sum_i\sum_j\frac{\partial Y^c}{\partial A^k_{ij}}\cdot Z
$$

$$
Z\,w^c_k=Z \sum_i\sum_j\frac{\partial Y^c}{\partial A^k_{ij}}
$$

$$
w^c_k=\sum_i\sum_j\frac{\partial Y^c}{\partial A^k_{ij}}
$$

## Reference

- [使用 Grad-CAM 解釋卷積神經網路的分類依據](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/grad-cam-introduction-d0e48eb64adb)
- [Grad-CAM 介紹 — Grad-CAM:Visual Explanations from Deep Networks via Gradient-based Localization](https://medium.com/@xiaosean5408/grad-cam-%E4%BB%8B%E7%B4%B9-grad-cam-visual-explanations-from-deep-networks-via-gradient-based-localization-5a91dc6003b8)
- [论文翻译】Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization ICCV 2017](https://blog.csdn.net/stu_sun/article/details/80628406?fbclid=IwAR3Krf-tlfLEPOBHzpAwI-cZavGnnKj0CzPzrCE3HZj1Qfp5YfZi8wF-vg8)
- [黑盒机器学习的可解释性](https://zhuanlan.zhihu.com/p/63959256?fbclid=IwAR1L6tdr-GEs103adOT1Te6eJmGzdrbRQwxdFzM50wJ9YU2CGk9ht5kjqLw)
