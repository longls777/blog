---
title: BART Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension
tags: prLMs
categories: Paper Reading Notes
date: 2023-08-12 16:29:00
index_img: 
banner_img: 
math: true
---

**标题：**《BART: Denoising Sequence-to-Sequence Pre-training for Natural
Language Generation, Translation, and Comprehension》

**论文来源：** ACL 2020

**原文链接：** https://arxiv.org/pdf/1910.13461v1.pdf



## 概述

训练seq2seq模型的降噪自编码器

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812151354794.png)



## 架构

- 使用标准Transformer结构
- 和GPT一样，将ReLU替换为GeLUs
- 模型参数初始化$N(0,0.02)$
- 比BERT多大约10%的参数



## 预训练任务

- Token Masking：与BERT相同
- Token Deletion：随机删除token，且不保留被删除token的位置信息
- Text Infilling：一段span被替换为一个[MASK]，span的长度服从$\lambda = 3$的poisson分布
- Sentence Permutation：打乱句子的顺序
- Document Rotation：统一随机选择一个token，并旋转文档以使其以该token开始。 此任务训练模型来识别文档的开头



## 微调

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812160619071.png)

- Sequence Classification Tasks：encoder和decoder接收相同的输入，decoder输出的最后一个token的最后隐层状态做分类（类似BERT的[CLS]分类），不同的是BART将附加token添加到末尾，以便解码器中令牌的表示可以关注完整输入中的解码器状态
- Token Classification Tasks：将完整的文本输入encoder和decoder，取decoder最上面的隐层状态来为每个token分类
-  Sequence Generation Tasks：encoder输入input，decoder自回归生成输出序列

- Machine Translation：用一个随机初始化的encoder替换BART的encoder的embedding层
  - 第一步先冻结BART绝大部分，只训练这个随机初始化的encoder，BART的positional embedding和BART encoder第一层的自注意力层
  - 第二步训练所有参数，但是只训练很少的iterations



## 预训练目标的对比

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812162224014.png)

结论：

- Performance of pre-training methods varies significantly across tasks ：在ELI5上表现得最好可能在SQuAD表现得最差
- Token masking is crucial
- Left-to-right pre-training improves generation
- Bidirectional encoders are crucial for SQuAD



![BART优于BERT，和RoBERTa互有优势](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230812162836537.png)