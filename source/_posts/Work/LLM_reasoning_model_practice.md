---
title: LLM Reasoning Model - Math 训练记录
tags: LLM
categories: Work
date: 2025-06-18 14:35:00
index_img: 
banner_img: 
math: true
comment: true
---



> Infra: https://github.com/volcengine/verl
>
> Train Data: https://huggingface.co/datasets/BytedTsinghua-SIA/DAPO-Math-17k
>
> Eval Data: https://huggingface.co/datasets/HuggingFaceH4/aime_2024
>
> https://huggingface.co/datasets/HuggingFaceH4/MATH-500
>
> Base Model: https://huggingface.co/Qwen/Qwen2.5-7B
>
> https://huggingface.co/Qwen/Qwen2.5-7B-Instruct
>
> Prompt Template:
>
> ```python
> template = f"""Solve the following math problem step by step. The last line of your response should be of the form Answer: $Answer (without quotes) where $Answer is the answer to the problem.\n\n{question}\n\nRemember to put your answer on its own line after \"Answer:\".""""
> ```
>

## 训练配置

首先对于Base模型，我们的配置如下：

```bash
python3 -m verl.trainer.main_ppo --config-path=config \
    --config-name='ppo_megatron_trainer.yaml'\
    algorithm.adv_estimator=grpo \
    data.train_files="$train_files" \
    data.val_files="$test_files" \
    data.train_batch_size=32 \
    data.max_prompt_length=1024 \
    data.max_response_length=15360 \
    data.filter_overlong_prompts=True \
    data.truncation='error' \
    actor_rollout_ref.model.path=Qwen/Qwen2.5-7B \
    actor_rollout_ref.actor.optim.lr=1e-6 \
    actor_rollout_ref.actor.ppo_mini_batch_size=16 \
    actor_rollout_ref.actor.ppo_micro_batch_size_per_gpu=1 \
    actor_rollout_ref.actor.megatron.pipeline_model_parallel_size=2 \
    actor_rollout_ref.actor.megatron.tensor_model_parallel_size=2 \
    actor_rollout_ref.actor.use_kl_loss=True \
    actor_rollout_ref.actor.kl_loss_coef=0.001 \
    actor_rollout_ref.actor.kl_loss_type=low_var_kl \
    actor_rollout_ref.actor.entropy_coeff=0 \
    actor_rollout_ref.model.enable_gradient_checkpointing=True \
    actor_rollout_ref.rollout.log_prob_micro_batch_size_per_gpu=1 \
    actor_rollout_ref.rollout.tensor_model_parallel_size=2 \
    actor_rollout_ref.rollout.name=vllm \
    actor_rollout_ref.rollout.gpu_memory_utilization=0.45 \
    actor_rollout_ref.rollout.n=3 \
    actor_rollout_ref.ref.log_prob_micro_batch_size_per_gpu=1 \
    actor_rollout_ref.ref.megatron.pipeline_model_parallel_size=2 \
    actor_rollout_ref.ref.megatron.tensor_model_parallel_size=2 \
    algorithm.use_kl_in_reward=False \
    trainer.critic_warmup=0 \
    trainer.logger=['console','wandb'] \
    trainer.project_name='verl_grpo_dapo17k' \
    trainer.experiment_name='qwen2_7b_base_use_kl_loss_megatron' \
    trainer.n_gpus_per_node=8 \
    trainer.nnodes=1 \
    trainer.save_freq=100 \
    trainer.test_freq=5 \
    trainer.total_epochs=15 $@
```

