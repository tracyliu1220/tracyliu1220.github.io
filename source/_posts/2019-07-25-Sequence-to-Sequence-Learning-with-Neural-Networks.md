---
title: Sequence to Sequence Learning with Neural Networks
categories: Paper Notes
tags:
  - deep learning
date: 2019-07-25 13:30:21
toc: true
---

> NIPS 2014
> @google.com

## Contents

[pdf](https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks.pdf)

### Abstract

- Task: an English to French translation task from the WMT-14 dataset
- Method:
    - a Deep LSTM: maps theinput sequence to a vector of a fixed dimensionality
    - another Deep LSTM: decodes the target sequence from the vector
- Result: BLEU score 34.8
- Additional Founds: reversing the order of the words in all source sentences (but not target sentences) improved the LSTM’s performance markedly

<!-- more -->

### 1 Introduction
- DNNs' inpus and targets: encoded with vectors of fixed dimensionality. 
- The idea: use one LSTM to read the input sequence, one timestep at a time, to obtain large fixed-dimensional vector representation, and then to use another LSTM to extract the output sequence from that vector

![](https://i.imgur.com/dTpDanW.png)

### 2 The model

#### RNN
- A natural generalization of feedforward neural networks to sequences
- Difficult to train the RNNs due to the resulting long term dependencies

![](https://i.imgur.com/F3A7LJP.png)

#### LSTM
![](https://i.imgur.com/1yv5fXP.png)
- 2 LSTM: one for the input sequence and another for the output sequence
- Deep LSTMs with 4 layers
- Reverse the order of the words of the input sentence

### 3 Experiments

#### 3.1 Dataset details
- 348M French words
- 304M English words
- Vector representation
    - 160000 most frequent words for the source language (English)
    - 80000 most frequent words for the target language (French)
    - Out-of-vocabulary: UNK

#### 3.2 Decoding and Rescoring
- Maximizing the log probability of a correct translation T given the source sentence S
![](https://i.imgur.com/o39XyCc.png)
- Beam search decoder

#### 3.3 Reversing the Source Sentences
- original source: has a large "minimal time lag"
- reversed source: the first few words in the source language are very close to the first few words in the target language

#### 3.4 Training details
- LSTMs
    - 4 layers
    - 1000 cells
    - Parameters: uniform distribution between -0.08 and 0.08
- input vocabulary: 160,000
- output vocabulary: 80,000
- SGD, without momentum, learning rate 0.7
- 7.5 epochs, after 5 epochs, halving the learning rate every half epoch
- Batches of 128
- Exploding gradients prevending
- Roughly same-length-sentences in the minibatch

#### 3.5 Parallelization
- 8-GPU machine
    - 4 for 4 layers of LSTMs
    - 4 for computing the softmax
- Training took about a ten days

#### 3.6 Experimental Results
![](https://i.imgur.com/SK9kL6J.png)
![](https://i.imgur.com/jY2FBaL.png)

#### 3.7 Performance on long sentences

![](https://i.imgur.com/5o2l2r0.png)

![](https://i.imgur.com/8G0POfe.png)

#### 3.8 Model Analysis

![](https://i.imgur.com/41qrhIh.png)


- Turn a sequence of words into a vector
    - Clustered by meaning
    - Sensitive to the order of words
    - Insensitive to the active and passive

### 4 Related work
- RNN-Language Model (RNNLM)
- Feedforward Neural Network Language Model (NNLM)
- Auli et al: combine an NNLM with a topic model of the input sentence
- Kalchbrenner and Blunsom: map sentences to vectors using convolutional neural networks -> lose the ordering

### 5 Concludsion
- A large deep LSTM with a limited vocabulary does well
- Improved by reversing the words -> has the greatest number of short term dependencies
- LSTM can correctly translate very long sentences

## Reference

- [LSTM](https://zhuanlan.zhihu.com/p/32085405)
- [Sequence to squence](http://zake7749.github.io/2017/09/28/Sequence-to-Sequence-tutorial/)
- [beam search](https://blog.csdn.net/batuwuhanpei/article/details/64162331)
- [SMT](https://zh.wikipedia.org/wiki/%E7%BB%9F%E8%AE%A1%E6%9C%BA%E5%99%A8%E7%BF%BB%E8%AF%91)
- [PCA](https://zh.wikipedia.org/wiki/%E4%B8%BB%E6%88%90%E5%88%86%E5%88%86%E6%9E%90)
