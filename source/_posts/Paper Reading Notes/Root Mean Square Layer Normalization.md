---
title: Root Mean Square Layer Normalization
tags: 
categories: Paper Reading Notes
date: 2023-08-09 15:31:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810154724678.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810154017115.png
math: true
---

**标题：**《Root Mean Square Layer Normalization》

**论文来源：** NeurIPS 2019

**原文链接：** https://arxiv.org/pdf/1910.07467.pdf



## 概述

层归一化（LayerNorm）已成功应用于各种深度神经网络，以帮助稳定训练并促进模型收敛，因为它能够处理输入和权重矩阵的重新居中和重新缩放。 然而，LayerNorm 引入的计算开销使得这些改进成本高昂，并且显着减慢了底层网络的速度

本文假设 LayerNorm中的重新居中不变性是可有可无的，并提出均方根层归一化（RMSNorm）。 RMSNorm 根据均方根（root mean square，RMS）对一层神经元的输入求和进行正则化，从而赋予模型重新缩放不变性和隐式学习率自适应能力。 RMSNorm 计算更简单，因此比 LayerNorm 更高效

![LayerNorm能加快模型收敛](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810151142179.png)



## Method

#### LayerNorm

$$
a_i = \sum_{j=1}^{m}w_{ij}x_j,\quad y_i=f(a_i+b_i)
$$



为了解决内部协变量偏移问题（internal covariate shift）
$$
\bar{a}_i=\frac{a_i-\mu}{\sigma} g_i, \quad y_i=f\left(\bar{a}_i+b_i\right),
$$


其中$g_i$是用于重新调整标准化总输入的增益参数，初始化为1

$\mu$和$\sigma$是均值和标准差
$$
\mu=\frac{1}{n} \sum_{i=1}^n a_i, \quad \sigma=\sqrt{\frac{1}{n} \sum_{i=1}^n\left(a_i-\mu\right)^2}
$$


#### RMSNorm

LayerNorm成功的一个众所周知的解释是其重新居中和重新缩放的不变性特性。前者使得模型对于输入和权重的平移噪声不敏感，而后者在输入和权重都被随机缩放时保持输出表示不变。本文假设**重新缩放的不变性是LayerNorm成功的原因，而不是重新居中的不变性**

公式如下：
$$
\bar{a}_i=\frac{a_i}{\operatorname{RMS}(\mathbf{a})} g_i, \quad \text { where } \operatorname{RMS}(\mathbf{a})=\sqrt{\frac{1}{n} \sum_{i=1}^n a_i^2}
$$
与LayerNorm相比，相当于去掉了减去均值这一步，也可以说假设均值为0



RMS计算总体输入的均值，并且对其进行$\sqrt{n}$的缩放（$RMS(\boldsymbol{a})$），然后再对每个输入应用$RMS(\boldsymbol{a})$进行缩放，通过这样做，无论输入和权重分布的缩放如何，输出分布都保持不变，从而有利于层激活的稳定性



## Experiments

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810153949740.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810154017115.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230810154117090.png)



可以看到RMSNorm在质量上与LayerNorm相当，但加快了运行速度，可以作为LayerNorm即插即用的替代