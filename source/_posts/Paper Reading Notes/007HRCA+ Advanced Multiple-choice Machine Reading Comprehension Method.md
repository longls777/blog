---
title: HRCA+ - Advanced Multiple-choice Machine Reading Comprehension Method
tags: MCQA
categories: Paper Reading Notes
date: 2023-02-27 15:34:00
index_img: 
banner_img:
math: true
---

**标题：**《HRCA+: Advanced Multiple-choice Machine Reading Comprehension Method》

**论文来源：** LREC 2022

**原文链接：** https://aclanthology.org/2022.lrec-1.651/

**源码：** 

## Method

![image-20230227154427405](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227154427405.png)

该模型模拟人类思考过程，即首先确认问题，然后查看候选选项，最终融合问题和候选选项的信息来阅读整篇文章

基于预训练模型，首先生成（文章，问题，候选选项）三元组$(P,Q,O)$的word embedding，每个word embedding划分为对应于（文章，问题，候选选项）的三部分

此外，K个HRCA层将以下三个步骤重复K次，从而模拟人类试图在阅读理解考试中获得高分的方式：

1. 对问题进行多头self-attention（将问题视为q、k和v），这一步的目的是让模型再次确认问题
2. 对步骤1中显示的选项和更新后的问题进行多头关注（将选项视为q，将更新后的问题视为k和v)。执行此步骤是为了模拟确认问题后对候选选项的理解
3. 对文章和步骤2中显示的更新选项执行多头关注（将文章视为q，并将更新的选项视为k和v)。这一步为了让模型在理解候选选项后，理解带有问题的文章和相应的候选选项

![image-20230227162248879](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227162248879.png)

经过预训练模型后，得到：
$$
[e_1,e_2,...,e_{k+l+m}]
$$


> HRCA使模型能够学习文章、问题和相应候选选项中每两个部分之间的关系，并将学习到的信息更新到下一个学习步骤

### HRCA

将得到的$E$分为$E^P,E^Q,E^O$

![image-20230227162549110](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227162549110.png)

### PQO Matrix

![image-20230227163021369](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227163021369.png)

作者发现HRCA中仅使用了三种attention关系，即问题和问题本身、候选选项和问题、文章和候选选项

上图中，红色的为HRCA中使用的，浅红色的为自注意力，灰色的为未使用的attention

### HRCA+

使用浅红色和灰色的关系，加强HRCA	

![image-20230227165008151](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230227165008151.png)

​	