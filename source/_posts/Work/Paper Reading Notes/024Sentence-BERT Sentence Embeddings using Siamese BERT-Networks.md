---
title: Sentence-BERT - Sentence Embeddings using Siamese BERT-Networks
tags: 
categories: Paper Reading Notes
date: 2023-08-25 12:55:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825130739299.png
banner_img: 
math: true
---

**标题：**《Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks》

**论文来源：** EMNLP 2019

**原文链接： **https://arxiv.org/abs/1908.10084

**源码：** https://github.com/UKPLab/sentence-transformers



## 概述

BERT和RoBERTa等需要在计算句子语义相似性时，需要将两个句子同时输入，不适合句子级别的语义相似度搜索

本文提出了Sentence-BERT，使用孪生和三元组网络结构来生成句子级别的表征，可以用来进行余弦相似度计算，大大降低了BERT搜索最相似的两个句子所用的时间



## Method

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825130739299.png)

### pooling

SBERT尝试了三种pooling操作：

- 取[cls]token
- 取均值
- 计算max-over-time

默认配置是取均值

### 分类目标函数

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825133040935.png)

### 回归目标函数

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825133812655.png)

### 三元组目标函数

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825133929233.png)

## Experments

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825140500047.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825140522106.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230825140607981.png)