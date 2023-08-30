---
title: Enriching Pre-trained Language Model with Entity Information for
Relation Classification
tags: RE
categories: Paper Reading Notes
date: 2023-08-30 13:30:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830132809565.png
banner_img: 
math: true
---

**标题：**《Enriching Pre-trained Language Model with Entity Information for Relation Classification》

**论文来源：** CIKM 2019

**原文链接： **https://arxiv.org/abs/1905.08284



## 概述

relation classification问题，在本文之前大多次采用卷积和RNN来做，本文首先提出BERT来解决



## Method

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830132809565.png)

- Entity 1前后加个$
- Entity 2 前后加个#
- 每一句前后加个[CLS]

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830133147303.png)

- Mean Pool 得到 Entity 的representation，注意是先activate，$W$和$b$共享参数

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830133429555.png)

- 同样的方法取句向量

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830133712545.png)

- 把它们三个拼接起来，经过一个MLP和Softmax层



## Experiments

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830135000823.png)

## Ablation Studies

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830135037102.png)

- **R-BERT-NO-SEP-NO-ENT**：去掉$ 和 # ，只用[CLS]分类
- **R-BERT-NO-SEP**：去掉$和#，依然拼接[cls]和两个entity的representation
- **R-BERT-NO-ENT**：去掉实体拼接，但是保留$和#，也是只用[CLS]分类
- **R-BERT**：原版

可以看到，只要标记了Entity的位置或者提供了Entity的表征，都能大幅提高分类的准确率