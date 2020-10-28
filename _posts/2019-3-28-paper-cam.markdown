---
layout:     post
title:      "论文笔记4:CAM"
subtitle:   " \"CV 经典入门 Class Acivation Map\""
date:       2019-03-28 13:00:00
author:     "neo"
header-img: "img/post-bg-sea.jpg"
catalog: true
tags:
    - CNN
    - 论文笔记
    - 机器学习
---

## 【论文笔记 4】CV 经典入门 CAM

<https://blog.csdn.net/qq_30159015/article/details/79765520>

### 1.背景

卷积神经网络里面的每一个卷积单元其实都扮演着一个个object detector的角色，本身就带有能够定位物体的能力。但是这种能力在利用全连接层进行classification的时候就丢失了。因此，像那些全卷积的神经网络，比如googlenet，都在避免使用全连接层，转而使用global average pooling，这样的话不仅可以减少参数，防止过拟合，还可以建立feature map到category之间的关联。

### 2.思路

global average pooling(gap)不仅仅是一个regularizer，还能够将卷积层的定位能力一直保持到最后一层。即使这个网络是训练来进行分类的，我们也可以在feature map上获取那些对于分类具有区分性(discriminative)的区域。比如对于一张分类成自行车的图片来说，feature map上面在车轮子，车把这样地方的激活值就会比较大。而且这种网络的训练是end-to-end的，只需要训练classification的网络，我们就可以在forward的时候获取localization的信息。

### 3.class activation map（CAM）

作者将VGG、GoogleNet、AlexNet的最后几层去掉，一般是去掉的全连接层，然后作者加入3*3，stride为1，pad为1的卷积层，之后是14*14的Average Pool层，然后就是softmax层，对这个新的网络进行fine-tuned。等到收敛之后，通过每一层卷积层的输出乘以这一层对应分类的权重，然后对结果加权，就可以得到热成像图，最后就得到了class activation map，叠加在原图上就可以有以上的效果。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1hmik7lm5j30rv0e10xx.jpg%29)



比如说有人预测澳大利亚犬的那组参数(w1,w2,,,wn)，对前面的最后一个卷积层的n张feature map进行加权，就可以得到澳大利亚犬的class activation map，这张map上激活值越大的地方，表示这个地方更有可能是属于澳大利亚犬，对这张class activation map求平均，其实就是这张图属于澳大利亚犬的概率，也就是图上的灰色圆对应的值。我们去找class activation map上的热区，其实就是在对物体进行定位。

​                   ![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1hmj7e49tj30d408l45x.jpg%29)

上面这张图截取了top-5预测类别对应的CAM，可以看到，对于不同的类别，网络可以找到各自的具有区分性或者有信息量的区域，比如对于palace，更多的集中在正面的围墙建筑，对于dome的话，就集中在圆屋顶那个位置。同样，对于狗和鸡的分类，热区更多地集中在他们的头部。对于杠铃，则几种在两边的铁块上。

### 4.总结

神经网络里面不同层的卷积单元都扮演者视觉特征检测器的角色，从底层的视觉特征，比如边缘，角点；到高层的视觉模型，比如物体或者是场景。最后把这些东西综合起来进行分类。我们可以把最后一个卷积层看成是一个词典。每一张feature map对应检测输入中的一个单词，如果有，就在对应位置有高的激活值。后面的fc层其实就是利用这个词典进行分类。比如对于自行车分类器，他对于车轮，车把，车座这三个单词会赋予高的权值，对其他的单词权值很低，那么当他检测到输入图片的词向量在车轮，车把，车座这三个单词的置信度很高的时候，就会把这张图判断成自行车。
在这项工作中，我们提出了一种称为类激活映射（CAM）的通用技术，用于具有全局平均池化的CNN。 这使分类训练的CNN能够学习执行对象定位，而无需使用任何边界框注释。 类激活映射允许我们在任何给定的图像上可视化预测的类别分数，突出显示由CNN检测到的区分性对象部分。 

1. 一般网络没有定位能力原因
   卷积神经网络（CNN）各层的卷积单元实际上表现为物体检测器，尽管没有提供对物体位置的监督。尽管在卷积图层中定位物体的能力非常出色，但使用完全连接的图层进行分类时，这种能力会丧失。
2. 使用GAP原因
   这是因为，在进行映射的平均值时，可以通过查找对象的所有区分性部分来最大化该值，因为所有低激活都会降低特定映射的输出。 另一方面，对于GMP，除了最具辨别性的图像区域以外，所有图像区域的低分不会影响分数，因为只是执行最大分数。

------