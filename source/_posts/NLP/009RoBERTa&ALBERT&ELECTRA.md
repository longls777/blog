---
title: RoBERTa & ALBERT & ELECTRA
tags: 八股
categories: NLP
date: 2023-08-05 12:43:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-e4a49c933af91793292ca6ba2e6c08db_r.jpg
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-e4a49c933af91793292ca6ba2e6c08db_r.jpg
math: true
---

> Bert/Albert/Roberta系列论文阅读对比笔记 - 中森的文章 - 知乎 https://zhuanlan.zhihu.com/p/416210663
>
> http://lishilong.xyz/2022/10/05/Paper%20Reading%20Notes/ALBERT%20A%20LITE%20BERT%20FOR%20SELF-SUPERVISED%20LEARNING%20OF%20LANGUAGE%20REPRESENTATIONS/
>
> http://lishilong.xyz/2022/10/06/Paper%20Reading%20Notes/RoBERTa%20A%20Robustly%20Optimized%20BERT%20Pretraining%20Approach/
>
> ELECTRA: 超越BERT, 19年最佳NLP预训练模型 - 李rumor的文章 - 知乎 https://zhuanlan.zhihu.com/p/89763176
>
> Bert之后，RoBERTa、XLNET、ALBERT、ELECTRA改进对比 - 虹膜小马甲的文章 - 知乎 https://zhuanlan.zhihu.com/p/486532878

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-e4a49c933af91793292ca6ba2e6c08db_r.jpg)



## RoBERTa

相比于BERT的改进：

1. 训练数据更多
2. 训练时间更长，batch size更大
3. 移除了next sentence prediction loss
4. 动态调整Masking机制
5. a larger byte-level BPE



#### 移除NSP

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007154545994.png)

可以看出来，没有NSP loss之后，模型的性能匹配甚至略微优于原来的模型

与此同时，发现将序列限制为来自单个文档（doc - sentence）的效果略好于从多个文档（full - sentence）打包序列



#### Dynamic Masking

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221007155117023.png)

static masking: 原本的BERT采用的是static mask的方式，就是在`create pretraining data`中，先对数据进行提前的mask，为了充分利用数据，定义了`dupe_factor`，这样可以将训练数据复制`dupe_factor`份，然后同一条数据可以有不同的mask。注意这些数据不是全部都喂给同一个epoch，是不同的epoch，例如`dupe_factor=10`， `epoch=40`， 则每种mask的方式在训练中会被使用4次

dynamic masking: 每一次将训练example喂给模型的时候，才进行随机mask



####  a larger byte-level BPE

BERT原型使用的是 character-level BPE vocabulary of size 30K, RoBERTa使用了GPT2的 byte BPE 实现，使用的是byte而不是unicode characters作为subword的单位

该编码方式虽然使模型表现略微下降，但显著减少了未登陆词，故仍受到采用





## ALBERT

两种减少参数技术：

- factorized embedding parameterization（矩阵分解）
- cross-layer parameter sharing（跨层参数共享）

句子间连贯性的目标损失：

-  sentence-order prediction，SOP（句子顺序预测）



#### factorized embedding parameterization

wordpiece embeedings的大小$E$  与 隐藏层embeedings的大小$H$的关系？

XLNet和RoBERTa都遵循了$E≡H$，但是这样有什么道理呢？本文作者也提出了这样的疑问，在作者看来，从建模的角度来看，WordPiece嵌入是为了学习与上下文无关的表示，而隐藏层嵌入是为了学习与上下文相关的表示。RoBERTa论文中阐述了类bert表示的力量来自于使用上下文为学习这种上下文相关表示提供信号。因此，将WordPiece嵌入大小E从隐藏层大小H中分离出来，可以使我们根据建模需求更有效地使用总模型参数，这就决定了$H>>E$

从实用的角度来看，自然语言处理通常要求词汇量V较大，如果$E≡H$，那么增加H就增加了嵌入矩阵$V$x$E$的大小，这很容易导致一个具有数十亿个参数的模型，其中大多数参数只在训练过程中很少更新

