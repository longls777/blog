---
title: BERT
tags: 八股
categories: NLP
date: 2023-08-04 16:19:30
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804202041393.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804202041393.png
math: true
---

​	

> 【NLP】Google BERT模型原理详解 - 李rumor的文章 - 知乎 https://zhuanlan.zhihu.com/p/46652512
>
> https://www.6aiq.com/article/1594176245885
>
> http://fancyerii.github.io/2019/03/09/bert-theory/
>
> BERT常见面试题总结（细节） - 魔法学院的Chilia的文章 - 知乎 https://zhuanlan.zhihu.com/p/438851458

![BERT](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804202041393.png)

## 预训练方法

- Masked LM
- NSP（Next Sentence Prediction）



## 模型结构

![image-20230804201953936](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804201953936.png)



## Embedding

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/v2-11505b394299037e999d12997e9d1789_1440w.webp)

- **Token Embeddings**是词向量，第一个单词是[CLS]标志，可以用于之后的分类任务
- **Segment Embeddings**用来区别两种句子，因为预训练不光做LM还要做以两个句子为输入的分类任务
- **Position Embeddings**和之前文章中的Transformer不一样，不是三角函数而是学习出来的



## 预训练任务1：MLM

随机mask 15%的token，其中10%的单词会被替代成其他单词，10%的单词不替换，剩下80%才被替换为[MASK]

由于注意力计算开销是输入序列长度的平方（$O(n^2 \cdot d)$），序列长度太大（512）会影响训练速度，所以90%的steps都用seq_len=128训练，余下的10%步数训练512长度的输入。

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804204154639.png)

## 预训练任务2：NSP

因为涉及到**QA和NLI**之类的任务，增加了第二个预训练任务，目的是让模型理解两个句子之间的联系。训练的输入是句子A和B，B有一半的几率是A的下一句，输入这两个句子，模型预测B是不是A的下一句。



## Fine-tunning

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804203259408.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804203327118.png)



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804204423989.png)

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804204442695.png)

> [BERT如何做QA](https://www.ylkz.life/deeplearning/p10265968/#21-%E6%9E%84%E5%BB%BA%E5%8E%9F%E7%90%86)



## 一、BERT的优缺点

**优点**

- 相对rnn更加高效、能捕捉更长距离的依赖
- 对比起之前的预训练模型，它捕捉到的是真正意义上的bidirectional context信息

**缺点**

- 生成任务表现不佳：预训练过程和生成过程的不一致，导致在生成任务上效果不佳
- 采取独立性假设：没有考虑预测[MASK]之间的相关性，是对语言模型联合概率的有偏估计（不是密度估计）
- 输入噪声[MASK]，造成预训练-精调两阶段之间的差异
- 无法处理文档级别的NLP任务，只适合于句子和段落级别的任务



## 二、自回归和自编码语言模型的优缺点

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804205048874.png)



> https://cloud.tencent.com/developer/article/1491568



## 三、为什么BERT在第一句前会加一个[CLS]标志?

论文中说：

> The first token of every sequence is always a special classification token [CLS]. The final hidden state corresponding to this token is used as the aggregate sequence representation for classification tasks.

因为这个无明显语义信息的符号会更**“公平”**地融合文本中各个词的语义信息，从而更好的表示整句话的语义



## 四、BERT的三个Embedding直接相加会对语义有影响吗？

> Embedding的数学本质，就是以one-hot为输入的单层全连接。也就是说，世界上本没什么Embedding，有的只是one-hot。（苏剑林）

在这里用一个例子再解释一下：

假设 token Embedding 矩阵维度是 [200000,768]；position Embedding 矩阵维度是 [512,768]；segment Embedding 矩阵维度是 [2,768]。

对于一个token，假设它的 token one-hot 是[1,0,0,0,...0]；它的 position one-hot 是[1,0,0,...0]；它的 segment one-hot 是[1,0]。那这个token最后的 word Embedding，就是上面三种 Embedding 的加和。

如此得到的 word Embedding，和**concat**后的特征：[1,0,0,0...0,1,0,0...0,1,0]，再过维度为 [200000+512+2,768] 的全连接层，得到的向量其实就是一样的。



> 为什么 Bert 的三个 Embedding 可以进行相加？ - 海晨威的回答 - 知乎 https://www.zhihu.com/question/374835153/answer/1506279757



## 五、在BERT中，token分3种情况做mask，分别的作用是什么？

15%token做mask；其中80%用[MASK]替换，10%用random token替换，10%不变。其实这个就是典型的**Denosing Autoencoder**的思路，那些被Mask掉的单词就是**在输入侧加入的所谓噪音**

这么做的主要原因是：① 在后续finetune任务中语句中并不会出现 [MASK] 标记；②预测一个词汇时，模型并不知道输入对应位置的词汇是否为正确的词汇（10% 概率），这就迫使模型更多地依赖于上下文信息去预测词汇，并且赋予了模型一定的纠错能力

原文：

> Although this (15% mask) allows us to obtain a bidirectional pre-trained model, a downside is that we are creating a mismatch between pre-training and fine-tuning, since the [MASK] token does not appear during fine-tuning. To mitigate this, we do not always replace "masked" word with the actual [MASK] token.



## 六、BERT训练时使用的学习率 warm-up 策略是怎样的？为什么要这么做？

warmup 需要在训练最初使用较小的学习率来启动，并很快切换到大学习率而后进行常见的 decay

这是因为，刚开始模型对数据的“分布”理解为零，或者是说“均匀分布”（当然这取决于你的初始化）；在第一轮训练的时候，每个数据点对模型来说都是新的，模型会很快地进行数据分布修正，如果这时候学习率就很大，极有可能导致开始的时候就对该数据“过拟合”，后面要通过多轮训练才能拉回来，浪费时间。当训练了一段时间（比如两轮、三轮）后，模型已经对每个数据点看过几遍了，或者说对当前的batch而言有了一些正确的先验，较大的学习率就不那么容易会使模型学偏，所以可以适当调大学习率。这个过程就可以看做是warmup

那么为什么之后还要decay呢？当模型训到一定阶段后（比如十个epoch），模型的分布就已经比较固定了，或者说能学到的新东西就比较少了。如果还沿用较大的学习率，就会破坏这种稳定性，用我们通常的话说，就是已经接近loss的local optimal了，为了靠近这个point，我们就要慢慢来



## 七、BERT训练过程中的损失函数是什么？

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804210940756.png)



## 八、bert的mask为何不学习transformer在attention处进行屏蔽score的技巧？

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804211614532.png)

_____________



- ![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804211452946.png)

_______________



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804211519405.png)

__________



![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230804211546540.png)

总结：

- 本身[MASK]自带位置信息，直接修改attention score的话在一定程度上忽略了位置信息
- 即使修改attention score，但是在此之前这个词已经参与了其他位置的score计算，有一定程度上的信息泄露
- BERT属于DAE（自编码模型）,[MASK]相当于引入噪声，而不同于Transformer中的mask（遮掩）



> bert的mask为何不学习transformer在attention处进行屏蔽score的技巧？ - 知乎 https://www.zhihu.com/question/318355038