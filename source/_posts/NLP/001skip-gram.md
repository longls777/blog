---
title: skip-gram详解
tags: nlp
categories: Machine Learning
date: 2022-07-03 9:45:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/20201223170652487.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/20201223170652487.png
math: true
---

## 跳字模型(skip-gram)

跳字模型的**概念**是在每一次迭代中都取一个词作为中心词汇，尝试去预测它一定范围内的上下文词汇

![v2-6224dddd34017aee696a88289245604c_720w](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-6224dddd34017aee696a88289245604c_720w.jpg)

目标函数为
$$
\prod_{t=1}^{T}{\prod_{-m\le j\le m,j=0}{P(W^{(t+j)}|w^{(t)})}}
$$

**该函数又称为似然函数，这里表示在给定中心词的情况下，在2m窗口内的所有其他词出现的概率（T表示词库里所有词的总数）。我们的目标是要通过调节参数，从而最大化这个函数（因为这个函数越大，表示与实际情况越吻合）。**（注意：这里假设给定中心词的情况下背景词的生成相互独立）

另外，根据平时的习惯，我们通常喜欢最小化损失函数，而不是最大化损失函数。因此我们对该函数取负对数，且除以T，得到新的损失函数（对数似然函数）：
$$
J(\theta)=-\frac{1}{T}\prod_{t=1}^{T}{\prod_{-m\le j\le m}{\log{P(W_{t+j}|w_{t})}}}
$$

那么具体的训练步骤是怎样的呢？请看下图

![v2-519ff918b896d441218013b043b6d443_720w](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-519ff918b896d441218013b043b6d443_720w.jpg)

例句为”勇哥爱跳舞“，中心词取”爱“，步长m取1，也就是范围内的词为“勇哥”和“跳舞”。

**第一步** ：t表示“爱”这个词在词典中的位置，那么“爱”用wt表示，“勇哥”用wt-1表示，“跳舞”用wt+1表示。

**第二步**：将“爱”这个词首先表示为one-hot编码，方便进行后续的矩阵操作。

接下来我们构建两个参数矩阵，分别为中心词矩阵和周围词矩阵，这两个矩阵分别是V×D维和D×V维，其中V表示词典的大小，D表示我们要构建的词向量的维度，是一个超参数，我们暂时认为其是固定的，不去管它。

以中心词矩阵为例，其为V×D维的，而我们的词表里一共有V个词，也就是说，该矩阵的每一行都表示一个单词的中心词向量（低维、稠密的），同理，周围词向量矩阵是D×V维的，每一列表示一个单词的周围词向量表示。

了解了中心词向量和周围词向量的概念后，我们来看第三步：

**第三步：**用第二步中的得到的，“爱”的one-hot编码乘以中心词向量矩阵W，得到一个1×D维的向量，这个向量可以认为是该词的**中心词向量表示**。

**第四步：** 用该中心词向量乘以周围词向量矩阵w*,该步骤可以理解为对于“爱”这个词，我们分别与每一个词作内积，最终得到的1×V向量中的每一个元素，便是该位置的词与“爱”这个词的内积大小。

**第五步**：对于最终的得到的向量，我们再进一步的做softmax归一化，归一化之后的概率越大，表示该词与“爱”的相关性越大，现在我们的目标就是要使得：“勇哥”这个词的概率较大，我们如何去实现这个目标呢？那就是通过调整参数矩阵w和w*,（这里就可以明白这两个矩阵其实只是辅助矩阵，我们根据损失函数，使用反向传播算法来对参数矩阵进行调节，最终实现损失函数的最小化。

**提问：为什么一个词汇要用两种向量表示（中心词向量和背景词向量）？**

1：**数学上处理更加简单**

让每个单词用两个向量表示，这两个表示是相互独立的，所以在做优化的时候，他们不会相互耦合，让数学处理更加简单。

2：**实际效果更好**

如果每个单词用一个向量来表示，那么中心词预测下一个词是自己本身的概率就会很大，因为我们是向量内积来定义；两个单词之间的相似性。所以用两种向量表示在通过效果上会比	一种向量表示更好。

**在训练结束后，对于词典中任一索引为i的词，我们都会得到两组词向量，在自然语言处理应用中，一般使用跳字模型的中心词向量作为词的表征向量。**

![20201223162556158](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201223162556158.png)

![20201223171359404](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201223171359404.png)

![20201223170652487](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201223170652487.png)

### 训练流程

计算隐藏层：

