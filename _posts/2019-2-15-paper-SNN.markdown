---
layout:     post
title:      "论文笔记3: Network in network"
subtitle:   " \"CAM中提出的GAP研究，出自这篇论文\""
date:       2019-02-15 13:00:00
author:     "neo"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - CNN
    - 论文笔记
    - 机器学习
---

## 【论文笔记 3】CV 经典入门 Network in network

### 1. 综述

总体来说这篇文章还是比较简单的，没有很复杂的推导。两大亮点：

- 利用微型神经网络可以代替线性滤波，从而增强模型在[感受野（receptive field）](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Fbaidu_32173921%2Farticle%2Fdetails%2F70049186)内对局部区域（local patches）的辨别能力
- 利用**GAP**进行location位置的判定，能做道更好的防止过拟合

### 2.MLPConv

传统的卷积滤波是广义线性模型GLM,而我认为它的抽象程度较低。这里我们转而利用非线性函数逼近器来增强局部模型的抽象能力

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1hl8ol221j30sa0dsjvs.jpg%29)

### 3.GAP

卷积神经网络最后的全连接层可以说作为了一个分类器，或者作为了一个 feature  clustering.   它把卷积层学习到的特征进行最后的分类；   intuitively, 根本不了解它是怎么工作的， 它就像一个黑盒子一样，并且它也引入了很多的参数，会出现 overfitting 现象；   （我认为其实最后的 全接层就是一个分类器）

本文，remove掉了 全连接层， 使用 global average pooling 来代替； 举个例子更容易说明白： 假设分类的任务有100 classes，  所以设置网络的最后的 feature maps 的个数为 100， 把每一个feature map 看作成 对应每一类的 概率的相关值 ，然后对每一个 feature map 求平均值（即 global average pooling), 得到了 100维的向量， 把它直接给 softmax层，进行分类；（其实100个数中最大值对应的类别即为预测值， 之所以再送给 softmax层是为了求 loss，用于训练时求梯度

当分类的类别有4种时，则最后的 global average pooling 应该是这样的：

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1hlp4438mj30ld0f50td.jpg%29)



### 4. GAP的可视化

通过GAP来增强NIN最后一个mlpconv层的特征图，使其作为分类是可信的，这可能会加强局部感受野的建模。为了知道这个目标实现了多少，我们提取和可视化了在CIFAR-10上训练的模型的来自最后一个mlpconv层的特征图。

下图展示了CIFAR-10上测试集上选择的10类的一些示例图和相关特征图。如预期，特征图的最大激活区域和输入的相关真实分类吻合，这明显是GAP加强过的。在真实分类的特征图内，可以看到最大的激活区域出现在与原物体相同的区域，在结构化物体中尤其如此

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1hlpwmn19j30qe0hgafv.jpg%29)

