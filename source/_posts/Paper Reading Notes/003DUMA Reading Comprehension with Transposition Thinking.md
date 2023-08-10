---
title: DUMA Reading Comprehension with Transposition Thinking
tags: MCQA
categories: Paper Reading Notes
date: 2022-09-29 15:48:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220929162023650.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220929162023650.png
math: true
---

**标题：**《DUMA: Reading Comprehension with Transposition Thinkin》

**论文来源：** IEEE 2021

**原文链接：** https://arxiv.org/pdf/2001.09415v5.pdf

**源码：**https://github.com/pfZhu/duma_code

## 概述

当前用来解决MRC问题的一般是两层结构：

1. 预训练模型的encoder表示层，如ALBERT
2. 对齐网络层，用来捕捉文本，问题，回答，三元组之间的关系，例如OCN和DCMN

![可以看到ALBERT非常强大](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220929155842159.png)

该文不注重设计复杂的对齐网络，而是从人类的思考方式中寻找经验：

1. 快速阅读全文和问题答案，建立整体印象
2. 基于问题和选项的重复信息，从文章中寻找支持证据
3. 基于文章信息，考虑问题和选项，并确定正确答案

该文使用attention来实现这种双向的换位思考模式，称之为passage-to-question attention 或者question-to-passage attention

![image-20220929162023650](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220929162023650.png)

## 模型

DUMA模型的具体设计是：将passage + question + option(1/2/3)输入encoder进行编码，其中passage + question 和每一个option对应，也就是说，如果有三个选项，那么就会有三个输入

![image-20221005142727594](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005142727594.png)

在编码之前，记录passage和question-option对的分界index，以便编码后split

![image-20221005142915197](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005142915197.png)

将总的token输入encoder进行编码：

![image-20221005143207016](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005143207016.png)

将编码后的输出进行分割，一部分是passage， 另一部分是question-option对

![image-20221005143825864](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005143825864.png)

将分割后的二者互作attention

![image-20221005144210917](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005144210917.png)

直接concat，然后通过一个（2*config.hidden_size, 1）的线性层

![image-20221005144349483](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221005144349483.png)

为什么直接concat呢？文章对这里的融合方法进行了探究，发现在点乘，相加和concat中，concat效果最好，作者解释是concat能最好的保留特征

## 复现

根据论文的源代码，复现了本文的实验后得到如下数据：

**albert + duma**

![image-20221002180701225](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221002180701225.png)

与文章中的实验数据进行对比：

![image-20221002180738630](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221002180738630.png)

发现误差在可接受范围内

