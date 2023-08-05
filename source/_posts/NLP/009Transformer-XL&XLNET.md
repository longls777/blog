---
title: Transformer-XL & XLNet
tags: 八股
categories: NLP
date: 2023-08-05 16:43:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805145917736.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805162007766.png
math: true
---

> https://carlos9310.github.io/2019/11/11/transformer-xl-and-xlnet/
>
> NLP系列之预训练模型（二）：Transformer-XL及其进化XLNet - 周星星的文章 - 知乎 https://zhuanlan.zhihu.com/p/355366424
>
> https://blog.csdn.net/weixin_43269174/article/details/94323036



## Transformer-XL

Transformer-XL 旨在于解决长期以来困扰 NLP 界的难题：**捕捉长距离依赖关系**，这也是其名称的由来 XL: extra long

有学者研究表明，LSTM 的平均上下文长度为 200（Transformer 低于 50）。而在同等的计算力下，Transformer-XL 最终可以捕获的依赖关系距离比循环神经网络多 80%，更超出 Transformer 450%。这还不够，next token prediction 速度更是 vanilla Transformer（Transformer 在字符级和深度上的改良版本）的 1,800+ 倍。这其中的秘诀在于 Transformer-XL 对自注意力机制引入的两项调整 —— **循环机制（recurrence mechanism）** 和 **相对位置编码（relative positional encoding）**



#### Vanilla Transformer

 Transformer-XL 并非直接从 2017 年发布的原始 Transformer 演化而来，而是一个叫 vanilla Transformer 的版本，Vanilla Transformer 将 Transformer 的深度扩展到 64 层，并应用了 标准 Transformer 架构（standard Transformer architecture）：将每一层中的 Decoder 剔除，并将 Encoder 中的自注意力层改为 Masked 自注意力层，由此一来模型结构大大简化，使得 Transformer 具备成为优秀特征提取器的资质

 vanilla Transformer 在训练时，首先将完整的语料分成若干个等长的 segment，计算 self-attention 不考虑本 segment 以外的语料信息，由此预测 segment 内部的 next token probability。而在推理 (预测) 时，逐字划分 segment 边界，每过一个时间步长则向右滑动一个单位。如下图所示：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805145857342.png)

上述编码策略有如下弊端：

- 捕捉长距离依赖关系依然受限于segment的长度
- 对长文本的分割破坏了语义边界，导致上下文碎片化（context fragmentation）



为了克服上述弊端，有效建模长距离依赖关系，就有了Transformer-XL，其与Vanilla Transformer相比，有如下两个特点：

- 片段级的循环机制（Segment-Level Recurrence with State Reuse），引入memory模块（cache之前一个或多个segment的隐状态信息），循环建模片段间的联系
  - 使超长距离依赖关系的编码成为可能
  - 使得片段之间产生交互，解决了上下文碎片化问题
- 相对位置编码（Relative Positional Encodings），代替绝对位置编码
  - 避免了memory中缓存的片段的位置信息与当前片段中的位置信息相互混淆



#### Segment-Level Recurrence with State Reuse

同 vanilla Transformer 一样，Transformer-XL将语料事先划分为等长的 segments，训练时将每一个 segment 单独投入计算 self-attention。每一层输出的隐藏状态作为记忆存储到内存中，并在训练下一个 segment 时，将其作为额外的输入，代表上文中的语境信息。这样一来便在上文与下文之间搭建了一座桥梁，使得模型能够捕获更长距离的依赖关系

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805145917736.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805150106246.png)



#### Relative Positional Encodings

recurrent机制使得先前的绝对位置编码方案不再适用，因为在多个segment中会出现多个同样的位置信息。 为此，作者们提出一种新的相对位置编码形式。其不仅与绝对位置一一对应，而且具有更好的泛化性

首先，在标准的transformer中，在同一个segment下的query向量$q_i$和key向量$k_j$的注意力得分可分解为：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/transformer-xl-03.png)

其中$E$表示词向量组成的矩阵，$U$表示绝对位置向量组成的矩阵。$(a)$表示内容间的交互，$(b)$和$(c)$分别是$i$位置的内容和位置信息分别与$j$位置的位置和内容信息之间的交互，$(d)$表示位置间的交互

基于仅依赖相对位置信息的思想，Transformer-XL将上述query与key的得分改为如下形式：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/transformer-xl-04.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805151923698.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805152049350.png)



#### 为什么Transformer-XL能有效解决BERT的长度限制问题？

- 因为BERT在预训练的时候，就把输入长度限制在512，BERT会把1~512位置映射到一个768维的position embedding（BERT并没有用原生Transformer的三角函数位置编码），因此没有512以上的position embedding。我们当然也可以重头训练一个最大长度为1000的BERT，但会很耗资源
- Transformer-XL输入是没有position embedding的，**相对位置信息是加在每层encoder的attention计算**中。通过循环机制和相对位置编码，Transformer-XL理论上能接受无限长的输入



