---
title: Transformer相关问题
tags: transformer
categories: NLP
date: 2022-09-15 21:20:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212106860.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212309079.png
---

> [关于Transformer的若干问题整理记录](https://zhuanlan.zhihu.com/p/82391768)

#### 有哪些不同类型的注意力，其公式及计算过程和复杂度差异有什么区别？

> 这个问题也会被分为几个子问题
>
> - 加性注意力(Additive Attention、乘性注意力(Dot Product Attention)、缩放点积注意力(Scaled Dot-Product Attention)的公式与计算过程？
> - Transformer计算attention的时候为何选择点乘而不是加法？
>   两者计算复杂度和效果上有什么区别？
> - 为什么在进行之前需要对attention进行scaled (attention计算时为什么要除 ) ？

首先我们来看一下三者的公式：

**Additive Attention**(加性注意力模型)
$$
S(x_i,q)=v^{T}tanh(Wx_i+Uq)
$$
**dot product attention** （乘性注意力/点积模型）
$$
S(x_i,q)=x_i^{T}q
$$
**Scaled Dot-Product Attention**(缩放点积模型)
$$
S(x_i,q)=x_i^{T}q/\sqrt{d_k}
$$
其他的还有双线性模型$S(x_i,q)=x_i^TWq$等

想象成一句中文与其对应的英文翻译，每个词为一个向量。两个序列也可以不等长，我们要计算两组序列彼此之间的attention权重。其过程大体可以被抽象为三步：

1. 信息输入：将语言输入模型，转换为对应的向量如 $q,k,v$ 或者$key, value$，我们这里假设存在两组向量序列来表示一句话与其对应的翻译。 $H:h_1,h_2,...h_n$和$S:s_1,s_2,...,s_n$，任意一个元素的维度为$d_k$
2. 对于任意的$h_i,s_j$,以某种attention函数计算两者之间的分数
3. 信息加权平均

##### Additive Attention

对于Additive Attention而言，这个过程为：

1. 计算任意向量$h_i$与向量$s_1,s_2,...,s_n$的attention权重
2. 然后拼接 $h_i−s_1$通过一个特定的线性层计算出其分数，再拼接 $h_i−s_2$，重复通过线性层计算，...，直到所有的均计算出一个结果
3. 再把所有结果通过 softmax 

其计算过程公式如下：
$$
\begin{aligned}
\boldsymbol{c}_t&=\sum_{i=1}^{n}{\alpha_{t,i}\boldsymbol{h}_i} \qquad ;输出的上下文向量\boldsymbol{y}_t\\
\alpha_{t,i}&=align(y_t,x_i)\qquad;两个单词y_t和x_i的对齐情况\\ &=\frac{exp(score(s_{t-1},\boldsymbol{h}_i))}{\sum_{i^{'}=1}^{n}{exp(score(s_{t-1},\boldsymbol{h}_{i^{'}}))}}\quad;对自定义的对齐分数进行softmax
\end{aligned}
$$

从这个角度讲，Additive Attention注意力的公式也可以看作：
$$
f_{score(i,j)}=v_a^Ttanh(W_a[h_i;s_j])=v^Ttanh(W_1s_i+W_2h_j)
$$
其中 $v_a$ 和 $W_a$ 是学习的注意参数。这里 $h$ 是指编码器的隐藏状态， $s$ 是解码器的隐藏状态。

##### Dot Product Attention

同样地，我们也可以将 $f_{score(i,j)}$ 函数替换为乘性注意力，即
$$
f_{score(i,j)}=F(s_i,h_j)=s_i^Th_j
$$
而上述的计算过程不变，依次对任意的 $s_i,h_j$ 求出其乘性注意力分数，然后统一softmax 。这个过程对于序列 $H，S$ 而言,就是矩阵相乘的过程

Dot Product Attention（Mul）和 Additive Attention（Add）两者在复杂度上是相似的。但是Additive Attention增加了三个可学习的矩阵，所以相比另外两个效果会更好，同时也增加了更多的模型参数，计算效率会较低。

至于为什么要用Mul来完成Self-attention，Transformer作者的说法是为了计算更快。因为虽然矩阵加法的计算更简单，但是Add套着$tanh$和$v$，相当于一个完整的隐层。在整体计算复杂度上两者接近，但是矩阵乘法已经有了非常成熟的加速实现。在$d_k$(即attention-dim）较小的时候，两者的效果接近。但是随着$d_k$增大，Add开始显著超越Mul。

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-4ce33c847c71c3092e1a557c857369fb_1440w.jpg)

