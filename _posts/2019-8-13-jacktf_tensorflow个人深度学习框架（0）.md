---
layout:     post
title:      "neotf个人深度学习框架_框架设计"
subtitle:   " \"设计编写自己能够完全使用的tf框架\""
date:       2019-8-13 17:00:00
author:     "neo"
header-img: "img/post-bg-tf.jpg"
catalog: true
tags:
    - CNN
    - 学习笔记
    - 深度学习
---

### 1. 框架立项前言

暑假这两周一直都在找代码，看懂代码，然后进行实验

但是总以为自己看懂了的代码，会有很多自己并没有看懂的地方，导致实验效果并不理想

因此，我苦思冥想要写一个自己的框架出来，用于以后进行代码实验

而不是在每一次有任务之后都是在 **github** 盲目寻找，最近也学了keras fastai 等等框架 跳来跳去也没啥意思

这套深度学习框架，也不能算的上一个框架 但是就算想把自己要用到的东西

完全自己写一遍，然后知道这个框架到底是怎么运行的，主要想法就算这些**Just do IT**

### 2. 设计目标

> 先总体的写一下目标，然后在进行模块化的编写

#### 1. Data Read and Process

主要利用 tensorflow data 的API 进行实现

#### 2. Model Create

可以使用 tf.slim.net 中 定义好的网络模型，也可以使用自己定义的网络结构

#### 3. Train 

模型训练，这部分还是比较简单的

#### 4.Save，Restore，Finetune

实现模型的存储，预训练参数的加载，freeze以及其他操作

#### 5. checkpoint and pb模型导出

将checkpoint 和 pb转化成接口进行导出，可以进行测试

#### 6. Tensorboard 可视化

对训练进行可视化，然后对网络进行对应的改进  

### 3. 整体设计结构

![](https://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/未命名文件 (3).png)

**开始实现吧！**

