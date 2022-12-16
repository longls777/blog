---
title: Graph Neural Networks for Natural Language Processing - A Survey
tags: 
categories: Paper Reading Notes
date: 2022-12-16 16:06:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221216163344103.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221216163344103.png
math: true
---

**标题：**《Graph Neural Networks for Natural Language Processing - A Survey》

**论文来源：** arxiv 2022

**原文链接：** https://arxiv.org/pdf/2106.06090.pdf

**源码：** 



## Introductions

##### 图神经网络在NLP中的应用：

- 文本序列中的句子结构信息（语法解析树，例如dependency and constituency parsing trees）用于加强原始序列信息
- 文本序列中的语义信息（如抽象意义表示图和信息提取图等语义解析图）用于增强原始序列信息

这些图结构数据可以编码实体token之间复杂的成对关系，以学习更多信息的表示

##### 在NLP任务中应用图神经网络的难点

- 自动将原始文本序列数据转换为高度图形结构的数据。这种挑战在自然语言处理中是深刻的，因为大多数自然语言处理任务都涉及使用文本序列作为原始输入。从文本序列自动构造图以利用底层结构信息是利用图神经网络解决自然语言处理问题的关键步骤。
- 正确确定图表示学习技术。提出专门设计的GNN来学习不同图结构数据(如无向图、有向图、多关系图和异构图)的独特特征是至关重要的
- 有效地建模复杂的数据。这样的挑战很重要，因为许多NLP任务涉及学习基于图的输入和其他高度结构化的输出数据之间的映射，如序列、树，以及节点和边都有多类型的图数据。

![图的应用分类](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20221216163344103.png)