作者分析Mul性能不佳的原因，认为是极大的点积值将整个softmax推向梯度平缓区，使得收敛困难。也就是出现了“梯度消失”。

这才有了scaled。所以，Add是天然地不需要scaled，Mul在$d_k$较大的时候必须要做scaled。

那么，极大的点积值是从哪里来的呢?

对于Mul来说，如果$s$和$h$都分布在[0,1]，在相乘时引入一次对所有位置的$\sum$求和，整体的分布就会扩大到[0, dk]。

反过来看Add，右侧是被$tanh()$钳位后的值，分布在[-1,1]。整体分布和$d_k$没有关系。

于是，这才出现了Scaled Dot-Product Attention

##### Scaled Dot-Product Attention

除以 $\sqrt{d_k}$ 的具体证明则如下，假设输入元素 $x_i$ 转化而来的 $q,k$ 每一维是均值为0、方差为1的独立随机变量，一共有 $d_k$ 维，那么它们的点积 $q·k=\sum_{i=1}^{d_k}{q_ik_i}$ 的均值为0、方差为 $d_k$
$$
var(\sum{x_iy_i})=kvar(x_1y_1)=kE[x_1^2]E[y_1^2]=k\sigma^4
$$
控制方差为1，所以就要除以$\sqrt{d_k}$，即：
$$
var(\frac{q·k}{\sqrt{d_k}})=\frac{d_k}{(\sqrt{d_k})^2}=1
$$
将方差控制为1，也就有效地控制了前面提到的梯度消失的问题



> https://zhuanlan.zhihu.com/p/366993073?utm_id=0
>
> https://zhuanlan.zhihu.com/p/31547842
>
> https://zhuanlan.zhihu.com/p/379033238
>
> https://arxiv.org/abs/1409.0473

#### Transformer为什么Q和K使用不同的权重矩阵生成，为何不能使用同一个值进行自身的点乘？

![image-20220915212106860](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212106860.png)

![image-20220915212309079](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212309079.png)

先从点乘的物理意义说，两个向量的点乘表示两个向量的相似度

Q，K，V物理意义上是一样的，都表示同一个句子中不同token组成的矩阵。矩阵中的每一行，是表示一个token的word embedding向量。假设一个句子"Hello, how are you?"长度是6，embedding维度是300，那么Q，K，V都是(6, 300)的矩阵

简单的说，K和Q的点乘是为了计算一个句子中每个token相对于句子中其他token的相似度，这个相似度可以理解为attetnion score，关注度得分。比如说 "Hello, how are you?"这句话，当前token为”Hello"的时候，我们可以知道”Hello“对于” , “, "how", "are", "you", "?"这几个token对应的关注度是多少。有了这个attetnion score，可以知道处理到”Hello“的时候，模型在关注句子中的哪些token。

这个attention score是一个(6, 6)的矩阵。每一行代表每个token相对于其他token的关注度。比如说上图中的第一行，代表的是Hello这个单词相对于本句话中的其他单词的关注度。添加softmax只是为了对关注度进行归一化。

然后我们拿这个attention score矩阵与代表着句子特征的V相乘，得到的是一个加权后结果。也就是说，原本V里的各个单词只用word embedding表示，相互之间没什么关系。但是经过与attention score相乘后，V中每个token的向量（即一个单词的word embedding向量），在300维的每个维度上（每一列）上，都会对其他token做出调整（关注度不同）。与V相乘这一步，相当于提纯，让每个单词关注该关注的部分。

好了，该解释为什么不把K和Q用同一个值了。

经过上面的解释，我们知道K和Q的点乘是为了得到一个attention score 矩阵，用来对V进行提纯。K和Q使用了不同的$W_k$,$ W_Q$来计算，可以理解为是在不同空间上的投影。

但是如果不用Q，直接拿K和K点乘的话，你会发现attention score 矩阵是一个对称矩阵。因为是同样一个矩阵，都投影到了同样一个空间，通过矩阵相乘的方式并不能获取每个token对其他token的attention score。俩个向量越相似，内积越大，当一个向量与自己做内积，再与其他不同词的向量做内积后(形成一个打分向量)，该向量经过softmax后，就会变为一个有一个位置的值很大(自己与自己相乘)，其他位置的值非常非常小的状况出现，比如[0.98,0.01,0.05,0.05]，那么，这样的得分再与V矩阵相乘后得出的加权向量就是一个基本上跟自己本身差不多的矩阵，那就失去了self attention的意义

> https://www.zhihu.com/question/319339652/answer/730848834
>
> https://medium.com/dissecting-bert/dissecting-bert-part-1-d3c3d495cdb3

