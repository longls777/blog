---
title: Reservoir computing approaches for representation and classification of multivariate time series
tags: 
categories: Paper Reading Notes
date: 2023-5-10 14:12:00
index_img: 
banner_img: 
math: true
---

**标题：**《Reservoir computing approaches for representation and classification of multivariate time series》

**论文来源：** arxiv 2020

**原文链接：** https://arxiv.org/pdf/1803.07870v3.pdf

**源码：** https://github.com/FilippoMB/Time-series-classification-and-clustering-with-Reservoir-Computing

## 概述

尽管具有无与伦比的训练速度，基于标准RC架构的MTS分类器无法达到完全可训练神经网络的相同精度。本文引入了储层模型空间，这是一种基于RC的无监督方法来学习MTS的向量表示。每个MTS都被编码在一个线性模型的参数中，该模型被训练用来预测储层动态的低维嵌入。与其他RC方法相比，由于中间降维过程，本文的模型空间产生更好的表示并获得可比的计算性能。作为第二个贡献，本文提出了一个用于MTS分类的模块化RC框架，并附带一个相关的开源Python库。该框架提供了不同的模块来无缝地实现高级RC架构。将该体系结构与其他MTS分类器（包括深度学习模型和时间序列核）进行比较。在基准和实际MTS数据集上获得的结果表明，RC分类器的速度大大提高，并且当使用本文提出的表示实现时，也实现了更高的分类精度。

**关键词：**储备池计算，模型空间，时间序列分类，循环神经网络



## 使用RNN处理多变量时间预测分类的一般方法

![使用RNN处理MTS问题](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510142836414.png)

- Encoder-Decoder架构：
  - encoder获取之前MTS的表征
  - decoder根据表征分类
- 只取最后状态$h(T)$

## 全部参数可训练的RNN && 门机制

![image-20230510143010817](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510143010817.png)

- 通过反向传播更新RNN部分的参数$\theta_{enc}$ && $\theta_{dec}$
- 尽管基本RNN在理论上有能力对任何动力系统建模，但在实践中，训练参数困难
- 为了保证稳定性，**RNN中循环函数的导数不能超过1 ？**。然而，**loss的梯度随着反向传播收缩**。使用RC模型是避免此问题的一种方法
-  另一种解决方案是使用**LSTM**，它利用门控机制来保持其内部记忆在长时间间隔内保持不变。然而，LSTM的灵活性是以更高的计算和架构复杂性为代价的。一种流行的变体是门控循环单元(GRU)，它通过使用比LSTM更少的参数来提供更好的记忆保持能力

## 储备池计算架构

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510143757899.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510143806267.png)

为了避免费时的反向传播操作，RC方法采取了一个完全不同的方向：

- 仍然使用RNN的编码函数：

  ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510143933335.png)

- 但是编码器的参数$\theta_{enc}= \{W_{in},W_{r}\}$**是随机生成，并且不训练的**
- **为了弥补这种适应性的不足，一个大的循环层，也就是储备池，产生了丰富的异构动态池，有助于解决许多不同的任务**。



**储备池的泛化能力**

储备池的泛化能力主要取决于三个因素：

- 循环层中大量的处理单元
- 循环连接的稀疏性
- 连接权矩阵$W_r$的谱半径，使系统处于稳定的边缘



**控制储备池**

通过修改以下超参数来控制储备池的行为：

- 谱半径$ρ$
- 非零连接的百分比$β$
- 隐藏单元的数量$R$
- 另一个重要的超参数是$W_{in}$缩放的值$w$，它控制着非线性处理单元的数量，并且与$ρ$一起将储备池内部动力学从混沌状态转移到收缩状态

- 为了正则化，还可以在状态更新函数（也就是上述RNN编码函数）中加入具有标准差$ξ$的高斯噪声



在ESN（Echo state network，回声状态网络）中，decoder一般是一个线性函数：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510145133037.png)

decoder 的参数可以通过一个岭回归的损失函数优化：

![image-20230510145215267](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510145215267.png)

**未经训练的储备池和线性decoder的组合定义了基本的ESN模型**



输出空间$r_x$ 是通过首先处理具有相同储层的每个MTS，然后训练岭回归模型来**提前一步预测输入**得到的:

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510145732020.png)

这两个参数就组成了输出空间$r_x$

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510145741203.png)

随后由decoder处理得到分类结果



# 创新点

#### 提出了储备池模型空间表示方法

1. **储层模型空间的公式**

储备池的泛化能力是建立在它从输入产生大量非均质动力学的基础上的。为了预测下一个输入值，根据感兴趣的预测范围选择不同的动态。

因此，当固定预测步长(例如，提前1步)时，所有那些对解决任务无用的动态都被丢弃。这在输出模型空间中引入了偏差，因为对预测任务不重要的特征仍然可以用于表征MTS。因此，本文提出了一个新的模型空间，其中每个MTS由线性模型的参数表示，该模型通过考虑所有储备池动态来预测下一个储备池状态。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150116294.png)

本文提出的输出空间：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150141601.png)

储备池模型空间表示表征了储备池层序的生成模型：

![image-20230510150255755](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150255755.png)

2. **储备池状态张量的降维**

原因：储备池的高维数，预测模型的参数数量会变得太大，使得所提出的表示变得难以处理

 解决方法：PCA降维

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150440104.png)

## 统一的储备池时间序列分类计算框架

包含四个模块：

- 储备池模块
- 降维模块
- 表示模块
- decoder模块（读出模块）



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150908823.png)

##### 储备池模块

使用双向储备池（类似于双向RNN？）双向性是通过向同一储层输入正反顺序的输入序列来实现的

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510151253240.png)

完整状态是通过连接上式中的两个状态向量获得的，并且可以通过在每一步总结最近和过去的信息来捕获更长的时间依赖性

当使用双向储备池时，本文提出的模型空间变为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510151435589.png)

> 原来是
>
> ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510150116294.png)

##### 降维模块

PCA

##### decoder模块

- MLP
- linear reg
- svm



# 实验结果

### MTS实验结果

#### 基准数据集准确率

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510152008532.png)

#### rm-ESN的一些不同配置的实验结果

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510152057856.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510152133880.png)



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510152213414.png)

### 单变量TS实验结果

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230510151749689.png)

