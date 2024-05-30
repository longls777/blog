---
title: DeBERTa - Decoding-enhanced BERT with Disentangled Attention
tags: prlm
categories: Paper Reading Notes
date: 2023-09-15 14:39:00
index_img: 
banner_img: 
math: true
---

**标题：**《DeBERTa: Decoding-enhanced BERT with Disentangled Attention》

**论文来源：**  ICLR  2021

**原文链接： **  https://arxiv.org/abs/2006.03654

**源码：** https://github.com/microsoft/DeBERTa

## 概述

采用两种技术提升BERT和RoBERTa的效果：

- Disentangled attention
- Enhanced mask decoder



## Disentangled attention

DeBERTa使用两个vector来表示一个token：

- content embedding
- position embedding

而不是像BERT一样把它们加起来

单词之间的注意力权重是分别根据单词的**内容和相对位置**使用解缠结矩阵计算的

>  the attention weights among words are computed using disentangled matrices based on their contents and relative positions, respectively. This is motivated by the observation that the attention weight of a word pair depends on not only their contents but their relative positions

对于第$i$个位置的token，使用两个向量表示：$H_i$和$P_{i|j}$，分别表示其内容和相对位置，$i$和$j$的cross attention score计算如下：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915153855581.png)

由四个分数组成：

- content-to-content
- content-to-position
- position-to-content
- position-to-position 



之前的方法使用分离的嵌入矩阵来计算相对位置偏差，这相当于仅用上式中的content-to-content和content-to-position来计算，本文认为position-to-content也很重要，因为word pair的注意力权重不仅取决于它们的内容，还取决于它们的相对位置，而这只能通过同时使用内容到位置和位置到内容项来完全建模

position-to-position并未提供太多额外的信息，在本文的实现中已从上式中删除

> Existing approaches to relative position encoding use a separate embedding matrix to compute the relative position bias in computing attention weights (Shaw et al., 2018; Huang et al., 2018). This is equivalent to computing the attention weights using only the content-to-content and content-toposition terms in equation 2. We argue that the position-to-content term is also important since the attention weight of a word pair depends not only on their contents but on their relative positions, which can only be fully modeled using both the content-to-position and position-to-content terms. Since we use relative position embedding, the position-to-position term does not provide much additional information and is removed from equation 2 in our implementation.

**standard self-attention：**

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915155237823.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915155217222.png)

**disentangled self-attention：**

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915155305067.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915155823999.png)



## Enhanced mask decoder

分离的注意力机制已经考虑了上下文单词的内容和相对位置，但并未考虑这些单词的绝对位置，而在许多情况下，这对于预测是至关重要的。

> 例如：a new **store** opened beside the new **mall**

DeBERTa在softmax层之前引入了绝对词位置嵌入，模型根据单词内容和位置的聚合上下文嵌入解码掩盖的单词

> DeBERTa incorporates absolute word position embeddings right before the softmax layer where the model decodes the masked words based on the aggregated contextual embeddings of word contents and positions.

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915160927249.png)

DeBERTa在所有Transformer层中捕捉相对位置，并且仅在解码掩盖的单词时使用绝对位置作为补充信息

> 此外，文章还做了一个看似微小但重要的改进。在encoder输出向量输入EMD进行 MLM 预训练时，原始的BERT并没有用 [MASK] token替换掉所有被mask掉的token，而是10%保持不变。因为在下游任务的输入中从来没有出现过 [MASK] token，这一设计的初衷是为了缓解预训练和微调之间的不一致性。但是这种方法导致了信息泄漏，即预测以token本身为条件的mask token。DeBERTa在将这些被mask但未改变的token(即那10%被泄露信息的token)输入decoder进行预测之前，将它们encoder输出向量替换为相应的绝对位置嵌入向量。

## Experiments

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915161151152.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915161212779.png)

### Ablation study

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915161305915.png)

基本上所有改进都有效

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915161657211.png)

- 共享映射矩阵并没有带来显著的性能下降
- Conv也是

### Scale up to 1.5 billion parameters

> First, we share the projection matrices of relative position embedding $W_{k,r}$,$W_{q,r}$ with $W_{k,c}$, $W_{q,c}$, respectively, in all attention layers to reduce the number of model parameters.
>
> Second, a convolution layer is added aside the first Transformer layer to induce n-gram knowledge of sub-word encodings and their outputs are summed up before feeding to the next Transformer layer

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230915161857482.png)

比11B的T5效果要好



> ICLR2021 | 微软DeBERTa：SuperGLUE上的王者 - HelloWorld的文章 - 知乎 https://zhuanlan.zhihu.com/p/344649850