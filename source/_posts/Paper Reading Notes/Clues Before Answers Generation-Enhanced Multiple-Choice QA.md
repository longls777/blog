---
title: Clues Before Answers  Generation-Enhanced Multiple-Choice QA
tags: MCQA
categories: Paper Reading Notes
date: 2022-09-27 13:52:00
index_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927142410395.png
banner_img: http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927142410395.png
math: true
---

**标题：**《Clues Before Answers: Generation-Enhanced Multiple-Choice QA》

**论文来源：** NAACL 2022

**原文链接：** https://arxiv.org/pdf/2205.00274v1.pdf

**源码：** https://github.com/nju-websoft/genmc



## 概述

目前MCQA的主要趋势是使用encoder-decoder的结构，然而，encoder-decoder结构的目标是生成文本，用此结构来做分类任务（MCQA）会使得预训练结构的知识得不到充分利用，

此文为了充分利用encoder-decoder预训练模型的潜在知识，提出了一种名为GenMC的MCQA模型，从问题中生成线索，利用线索来辅助选择答案。

![image-20220927140124292](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927140124292.png)

随着text2text模式在nlp领域的快速应用，有人使用此结构通过大量语料的训练来达到了MCQA任务的SOTA，训练方法是将问题和所有选项作为输入，将正确选项作为生成目标

然而，使用生成任务预训练，再用分类任务做微调，这会导致不能充分利用预训练的知识。

出了text2text模式之外，还有encoder-only模式

## 模型设计

![image-20220927142410395](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20220927142410395.png)