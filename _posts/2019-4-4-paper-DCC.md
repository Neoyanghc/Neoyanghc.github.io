---
layout:     post
title:      "论文笔记6:Deep Domain Confusion"
subtitle:   " \"迁移学习网络和方法学习\""
date:       2019-04-9 15:00:00
author:     "jack"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - 迁移学习
    - 论文笔记
    - 深度学习
---

## 论文笔记6 Deep Domain Confusion

论文Deep Domain Confusion: Maximizing for Domain Invariance 的研读笔记

### 1. 简介

1. 深度监督神经网络在大量数据的训练下，产生很多的数据偏差bias，很难remove到新的任务上去。

2. 提出一种新的网络结构，能够应对数据bias问题。
3. 就适应层的选择问题进行了实验和对比

### 2. 主要思想

> Optimizing for domain invariance, therefore, can be considered equivalent to the task of learning to predict the class labels while simultaneously finding a representation that makes the domains appear as similar as possible.

数据样本不够怎么使用深度学习？大家第一时间想到的肯定是微调已经训练好的模型，像VGG、Inception、Resnet这样的模型，但是有时我们可能会发现，有时微调后的效果并不是很好，可能会需要微调好多层才能得到较好的效果，但是这往往需要大量的样本，但当我们仅有少量或没有带标注的数据时，我们就无法有效的通过微调网络来实现对新样本的识别。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1wmsaluwsj30f5089mxx.jpg%29)

存在这一问题的主要原因是源数据与目标数据之间的分布情况不同，所以，在面对新样本时，我们需要使源数据与目标数据之间具有相似的分布情况，这也就是所说的域适应问题，如图所示。这篇论文认为，迁移学习中优化域不变性的任务，可以被认为等同于学习预测类标签的任务，同时找到使域尽可能相似的表示形式。

论文主要通过loss的组合来实现。通过在源域与目标域之间添加了一层适应层及添加域混淆损失函数来让网络在学习如何分类的同时来减小源域及目标域之间的分布差异，从而实现域的自适应，成功的解决了这类问题。

### 3. 网络结构

将网络不仅在源域上面训练，而且将目标域和源域的差距考虑进行，对网络进行再训练

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1wn4h65lnj309r0b674p.jpg%29)

loss函数，不仅需要对分类完成的好，而且要能缩小两者差距。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1wn5a65s3j30c5013t8k.jpg%29)

网络结构设计时做了两个选择题。

- adaptation layer 在哪里设置
- 输出的维度，MMD的维度

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1wncyfduxj30r80a3wfy.jpg%29)

### 4. 如何训练网络

综上所述，两个选择对比出来之后，应该来对网络进行训练

利用上面给出的loss function ，通过 lamba 来控制进行 fine-tuning，

并且是对两边的网络结构都进行fine-tuning.

还包括对MMD 加上一些正则化手段，来辅助训练。

### 5. 利用 t-she 查看数据的相关性

figure 中展示了fc7 对数据相关性的比较。可以看到此网络显著减少了不同数据之间的相关性

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g24uym6wjgj30zq0zmnmr.jpg%29)

