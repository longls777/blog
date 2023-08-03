---
title: Transformer
tags: 八股
categories: NLP
date: 2023-08-02 14:12:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230802141733150.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212309079.png
math: true
---

> 详解Transformer （Attention Is All You Need） - 大师兄的文章 - 知乎 https://zhuanlan.zhihu.com/p/48508221
>
> https://blog.csdn.net/qq_27590277/article/details/106263994\
>
> 万字长文帮你彻底搞定Transformer-不要错过！！ - DASOU的文章 - 知乎 https://zhuanlan.zhihu.com/p/153183322
>
> 答案解析(1)—史上最全Transformer面试题：灵魂20问帮你彻底搞定Transformer - DASOU的文章 - 知乎 https://zhuanlan.zhihu.com/p/149799951
>
> 关于Transformer的若干问题整理记录 - 小白丶的文章 - 知乎 https://zhuanlan.zhihu.com/p/82391768

![Transformer](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230802141733150.png)

## 一、Transformer为何使用多头注意力机制？（为什么不使用一个头）

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915201320897.png)

原论文中说到进行Multi-head Attention的原因是将模型分为多个头，形成多个子空间，可以让模型去关注不同方面的信息，最后再将各个方面的信息综合起来。其实直观上也可以想到，如果自己设计这样的一个模型，必然也不会只做一次attention，多次attention综合的结果至少能够起到增强模型的作用，也可以类比CNN中同时使用**多个卷积核**的作用，直观上讲，多头的注意力**有助于网络捕捉到更丰富的特征/信息。**

> 具体可以看这篇文章http://lishilong.xyz/2022/09/15/NLP/transformer%E7%9A%84multi-head/



## 二、Transformer为什么Q和K使用不同的权重矩阵生成，为何不能使用同一个值进行自身的点乘？

> https://www.zhihu.com/question/319339652
>
> https://medium.com/dissecting-bert/dissecting-bert-part-1-d3c3d495cdb3

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212106860.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915212309079.png)

K和Q的点乘是为了得到一个attention score 矩阵，用来对V进行提纯。K和Q使用了不同的$W_k, W_Q$来计算，可以理解为是在不同空间上的投影。正因为有了这种不同空间的投影，增加了表达能力，这样计算得到的attention score矩阵的泛化能力更高。这里解释下泛化能力，因为K和Q使用了不同的$W_k, W_Q$来计算，得到的也是两个完全不同的矩阵，所以表达能力更强。

但是如果不用Q，直接拿K和K点乘的话，你会发现attention score 矩阵是一个对称矩阵。因为是同样一个矩阵，都投影到了同样一个空间，所以泛化能力很差。这样的矩阵导致对V进行提纯的时候，效果也不会好。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230802164631292.png)



## 三、有哪些不同类型的注意力，其公式及计算过程和复杂度差异有什么区别？

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



> 关于Transformer的若干问题整理记录 - 小白丶的文章 - 知乎 https://zhuanlan.zhihu.com/p/82391768
>
> additive attention与dot-product attention - 人帅也要多读书的文章 - 知乎 https://zhuanlan.zhihu.com/p/366993073
>
> 模型汇总24 - 深度学习中Attention Mechanism详细介绍：原理、分类及应用 - 深度学习与NLP的文章 - 知乎 https://zhuanlan.zhihu.com/p/31547842
>
> The attention in transformer （面经干货总结） - 算法吐槽菌的文章 - 知乎 https://zhuanlan.zhihu.com/p/379033238
>
> https://arxiv.org/abs/1409.0473



## 四、在计算attention score的时候如何对padding做mask操作？

padding位置置为负无穷(一般来说-1000就可以)



## 五、为什么在进行多头注意力的时候需要对每个head进行降维？

将原来高维的空间转化为多个低维空间并进行拼接，这样可以既可以丰富特征信息，也减少计算量

