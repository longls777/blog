---
title: ALBERT A LITE BERT FOR SELF-SUPERVISED LEARNING OF LANGUAGE REPRESENTATIONS
tags: prLMs
categories: Paper Reading Notes
date: 2022-10-05 15:46:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006163318194.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006163318194.png
math: true
---

**标题：**《ALBERT: A LITE BERT FOR SELF-SUPERVISED LEARNING OF LANGUAGE REPRESENTATIONS》

**论文来源：** ICLR 2020

**原文链接：** https://arxiv.org/pdf/1909.11942.pdf

**源码：** https://github.com/google-research/ALBERT



## 概述

在预训练自然语言表示时增加模型大小通常会提高下游任务的性能。然而，由于GPU/TPU内存的限制和更长的训练时间，在某些时候进一步增加模型变得更加困难。为了解决这些问题，本文提出了两种减少参数的技术，以降低内存消耗并提高BERT的训练速度

除此之外，本文还使用了一种专注于建模句子间连贯性的自我监督loss，并表明它始终有助于多句输入的下游任务

两种减少参数技术：

- factorized embedding parameterization（矩阵分解）
- cross-layer parameter sharing（跨层参数共享）

## 模型细节

### factorized embedding parameterization（矩阵分解）

wordpiece embeedings的大小$E$  与 隐藏层embeedings的大小$H$的关系？

XLNet和RoBERTa都遵循了$E≡H$，但是这样有什么道理呢？本文作者也提出了这样的疑问，在作者看来，从建模的角度来看，WordPiece嵌入是为了学习与上下文无关的表示，而隐藏层嵌入是为了学习与上下文相关的表示。RoBERTa论文中阐述了类bert表示的力量来自于使用上下文为学习这种上下文相关表示提供信号。因此，将WordPiece嵌入大小E从隐藏层大小H中分离出来，可以使我们根据建模需求更有效地使用总模型参数，这就决定了$H>>E$

从实用的角度来看，自然语言处理通常要求词汇量V较大，如果$E≡H$，那么增加H就增加了嵌入矩阵$V$x$E$的大小，这很容易导致一个具有数十亿个参数的模型，其中大多数参数只在训练过程中很少更新

由此，作者提出了矩阵分解，将一个非常大的词汇向量矩阵分解为两个小矩阵，例如词汇量大小是V，向量维度是E，隐藏层向量为H，则原始词汇向量参数大小为V x H，ALBert想将原始embedding映射到V x E（低纬度的向量），然后映射到隐藏空间H，这样参数量从 V x H下降到V x E + E x H，参数量大大下降。

### cross-layer parameter sharing（跨层参数共享）

在Bert中，我们共有12个self-attention层，每一层的结构如下所示：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/2906939-20220815131824715-431109172.png)

Albert的作者经过研究发现，虽然Bert中有着12个self-attention层，但是，如果把每一层的参数都提取出来，会发现每一层的参数都基本相似。因此Albert的作者索性将一个self-attention层复制12次，用这12个完全相同的self-attention层取代原先12个不同的self-attention层。在训练时，我们其实只对一层self-attention进行训练，但在计算时，由于我们将这一层计算了12次，所以计算速度并没有显著地降低。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006162451080.png)

上图是几种不同的共享形式，all-shared降低的参数量最大，但同时会对最后的效果产生一定的影响，而通过观察发现，shared-FFN可能是降低模型性能的主要原因，shared-attention影响并不大，如果担心影响实际效果，可以选择shared-attention，Albert默认使用的是all-shared

### sentence-order prediction, SOP

我们知道原始的Bert预训练的loss由两个任务组成，maskLM和NSP(Next Sentence Prediction)，maskLM通过预测mask掉的词语来实现真正的双向transformer，NSP类似于语义匹配的任务，预测句子A和句子B是否匹配，是一个二分类的任务，其中正样本从原始语料获得，负样本随机负采样。NSP任务可以提高下游任务的性能，比如句子对的关系预测。但是也有论文指出NSP任务其实可以去掉，反而可以提高性能，比如RoBert。

本文以为NSP任务相对于MLM任务太简单了，学习到的东西也有限，因此本文提出了一个新的loss，sentence-order prediction(SOP)，SOP关注于句子间的连贯性，而非句子间的匹配性。SOP正样本也是从原始语料中获得，负样本是原始语料的句子A和句子B交换顺序。举个例子说明NSP和SOP的区别，原始语料句子 A和B， NSP任务正样本是 AB，负样本是AC；SOP任务正样本是AB，负样本是BA。可以看出SOP任务更加难，学习到的东西更多了（句子内部排序）

## 模型性能

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20190929114615757.png)

1. 对于non-shared下(BERT-style)，更大的嵌入尺寸能够取得更好的结果，但是提升的幅度其实不大。
2. 对于all-shared(ALBERT-style)，嵌入大小128是最好的。

基于上述这些结果，本文在后续的实验中的嵌入大小统一选用 E = 128 。



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20190929114712400.png)

本文这里对比了3种策略：没有句子间损失(比如XLNet和RoBERTa)、NSP(比如BERT)，SOP(ALBERT)。这里采用的ALBERT也是ALBERT-base。对比过程，一方面对比自身任务中的准确率，另一方面是下游任务的性能表现。

- 在自身任务这一维度，可以看出NSP损失对于SOP几乎是没有任何益处，NSP训练后，在SOP上的表现只有52%，这跟瞎猜差不了多少。据此，可以得出结论：NSP建模止步于主题识别。反观SOP损失，确实一定程度上能够解决NSP任务，其准确率为78.9%，而自身的准确率为86.5%。
- 更为重要的是，在下游任务上SOP损失统统起到促进作用，具体表现在SQuAD1.1提升1%，SQuAD 2.0提升2%，RACE提升1.7%。



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006163318194.png)在上图中，计算速度体现在Speedup列，并以BERT-large的基准。例如，Bert-base的计算速度为4.7x，就代表Bert-base的计算速度时Bert-large的4.7倍（以此类推），但上表也表现出一个较为重要的问题：**Albert的计算速度对比Bert其实并没有多大的提升，但同时，由于减少了参数的量，还会对模型的性能产生一定的影响。**

考虑到这一点，作者还拿Bert-large与Albert-xxlarge进行了对比，结果如下表所示:

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006163511631.png)

在上表中，Albert以较小的step，差不多的训练时间，在不同的数据集上取得了比Bert更好的效果。但考虑到实际使用时，参数量减少给训练结果带来的负面影响，Albert是否比Bert优秀还是要另当别论。



> https://www.cnblogs.com/LAKan/p/16587407.html
>
> https://zhuanlan.zhihu.com/p/139174525
>
> https://blog.csdn.net/Decennie/article/details/119820945