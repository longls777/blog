---
title: RE-Matching: A Fine-Grained Semantic Matching Method for Zero-Shot Relation Extraction
tags: RE
categories: Paper Reading Notes
date: 2023-08-23 14:56:00
index_img: 
banner_img: 
math: true
hide: true
---

**标题：**《RE-Matching: A Fine-Grained Semantic Matching Method for Zero-Shot Relation Extraction》

**论文来源：** ACL 2023

**原文链接：** https://arxiv.org/pdf/2306.04954.pdf

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
- 分别编码$x$和n个候选的$d$
- 对于$d$ ： 为了增强$d$中不足的实体信息，使用Sentence-BERT分别建模$d^h$（head entity），$d^t$（tail entity），和$d$（描述本身）
- 对于$x$ ： 使用BERT编码$x^h$，$x^t$,  $x$
- 联合编码 -> 分别编码 ： $O(mn) \to O(m + n)$
- 实体间的相似度由余弦相似度计算：$cos(x^h,d^h)$  &&  $cos(x^t,d^t)$
- 为了减少$x$中冗余文本的影响，将$x$输入一个蒸馏模块，投影到一个不相关特征的正交空间，得到精细化表示$x^p$
- context matching score = $cos(x^p,d)$
- final score = context matching score + entity matching score



