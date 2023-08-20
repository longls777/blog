---
title: LLMScience
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



## STEP1

使用官方的200条数据和收集到的额外数据，基于llama2-7b lora微调，然后推理



## STEP2

依然是LORA微调，但是修改模型结构，用最后一个token的embedding + 分类器做分类来训练和推理



## STEP3

改进分类器，可以套DUMA或者CDUMA，还有就是损失函数，因为评估指标是MAP@3



## 2023.8.19

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230820163347651.png)

使用全部39600条数据进行LoRA微调

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230819164406358.png)

参数如下：

```sh
CUDA_VISIBLE_DEVICES=0 python src/train_bash.py \
    --stage sft \
    --model_name_or_path /home/lsl/projects/Finetune_LLAMA/LLAMA_Model/llama-2-7b-chat-hf \
    --do_train \
    --dataset llmscience \
    --template llama2 \
    --finetuning_type lora \
    --output_dir sft_checkpoint \
    --overwrite_cache \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 4 \
    --lr_scheduler_type cosine \
    --logging_steps 10 \
    --save_steps 1000 \
    --learning_rate 5e-5 \
    --num_train_epochs 3.0 \
    --plot_loss \
    --fp16
```

推理结果

```json
{
    "predict_accuracy": 0.36,
    "predict_bleu-4": 12.353524999999998,
    "predict_rouge-1": 36.0,
    "predict_rouge-2": 0.0,
    "predict_rouge-l": 35.166667,
    "predict_runtime": 29.113,
    "predict_samples_per_second": 3.435,
    "predict_steps_per_second": 3.435
}
```

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/training_loss1.png)

效果一般

## 2023.8.20

推测可能是有些数据质量比较差，所以选用高质量的数据集（作者说的质量比较高）

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230820163801773.png)

10500条数据，epoch改成2

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/image-20230820163856986.png)
