---
title: <Paper Reading> LMM-R1: Empowering 3B LMMs with Strong Reasoning Abilities Through
Two-Stage Rule-Based RL
tags: LLM RLHF
categories: PaperReading
date: 2025-08-11 14:26:00
index_img:
banner_img:
math: true
comment: true
hide: true
---

> https://arxiv.org/pdf/2503.07536

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811144811114.png)

两阶段训练

- **Foundational Reasoning Enhancement (FRE)**
  - Text-Only Reasoning Enhancement
    - Data Source
      - [DeepScaler-Preview-Dataset](https://huggingface.co/datasets/agentica-org/DeepScaleR-Preview-Dataset)  math 40k 
  - Multimodal Reasoning Enhancement
    - Data Source
      - [MathV360K](https://huggingface.co/datasets/Zhiqiang007/MathV360K) filtered prompts that have verifiable answers **65k**
- **Multimodal Generalization Training (MGT)**
  - Based on FRE-Text model
  - General Multimodal Reasoning Domain
    - Visual Reasoning-Centric Geometric Domain (Geo)
      - Extract 15k geometry problems from upper **65k MM reasoning Data**
    - Perception-Reasoning Balanced Domain (PerceReason)
      - VQA, DocQA, Math reasoning, Scientific reasoning
      - **Use total 65k MM reasoning data**
  - Agent-Related Reasoning Domain
    - Sokoban Planning Domain
    - Football Game Domain

### MultiModal RLVR  vs Text-only RLVR

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811151934346.png)

base model: Qwen2.5-VL-Instruct-3B

对比分别使用多模态数据RLVR的FRE-Multi和使用纯文本数据RLVR的FRE-Text，可以发现二者在多模态推理benchmark (MM Reasoning Dominated) 相比initial model都有比较明显的提升，但是二者之间差距不大

在各自的推理领域，则各自表现优秀（符合预期

在Text-only上的结果显示：多模态推理cold start损害了在文本推理上的能力；而在MM General上的结果则显示：纯文本cold start则能够泛化到多模态推理



MGT-PerceReason泛化性更好一点，也合理，因为任务多样性比较高



### Two-Stage Advantage

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811160746104.png)

Direct-RL-Geo指从Qwen2.5-VL直接RLVR训练，MGT-Geo是基于FRE-Text冷启动模型进行RLVR训练

从结果上看，两阶段的训练带来了比较明显的提升，make sense



### Further Analysis

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811161320421.png)

MM cold start的时候，response length竟然持续下降？



![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811161729068.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811161757094.png)

![](https://longls777.oss-cn-beijing.aliyuncs.com/img/image-20250811161820804.png)

观察到其他的MM相关训练，也出现了response length逐渐下降的趋势，感觉不是很合理，可能是用了PPO的原因？