## 预训练中的三种语言模型

#### 自回归语言模型（Autoregressive LM，AR）

![AR](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-679ea82005147050ed59c793d5b88e11_1440w.webp)

- 如Transformer-XL，GPT等，任务为根据上文内容预测下一个token，预训练任务为单向的语言模型任务
- 优点：可以完成生成类的NLP任务
- 缺点：只能利用单向信息



#### 自编码语言模型（Autoencoder LM，AE）

![AE](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-ae16b70e375109cc357d02c0abe159ee_1440w.webp)

- 如BERT等，根据上下文单词来预测被[MASK]掉的token
- 优点：能利用到双向信息
- 缺点：预训练和Fine-tuning输入不一致，预训练时输入有[MASK]，但在具体下游任务时，输入是没有[MASK]的



#### 预训练方法中AR和AE的优缺点

- 独立假设。BERT基于**所有被mask的token间是相互独立的**假设对联合条件概率$p(\overline{x}|\hat{x})$进行因式分解的。AE式中的$\approx$强调了这里有独立假设导致等号不成立，而AR式中则没有这种假设
- 输入噪声。AE引入了mask，使得预训练与下游任务不一致，而AR中则没有这种输入噪声(**此处虽然在原始输入中没有直接引入噪声，但是在内部处理的时候会用到掩码矩阵进行token的预测**)
- 上下文的依赖。AR只能依赖当前位置左边的token，而AE可同时依赖左右两边的token，**这使得bert在NLU方面的性能要好于GPT**



#### 排列语言模型（Permutation Language Modeling，PLM）

**XLNET结合自回归语言模型和自编码语言模型的优点，提出了排列语言模型**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image_20230805154228.png)



## XLNet

- 排序语言模型（PLM）
- 双流自注意机制
- 部分预测（Partial Prediction）
- 相对segment编码
- Transformer-XL中（基于memory单元）的segment循环机制和相对位置编码

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/xlnet-04.png)



#### Two-Stream Self-Attention for Target-Aware Representations

![原始PLM目标函数](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805160801392.png)

虽然上述提出的PLM的目标函数具有很好的特性（**充分利用双向上下文信息且不(显式地)引入外部噪声**），但用传统的transformer计算式中的$p_{\theta}$无法work。

**因为在未引入排列机制前，每个输入序列的顺序是确定的。而引入排列机制后，同样的序列（目标token之前的序列）后要预测的token可能不同，如果还是用那种经典的AR方式计算下一个token的分布情况，会导致不同的token却有相同的分布。** 

> 举个例子：
>
> I love New York这四个token，现在有两个序列1-> 2 -> 3 ->4和1-> 2 -> 4 ->3，假设已经知道前面两个token是I love，要预测下一个token，很明显，在两个序列中，下一个token为New的概率都是一样的，这是非常不合理的，因为New作为位置3和位置4的概率应该是不一样的

为了避免上述问题，PLM在预测下一个token的分布时将目标token的位置也考虑进来：

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805160827864.png)

其中$g_{\theta}(X_{z < t},z_t)$表示一种附加了目标位置$z_t$作为输入的新型表示

虽然$g_{\theta}$解决了预测目标的歧义，但如何定义$g_{\theta}$仍是个不小的问题。而在传统的transformer结构中有两个相互矛盾的要求：

1. 在预测token $x_{z_t}$时，$g_{\theta}(X_{z < t},z_t)$只能利用位置信息$z_t$和上文信息$X_{z < t}$，不能利用内容信息$x_{z_t}$
2. 在预测其他token时，如$x_{z_j}$，其中$j>t$，又希望$g_{\theta}(X_{z < t},z_t)$能将内容信息$x_{z_t}$也编码进来，以提供完整的上下文信息。为了解决上述矛盾，提出**双流机制**（**用两种隐状态表示而不是像传统transformer中那样只有一种**）

> 简而言之：
>
> - 当token用于预测后面的字符，这时候，它应该能看到自己的内容信息和位置信息
> - 当token用于预测自己到底是哪个字符，这时候，它应该只能看到自己的位置信息

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805162007766.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805162823421.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805163443235.png)



#### Partial Prediction

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805163643559.png)



#### 与Transformer-XL的结合

这里就是把Transformer-XL的循环机制和相对位置编码引进XLNet中，值得注意的是，Permutation只争对当前的segment，上一个segment的排序是原文本的顺序排列



#### Modeling Multiple Segments

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230805163939394.png)



#### XLNet与BERT对比

BERT和XLNet都只预测一个序列中的部分token。对BERT来说，这是必须的，因为如果mask了所有的token，就不可能做出有意义的预测。此外，对BERT和XLNet而言，部分预测通过仅预测具有足够上下文的token来降低优化难度。然而**BERT中由于独立性假设无法对mask的token间的依赖关系进行建模，XLNet就没有这种缺陷**