---
title: Auto-Encoding Variational Bayes
categories: Paper Notes
tags:
  - deep learning
date: 2019-07-31 13:30:21
toc: true
---

> ICLR 2014
> @google.com

## Contents

[pdf](https://arxiv.org/pdf/1312.6114.pdf)

### Introduction

- Variational Bayesian (VB)
- Approximate posterior using MLP
- Stochatic Gradient Variational Bayes (SGVB)
- Auto-Encoding VB (AEVB) algorithm
- Variational auto-encoder (VAE)

### Method

#### problem

- the integral of the marginal likelihood $p_\theta(x) = \int p_\theta(z)p_\theta(x|z) dz$ is intractable
- a large dataset: the need of updating using small minibatches

<!-- more -->

#### graphic

![](https://i.imgur.com/u3EKmgm.png)

- Sold lines: generative model $p_\theta(z)p_\theta(x|z)$
- Dashed lines: variational approximation $q_\phi(z|x)$ to the posterior $p_\theta(z|x)$

#### encoder

$q_\phi(z|x)$: an approximation to the intractable true posterior $p_\theta(z|x)$

#### decoder

$p_\theta(x|z)$

### The variational bound

#### the marginal likelihoods of $x^{(i)}$

$log\ p_\theta(x^{(i)}) = log\ p_\theta(x^{(i)},z) - log\ p_\theta(z|x^{(i)})$

$log\ p_\theta(x^{(i)}) = D_{KL}(q_{\phi}(z|x^{(i)})||p_{\theta}(z|x^{(i)})) + \mathcal{L}(\theta,\phi;x^{(i)})$

$log\ p_\theta(x^{(i)}) \geq \mathcal{L}(\theta,\phi;x^{(i)}) = \mathbb{E}{q_{\phi}(z|x^{(i)})}[-log\ q_\phi(z|x^{(i)}) + log\ p_{\theta}(x^{(i)},z)]$

$\mathcal{L}(\theta,\phi;x^{(i)}) = \mathbb{E}{q_{\phi}(z|x^{(i)})}[-log\ q_\phi(z|x^{(i)}) +$ $log\ p_{\theta}(x^{(i)}|z) + log\ p_{\theta}(z)$ $]$

$\mathcal{L}(\theta,\phi;x^{(i)}) = -D_{KL}(q_\phi(z|x^{(i)}) || p_\theta(z)) + \mathbb{E}{q_{\phi}(z|x^{(i)})}[log\ p_\theta(x^{(i)}|z)]$

### The SGVB estimator and AEVB algorithm

do gradient ascent

#### differentiable transformation for $z$

reparameterization

$\tilde{z} = g_\phi(\epsilon, x)$  with $\epsilon \sim p(\epsilon)$

#### Monte Carlo estimates

$\mathbb{E}{q_\theta(z|x^{(i)})}[f(z)] = \mathbb{E}\_{p(\epsilon)}[f(g_\phi(\epsilon, x^{(i)}))] \simeq \frac{1}{L} \Sigma_{l=1}^L f(g_\phi(\epsilon^{(l)}, x^{(i)}))$

where $\epsilon^{(l)} \sim p(\epsilon)$, sampling $L$ datapoints

#### generic SGVB

from

$\mathcal{L}(\theta,\phi;x^{(i)}) = \mathbb{E}{q_{\phi}(z|x^{(i)})}[-log\ q_\phi(z|x^{(i)}) + log\ p_{\theta}(x^{(i)},z)]$

to

$\tilde{\mathcal{L}}^A(\theta,\phi;x^{(i)}) = \frac{1}{L}\Sigma_{l=1}^Llog\ p_\theta(x^{(i)}, z^{(i,l)}) - log\ q_\phi(z^{(i,l)}|x^{(i)})$

where $z^{(i,l)} = g_\phi(\epsilon^{(i,l)}, x^{(i)})$ and $\epsilon^{(l)} \sim p(\epsilon)$

#### second version of the SGVB

- $D_{KL}(q_\phi(z|x{(i)})||p_\theta(z))$ can be integrated analytically
- has less variance than the generic one

from

$\mathcal{L}(\theta,\phi;x^{(i)}) = -D_{KL}(q_\phi(z|x^{(i)}) || p_\theta(z)) + \mathbb{E}{q_{\phi}(z|x^{(i)})}[log\ p_\theta(x^{(i)}|z)]$

to

$\tilde{\mathcal{L}}^B(\theta,\phi;x^{(i)}) = -D_{KL}(q_\phi(z|x^
{(i)})||p_\theta(z)) + \frac{1}{L}\Sigma_{l=1}^L(log\ p_\theta(x^{(i)}|z^{(i,l)}))$

where $z^{(i,l)} = g_\phi(\epsilon^{(i,l)}, x^{(i)})$ and $\epsilon^{(l)} \sim p(\epsilon)$

#### multiple datapoints

$N$ datapoints with $M$ features ($x^{(1)} \sim x^{(M)}$ )

$\mathcal{L}(\theta,\phi;X) \simeq \tilde{\mathcal{L}}^M(\theta,\phi;X^M) = \frac{N}{M}\Sigma_{i=1}^M \tilde{\mathcal{L}}(\theta,\phi;x^{(i)})$

#### AEVB

![](https://i.imgur.com/YAEkjJE.png)

### VAE

$log\ q_\phi(z|x^{(i)}) = log\ N(z;\mu^{(i)},\sigma^{2(i)}I)$

$\mu^{(i)}$ and $\sigma^{(i)}$ are output from MLP
MLP: nonlinear function of $x^{(i)}$ and $\phi$

#### sampling $z^{(i, l)}$

$z^{(i, l)} \sim q_\theta(z|x^{(i)})$

$z^{(i, l)} = g_\phi(x^{(i)}, \epsilon^{(l)}) = \mu^{(i)} + \sigma^{(i)} \times \epsilon^{(l)}$

where $\epsilon^{(l)} \sim N(0, I)$

$\mathcal{L}(\theta,\phi;x^{(i)}) \simeq \frac{1}{2} \Sigma_{j=1}^J(1 + log((\sigma_j^{(i)})^2) - (\mu_j^{(i)})^2 - (\sigma_j^{(i)})^2) + \frac{1}{L}\Sigma\_{l=1}^Llog\ p\_\theta(x^{(i)}|z^{(i,l)})$

![](https://i.imgur.com/F2kKNwf.png)

### Experiments

#### likelihood lower bound
![](https://i.imgur.com/zaNWqtH.png)

#### marginal likelihood
![](https://i.imgur.com/d3yizfV.png)


## Reference

- [VAE](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/585654/)
