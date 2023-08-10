---
title: Learning to Ask Good Questions:Ranking Clarification Questions using Neural Expected Value of Perfect Information
tags: CQGen
categories: Paper Reading Notes
date: 2022-08-26 14:56:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220826145712032.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220826145724722.png
math: true
---

**标题：**《Learning to Ask Good Questions:Ranking Clarification Questions using Neural Expected Value of Perfect Information》

**论文来源：** ACL 2018

**原文链接：** https://arxiv.org/pdf/1805.04655.pdf

**源码：** https://github.com/raosudha89/ranking_clarification_questions

### Introduction

在QA中，机器通过提问来获取补充信息，这种问题就是澄清问题(clarifying question)，澄清问题包括生成（CQGen）和评估（CQR），本文针对于评估任务，提出了一个模型用于对澄清问题进行排序。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220826145712032.png)

如上图所示，提问者提出的问题缺少一些信息，所以机器通过澄清式提问来试图补全缺失的信息，而生成的潜在澄清问有很多，如何从中选择最好的？评价标准是什么？这都是本文试图解决的问题

> 可选择的澄清问题：
>
> 1. What version of Ubuntu do you have?
> 2. What is the make of your wifi card?
> 3. Are you running Ubuntu 14.10 kernel 4.4.0-59-
>    generic on an x86 64 architecture?

基本上可以认为，好的澄清问题的答案可以提供更多更准确的信息

针对以上问题，本文建立了一个基于完全信息期望值（EVPI，expected value with perfect information）的决策框架的神经网络模型，利用问答网站 StackExchange构建的(post, question,answer) 三元组数据集进行训练，通过计算哪个问题能给post提供更多信息量，更有可能得到答案，来对澄清问题进行排序。

### **Contributions**

1. 基于完全信息期望值（EVPI）架构构建了一个用于解决澄清问题排序的神经网络模型。

2. 从StackExchange问答网站构建了一个新的数据集，用于训练一个能通过人们的问题，来提出澄清问题的模型。

### Model

对于每一个问题q有一组可能的答案，每个答案提供的信息的价值不同。EVPI用来衡量一个问题q的所有可能回答带来的收益的期望值。
$$
EVPI(q_i|p)=\sum_{a_j\in A}{P[a_j|p,q_i]U[p+a_j]} \tag{1}
$$
对于问题$q_i$，可能的回答的集合为$A$，$a_j$为其中的一个回答，$p$为post，$P[a_j|p,q_i]$为给定post $p$和问题$q_i$时，得到答案$a_j$的概率，$U[p+a_j]$是用来计算通过回答$a_j$增广后post会更完整多少的效用函数

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220826145724722.png)

在测试时，给定一个帖子 p，用Lucene（检索系统）抽取10个与p相似的帖子，得到对应的澄清问题q和对应的答案a，构成(post, question,answer)的候选三元组数据集。

对于每个问题$q_i$，生成答案表示$F(p,q_i)$，计算答案$a_j$和我们的答案表示$F(p,q_i)$有多接近。然后, 我们计算$p$因为$a_j$而增加的效用。最后，使用式(1)来对所有的澄清问题进行排序。

### **候选问题和答案的生成**

给定一个帖子$p$，第一步是生成一组候选问题和答案，本文的做法是通过信息检索，从数据集中抽取10个与帖子$p$最相似的帖子（包含$p$自身），帖子里面对应的问题和答案作为候选的问题和答案

### 答案模型

