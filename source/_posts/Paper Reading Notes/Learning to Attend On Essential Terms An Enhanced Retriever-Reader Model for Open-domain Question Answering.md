---
title: Learning to Attend On Essential Terms - An Enhanced Retriever-Reader Model for Open-domain Question Answering
tags: MCQA
categories: Paper Reading Notes
date: 2023-02-27 14:13:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227142809881.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227142809881.png
math: true
---

**标题：**《Learning to Attend On Essential Terms: An Enhanced Retriever-Reader Model for Open-domain Question Answering》

**论文来源：** NAACL 2019

**原文链接：** https://aclanthology.org/N19-1030/

**源码：** 

## METHOD

![image-20230227142623454](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227142623454.png)

### 基本术语感知检索-阅读器模型(ET-RR)结构

![image-20230227142809881](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227142809881.png)

问题$Q$，N个answer choices $C_n$，通过基本术语选择器选出$Q$的一个子集，包含基本术语，称为$E$

对于每个answer，$E + C_n$送入检索器，选出top k个sentences，连接这k个sentences，称为$P_n$

然后，对于$\{Q, C_n, P_n\}$三元组，阅读器匹配并获取匹配分数，分数最高的choice作为选择结果

## Reader Model

### Input Layer

- **word embedding**：Glove
- **Part-of-Speech Embedding and Named-Entity Embedding**
- **Relation Embedding**：ConceptNet
- **Feature Embeddings**：
  - word match：单词词目在$Q$或$C$中就为1，其他为0
  - word frequency：应用词频算法
  - essential term：$[e_1,e_2,...]$，在Q中，如果这一个单词是essential term就为1，其他为0

### Attention Layer

获得word-level的embedding之后，使用attention加强单词表示。 两个单词$W_U,W_V$，word层面的attention计算如下：

![image-20230227150133521](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227150133521.png)

使用上式计算三种类型的attention：

- question-aware passage representation
- question-aware choice representation
- passage-aware choice representation

...