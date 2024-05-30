---
title: ZS-BERT - Towards Zero-Shot Relation Extraction with Attribute Representation Learning
tags: RE
categories: Paper Reading Notes
date: 2023-08-25 14:43:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825160632508.png
banner_img: 
math: true
---

**标题：**《ZS-BERT: Towards Zero-Shot Relation Extraction with Attribute Representation Learning》

**论文来源：** NAACL  2021

**原文链接： **  https://aclanthology.org/2021.naacl-main.272

**源码：** https://github.com/dinobby/ZS-BERT



## 概述

很少有研究关注于如何预测训练阶段没见过的实体关系，本文提出了zero-shot BERT来解决这一问题

给定输入的句子和其中实体关系的描述，zs-bert学习两个函数：

- 最小化它们之间的距离
- 分类可见关系

以此来将句子和关系描述映射到一个向量空间

通过基于这两个函数生成未见过的关系和新出现的句子的嵌入，使用最近邻搜索来获得未见过的关系的预测

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825155107275.png)

## Method

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825160632508.png)

ZS-BERT Encoder

- 取[cls]，entity对应token的embedding，得到a

Sentence-BERT Encoder

- 编码关系描述，得到a'



多任务学习架构

- 最小化a和a’之间的距离
- 在训练阶段最小化类别分类损失，测试阶段通过新句子的表征a寻找最接近的关系表征，来得到分类结果

### Learning Relation Attribute Vectors

使用Sentence-BERT，固定参数



### Input Sentence Encoder

提取三部分：

- [cls]对应的embedding（句向量）
- head entity对应的向量
- tail entity对应的向量

实现方式是均值pooling

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825201528010.png)



最后总的输入sentence表示：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825201709763.png)

### 训练Loss

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825201755637.png)

$\gamma$的作用？

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825201852129.png)



## Experiments

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825164018590.png)
