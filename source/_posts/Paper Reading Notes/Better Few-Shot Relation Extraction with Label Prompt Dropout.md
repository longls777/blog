---
title: Better Few-Shot Relation Extraction with Label Prompt Dropout
tags: Few-Shot
categories: Paper Reading Notes
date: 2022-12-18 22:19:00
index_img: 
banner_img: 
math: true
---



**标题：**《Better Few-Shot Relation Extraction with Label Prompt Dropout》

**论文来源：** EMNLP 2022

**原文链接：** https://arxiv.org/abs/2210.13733

**源码：**  https://github.com/jzhang38/LPD

## 论文动机

Few-Shot关系提取的目的是基于非常有限的训练，学习识别两个实体之间的关系。最近的研究发现，文本标签(即关系名称和关系描述)对于学习类别表示非常有用，然而，在学习过程中如何更好地利用这些标签信息是一个重要的研究问题。现有的工作很大程度上假设这种文本标签在学习和预测过程中总是存在的，相反，我们提出了一种称为label prompt dropout的新方法，它在学习过程中随机删除标签描述。实验证明，我们的方法能够导致改进类别表示，在Few-Shot关系提取任务上产生明显更好的结果。

## 研究背景

提取实体关系这个任务一般被作为多分类问题处理，使用LSTM或者BERT进行大语料监督学习是主流处理方法，然而一个新的问题是如何使用少量数据训练以识别新的关系，这个新的研究任务称为few-shot relation extraction（FSRE）。



> [FewRel: A Large-Scale Supervised Few-Shot Relation Classification Dataset with State-of-the-Art Evaluation](https://aclanthology.org/D18-1514.pdf)
>
> [FewRel 2.0: Towards More Challenging Few-Shot Relation Classification](https://aclanthology.org/D19-1649.pdf)

得益于CV领域few-shot的成功，许多人使用元学习框架处理FSRE，从训练数据中随机采样具有不同标签集的事件，以模拟测试阶段的few-shot场景。

基于bert的原型网络性能很好，但是类别原型仅通过每个类别的支持实例的平均表示来构造，忽略了可能提供额外有用信息的文本标签。因此，最近的努力方向是试图修改原型网络，使其也可以使用标签信息。

[Yang et al. (2020)](https://dl.acm.org/doi/10.1145/3340531.3412153)将实体类型信息和关系描述插入到模型中。[Dong et al. (2021)](https://aclanthology.org/2021.emnlp-main.212/)除语句编码器外，还使用关系编码器生成关系表示。[Han et al. (2021a)](https://aclanthology.org/2021.emnlp-main.204/)提出了一种混合原型网络，可以从上下文句子和关系描述中生成混合原型。尽管如此，这些方法在很大程度上假设在学习和预测过程中，每个支持实例都在支持集中提供了相应的文本标签。我们认为，向所有支持实例注入文本标签可能会使训练任务变得没有挑战性，因为模型在训练过程中很大程度上依赖于文本标签，从而导致在测试过程中面对看不见的关系和文本标签时性能较差。理想情况下，文本标签应该被视为额外的信息源，这样模型可以使用或不使用文本标签。



> [Enhance Prototypical Network with Text Descriptions for Few-shot Relation Classification](https://dl.acm.org/doi/10.1145/3340531.3412153)
>
> [MapRE: An Effective Semantic Mapping Approach for Low-resource Relation Extraction](https://aclanthology.org/2021.emnlp-main.212.pdf)
>
> [Exploring Task Difficulty for Few-Shot Relation Extraction](https://aclanthology.org/2021.emnlp-main.204.pdf)

## 本文贡献

- 提出了LPD（label prompt dropout），一种新的标签提示dropout方法，可以更好地利用FSRE中的文本标签。这种简单的设计明显优于先前使用复杂网络结构融合文本标签和上下文句子的尝试。
- 确定了文献中先前实验设置的局限性，并提出了一个更严格的FSRE评估设置。对于这两种设置，我们都展示了相对于之前的技术状态的强大改进。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221219221843700.png)

## 任务定义

FSRE任务：

对于每个实例$(x,e,y)$，句子$x=\{x_1,x_2,x_3,...,x_m\}$，其中每一个$x_i$是一个token，$e=\{e_{head}, e_{tail}\}$是实体的位置，标签$y=\{y_{text},y_{num}\}$，其中$y_{text}$是文本标签，$y_{num}$是数字标签。

在元学习框架下，每个数据集由多个片段组成，每一个片段包括一个支持集 $S$和查询集 $Q$，在N-way-K-shot学习中，$S=\{s_n^k;n=1,...,N,k=1,...,K\}$，包含N个不同的类别，在每个类别中有K个不同的支持实例。

我们的任务是对于查询集$Q$中的每个实例$q$，预测其正确的标签$y \in \{y^1,...,y^N\}$。





## Approach

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221219230137709.png)

### 使用LPD训练

对于每一个支持实例，我们使用 $:$ 将关系描述和文本内容直接连接。为了避免模型过度关注关系描述，我们以概率$\alpha_{train}$随机去掉关系描述这一标签提示。对于查询实例，我们直接将输入其文本。

一个支持实例：

**[CLS] location of event: [E1] Beijing [/E1] held the [E2] 2022 winter Olympics [/E2**]​

随后将其输入Transformer Encoder，将**[E1]**和**[E2]**最后一层的层表示连接起来，作为这一个实例的关系表示$r$：
$$
r=[Encoder(x)_h;Encoder(x)_t]
$$
$h$表示[E1]的位置，$t$表示[E2]的位置。

在K-way-N-shot学习中，我们将一个类别的K个支持实例的关系表示平均一下，得到这个类别的原型。

然后计算查询实例和每个类别原型的点积，作为交叉熵损失中的logit：
$$
u^n=\frac{1}{K}\sum_{k=1}^K{r_k^n}
$$

$$
\mathcal{L}_{train}=-\sum^N_{n=1}{log\frac{exp(r_q^Tu_n)}{\sum_{n'=1}^N{exp(r_q^Tu^{n'})}}}
$$

其中$r_k^n$表示类别$n$的第$k$个支持实例的关系表示，$u^n$是类别$n$的关系原型，$r_q$是查询实例的关系表示。

### 使用prompt指导原型测试