其中`lr`为`1e-6`，来自[verl官方的默认实现](https://github.com/volcengine/verl/blob/8b33abd84f360473f05e5a750aef36e974340cce/examples/grpo_trainer/run_qwen2-7b.sh#L14C5-L14C44)，没有`warm_up`以及`decay`，根据Qwen2.5 Technical Report，Qwen2.5-Base模型训练时的LR没有貌似具体说明(pretrain)，但是SFT从`7e-6`开始decay到`7e-7`，offline RL(DPO)阶段的LR为`7e-7`，所以看起来`1e-6`也比较合理。

> [Qwen2.5 Technical Report](https://arxiv.org/pdf/2412.15115)

以下这些参数经过了调整，目前的配置可以在单机8卡(80G VRAM per GPU)上运行。

| param                             | offical | mine  |
| --------------------------------- | ------- | ----- |
| train_batch_size                  | 1024    | 32    |
| ppo_mini_batch_size               | 256     | 16    |
| ppo_micro_batch_size_per_gpu      | 40      | 1     |
| max_prompt_length                 | 512     | 1024  |
| max_response_length               | 1024    | 15360 |
| rollout.n                         | 5       | 3     |
| rollout.gpu_memory_utilization    | 0.6     | 0.45  |
| log_prob_micro_batch_size_per_gpu | 40      | 1     |

显存占用应该来自三部分：**训练时激活/KV Cache 占用 + vLLM KV + 模型权重/优化器**

由于使用`flash-attn`，**训练时激活显存**应该和**KV Cache**一样，均为线性增长。官方配置中启动的模型上下文窗口应该是 2k (512 + 1024 ceil)，而我修改后应该是16k，相当于8x增长，很容易OOM，因此我降低了`ppo_micro_batch_size_per_gpu` from 40 to 1, 相当于40x的降低，同时减少`rollout.gpu_memory_utilization`，给`Actor`分配更多的显存。

`ppo_mini_batch_size`在这里只影响收敛速度和稳定性，其实可以不用改。(as well as rollout.n)



此外，遵循verl官方的设置：

```bash
    actor_rollout_ref.actor.use_kl_loss=True \
    actor_rollout_ref.actor.kl_loss_coef=0.001 \
    actor_rollout_ref.actor.kl_loss_type=low_var_kl \
```

[/verl/verl/workers/actor/megatron_actor.py](https://github.com/volcengine/verl/blob/main/verl/workers/actor/megatron_actor.py#L393C16-L401C72)

```python
if self.config.use_kl_loss:
    ref_log_prob = data["ref_log_prob"]
    # compute kl loss
    kld = kl_penalty(logprob=log_prob, ref_logprob=ref_log_prob, kl_penalty=self.config.kl_loss_type)
    kl_loss = agg_loss(loss_mat=kld, loss_mask=response_mask, loss_agg_mode=self.config.loss_agg_mode)

    policy_loss = policy_loss + kl_loss * self.config.kl_loss_coef
    metrics["actor/kl_loss"] = kl_loss.detach().item()
    metrics["actor/kl_coef"] = self.config.kl_loss_coef
```

这里使用`low_var_kl`，也就是`k3`估计：

```python
kl = ref_logprob - logprob
ratio = torch.exp(kl)
kld = (ratio - kl - 1).contiguous()
return torch.clamp(kld, min=-10, max=10)
```

最终的`policy_loss` 要加上`k3`估计的` kl_loss`，`kl_loss_coef`为系数，这里使用`0.001`

## Performance

![Qwen-7B-Base response_length/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_43_51.png)

![Qwen-7B-Base MATH-500 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_46_18.png)

![Qwen-7B-Base AIME_2024 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_45_50.png)



- response length持续增加
- MATH-500 acc在前25个steps快速rise到ceiling，后续波动
- AIME-2024 持续波动

![Qwen-7B-Base critic/rewards/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_58_49.png)

![Qwen-7B-Base actor/kl_loss](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_56_49.png)

![Qwen-7B-Base training/rollout_probs_diff_mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2019_59_23.png)

随训练step增加：

- mean reward 增加：符合预期
- kl_loss增加：policy model 和 ref model 差异增大
- `rollout_probs_diff_mean`降低：policy model收敛

> training/rollout_probs_diff_mean:
> $$
> \frac1{N_\text{tok}}\sum_{t=1}^{N_\text{tok}}
> 
> \bigl|\,p_{\text{actor}}^{(t)}-p_{\text{rollout}}^{(t)}\bigr|,
> $$
> ——当前 Actor 对batch内 **每个 token** 的概率 $p_\text{actor}$ 与 **生成当时缓存的概率** $p_\text{rollout}$ 之间的 **绝对差** 的平均值

## w/ vs w/o kl_loss on Base Model

设置`actor.kl_loss_coef=0 & 0.001`，紫色为`w/ kl_loss`，蓝绿色为`w/o kl_loss`

![Qwen-7B-Base w/ vs w/o kl_loss response_length/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_05_35.png)

![Qwen-7B-Base w/ vs w/o kl_loss MATH-500 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_06_44.png)



![Qwen-7B-Base w/ vs w/o AIME_2024 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_06_30.png)

![Qwen-7B-Base w/ vs w/o kl_loss critic/rewards/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_05_19.png)

差异不大，推测是`actor.kl_loss_coef=0.001`太小了



## w/ vs w/o kl_loss  Instruct Model

设置`actor.kl_loss_coef=0 & 0.01`，橙色为`w/ kl_loss`，绿色为`w/o kl_loss`

![Qwen-2.5-7B-Instruct w/ vs w/o kl_loss response_length/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_12_56.png)

![Qwen-2.5-7B-Instruct w/ vs w/o kl_loss MATH-500 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_13_06.png)

![Qwen-2.5-7B-Instruct w/ vs w/o kl_loss AIME-2024 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_13_14.png)

![Qwen-2.5-7B-Instruct w/ vs w/o kl_loss critic/rewards/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_13_31.png)

![Qwen-2.5-7B-Instruct w/ vs w/o kl_loss training/rllout_probs_diff_mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_16_58.png)

- `mean response_length`差别不是很大，训练后期`w/o kl_loss`出现超长response的情况更多
- `MATH-500`上 `w/o kl_loss`上限更高，但是更不稳定
- `AIME-2024 `差别不大
- `critic/reward/mean` 差别不大
- `training/rollout_probs_diff_mean`上，`w/o kl_loss` 150 steps后更低

## Base Model vs Instruct Model (both w/o kl_loss)

![Qwen-2.5-7B-Base vs Qwen-2.5-7B-Instruct response_length/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_23_54.png)

![Qwen-2.5-7B-Base vs Qwen-2.5-7B-Instruct training/rollout_probs_diff_mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_24_08.png)

![Qwen-2.5-7B-Base vs Qwen-2.5-7B-Instruct AIME_2024 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_24_18.png)

![Qwen-2.5-7B-Base vs Qwen-2.5-7B-Instruct MATH-500 acc mean@1](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_24_27.png)

![Qwen-2.5-7B-Base vs Qwen-2.5-7B-Instruct critic/rewards/mean](https://longls777.oss-cn-beijing.aliyuncs.com/img/W%26B%20Chart%202025_6_30%2020_22_53.png)

- Instruct模型最开始时reward更高
- 其余差别不是很大（可能是评测集展现不出差别，按理来说base模型多样性更高，上限更高）