![20201224104950761](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224104950761.png)

计算输出层：

![2020122410585787](http://longls777.oss-cn-beijing.aliyuncs.com/img/2020122410585787.png)

反向传播：计算 prediction errors

![20201224110848658](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224110848658.png)

反向传播：计算$\nabla W_{input}$

![20201224111911128](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224111911128.png)

![20201224111946423](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224111946423.png)

反向传播：计算$\nabla W_{output}$

![20201224112405459](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224112405459.png)

反向传播：更新矩阵$W_{input}$

![20201224113549852](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224113549852.png)

反向传播：更新矩阵$W_{output}$

![20201224142640867](http://longls777.oss-cn-beijing.aliyuncs.com/img/20201224142640867.png)

可以观察到，每次迭代，$W_{input}$ 中只有对应中心词的一行在更新，而$W_{output}$ 里的权重都在更新。

> https://aegis4048.github.io/demystifying_neural_network_in_skip_gram_language_modeling

## 负采样(Negative Sampling)

### 为什么采用负采样？

![vanilla-skip-gram-complexity](http://longls777.oss-cn-beijing.aliyuncs.com/img/vanilla-skip-gram-complexity.png)

采用softmax来归一化，分母上需要计算V次矩阵乘积，V是语料库的大小，一般很大，时间复杂度是$O(V)$，除此之外，对于每个中心词，都要计算一次normalization factor，时间复杂度达到$O(V^2)$，但是在代码实现中，一般只计算一次normalization factor（可能是因为SGD，计算多个中心词之后才更新参数）

负采样使用sigmoid，可以使问题转化为多个独立的二分类任务，复杂度为$O(K+1)$，K的范围一般是$[5, 20]$

### 负采样的具体做法？

负采样每次会更新$W_{output}$中的$(K+1)*N$个参数

假设隐藏层维度$N=3$，词汇总数$V=11$，负样本$K=3$

softmax和negative sampling的对比如下：

![neg_vs_skip](http://longls777.oss-cn-beijing.aliyuncs.com/img/neg_vs_skip.png)



在负采样中，不再通过给定中心词预测周围的词来学习word vector，而是通过区分positive words和negative words来训练

![neg_binomial](http://longls777.oss-cn-beijing.aliyuncs.com/img/neg_binomial.png)

例如，对于中心词regression来说，“logistic", "machine", "sigmoid", "supervised", "neural" 就是positive words，而”zebra", "pimples", "Gangnam-Style", "toothpaste", "idiot"就是negative words

模型训练的目标是最大化$p(D=1|w,c_{pos})$，最小化$p(D=1|w,c_{neg})$

> $w$是center word
>
> negative words来自噪声分布$P_{n}(w)$

如果模型可以很好的区分positive words和negative words，就可以得到well learned word vectors

对于给定的word pair$(w,c)$，判断word $c$是不是来自center word $w$的context words，True($D=1$) or False($D=0$)，使用sigmoid函数来表示：

![image-20220705184738156](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220705184738156.png)

$\overline{c}_{output_{(j)}}$是来自$W_{output}$的word vector

![image-20220705185457497](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220705185457497.png)

对于每个word，需要计算$k+1$次上面的sigmoid，即一个来自context words的$c_{pos}$和$K$个来自$P_{n}(w)$的negative words

![image-20220705185805468](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220705185805468.png)

> $h$是center word的隐藏层表示

![negative_sample_text](http://longls777.oss-cn-beijing.aliyuncs.com/img/negative_sample_text.png)

对于上图，center word是drilling，$c_{pos}$是engineer，橙色的是context words，黄色的是从噪声分布$P_{n}(w)$随机选出的negative words

下面是训练过程：

![neg_opt_1](http://longls777.oss-cn-beijing.aliyuncs.com/img/neg_opt_1.png)

注意，对于每个context word 都要训练，所以上面的例子中，一共要训练$6*5=30$个word vector

### 噪声分布$P_{n}(w)$是什么？

$P_{n}(w)$可以表示为如下的形式：
$$
P_{n}(w) = (\frac{U(w)}{Z})^\alpha
$$
其中每个单词在语料库中出现的次数组成的分布叫做$U(w)$，也叫做unigram分布；$Z$是normalization factor，$\alpha$是平滑因子，其作用如下：

![image-20220705192220777](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220705192220777.png)

> https://aegis4048.github.io/optimize_computational_efficiency_of_skip-gram_with_negative_sampling

