---
title: GPT Understands, Too
tags: fine tuning
categories: Paper Reading Notes
date: 2023-08-17 19:19:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818115703123.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818115703123.png
math: true
---

**标题：**《GPT Understands, Too》

**论文来源：** arXiv:2103.10385

**原文链接：** https://arxiv.org/abs/2103.10385

**源码：** https://github.com/THUDM/P-tuning



## 概述

本文提出了P-tuning方法，证明了GPT等自回归的生成式模型也可以具有和BERT等MLM相同甚至更好的NLU能力

随着GPT3 175B等的大模型陆续出现，全参数微调是非常困难的，很多prompt微调依靠手工设计的模板，但是这不靠谱

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230817222040174.png)

本文做的相关实验证明了prompt的设计对于LLM的效果具有很大影响



## Method

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818115703123.png)

给定上下文（$x$），目标（$y$），要求搜索prompt

相比于离散prompt搜索，P-tuning使用可训练的Prompt Encoder生成prompt embedding，比LLM vocab space的词汇可能具有更好的效果



**优化过程**

两个问题：

- 离散性：LLM的原始词嵌入$e$经过预训练后已变得高度离散化。如果$h$用随机分布初始化，然后通过随机梯度下降（SGD）进行优化（已被证明仅在一个小的邻域内改变参数），优化器很容易陷入局部最小值
- 关联性：直觉上认为提示嵌入$h$的值应该是相互依赖而不是独立的，需要一些机制来将提示嵌入彼此关联起来



**解决方法**

P-tuning使用一个包含轻量级神经网络的Prompt Encoder将$h$建模为一个序列，以解决离散性和关联性问题

> LSTM + ReLU + 2-layer MLP

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818121124142.png)

LSTM的参数比LLM小了好几个数量级，而且推理时只需要$h$，可以丢弃LSTM

同时，作者发现增加几个anchor tokens可以提高模型在NLU任务上的表现，例如capital



## Experiments

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230818121735943.png)

效果超群