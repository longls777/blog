---
title: RoBERTa  A Robustly Optimized BERT Pretraining Approach
tags: prLms
categories: Paper Reading Notes
date: 2022-10-06 21:22:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927142410395.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927142410395.png
math: true
---

**标题：**《RoBERTa:  A Robustly Optimized BERT Pretraining Approach》

**论文来源：** arxiv 2019

**原文链接：** https://arxiv.org/pdf/1907.11692.pdf

**源码：** https://github.com/facebookresearch/fairseq

## 概述

该文章通过对BERT的实验的复现发现，超参数的选择对BERT的最终结果有重大影响，该研究仔细衡量了许多关键超参数和训练数据大小的影响，发现BERT明显训练不足。

从模型上来说，RoBERTa基本没有什么太大创新，主要是在BERT基础上做了几点调整：

1. 训练数据更多
2. 训练时间更长，batch size更大 
3. 移除了next sentence prediction loss 
4. 动态调整Masking机制
5.  a larger byte-level BPE 

RoBERTa is trained with dynamic masking (Section 4.1), FULL - SENTENCES without NSP loss (Section 4.2), large mini-batches (Section 4.3) and a larger byte-level BPE (Section 4.4).

## 创新细节

### 训练数据更多

BERT：BOOKCORPUS  plus English WIKIPEDIA 16GB

RoBERTa：

- BOOKCORPUS  plus English WIKIPEDIA 16GB
- CC-NEWS 76GB
- OPEN WEB TEXT 38GB
- STORIES 31GB

### 训练时间更长，batch size更大 

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007152640133.png)

在同样运算消耗的条件下，batch-size增大，PPL（困惑度）变小，效果更好

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007154409170.png)

可以看到，预训练时间越长，效果越好

### 移除了next predict loss 

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007154545994.png)

可以看出来，没有NSP loss之后，模型的性能匹配甚至略微优于原来的模型

与此同时，发现将序列限制为来自单个文档(doc - sentence)的效果略好于从多个文档(full - sentence)打包序列

## 动态调整Masking机制 Dynamic Masking

static masking: 原本的BERT采用的是static mask的方式，就是在`create pretraining data`中，先对数据进行提前的mask，为了充分利用数据，定义了`dupe_factor`，这样可以将训练数据复制`dupe_factor`份，然后同一条数据可以有不同的mask。注意这些数据不是全部都喂给同一个epoch，是不同的epoch，例如`dupe_factor=10`， `epoch=40`， 则每种mask的方式在训练中会被使用4次

dynamic masking: 每一次将训练example喂给模型的时候，才进行随机mask

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007155117023.png)

## a larger byte-level BPE

BERT原型使用的是 character-level BPE vocabulary of size 30K, RoBERTa使用了GPT2的 byte BPE 实现，使用的是byte而不是unicode characters作为subword的单位。

> learn a subword vocabulary of a modest size (50K units) that can still encode any input text without introducing any “unknown” tokens.



> https://blog.csdn.net/luojie140/article/details/112306801
>
> https://zhuanlan.zhihu.com/p/103205929
>
> https://blog.csdn.net/Decennie/article/details/120010025