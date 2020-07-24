---
layout:     post
title:      "论文笔记5:Methods for Interpreting DNN"
subtitle:   " \"CV 综述文章分析和介绍现有方法\""
date:       2019-04-4 15:00:00
author:     "neo"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - 可解释性
    - 论文笔记
    - 深度学习
---


## 论文笔记5 网络可解释性方法综述

论文Methods for Interpreting and Understanding Deep Neural Networks 的研读笔记

### 1. 简介

+ 提出**Interpreting**和**Explaining**的定义和区别
+ 分析和介绍现有方法
+ 总结现在都是事后解释和针对网络结构解释的现状

### 2. 概念提出

`Deﬁnition 1. An interpretation is the mapping of an abstract concept (e.g. a predicted class) into a domain that the human can make sense of.`

`Deﬁnition 2. An explanation is the collection of features of the interpretable domain, that have contributed for a given example to produce a decision (e.g. classiﬁcation or regression.`

个人理解：针对interpretation ：模型解释

1. find **prototypical example** of a category 其实就是将黑盒看成一个f(x),看它输出概率最大的那种图片是什么来找到一个典型样本
2. find **pattern** maximizing activity of a neuron 找到能够最大化激活网络的pixel空间组合方式

这两者都是针对pixel像素来解释的，倾向于找到“典型特征来代表类”

针对explanation：决策解释

1. why” does the model arrive at this particular prediction 解释为什么我们的模型能够成功预测

2. verify that model behaves as expected 当模型将一个输入x预测为“1”时，起主要作用的特征（pixels）应该是组成数字“1”的附近的pixels

### 3. Interpretation 方法

主要是利用**activation maximum** ,通过反向推导找到对应类最典型的图片（生成的）。

![](http://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1p6tvsjzgj31760jte42.jpg%29)

缺点: 没有办法针对**细节**(颜色，质地等)

### 4. Explanation 方法

#### 4.1 Sensitivity Analysis

主要原理是通过输入x对输出y的影响程度来对x的每一个特征(分量)进行打分, 通过对打分进行可视化,最终将形成一张heatmapping.而打分的方法, 使用非常符合直觉的梯度.

![](http://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1pm5spmcvj31a70u4dsd.jpg%29)

有三种典型的SA方法，如下所示，但是通过不同的方法引入概率，来对SA进行改良可以使结果变得更好。也可以用图像编码的方式来改善。

![](http://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1pmg67ps4j30wp11ldom.jpg%29)

缺点：噪声比较大，不适合用于深层的梯度爆炸网络

#### 4.2 Simple Taylor Decomposition 

通过将函数视为一个相关性问题来解决。

#### 4.3  Relevance Propagation 

包括 

+ deconvolution 论文研读一
+ guided backprop  论文研究二

### 5. LRP

后面会单出一篇来解释这种方法

### 6. 衡量解释能力

#### 6.1 Explanation Continuity  解释的连续性

`If two data points are nearly equivalent, then the explanations of their predictions should also be nearly equivalent.`

可以通过看预测方程的导数来决定。

#### 6.2 Explanation Selectivity 

一种直观的想法是，我们通过对heatmap中的位置按重要程度降序排列并取前L个元素，得到一个序列O，依次对这些位置的数据点通过函数g(x,rk) g(x, r_k)g(x,r k )进行扰动（比如屏蔽或替换等）。来观察它的变化

举一个图片flipping的例子：

![](http://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1pnqsid26j30xn0rj0zj.jpg%29)













