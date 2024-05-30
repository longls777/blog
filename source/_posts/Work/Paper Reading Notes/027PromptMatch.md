---
title: Label Verbalization and Entailment for Effective Zero- and Few-Shot Relation ExtractionLearning
tags: RE
categories: Paper Reading Notes
date: 2023-08-30 14:33:00
index_img: https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830150002407.png
banner_img: 
math: true
---

**标题：**《Label Verbalization and Entailment for Effective Zero- and Few-Shot Relation ExtractionLearning》

**论文来源：** EMNLP  2021

**原文链接： **  https://aclanthology.org/2021.emnlp-main.92/

**源码：** https://github.com/osainz59/Ask2Transformers



## 概述

把关系抽取任务当作entailment任务处理



## Method

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830150002407.png)

- 首先根据entity生成假设
- 通过NLI模型推断sentence是否包含假设
- 根据上一步的概率分布，返回概率最大的relation，这些relation中存在NO-RELATION，也就是有一种relation是没关系

### Zero-shot relation extraction

sentence（包含两个实体）作为前提premise，关系描述作为假设hypothesis，任务是判断premise是否蕴含hypothesis



#### Verbalizing relations as hypothesis.

- 通过一些template自动生成
  - 比如 relation：PER:DATE_OF_BIRTH
  - template：{subj}’s birthday is on {obj}.

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830152755510.png)

- 设定一个函数判断实体类型是否符合template的规定，例如：

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830152847039.png)

好像这个数据集的第一个entity都是person？ 参数类型只限定了第二个entity



#### NLI for inferring relations

- 输入一个sentence $x$，包含两个entity $x_{e1}$，$x_{e2}$，NLI得到可能的关系集合$R$中概率最大的$\hat{r}$

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830154426833.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830154706713.png)

- 如果entity不符合template要求，概率置为0

- 虽然蕴涵模型返回蕴涵、矛盾和中性的概率，但 $P_{NLI}$ 只是利用了蕴涵概率



#### Detection of no-relation

两种检测办法：

- **template-based detection**：增加一种无关系的template
  - NO-RELATION：{subj} and {obj} are not related
- **threshold-based detection**：给$P_r$定一个阈值，都没超过就是NO-RELATION，如果没验证集数据就设为0.5



### Few-Shot relation extraction

生成labelled entailment pairs来fine-tuning NLI model

- 对于每个positive的instance，生成至少一个正确的entailment instance（使用正确的template）
- 对于每个positive的instance，生成一个中性的entailment instance（从不符合关系的template中随机选一个）
- 对于每个negative的instance，生成一个矛盾的entailment instance（从剩下的template中随机选一个）

如果template用于NO-RELATION情况，执行以下操作：

- 首先，对于每个NO-RELATION示例，使用NO-RELATION template生成一个蕴含示例
-  然后，对于每个positive relation示例，使用NO-RELATION template生成一个矛盾示例



## Experiments

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830160202483.png)

数据集：TACRED，一共包含42个关系，其中有NO-RELATION，本文对其split以生成不同的场景

- Zero-Shot：
  - 没有验证集 (0% split)
  - 2 examples per relation (1% split)
  - 验证集不能用来训练，但可以用来调整超参数，也就是那个阈值
- Few-Shot：
  - 4 examples per relation (1% split)
  - 16 examples per relation (5% split)
  - 32 examples per relation (10% split)
  - 使用相同的比例reduce 验证集
- Full Training：使用全部数据，包括全部的训练集和验证集
- Data Augmentation：在无标签数据集上使用本文的方法（模板生成）生成的数据是否可以用来训练监督RE model
  - TACRED中75%的数据被设置为无标签，称为银数据
  - 剩下的25%用作split，称为金数据
  - 产生两种场景：
    - Zero-Shot (0% split)：使用本文的方法注释银数据，然后微调RE model
    - Few-Shot (1% / 5% / 10% split)：首先用金数据微调，然后注释银数据，最后在两个数据集上微调RE model



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830161847447.png)



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830162004979.png)

当用于设定NO-RELATION阈值的验证集大小增长时，model的F1也逐渐增长



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830162124205.png)

取三个种子运行的均值和标准差，用于对比的模型结果是自己实现的



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830162337493.png)

全数据集训练的结果



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830162448897.png)





![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230830162635512.png)

两个任务：

- 二元任务，即区分positive关系和无关系的情况
- 多类问题，即在positive案例中确定具体的关系类型

Zero-Shot NLI在区分positive方面非常有效，但在检测NO-RELATION时仍然有改进的空间