> 在**不增加时间复杂度**的情况下，同时，借鉴**CNN多核**的思想，在**更低的维度**，在**多个独立的特征空间，**更容易学习到更丰富的特征信息
>
> transformer中multi-head attention中每个head为什么要进行降维？ - 海晨威的回答 - 知乎 https://www.zhihu.com/question/350369171/answer/1718672303
>
> 扩展：Self-Attention时间复杂度： $O(n^2 \cdot d)$，$n$为max_seq_length，$d$为embedding_dim
>
> ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230803144223933.png)
>
> [矩阵、张量乘法（numpy.tensordot）的时间复杂度分析](https://liwt31.github.io/2018/10/12/mul-complexity/)



## 六、为何在获取输入词向量之后需要对矩阵乘以embedding size的开方？意义是什么？

embedding matrix的初始化方式是xavier init，这种方式的方差是1/embedding size，因此乘以embedding size的开方使得embedding matrix的方差是1，在这个scale下可能更有利于embedding matrix的收敛

> Transformer 3. word embedding 输入为什么要乘以 embedding size的开方 - 屋顶菌的文章 - 知乎 https://zhuanlan.zhihu.com/p/442509602



## 七、简单介绍一下Transformer的位置编码？有什么意义和优缺点？

因为self-attention是位置无关的，无论句子的顺序是什么样的，通过self-attention计算的token的hidden embedding都是一样的，这显然不符合人类的思维。因此要有一个办法能够在模型中表达出一个token的位置信息，transformer使用了固定的positional encoding来表示token在句子中的绝对位置信息。

具体计算方法如下：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230803151604300.png)

其中$t$表示位置，$i$表示维度

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230803151649141.png)

也即第$t$个位置的位置编码为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230803151728600.png)



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/fd9871ec1e564486a546fc47caf4b988.png)

上图横坐标代表维度$i$，纵坐标代表位置$k$。可以看到，对于句子长度最大为50的模型来说，前60维就可以区分位置了

> 奇数维度之间或者偶数维度之间周期不同

给定$k$，存在一个固定的与$k$相关的线性变换矩阵，从而由$pos$的位置编码线性变换而得到$pos+k$的位置编码。这个相对位置信息可能可以被模型发现而利用。因为绝对位置信息只保证了各个位置不一样，但是并不是像0，1，2这样的有明确前后关系的编码，证明如下：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/e5ee3e90bc71dce15245eb2bca5516db.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/86a16423bef4f5f04e02fe66cf08e067.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/434f7c183fc6764b5a5c2995aad18e6c.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/aa04c495cde63f8cc687a2dcc21b0e63.png)

其中$k$为常数，因此我们可以用两个常数来表示：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/34c0742b091d3245dcc81eba57d67c4c.png)

所以可以得到位置$pos$的编码与位置$pos+k$的编码是线性关系：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/05fc26c69fb795ff833d9afb29fe3c96.png)

结论：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/8e789649da45467d9e15088829cf8cab.png)



其中M为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/c76a2c40b6c5478e89e4dbd7d0c08e0c.png)

> 拓展：上面的操作也只可以看到线性关系，怎么可以更直白地知道每个token的距离关系？
>
> 为了解答上面的问题，我们将$PE_{pos}$和$PE_{pos+k}$相乘 (两个向量相乘)，可以得到如下结果：
>
> ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/d71060a995283ccb4589f8022836f345.png)
>
> 我们发现相乘后的结果为一个余弦的加和。这里影响值的因素就是$k$，如果两个token的距离越大，也就是$k$越大，根据余弦函数的性质可以知道，两个位置的$PE$相乘结果越小。这样的关系可以得到，如果两个token距离越远则乘积的结果越小
>
> 这样的方式虽说可以表示出相对的距离关系，但是也是有局限的。其中一个比较大的问题是：只能得到相对关系，无法得到方向关系。所谓的方向关系就是，对于两个token谁在谁的前面，或者谁在谁的后面是无法判断的。数学表示如下：
>
> ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/4382d7b93a5ed7b38608be6a66d51d1c.png)





> https://blog.csdn.net/zhuzyibooooo/article/details/126063398
>
> https://blog.csdn.net/Kaiyuan_sjtu/article/details/119621613
>
> https://blog.csdn.net/qq_43391414/article/details/121061766
>
> https://www.cnblogs.com/gogoSandy/archive/2021/11/18/15565803.html



## 八、还有哪些关于位置编码的技术，各自的优缺点是什么？

**位置编码方式一：**

![[0,1]区间内编码](http://longls777.oss-cn-beijing.aliyuncs.com/img/2e596770167143bba3ca319a987265a4.png)

**位置编码方式二：**

![一种绝对位置编码](http://longls777.oss-cn-beijing.aliyuncs.com/img/538bc20c223e4814b0b87f97eeda6a1b.png)



> https://blog.csdn.net/qq_43391414/article/details/121061766
>
> [让研究人员绞尽脑汁的Transformer位置编码](https://kexue.fm/archives/8130)  膜拜，这个还没看，aaron有空记得看！！！！！
>
> https://zhuanlan.zhihu.com/p/105001610 还有这个，因为他们说面试不会问的这么仔细所以暂时先不看了555



未完待续...