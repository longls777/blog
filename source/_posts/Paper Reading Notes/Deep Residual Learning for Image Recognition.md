---
title: Deep Residual Learning for Image Recognition
tags: CV
categories: Paper Reading Notes
date: 2022-09-18 20:35:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220918203856759.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220918203856759.png
math: true
---

**标题：**《Deep Residual Learning for Image Recognition》

**论文来源：** CVPR 2016

**原文链接：** https://openaccess.thecvf.com/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf



## 概述

对于传统的CNN网络，简单的增加网络的深度，容易导致梯度消失和爆炸。针对梯度消失和爆炸的解决方法一般是正则初始化(normalized initialization)和中间的正则化层(intermediate normalization layers)，但是这会导致另一个问题，退化问题，随着网络层数的增加，在训练集上的准确率却饱和甚至下降了。

按照常理更深层的网络结构的解空间是包括浅层的网络结构的解空间的，也就是说深层的网络结构能够得到更优的解，性能会比浅层网络更佳。但是实际上并非如此，深层网络无论从训练误差或是测试误差来看，都有可能比浅层网络更差。

这就引出了退化问题，**当网络层数加深，我们的训练损失会变得更大（训练集与测试集的损失都增大，所以不是过拟合的问题），既然深层网络相比于浅层网络具有退化问题，那么是就保留深层网络的深度，同时避免退化问题，于是就出现了残差网络。**

