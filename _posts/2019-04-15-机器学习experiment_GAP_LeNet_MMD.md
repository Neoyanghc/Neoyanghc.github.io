---
layout:     post
title:      "机器学习experiment_GAP_LeNet_MMD"
subtitle:   " \"将MMD应用在LeNet上查看GAP下不同类的差距\""
date:       2019-04-15 13:00:00
author:     "jack"
header-img: "img/post-bg-road1.jpg"
catalog: true
tags:
    - 可解释性
    - 实验记录
    - 深度学习
---

## 机器学习experiment_GAP_LeNet_MMD

> ‘"tensorflow + torch "

### 1. 实验目标

+ 直观观察不同类中的差距
+ 利用MMD进行差距度量

### 2. 实验环境

数据集：**MNIST cluttered dataset**

```bash
pip3 install -r requirements.txt
jupyter notebook mnist.py # The heatmaps are available in out/
```

原始数据：

