---
title: LLMScienc
tags: 
categories: Practice
date: 2023-08-19 14:53:00
index_img: 
banner_img: 
hide: true
---



> LLM PEFT https://github.com/hiyouga/LLaMA-Efficient-Tuning
>
> Kaggle competition https://www.kaggle.com/competitions/kaggle-llm-science-exam/overview



## Data

| Dataset     | train | test |
| ----------- | ----- | ---- |
| LLMscience  | 200   | 200  |
| 6.5k GPT3.5 | 6500  |      |
| 15k GPT3.5  | 15000 |      |
| 6k          | 6000  |      |
| **ALL**     |       |      |

> LLMscience https://www.kaggle.com/competitions/kaggle-llm-science-exam/data
>
> 6.5k https://www.kaggle.com/datasets/radek1/additional-train-data-for-llm-science-exam
>
> 15k https://www.kaggle.com/datasets/radek1/15k-high-quality-examples 
>
> 6k https://www.kaggle.com/datasets/radek1/sci-or-not-sci-hypthesis-testing-pack



39600

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230819164406358.png)



## STEP1

使用官方的200条数据和收集到的额外数据，基于llama2-7b lora微调，然后推理



## STEP2

依然是LORA微调，但是修改模型结构，用最后一个token的embedding + 分类器做分类来训练和推理



## STEP3

改进分类器，可以套DUMA或者CDUMA，还有就是损失函数，因为评估指标是MAP@3

