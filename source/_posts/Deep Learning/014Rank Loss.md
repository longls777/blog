---
title: Rank Loss
tags: 
categories: Deep Learning
date: 2023-9-15 13:11:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-9a9fdfb71c761db343858a408a5060f1_1440w.webp
banner_img: 
math: true
---



> 一文理解Ranking Loss/Margin Loss/Triplet Loss - 徐土豆的文章 - 知乎 https://zhuanlan.zhihu.com/p/158853633

## Rank Loss

不像其他损失函数，比如交叉熵损失和均方差损失函数，这些损失的设计目的就是学习如何去直接地预测标签，或者回归出一个值，又或者是在给定输入的情况下预测出一组值，这是在传统的分类任务和回归任务中常用的。ranking loss的目的是去预测输入样本之间的相对距离。这个任务经常也被称之为**度量学习**(metric learning)

在训练集上使用ranking loss函数是非常灵活的，我们只需要一个可以衡量数据点之间的相似度度量就可以使用这个损失函数了。这个度量可以是二值的（相似/不相似）。比如，在一个人脸验证数据集上，我们可以度量某个两张脸是否属于同一个人（相似）或者不属于同一个人（不相似）。通过使用ranking loss函数，我们可以训练一个CNN网络去对这两张脸是否属于同一个人进行推断。



## 表达式

主要针对以下两种不同的设置：

- 使用一对的训练数据点（即是两个一组）
- 使用三元组的训练数据点（即是三个数据点一组）



### 二元组样本对

![二元组样本的rank loss](https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-f926222841a5376e85a15c31c5e157f2_720w.webp)

在这个设置中，CNN的权重值是共享的，称之为Siamese Net

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915131614233.png)

成对样本的ranking loss强制样本的表征在正样本对中拥有趋向于0的度量距离，而在负样本对中，这个距离则至少大于一个阈值

> 这里设置阈值的目的是，当某个负样本对中的表征足够好，体现在其距离足够远的时候，就没有必要在该负样本对中浪费时间去增大这个距离了，因此进一步的训练将会关注其他更加难分别的样本对

假设用$r_0$，$r_1$分别表示样本对两个元素的表征，$y$是一个二值的数值，在输入的是负样本对时为0，正样本对时为1，距离$d$是欧式距离，就能得到最终的loss函数表达式：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915132001145.png)

### 三元组样本对

![三元组样本的rank loss](https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-c573b95fcc6e55bc1c17ade5256d6aed_1440w.webp)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915132144837.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915132224898.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/v2-9a9fdfb71c761db343858a408a5060f1_1440w.webp)

## Rank Loss的别名

- **ranking loss**：这个名字来自于信息检索领域，在这个应用中，我们期望训练一个模型对项目（items）进行特定的排序。比如文件检索中，对某个检索项目的排序等
- **Margin loss**：这个名字来自于一个事实——我们介绍的这些loss都使用了边界去比较衡量样本之间的嵌入表征距离
- **Contrastive loss**：我们介绍的loss都是在计算类别不同的两个（或者多个）数据点的特征嵌入表征。这个名字经常在成对样本的ranking loss中使用。但是我从没有在以三元组为基础的工作中使用这个术语去进行表达
- **Triplet loss**：这个是在三元组采样被使用的时候，经常被使用的名字
- **Hinge loss**：也被称之为**max-margin objective**。通常在分类任务中训练SVM的时候使用。他有着和SVM目标相似的表达式和目的：都是一直优化直到到达预定的边界为止。