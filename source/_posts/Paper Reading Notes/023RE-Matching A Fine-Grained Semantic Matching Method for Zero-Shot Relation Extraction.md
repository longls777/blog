---
title: RE-Matching - A Fine-Grained Semantic Matching Method for Zero-Shot Relation Extraction
tags: RE
categories: Paper Reading Notes
date: 2023-08-23 14:56:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823164348732.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/14.jpg
math: true
---

**标题：**《RE-Matching: A Fine-Grained Semantic Matching Method for Zero-Shot Relation Extraction》

**论文来源：** ACL Long 2023

**原文链接：** https://aclanthology.org/2023.acl-long.369/

**源码：** https://github.com/zweny/RE-Matching



## 概述

语义匹配（senmantic matching）是zero-shot RE的主流范式，它将给定的输入与相应的标签描述进行匹配。输入中的实体应与描述中的上位词精确匹配，而与匹配无关的上下文应在匹配时被忽略。然而，一般的匹配方法缺乏对上述匹配模式的明确建模。

本文提出了一种专门用于零样本关系提取的精细语义匹配方法。遵循上述匹配模式，**将句子级别的相似度分数分解为实体和上下文匹配分数**。由于缺乏对冗余组件的明确注释，本文设计了一个**特征蒸馏模块**，**以自适应方式识别与关系无关的特征并减少其对上下文匹配的负面影响**。



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823162629351.png)

ZeroRE的两种范式：

- 分别编码Des和Input，这样可以复用，只需要编码$m+n$次
- 同时编码Des和Input，这样需要编码$m \times n$次



对于上图，有一些冗余文本对于语义匹配并无帮助，所以应该识别并且消除冗余文本的影响



## TASK

#### Training

给定$\{ x,e1,e2,y,d \}$，训练模型输出$x$和$d$的语义相似度

- $x$ ： 输入实例（input sentence）
- $e1$ ： 目标实体1
- $e2$ ： 目标实体2
- $y$ ： e1和e2的关系
- $d$ ： 对应的关系描述

#### Testing

给定$\{ x', e1',e2' \}$，预测$e1$和$e2$的关系$y'$，其中$y'$与$y$的交集为空，也就是说预测的关系在训练的时候从来没有出现过，也就是zero-shot

预测的方式是选择最相似的$d$对应的关系$y'$



## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823164348732.png)

- 将关系描述和输入实例分成两个模块，分别建模关系匹配模式，这样的好处是可以复用$d$和$x$，推理更高效
- 分别编码$x$ 和n个候选的$d$
- 对于$d$ ： 为了增强$d$中不足的实体信息，使用Sentence-BERT分别建模$d^h$（head entity），$d^t$（tail entity），和$d$（描述本身）
- 对于$x$ ： 使用BERT编码$x^h$，$x^t$,  $x$
- 联合编码 -> 分别编码 ： $O(mn) \to O(m + n)$
- 实体间的相似度由余弦相似度计算：$cos(x^h,d^h)$  &&  $cos(x^t,d^t)$
- 为了减少$x$中冗余文本的影响，将$x$输入一个蒸馏模块，投影到一个不相关特征的正交空间，得到精细化表示$x^p$
- context matching score = $cos(x^p,d)$
- final score = context matching score + entity matching score



### Relation Description Encoder

使用多种方法构建实体描述：

e.g. $y$ = headquartered_in  $d$ = “the headquarters of an organization is located in a place”   

- 使用关键词：$d$中的entity hypernym直接用于实体表示，$d^h$ = organization， $d^t$ = place
- 使用同义词：$d^h$ = organization, institution, company
- 基于规则的模板填充：收到prompt的启发，使用类似的描述来生成更好的实体表示
  - the head/tail entity types including [S], [S], ...
  - $d^h$ = the head entity types including organization, institution, company



###  Input Instance Encoder

- 插入四个special token来标记head entity和tail entity 
  - [Eh], [\Eh], [Et], [\Et]

- 使用MAXPool来获取对应的entity表征
- context表征$x$由拼接[Eh]和[Et]的隐藏层得来

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823184030911.png)

$\phi$是一个MLP，降维用的



###  Contextual Relation Feature Distillation

这块冗余文本是自己定义的，需要检测

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823184945909.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823184955065.png)

使用一个分类器对编码后的$x$进行分类，但是通常来说分类器很难判断哪些token是冗余的，对于判断实体关系没有用的，所以这里使用了一个梯度反转层

> Gradient Reverse Layer（GRL）：https://arxiv.org/abs/1409.7495
>
> （来自chatGPT）
>
> 梯度反转层（Gradient Reverse Layer）是深度学习中的一个技巧，通常用于域自适应（Domain Adaptation）或领域不变性学习（Domain Invariant Learning）等任务。它的主要作用是通过反转（翻转）梯度的方向，来减小源域和目标域之间的领域差异，从而帮助模型更好地泛化到不同的领域。
>
> 梯度反转层的工作原理如下：
>
> 1. **前向传播**：在前向传播过程中，梯度反转层不会对输入数据进行任何改变，仅仅将输入传递给下一层。
> 2. **反向传播**：在反向传播过程中，梯度反转层会将梯度乘以一个负的标量权重（通常是-1），从而改变了梯度的方向。这样，在反向传播过程中，梯度将朝着相反的方向传递回来。
>
> 梯度反转层通常用于域自适应中的深度神经网络模型，其中源域和目标域之间存在领域差异，但希望模型能够在目标域上表现良好。通过在源域和目标域之间的共享特征提取层之后添加梯度反转层，可以鼓励模型学习到对领域差异不敏感的特征表示。这有助于使模型更好地适应目标域的数据，同时保留对源域数据的有用特征。



### Relation Feature Distillation Layer

旨在减少冗余文本对于matching的影响

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823185730580.png)

- 首先获取将输入$x$投影到冗余$x^*$（上一步检测出来的）方向
- 然后从$x$减去$\hat{x}$获取精细化的文本表征$x^p$

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823190024565.png)



### Fine-Grained Matching and Training

总的匹配分数 = 实体匹配分数 + 文本匹配分数，计算方式如下，：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823190141897.png)

损失函数：

![=](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823190309508.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230823190419879.png)

## Experments

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230824122941594.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230824123659523.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230824123711639.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230824123735657.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230824123948937.png)