由此，作者提出了矩阵分解，将一个非常大的词汇向量矩阵分解为两个小矩阵，例如词汇量大小是V，向量维度是E，隐藏层向量为H，则原始词汇向量参数大小为V x H，ALBert想将原始embedding映射到V x E（低纬度的向量），然后映射到隐藏空间H，这样参数量从 V x H下降到V x E + E x H，参数量大大下降



####  cross-layer parameter sharing

在Bert中，共有12个self-attention层，每一层的结构如下所示：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/2906939-20220815131824715-431109172.png)

Albert的作者经过研究发现，虽然Bert中有着12个self-attention层，但是，如果把每一层的参数都提取出来，会发现每一层的参数都基本相似。因此Albert的作者索性将一个self-attention层复制12次，用这12个完全相同的self-attention层取代原先12个不同的self-attention层。在训练时，我们其实只对一层self-attention进行训练，但在计算时，由于我们将这一层计算了12次，所以计算速度并没有显著地降低。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221006162451080.png)

上图是几种不同的共享形式，all-shared降低的参数量最大，但同时会对最后的效果产生一定的影响，而通过观察发现，shared-FFN可能是降低模型性能的主要原因，shared-attention影响并不大，如果担心影响实际效果，可以选择shared-attention，Albert默认使用的是all-shared



####  sentence-order prediction, SOP

我们知道原始的Bert预训练的loss由两个任务组成，maskLM和NSP（Next Sentence Prediction），maskLM通过预测mask掉的词语来实现真正的双向transformer，NSP类似于语义匹配的任务，预测句子A和句子B是否匹配，是一个二分类的任务，其中正样本从原始语料获得，负样本随机负采样。NSP任务可以提高下游任务的性能，比如句子对的关系预测。但是也有论文指出NSP任务其实可以去掉，反而可以提高性能，比如RoBert

本文以为NSP任务相对于MLM任务太简单了，学习到的东西也有限，因此本文提出了一个新的loss，sentence-order prediction（SOP），SOP关注于句子间的连贯性，而非句子间的匹配性。SOP正样本也是从原始语料中获得，负样本是原始语料的句子A和句子B交换顺序。举个例子说明NSP和SOP的区别，原始语料句子 A和B， NSP任务正样本是 AB，负样本是AC；SOP任务正样本是AB，负样本是BA。可以看出SOP任务更加难，学习到的东西更多了（句子内部排序）

**ALBERT证明了NSP不仅过于简单，而且更多的注重于主题预测而不是句子连贯性预测**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/20190929114712400.png)

NSP模型在处理SOP时的准确率和随机猜测的概率非常接近（52%），验证了NSP更多是主题预测的说法，并且SOP对一系列下游任务都有显著的效果提升



## ELECTRA

全称是Efficiently Learning an Encoder that Classifies Token Replacements Accurately



#### NLP式的Generator-Discriminator

ELECTRA最主要的贡献是提出了新的预训练任务和框架，把生成式的Masked language model（MLM）预训练任务改成了判别式的Replaced token detection（RTD）任务，判断当前token是否被语言模型替换过。那么问题来了，随机替换一些输入中的字词，再让BERT去预测是否替换过可以吗？可以的，但效果并不好，因为随机替换**太简单了**

那怎样使任务复杂化呢？

作者使用一个MLM的G-BERT来对输入句子进行更改，然后丢给D-BERT去判断哪个字被改过，如下：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-2d8c42a08e37369b3b504ee006d169a3_1440w.webp)



#### Replaced Token Detection

上述结构有个问题，输入句子经过生成器，输出改写过的句子，因为句子的字词是离散的，所以梯度在这里就断了，判别器的梯度无法传给生成器，于是生成器的训练目标还是MLM，判别器的目标是序列标注（判断每个token是真是假），两者同时训练，但**判别器的梯度不会传给生成器**，目标函数如下：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805133353516.png)



