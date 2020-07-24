---
layout:     post
title:      "论文笔记9:KSE for Interpretable CNN Compression"
subtitle:   " \"网络压缩与可解释性\""
date:       2019-08-27 13:00:00
author:     "neo"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - CNN
    - 论文笔记
    - 机器学习

---

## 论文笔记9: Exploiting Kernel Sparsity and Entropy for Interpretable CNN Compression

>  最近实验做的没有头绪，准确率没啥太大长进，看看论文吧

### 1.Introduction

这篇文章提出了网络压缩的一个新方法，相比于之前的剪支 **pruning**， 参数量化 **parameter quantization**

虽然有注意力的剪支已经有了比较好的效果，但是在一些结构比较复杂的网络上。

比如**Resnet**,可能会在**block_add**中出现**mismatch**的现象。所以就没法很好的进行实现与压缩

本文从网络可解释性入手去分析CNN的可压缩性，

通过分析以为可解释性的文章，我们得知feature map在不同的层上扮演不同的角色，

比如在底层通过kerner卷积核先找到一些低维度的特征，然后在topconv上面得到一些高维的特征。

![](https://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/20190829143432.png)

因此就可以从kernel入手，去分析每个层上面每个kernel卷积核的重要性，然后根据衡量出来的重要性去取舍

### 2. Methods

这篇文章提出了使用**KSE** (Kernel Sparsity and Entropy) 的方式进行计算，信息重要程度的衡量和计算

![](https://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/20190829143903.png)

**Kernel Sparsity：**这部分直接使用**L1-norms**将不重要的参数置为零，然后求和sum

**Kernel Entropy：**这使用 k nearest neighbours 来计算每个点的重要程度（不是很懂）

**KSE indicator：**定义对应方程，控制值在[0-1]之前

![](https://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/20190829145547.png)

这部分使用binary mask，来计算kernel的信息丰富度，主要是使用双线性插值方法将特征映射缩放到输入图像的分辨率

 **Kernel Clustering：** 相较于前面的直接取0或1问题，这里我们采用更加能够进行改善的方式进行实现

### 3. Experiment

![](https://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/20190829145940.png)

### 4.Visualization Analysis

虽然是在做压缩，但是这种方法也是利用到了解释性中的重要程度，利用heatmap的好坏去衡量这个feature map的重要性，是非常值得学习与借鉴的。

![](https://neoyanghc-picture.oss-cn-beijing.aliyuncs.com/20190829150254.png)