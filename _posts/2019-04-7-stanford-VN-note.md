---
layout:     post
title:      "Standford CS231n 李飞飞可视化和理解DNN学习笔记"
subtitle:   " \"卷积神经网络用于目标检测等等\""
date:       2019-04-3 17:00:00
author:     "jack"
header-img: "img/post-bg-road.jpg"
catalog: true
tags:
    - CNN
    - 学习笔记
    - 机器学习
---

## Standford CS231 李飞飞可视化DNN学习笔记

### 1.what‘s going on inside ConvNets？

#### 1.1 what are the middle conv do?

我们通过可视化各层卷积层的权值，来告诉我们卷积核在寻找什么。why?

这种想法是来源于 template match 和 inner products 。只有和你卷积核相似的时候才能得到自己的内积（正正得正，负负也得正，相加不是最大嘛），我们的卷积核就是在跟图像做线性乘加。所有这种思想就比较好理解。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qld32ocjj30mf0bzgvp.jpg%29)

这里面的64个图片，分别代表64个卷积（3x11x11）的表示。

但是我们想第二层走的时候，由于这些filters不是直接连接到input image的所以就会有不太可视化的情形，有些filters可能都不是3维的，比如 16x11x11的这种。但是我们可以把16x11x11展开成16张11x11的灰度图片，

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qlgds99gj30s00dmh0b.jpg%29)

尝试使用近邻法，就是将两个相似的照片，在最高层的输出进行对比，现有的研究已经在fc层已经可以得知下列图片中第二行第一个图片（左边的大象）和第二个图片（右边的大象）在最后个全连层极其相似。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qltnlhlcj30md0dfgym.jpg%29)

#### 1.2  Ocoluding 实验 和 Saliency Map

是关注图片的那部分帮助我们进行了决策。哪部分是重要的，哪部分不重要

#### 1.3 guided backprop

也是关注像素级的一些，关注每个神经元。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qmjmctqij30rs0d6k0d.jpg%29)

#### 1.4 Gredient Ascent

加入正则化其实是为了，让我们能够更好的的理解。 更好的正则化，能生成更好的图片

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qmt2mlu8j30ru0d7tjg.jpg%29)

未完待续