第二步就是计算对于给定的帖子$p_i$和问题$q_i$，答案是$a_j$的概率。首先结合帖子$p_i$和问题$q_i$来生成答案表达式$Fans(\overline{p}_i，\overline{q}_i)$，是含有5个隐层的前馈神经网络，词向量用LSTM编码。我们用如下函数衡量答案表达式与候选答案$a_j$之间的距离：
$$
dist(F_{ans}(\overline{p}_i，\overline{q}_i),\hat{a}_j)
=
1-cos\_sim(F_{ans}(\overline{p}_i，\overline{q}_i),\hat{a}_j)
$$
再结合上问题$q_i$与$q_j$（和$a_j$配对）之间的余弦相似度，最终计算出$a_j$是$q_i$的答案的可能性：
$$
P[a_j|p,q_i]=exp^{-dist(F_{ans}(\overline{p}，\overline{q}_i),\hat{a}_j)}*cos\_sim(\hat{q}_i,\hat{q}_j)
$$
$\hat{a}_j$，$\hat{q}_i$，$\hat{q}_j$是对应的平均词向量，用GloVe embedding

因为问题和答案都可以用几种不同的方式来表达相同的意思，为了归纳进相似的答案表示，提高答案的泛化能力，使用如下损失函数训练答案生成器，迫使生成与正确答案$a_i$相近的答案：
$$
loss_{ans}(p_i,q_i,a_i,Q_i)=dist(F_{ans}(\overline{p}_i，\overline{q}_i),\hat{a}_j)+\\
\sum_{j\in Q}{(dist(F_{ans}(\overline{p}_i，\overline{q}_i),\hat{a}_j)*cos\_sim(\hat{q}_i,\hat{q}_j))}
$$
![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220826203720659.png)

上图是答案生成器的训练过程，给定post $p$和问题$q_i$，生成一个答案表示，不仅接近它的原始答案$a_i$，而且如果候选问题$q_j$接近原始问题$q_i$，它也会接近它的一个候选答案$a_j$

### **有效性计算**

给定一个帖子$p_i$和一个候选答案$a_j$，第三步是计算更新的帖子有效性$U(p_i+a_j)$，有效性是指对于帖子$p_i$，答案$a_j$和不同的问题$q_j$叠加对于$p_i$的更新有多有用，因为一个$a_j$可能对应多个$q_j$。对数据集中所有的$(p_i,q_i,a_i)$元组，我们标注为$y=1$，对于候选数据$(p_i,q_j,a_j)$元组，我们标注为$y=0$，除非$a_j=a_i$

> 虽然理论上，更新后的帖子的效用只能通过给定的帖子$p$和候选答案$a_j$来计算，但在实验中，本文发现当候选问题$q_j$与候选答案配对成为效用函数的一部分时，神经EVPI模型表现得更好。本文把这归因于这样一个事实，即关于一个答案是否增加了帖子的效用的很多信息也包含在向帖子提问的问题中

有效性用交叉熵损失函数来衡量：
$$
loss_{util}(y_i,\hat{p}_i,\hat{q}_j,\hat{a}_j)=
y_ilog(\sigma(F_{util}(p_i,q_j,a_j)))
$$
其中$\sigma(F_{util}(p_i,q_j,a_j))=U(p_i+a_j)$，即更新后$p$的效用，是含有5个隐层的前馈神经网络，词向量用LSTM编码

### 联合神经网络模型

总损失函数如下：
$$
\sum_{i}\sum_{j}=loss_{ans}(p_i,q_i,a_i,Q_i)+loss_{util}(y_i,\hat{p}_i,\hat{q}_j,\hat{a}_j)
$$

## **Conclusion**

作者构建了一个用于澄清问题排序的新数据集，并提出了一个新的模型来解决这一任务。模型集成了深度网络架构和EVPI（完美信息期望值）概念，这有效地模拟了提问者的务实选择:如果我问这个问题，我如何想象对方会回答这个问题。

### 总结

本文为了解决对澄清问题排序的问题，首先对于给定的$p_i$，通过检索类似的$p$，建立10个$(p_i,q_i,a_i)$的“文本，问题，答案”三元组，然后通过embedding和余弦相似度，计算对于指定的$p_i,q_i$，其答案是$a_i$的概率，接着再计算对于给定的$p_i,a_j$，$p_i$更新后的效用$U(p_i+a_j)$，最后通过式1（EVPI）来计算候选答案的价值并排序

