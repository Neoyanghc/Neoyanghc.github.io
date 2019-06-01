---
layout:     post
title:      "idea_可解释性研究规划导图"
subtitle:   " \"经过和景老师以及学长的讨论对下一步工作的想法进行一个初步总结\""
date:       2019-04-6 13:00:00
author:     "jack"
header-img: "img/post-bg-forest.jpg"
catalog: true
tags:
    - 可解释性
    - 计划
    - 深度学习
---

## 可解释性研究想法规划导图

### 1.Question提出

#### 1.1网络可解释性研究重点的变化

问题来源于景老师组会对家琪学长说的思考点：即现在的很多图像网络已经能够具备两个功能，即一方面能够进行**分类**，指出图片中的信息，另一方面也能够对**目标进行定位**。那下步的研究要何去何从？

并且还参考了综述文章Methods for Interpreting and Understanding Deep Neural Networks 中提出的衡量解释性的角度，来看待这个问题。

#### 1.2 背景简介（Related work）

- 以往的LRP，SA等是对重要决策信息在图像上进行反馈。来验证网络为什么好，是pixel像素级别的
- CAM,grad-CAM 是通过GAP的定位能力来进行重要决策信息的位置热力图还解释网络，是单图片级别的

还有很多解释方法，但是基本都在解释单图片，解释神经元，甚至在每个pixel上面打标签做文章。

而我的重点是是想做对比，对比同类之间的输入差距小，结果是输出差距也小，不同类之间当然都是大。

而且我们希望我们的研究是一个具有**end to end manner** 的端到端过程。这样方便我们去拓展到其他网络。

### 2.Inspiration

#### 2.1 三元组模型网络

受人脸识别的三元组启发，我的个人想法是将单图片级别上升到多图片的对比，大方向就是根据不同图片之间的差距来比较网络输入的**原始图片**和网络输出的**feature map**。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1onvyxxnkj31b60j44bi.jpg%29)

#### 2.2 定义输出差距判断函数

相比输入的原始图片，我们可以通过类标签，或者人工肉眼来其标识其差距。

但是输出的特征空间，因为它经过多次卷积，输出的东西也比较难被我们理解。但是值得一提的是：**临近法**现有的研究已经在fc层已经可以得知下列图片中第二行第一个图片（左边的大象）和第二个图片（右边的大象）（可以说他们在每个像素上面都不同）是极其相似的。（具体是怎么判断差距有待了解）我们的输出也可以是多维的，可以用PCA，t-she等降维来减少计算量。但是这里就有一个难点就是**怎么定义我们的输出差距判断函数**

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qltnlhlcj30md0dfgym.jpg%29)



#### 2.3 输出feature map 选择

这次是受CAM启发。对于网络的输出，我认为应该利用最后一个卷积来做文章，相比去送入**softmax**层的**fc**。**last conv** 具有更多和更好的图片特征。这一点无论是*network in network* 还是 *CAM* 都是这么做的，而且都比较好。当然也可以尝试用fc层。

#### 2.4 衡量可解释的标准

综述文章Methods for Interpreting and Understanding Deep Neural Networks中7.1节提到**Explanation Continuity** 

> If two data points are nearly equivalent, then the explanations of their predictions should also be nearly equivalent

刚好对我们的这个解释做了一个对比说明，先对输入做一个对比，再对输出的做一个对比。如果两个基本一致，那这个不就是有解释性的网络吗？

### 3.两种思路方法

我个人刚开始的想法是3.1第一种事后解释性。但是跟老师头脑风暴后，产生了3.2第二种想法。

#### 3.1 事后解释性

当我们的网络训练好了之后。

通过我们上述提到的定义好的差距function来解释网络的输出。比如下面两幅图片，虽然他们的背景完全不同，但是我们在输入的时候分类认为是相同的。而输出的feature spaces 虽然数值上有很大差距，但是经过我们定义的函数对比后应该是相同的，或者比较接近。

当这个网络的所有样本都能用这个函数衡量的时候，我们算一个输出的相同度的准确率就可以看这个网络是否是比较好的网络。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qo9slwpij30ki0alaji.jpg%29)

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qph5ey8kj30wn0jbadw.jpg%29)

#### 3.2 事中解释性

在网络训练的最后一层上，对比他们的feature map之间的差距，相同类的feature map 应该尽可能的相似，不同类的应该最大化不相似。通过跟 interpret CNN 一样加 loss来惩罚训练。如果整个feature 不好用的话，可以利用cam 将 具体位置定位出来，只针对那一部分内容来让惩罚，让其相似。

![](http://jackyanghc-picture.oss-cn-beijing.aliyuncs.com/007bgNxTly1g1qpiqvfswj311m0kmdki.jpg)



+ 缺点可能会比较难收敛。

### 4. future work 与总结

#### 4.1 总结

+ 摆脱单图片的束缚，用多个的对比来解释网络
+ 具体实现细节还有待推敲

#### 4.2 future work 

 正如上面所述，提到的问题，我们要对feature map 之间的差距进行一个度量，那下一步工作

+ 调研feature map的度量方法
+ 可以将典型网络的feature map的数据导出来看看，看能不能发现什么规律

