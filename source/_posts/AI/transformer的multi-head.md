---
title: Transformer为什么使用Multi-Head？
tags: transformer
categories: AI
date: 2022-09-15 19:19:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915201320897.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915201320897.png
math: true

---

## Transformer为何使用多头注意力机制？

![image-20220915201320897](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915201320897.png)

在原文中，作者发现采用不同的$d_q, d_k, d_v$去线性投射QKV对结果有正向的影响：

![image-20220915201642151](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915201642151.png)

至于为什么会有正向的影响，作者的理解是多头注意力模型可以关注多个子空间的不同位置的信息，

![image-20220915202009494](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220915202009494.png)有大量的paper表明，Transformer，或Bert的特定层是有独特的功能的，底层更偏向于关注语法，顶层更偏向于关注语义。假设在同一层Transformer关注的方面是相同的，那么对该方面而言，不同的头关注点应该也是一样的。但是我们发现，同一层中，总有那么一两个头独一无二，和其他头的关注pattern不同，比如下图：

![From：https://arxiv.org/pdf/1906.05714.pdf](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-7ffa5458509e2ea7719774a0f1e6ecdc_720w.jpg)

从这张图中我们可以发现，同一层中的多数头的关注模式是一样的（比如第3层的1234头），但是又总会有那么一两个头与众不同（比如第2层的0头）。这种模式是很普遍的，为什么会出现这种情况？

首先，如果所有头的初始化参数一样，那么最终得到的收敛的参数是一样的，关注模式也就是一样的，那么，关注模式的不同就来自于初始化的不同。

那么问题来了：

- 在一层中，不同头之间的差距有多少（用 $h_i$去度量），怎么去解释这个差距？
-  $h_i$是否随着层数的变化而变化?

[arxiv.org/pdf/1906.04341...](https://arxiv.org/pdf/1906.04341v1.pdf) 可视化了这两个问题，如图所示，不同的颜色代表不同的层，同一颜色的分布代表了同一层的头差距。我们可以先看看第一层，也就是深蓝色。在左边出现了一个点，右边和下边都有点出现，分布是比较稀疏的。再看看第六层浅蓝色的点，相对来说分布比较密集了。再看看第十二层，深红色，基本全部集中在下方，分布非常密集。

根据这篇文章我们可以推论：头之间的差距随着所在层数变大而减少，也就是说，头之间的方差随着所在层数的增大而减小。

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-93198d088182bad94f16aa0802acbd62_720w.jpg)

但是为什么会有这种差距呢？一种可能的解释是，它类似一种noise，或者dropout，而不是去关注不同的方面，也就是说，无论多少层，既然都会出现与众不同的头，那么这个（些）头就是去使得模型收敛（效果最优）的结果。

最后来看看Transformer原论文中的结果，我们主要看base那一行和（A）组。对于PPL和BLEU，确是8个头/16个头最好，1个头最差，4个和32个头稍微差一点，但是差的不多。从这里来看，head也不是越多越好或者越少越好。

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-7aaa68b09cebf9687892958d1cb86859_720w.jpg)

[arxiv.org/pdf/1905.09418...](https://arxiv.org/pdf/1905.09418.pdf)这篇文章探究了几个问题，其中包括：

- 翻译的质量在何种程度上依赖单个头？
- 头的模型是否一致？其中对翻译最重要的是什么模式？
- 能否去掉一些头而不失效果？

文章的结论是：

- 只有一小部分头对翻译而言是重要的，其他的头都是次要的（可以丢掉）。
- 重要的头有一种或多种专有的关注模式。

作者使用了自信度和LRP（Layer-wise Relevance Propagation）来描述头的重要度。结果见图1a,1b,2a,2c。除此之外，作者考虑了不同头的功能(functions)。头的功能可以分为三类：

- 关注左右的Token（紫色）
- 关注语法（绿色）
- 关注罕见词（橙色）

结果见图1c,2b,2d。

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-4b8ac4d733f77898e7ae0b07df52c45b_720w.jpg)

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-e77bdeed4996ee262fe6f931783d9fdc_720w.jpg)

那么回到问题，为什么需要有Multi-Head。从这篇文章的结果来看，Multi-Head其实不是必须的，去掉一些头效果依然有不错的效果（而且效果下降可能是因为参数量下降），这是因为在头足够的情况下，这些头已经能够有关注位置信息、关注语法信息、关注罕见词的能力了，再多一些头，无非是一种enhance或noise而已。

简单回答就是，Multi-Head可以让transformer可以注意到不同子空间的信息，捕捉到更加丰富的特征信息。其实本质上是论文原作者发现这样效果确实好。

>  https://www.zhihu.com/question/341222779
>
> https://arxiv.org/pdf/1906.04341v1.pdf
>
> https://arxiv.org/pdf/1905.09418.pdf