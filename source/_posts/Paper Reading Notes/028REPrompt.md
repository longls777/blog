---
title: RelationPrompt - Leveraging Prompts to Generate Synthetic Data for Zero-Shot Relation Triplet Extraction
tags: RE
categories: Paper Reading Notes
date: 2023-08-30 21:20:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830220353071.png
banner_img: 
math: true
---

**标题：**《RelationPrompt: Leveraging Prompts to Generate Synthetic Data for Zero-Shot Relation Triplet Extraction》

**论文来源：** ACL Findings 2022

**原文链接： **  https://aclanthology.org/2022.findings-acl.5/

**源码：** https://github.com/declare-lab/RelationPrompt



## 概述

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830212615881.png)

RE三元组抽取任务，只给sentence，要求识别两个entity和relation

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830213826241.png)

- (a)：注释后的sentence用来训练
- (b)：使用没见过的relation用来验证
- (c)：利用relation，使用PrLMs，prompt生成合成训练samples

> We propose RelationPrompt which reframes the zero-shot problem as synthetic data generation. The core concept is to leverage the semantics of relation labels, prompting language models to generate synthetic training samples which can express the desired relations



## Method

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830220353071.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830215720215.png)

### Relation Generator

- 首先在训练集上微调关系生成器$M_g$，使其学会如何生成合成数据$D_{synthetic}$，$M_g$结构如下图(a)
- 生成合成数据的时候，使用temperature t来控制多样性；遇到生成错误的数据（没有Context:等标记），丢弃，重复生成直到成功





### Relation Extractor

- 首先在训练集上微调关系提取器$M_e$，然后继续在合成数据$D_{synthetic}$上微调，最终在没见过的relation的数据集$S_u$上测试
- 预测single relation triplet时，decoder不需要输入来初始化，如下图(a)
- 如果实体或关系无效，将其视为该样本的空预测
- 换句话说，ZeroRC就相当于给定entity作为decoder的初始输入，如下图(b)
- 本文的方法自然包含ZeroRC和ZeroRTE，因为只在推理阶段有变化，训练阶段是统一的

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830221911288.png)

### Extracting Multiple triplets using Triplet Search Decoding

对于合成数据的 RelationPrompt 生成，每个样本仅限于包含单个关系三元组。 因此，传统的三元组提取模型很可能无法在本文的多三元组 ZeroRTE 框架中表现良好，因为它们通常假设训练样本每个句子可能包含多个三元组

对此本文提出了multi-triplet ZeroRTE

- 如上图(c)
- 取概率最高的top b个$e_{head}$，分别指导后续解码，之后的$e_{tail}$和$y$也是一样
- 所以一共会生成$b^3$个不同的relation triplet

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830223257940.png)

- 通过在验证集上微调得到的probability threshold来筛选这些生成的relation triplet



## Experiments

关系生成器用GPT-2，关系提取器用BART

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830223853282.png)

这个点，真低呀

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830223942506.png)

可能因为生成了数据，所以比ZS-BERT高很多呀，NoGen的时候还略微弱于ZS-BERT（m=5/10）

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830224137378.png)

两种方法都有效

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830224254365.png)

生成的效果不错

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830224316196.png)

后来下降可能是因为生成的数据质量不够好，因为后来选取的都是概率比较小的token

> 125 = 5^3，这里top b的b应该是5



## 总结

感觉换个prompt方式还有PrLM，应该会比RE-Matching要高

RE-Matching用的是BERT-base，110M参数量，本文用的GPT-2 124M和BART 140M，感觉差了也不是很多

本文的提升比较大应该主要是因为用GPT-2造了新数据