![残差块](http://longls777.oss-cn-beijing.aliyuncs.com/img/7c04dd8aefda47ab94902e20fb8b3b05.png)



从信息论的角度讲，由于DPI（数据处理不等式）的存在，在前向传输的过程中，随着层数的加深，Feature Map包含的图像信息会逐层减少，而ResNet的直接映射的加入，保证了 $l+1$层的网络一定比 $l$ 层包含更多的图像信息。

基于这种使用直接映射来连接网络不同层直接的思想，残差网络应运而生。

## 残差和误差

简单来说，残差是观察值与模型估计值之间的差，误差是观察值与真实值之间的差

假设真实值为$x$，模型为映射$f(x)=b$，观测值为$x_0$，那么残差就是$f(x_0)-b$，误差就是$x_0-x$

通常我们只有观测值$x_0$，所以一般残差是可以求得的，但是误差不能

## 残差网络

对于残差网络的命名原因，作者给出的解释是，网络的一层通常可以看做$ y=H(x)$ , 而残差网络的一个残差块可以表示为 $H(x)=F(x)+x$，也就是 $F(x)=H(x)−x$，在单位映射中， $y=x$便是观测值，而 $H(x)$ 是预测值，所以 $F(x)$ 便对应着残差，因此叫做残差网络。



残差网络是由一系列残差块组成的。一个残差块可以表示为：
$$
x_{l+1}=x_l+\mathcal{F}(x_l,W_l)
$$


![残差块](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-bd76d0f10f84d74f90505eababd3d4a1_b.jpg)

图中的Weight指卷积操作，addition是指单位加操作

残差块分成两部分直接映射部分和残差部分。$h(x_l)$是直接映射，反应在图1中是左边的曲线； $\mathcal{F}(x_l, {W_l})$ 是残差部分，一般由两个或者三个卷积操作构成，即图中右侧包含卷积的部分。

在卷积网络中， $x_l$ 可能和 $x_{l+1}$ 的Feature Map的数量不一样，这时候就需要使用 $1\times1$ 卷积进行升维或者降维。这时，残差块表示为：
$$
x_{l+1}= h(x_l)+\mathcal{F}(x_l, {W_l})
$$
其中$h(x_l) = W'_lx$ 。其中$W'_l$ 是$1\times1$卷积操作，但是实验结果 $1\times1$卷积对模型性能提升有限，所以一般是在升维或者降维时才会使用。

![1*1升降维](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-54d11fdb5da318615fae5f579f68c31a_b.jpg)

## 残差网络的原理

残差块一个更通用的表示方式是：
$$
y_l= h(x_l)+\mathcal{F}(x_l, {W_l})\\
x_{l+1} = f(y_l)
$$
现在我们先不考虑升维或者降维的情况，那么$h(\cdot)$ 是直接映射， $f(\cdot)$ 是激活函数，一般使用ReLU。我们假设：

- $h(⋅)$ 是直接映射
- $ f(⋅)$ 是直接映射

那么这时候残差块可以表示为：
$$
x_{l+1}= x_l+\mathcal{F}(x_l, {W_l})\\
$$
对于一个更深的层 $L$ ，其与 $l$ 层的关系可以表示为
$$
x_L = x_l + \sum_{i=l}^{L-1}\mathcal{F}(x_i, {W_i})
$$


这个公式反应了残差网络的两个属性：

1.  $L$层可以表示为任意一个比它浅的$l$层和他们之间的残差部分之和；
2. $x_L= x_0 + \sum_{i=0}^{L-1}\mathcal{F}(x_i, {W_i}) $，$L$ 是各个残差块特征的单位累和，而MLP是特征矩阵的累积。

根据BP中使用的导数的链式法则，损失函数 $\varepsilon $关于 $x_l$ 的梯度可以表示为
$$
\frac{\partial \varepsilon}{\partial x_l} = \frac{\partial \varepsilon}{\partial x_L}\frac{\partial x_L}{\partial x_l} = \frac{\partial \varepsilon}{\partial x_L}(1+\frac{\partial }{\partial x_l}\sum_{i=l}^{L-1}\mathcal{F}(x_i, {W_i})) = \frac{\partial \varepsilon}{\partial x_L}+\frac{\partial \varepsilon}{\partial x_L} \frac{\partial }{\partial x_l}\sum_{i=l}^{L-1}\mathcal{F}(x_i, {W_i})
$$
上面公式反映了残差网络的两个属性：

1. 在整个训练过程中， $\frac{\partial }{\partial x_l}\sum_{i=l}^{L-1}\mathcal{F}(x_i, {W_i}) $不可能一直为 $−1$，也就是说在残差网络中不会出现梯度消失的问题。
2. $\frac{\partial \varepsilon}{\partial x_L}$ 表示 $L$ 层的梯度可以直接传递到任何一个比它浅的 $l$层。

通过分析残差网络的正向和反向两个过程，我们发现，当残差块满足上面两个假设时，信息可以非常畅通的在高层和低层之间相互传导，说明这两个假设是让残差网络可以训练深度模型的充分条件。那么这两个假设是必要条件吗？

## 直接映射是最好的选择

对于假设1，我们采用反证法，假设 $h(x_l) = \lambda_l x_l$ ，那么这时候，残差块表示为
$$
x_{l+1} = \lambda_lx_l + \mathcal{F}(x_l, {W_l})
$$
对于更深的L层
$$
x_{L} = (\prod_{i=l}^{L-1}\lambda_i)x_l + \sum_{i=l}^{L-1}\mathcal{F}(x_i, {W_i})
$$
为了简化问题，我们只考虑公式的左半部分 $x_{L} = (\prod_{i=l}^{L-1}\lambda_l)x_l$ ，损失函数 $\varepsilon$ 对 $x_l$ 求偏微分得：
$$
\frac{\partial\varepsilon}{\partial x_l} = \frac{\partial\varepsilon}{\partial x_L} \left( (\prod_{i=l}^{L-1}\lambda_i) + \frac{\partial}{\partial x_l} \hat{\mathcal{F}}(x_i, \mathcal{W}_i)\right)
$$
上面公式反映了两个属性：

1. 当 $\lambda>1$时，很有可能发生梯度爆炸；
2. 当 $\lambda<1$时，梯度变成0，会阻碍残差网络信息的反向传递，从而影响残差网络的训练。

所以 $\lambda$必须等于1。同理，其他常见的激活函数都会产生和上面的例子类似的阻碍信息反向传播的问题。

## 激活函数移动到残差部分更好

上面我们得出结论”直接映射是最好的选择”，所以我们希望构造一种结构能够满足直接映射，即定义一个新的残差结构 $\hat{f}(\cdot)$：
$$
y_{l+1} = y_l + \mathcal{F}(\hat{f}(y_l), w_{l+1})
$$
上面公式反应到网络里即将激活函数移到残差部分使用，即下图中c

![激活函数在残差网络中的使用](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-1c02c8b95a7916ad759a98507fb26079_b.jpg)

![基于激活函数位置的变异模型在Cifar10上的实验结果](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-ffe81dab49de5306fb001e1da7de7ce3_b.jpg)



实验结果也表明将激活函数移动到残差部分可以提高模型的精度

## 残差网络与模型集成

[Andreas Veit等人的论文](https://arxiv.org/pdf/1605.06431.pdf)指出残差网络可以从模型集成的角度理解。如下图所示，对于一个3层的残差网络可以展开成一棵含有8个节点的二叉树，而最终的输出便是这8个节点的集成。而他们的实验也验证了这一点，随机删除残差网络的一些节点网络的性能变化较为平滑，而对于VGG等stack到一起的网络来说，随机删除一些节点后，网络的输出将完全随机。

![残差网络展开成二叉树](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-cbf6ce3da2669335e119a2f222afa6f5_b.jpg)



> https://blog.csdn.net/m0_47146037/article/details/124299668
>
> 为什么残差连接的网络结构更容易学习？ - 人间白头的回答 - 知乎 https://www.zhihu.com/question/306135761/answer/2491142607
>
> [基础知识复习：残差(residuals)是什么 - Daniel Liu的文章 - 知乎](https://zhuanlan.zhihu.com/p/98643701)
>
> 详解残差网络 - 大师兄的文章 - 知乎 https://zhuanlan.zhihu.com/p/42706477
>
> https://arxiv.org/pdf/1605.06431.pdf

