---
layout:     post
title:      "论文笔记1:Visualizing and Understanding Convolution Networks"
subtitle:   " \"第一篇提出可解性研究的论文，开山之作\""
date:       2019-02-10 13:00:00
author:     "jack"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - CNN
    - 论文笔记
    - 机器学习
---

## 【论文笔记 1】CNN经典入门Visualizing and Understanding Convolution Networks

### 1. 综述

背景：神经网络现在表现的越来越好，但是没办法知道为啥好。导致没法进行有效改进。

- 现状由于各种能力的提升（GPU）, NN越来越好,  正则化策略越来越好（dropout）
- 但是训练模型还通过**试错法**，所以要开发理论研究

两种方法进行研究

1. 分解每一中间层层，每一个分类器的作用利用 **deconvnet**(反卷积)
2. ablation study（拆分研究）, occluding portions(遮挡部分)

### 2.网络描述

   基本使用alexnet没变化，只是对softmax分类器进行了修改，什么relu啊，pooling基本没变，还改了步长

### 3.反卷积的说明

**deconvnet**在前面的论文中，其实是一种半监督方法，通过每一次的输入map和输出map的反卷积的比对来进行学习和调整参数。这里不用于学习，只用于可视化。

具体方法利用unpool，fillter.T, Rectification 来实现

1. 先进行输入，比如从output那参数进来，然后进行unpool，放大到邻居

   pool的时候利用switch记住最大值的位置location，然后到下一步，反卷积时其他位置直接为0

2. 然后利用relu进行修正

3. 然后filter的转置进行恢复，

### 4.卷积网可视化

#### 4.1 特征可视化

- 一组特定的输入特征（通过重构获得），将刺激神经网络产生一个固定的输出特征。
- 每一层都有其层次化特点

#### 4.2 特征不变性

通过平移，旋转和缩放来判断鲁棒性

- 越往高层走，平移和尺度的变化对最终结果影响越小
- 但是旋转对结果影响很大。

### 5.遮挡的敏感度

这里遮挡块是可以滑动的，对图片中的某一部分作遮挡处理，然后向卷积神经网络输入遮挡过的图片，查看网络将其分为正确类的可能性

### 







