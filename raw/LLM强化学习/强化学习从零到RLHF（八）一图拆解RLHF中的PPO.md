---
title: "强化学习从零到RLHF（八）一图拆解RLHF中的PPO"
source: "https://zhuanlan.zhihu.com/p/635757674"
author:
  - "[[低级炼丹师]]"
published:
created: 2026-05-01
description: "众所周知，chatgpt的RLHF三阶段分别是： 1、SFT，也就是 Supervised Fine-Tuning，是 GPT-3 以及 ChatGPT 训练过程中的一个重要步骤。在这个步骤中，采用有监督的方式对预训练的语言模型进行微调。这又被称为行为…"
tags:
  - "clippings"
---
[收录于 · 强化学习基础系列](https://www.zhihu.com/column/c_1638958028161433600)

452 人赞同了该文章

目录

众所周知，chatgpt的RLHF三阶段分别是：

1、 [SFT](https://zhida.zhihu.com/search?content_id=229410969&content_type=Article&match_order=1&q=SFT&zhida_source=entity) ，也就是 [Supervised Fine-Tuning](https://zhida.zhihu.com/search?content_id=229410969&content_type=Article&match_order=1&q=Supervised+Fine-Tuning&zhida_source=entity) ，是 GPT-3 以及 ChatGPT 训练过程中的一个重要步骤。在这个步骤中，采用有监督的方式对预训练的语言模型进行微调。这又被称为行为克隆（Behavioral Cloning，简称BC），即直接使用专家的行为数据（例如，专家在特定情况下采取的动作）来训练模型。在行为克隆中，模型的目标是尽可能地复制专家的行为，而不是尝试优化某种奖励函数，所以它可能无法处理那些专家数据中没有覆盖到的情况，因为它完全依赖于专家的行为数据。

2、RM，奖励模型（ [Reward Model](https://zhida.zhihu.com/search?content_id=229410969&content_type=Article&match_order=1&q=Reward+Model&zhida_source=entity) ）是ChatGPT训练过程的第二阶段，它的目标是训练一个模型来适应人类的偏好。在这个阶段，首先从提示库中进行采样，并使用大型语言模型生成多个响应。然后，人工对这些响应进行排名，根据这些排名训练一个奖励模型。奖励模型的目标是学习人类对于不同响应的偏好，并将这些偏好编码到模型中。这样，奖励模型可以用来为模型生成的新响应打分，从而在后续的训练中引导模型生成更符合人类偏好的内容。这种方式不仅能帮助模型处理训练数据中未覆盖的情况，也能减少模型生成不确定或模棱两可的回答，从而打破行为克隆的影响。

3、 [PPO](https://zhida.zhihu.com/search?content_id=229410969&content_type=Article&match_order=1&q=PPO&zhida_source=entity) ，PPO（Proximal Policy Optimization）是一种强化学习算法，它通过引入奖励信号来调整模型的行为，使模型生成的内容更符合人类的偏好。具体来说，PPO通过最大化预期奖励来调整模型的策略，使模型在选择行为时更倾向于选择可以得到更高奖励的行为。在这个阶段中，我们首先使用在第一阶段训练的有监督微调模型和第二阶段训练的奖励模型来生成一个初始的策略。然后，我们使用PPO算法来调整这个策略，使模型在生成内容时更考虑人类的偏好。通过这个阶段的训练，模型不仅可以理解人类的语言，还可以理解人类的偏好，并生成更符合人类偏好的内容。

本节我们将介绍目前在整个RLHF流程中，他的第三阶段强化学习是怎么做的。

## PPO整体架构

### 宏观表述

PPO 是一种用于训练强化学习模型的算法。它可以用于调整语言模型，使得模型生成的结果更符合人类的偏好。具体来说，过程可以分为三个阶段：

1. **[Rollout and Evaluation](https://zhida.zhihu.com/search?content_id=229410969&content_type=Article&match_order=1&q=Rollout+and+Evaluation&zhida_source=entity) ：** 在这个阶段，我们从prompt库里抽样，使用语言模型生成response，然后使用奖励模型（Reward Model, RM）给出奖励得分。这个得分反映了生成的response的质量，比如它是否符合人类的偏好，是否符合任务的要求等。
2. **Make experience：** 在这个阶段，我们收集了一系列的“经验”，即模型的行为和对应的奖励。这些经验包括了模型生成的response以及对应的奖励得分。这些经验将被用于下一步的优化过程。
3. **Optimization：** 在这个阶段，我们使用收集到的经验来更新模型的参数。具体来说，我们使用PPO算法来调整模型的参数，使得模型生成的response的奖励得分能够增加。PPO算法的一个关键特性是它尝试保持模型的行为不会发生太大的改变，这有助于保证模型的稳定性。

通过这三个阶段的微调，我们可以使得语言模型的输出更符合我们的期望，例如更有创造性，更符合人类的偏好等。

### 微观拆解

为了将transformer的ppo过程更为清晰的表达出来，我将整个过程拆解为了10个步骤，如下图所示：

![](https://pica.zhimg.com/v2-4cb8160fb18b43eefc3bdaa9d09ff3b2_1440w.jpg)

十个步骤依次是：

1. **Rollout** ：根据策略（LM）生成轨迹（文本）。
2. **Evaluate** ：对生成的轨迹进行评估（RM）。
3. **Old Policy Sampling** ：从旧的策略（initial actor）中采样概率等信息。
4. **KL Penalty** ：计算当前策略和原始LM之间的KL散度，用作对策略改变过快的惩罚项。
5. **Generalized Advantage Estimation (GAE)** ：GAE是一种优势函数的估计方法，它结合了所有可能的n-step 进行advantage估计。
6. **New Policy Sampling** ：从新的策略中采样概率等信息。
7. **Critic Loss** ：Critic的目标是估计状态的价值函数，Critic loss就是价值函数预测值和实际回报之间的差距。
8. **Actor Loss** ：Actor的目标是优化策略，Actor loss就是基于优势函数的策略梯度。
9. **Entropy Loss** ：为了增加探索性，通常会添加一个基于策略熵的正则项，它鼓励策略保持多样性。
10. **Policykl** ：这是对策略迭代过程的一个度量，它度量新策略和旧策略之间的差距。

在图中，我已经把所有的模型和变量都写出来了，其中黄色代表模型，红色代表值，绿色是步骤。朝向步骤的箭头表面该步骤需要该值或者模型，朝向步骤外部的箭头表示该步骤的输出。

下面我们开始依次讲解这十个步骤：

## Rollout

**介绍**

在强化学习中，Rollout是指在给定的策略下模拟环境的过程。在PPO中，Rollout的过程对应于根据当前的语言模型（策略）生成文本（轨迹）。

这个过程依赖于在prompt库中抽取的一个batch的数据Batch Prompt和当前的语言模型LM。

语言模型接收一个prompt作为输入，并生成一个Response。这些Response就构成了我们的"轨迹"。

**输入输出**

**输入：** Batch Prompt、LM

**输出：** Prompt+Response

**代码**

在代码中，这个过程如下，我们使用huggingface的官方强化学习库TRL作为示例：

```
# 创建PPOTrainer实例。需要提供的参数包括：配置信息，模型，引用模型，分词器，数据集，数据整理器，优化器。
ppo_trainer = PPOTrainer(
    config,  # 配置信息
    model,  # 要训练的模型
    ref_model=None,  # 引用模型，通常是微调之前的预训练模型
    tokenizer=tokenizer,  # 用于文本编码和解码的分词器
    dataset=dataset,  # 用于训练的数据集
    data_collator=collator,  # 用于批量处理数据的数据整理器
    optimizer=optimizer,  # 用于模型优化的优化器
)

# 定义生成模型响应时的参数
generation_kwargs = {
    # "min_length": -1,  # 最小生成长度
    "top_k": 0.0,  # 在生成时，仅考虑前k个最可能的词
    "top_p": 1.0,  # 在生成时，仅考虑概率累计到某个阈值的词
    "do_sample": True,  # 是否进行抽样
    "pad_token_id": tokenizer.pad_token_id,  # 填充词的词ID
    "eos_token_id": 100_000,  # 句子结束词的词ID
}

# 使用定义好的参数生成模型的响应
response_tensors = ppo_trainer.generate(
        question_tensors,  # 输入的问题
        return_prompt=False,  # 是否返回提示
        length_sampler=output_length_sampler,  # 输出长度的抽样器
        **generation_kwargs,  # 生成参数
    )

# 将生成的响应从张量转换为文本，并存储在batch字典中
batch["response"] = tokenizer.batch_decode(response_tensors, skip_special_tokens=True)

# 这里将问题和相应的回答拼接起来，然后准备对拼接后的文本进行情感打分
texts = [q + r for q, r in zip(batch["query"], batch["response"])]
```

## Evaluate

**介绍**

Evaluate是在强化学习中对生成的轨迹（在我们的例子中就是文本）进行评估的步骤。在PPO中，这个评估过程由一个RM模型来完成，来为每一对Prompt+Response产生一个标量奖励值，这个值表示生成的轨迹的好坏，优化过程会试图最大化这个值。

**输入输出**

输入：Prompt+Response、RM

输出：Reward

**代码**

```
# 创建一个情感分析pipeline，也就是RM模型。需要提供的参数包括：模型类型，模型名称，设备映射，模型参数，分词器。
sentiment_pipe = pipeline(
    "sentiment-analysis",  # 模型类型，这里是情感分析
    model=reward_model_name,  # 模型名称
    device_map={"": current_device},  # 设备映射，这里将模型加载到当前设备
    model_kwargs={"load_in_8bit": True},  # 模型参数，这里是加载8位模型
    tokenizer=tokenizer,  # 用于文本编码和解码的分词器
)

# 使用情感分析对文本进行打分
pipe_outputs = sentiment_pipe(texts, **sent_kwargs)  # 'texts'是之前拼接好的问题和响应文本，'sent_kwargs'是情感分析的参数

# 计算奖励值。这里的奖励值是情感分析的得分减去一个基线值。
rewards = [torch.tensor(output[0]["score"] - script_args.reward_baseline) for output in pipe_outputs]
```

## Old Policy Sampling

**介绍**

Old Policy Sampling 这个步骤是make experience的过程，计算并存储旧策略的概率、价值等值，来为后面更新的过程服务。

**Old Logprobs**

这个步骤中，我们从“旧的”策略，即在这个batch数据中初始的LM（initial actor）中计算每个token在旧的策略下的概率Old Logprobs。

```
actor：从策略网络拷贝来的模拟整个智能体在环境中行动的网络，
详细看本系列第五章中Actor-Critic经典架构小节
```

这个步骤的重要性在于，我们在优化策略的时候，需要比较新旧策略下动作的概率，以此来更新我们的策略。因此，我们需要存储旧的策略的动作概率作为参考。

```
之所以要比较这个概率是为了算一个叫ratio的值，用这个值更新策略梯度，能限制更新率，
详细看本系列第七章中PPO损失函数小节
```

**Old Values**

Old Values的含义是旧策略中每个时间步（每个token的预测结果）的价值，这个值由critic网络进行预测，critic网络就是actor上加几个线性层能够给每个token预测一个值。需要这个值的原因是advantage的计算依赖于Old Values。

```
critic：专门用来预测actor轨迹每一步价值的网络，
详细看本系列第五章中Actor-Critic经典架构小节
```

**Ref Logprobs**

Ref Logprobs的含义是最最原始的LM对于每个时间步的概率预测，一般就是固定不变的gpt3，计算这个值的目的是限制actor的更新，防止其偏离原始gpt3太远，他的实现在下一个步骤中。

**输入输出**

**输入：** Ref\_model、Actor、Critic、Prompt+Response

**输出：** Ref Logprobs、Old Logprobs、Old Values

**代码**

```
all_logprobs, _, values, masks = self.batched_forward_pass(self.model, queries, responses, model_inputs)
ref_logprobs, _, _, _ = self.batched_forward_pass(self.ref_model, queries, responses, model_inputs)
```

batched\_forward\_pass方法的代码就是一个前向计算的过程。

## KL Penalty

**介绍**

在PPO 实现中，KL Penalty是在模型优化过程中添加的一个惩罚项，用于保证经过强化学习后的模型（新策略actor）不会过于偏离原始预训练模型（ref model）。

具体来说，首先使用微调过程中的模型（新策略actor）和预训练模型（ref model）来计算序列中每个词的对数概率。然后，我们计算两个模型输出之间的 Kullback-Leibler (KL) 散度，这是一种衡量两个概率分布差异的方法。该KL散度被用作一个额外的奖励信号，并作为优化过程中的惩罚项，用于确保微调后的模型生成的响应不会偏离太远于预训练模型。这样可以保证模型在微调的过程中不会丢失预训练模型学习到的有用的知识和模式。

在图中的KL Penalty步骤中，我们会在reward上增加这个kl惩罚项来实现这个过程。

**输入输出**

**输入：** Ref Logprobs、Old Logprobs、Reward

**输出：** Token Reward

**代码**

```
# 初始化两个列表来分别存储奖励和非得分奖励
rewards, non_score_rewards = [], []

# 使用 zip 函数并行遍历输入的得分、对数概率、参考模型的对数概率以及mask
for score, logprob, ref_logprob, mask in zip(scores, logprobs, ref_logprobs, masks):
    # 计算 KL 散度，即模型的对数概率与参考模型的对数概率之间的差值
    kl = logprob - ref_logprob

    # 计算非得分奖励，即 KL 散度乘以 KL 控制器值的负值
    non_score_reward = -self.kl_ctl.value * kl
    non_score_rewards.append(non_score_reward)

    # 复制非得分奖励为新的奖励
    reward = non_score_reward.clone()

    # 找到mask中最后一个非零元素的索引，这表示输入序列的实际长度
    last_non_masked_index = mask.nonzero()[-1]

    # 对于最后一个非mask部分的token，其奖励是偏好模型的得分加上 KL 散度
    reward[last_non_masked_index] += score

    # 将计算的奖励添加到奖励列表中
    rewards.append(reward)

# 返回包含所有奖励的张量以及包含所有非得分奖励的张量
return torch.stack(rewards), torch.stack(non_score_rewards)
```

## Generalized Advantage Estimation (GAE)

**介绍**

在强化学习中，我们会使用一个advantage的概念，来衡量每个时间步动作的含金量，他的本质含义是当前采样到的动作（生成的文本）的价值比平均的数学期望价值高的部分。

而在PPO中，我们一般使用GAE来进行advantage的计算。GAE是一种多步优势估计方法。它通过引入一个权衡参数λ，在单步TD误差和多步TD误差之间进行权衡，从而减小估计的方差，提高学习的稳定性。

GAE的主要目标是希望找到一种策略，使得从当前状态开始，采取该策略能够获得的未来奖励最大，GAE使用了一种名为TD误差的概念，这是一种预测未来奖励的方法。然后，GAE将这些TD误差组合成一个加权和，权重由一个衰减因子λ决定。当λ=0时，GAE就退化为普通的优势函数估计；当λ=1时，GAE就变成了一种名为"蒙特卡洛"的方法。总的来说，GAE的本质就是把优势估计为后续时间步TD误差的加权和，公式为：

的含义是估计的是在t时刻的优势，k的含义是k步一更新，所以累加的TD误差加权和是k步之内的。λ 是权衡参数，用于在偏差和方差之间进行权衡。γ 是折扣因子，用于衰减未来的回报。 是在时间步t+l的TD误差，计算公式为 ，这里的v是旧价值。在GAE步骤中，另外一个输出的值是returns，他的含义是每一步真实的后续累计折扣奖励，，这里使用advantage和value的和计算得到。

**输入输出**

**输入：** Token Reward、Old Values

**输出：** Advantages、Returns

**代码**

```
# 从后往前遍历整个生成的序列
for t in reversed(range(gen_len)):
    # 计算下一个状态的价值，如果当前状态已经是最后一个状态，则下一个状态的价值为0
    nextvalues = values[:, t + 1] if t < gen_len - 1 else 0.0

    # 计算 δ，它是奖励加上衰减后的下一个状态的价值，然后减去当前状态的价值
    delta = rewards[:, t] + self.config.gamma * nextvalues - values[:, t]

    # 使用 δ 更新 lastgaelam，这是 GAE 公式的一部分
    lastgaelam = delta + self.config.gamma * self.config.lam * lastgaelam

    # 将计算的优势值添加到优势值列表中
    advantages_reversed.append(lastgaelam)

# 将优势值列表反向并转换为张量
advantages = torch.stack(advantages_reversed[::-1]).transpose(0, 1)

# 计算回报值，它是优势值加上状态值
returns = advantages + values
```

## New Policy Sampling

**介绍**

New Policy Sampling是PPO算法中的一个关键步骤。在PPO中，策略优化的过程涉及到两个策略：一个是"旧的"策略，这是我们在开始每次优化迭代时使用的策略，另一个是"新的"策略，这是我们在优化过程中不断更新的策略。

New Policy Sampling就是在新的策略（更新后的actor）下对轨迹（文本）计算概率的过程。这个信息会被用于计算"Actor Loss"，也就是策略梯度的损失。在我们的步骤中，Old Logprobs是一次性一个batch的数据计算的，这是因为在一个batch中旧策略都是不变的；而New Logprobs是一个mini batch计算一次，这是因为新策略每个mini batch变一次。

此外这个步骤还会输出New Values和Logits分别用于critic loss和entropy loss的计算。

**输入输出**

**输入：** Ref\_model、Actor、Critic

**输出：** New Logprobs、New Values、Logits

**代码**

```
logprobs, logits, vpreds, _ = self.batched_forward_pass(
                        self.model, batch["queries"], batch["responses"], model_inputs, return_logits=True
                    )
```

## Critic Loss

**介绍**

在强化学习中，Critic是一个模型，其任务是估计状态的价值函数，也就是预测从当前状态开始，通过遵循某个策略，期望能得到的总回报。Critic的训练目标是最小化它的预测价值与实际回报之间的差距。这个差距被称为Critic Loss。

Critic Loss通常通过均方误差（Mean Squared Error, MSE）来计算。对于每一个状态，我们都有一个由Critic预测出的预期回报值V（s），以及一个真实的回报值G（returns）。Critic Loss就是这两个值之间差的平方。在一个批量的数据中，Critic Loss是所有状态的这个差的平方的平均值。公式如下：

其中E\[.\]表示期望值， 是Critic对状态s（这个时间步的token）的价值预测New Values，G是真实的回报值Returns。

通过最小化Critic Loss，Critic的预测能力会逐渐提升。因为Critic的预测结果会被用来估计每个行动的优势（Advantage），这个优势值又会被用来计算策略的更新（Actor Loss）。

**输入输出**

**输入：** New Values、Returns

**输出：** 梯度更新

**代码**

```
# 将价值函数的预测值裁剪到一个范围内
vpredclipped = clip_by_value(
            vpreds, values - self.config.cliprange_value, values + self.config.cliprange_value
        )

# 计算裁剪前和裁剪后的价值函数损失
vf_losses1 = (vpreds - returns) ** 2
vf_losses2 = (vpredclipped - returns) ** 2

# 最终的价值函数损失是裁剪前和裁剪后损失的最大值的平均值的一半
vf_loss = 0.5 * masked_mean(torch.max(vf_losses1, vf_losses2), mask)

# 计算裁剪操作实际发生的频率
vf_clipfrac = masked_mean(torch.gt(vf_losses2, vf_losses1).double(), mask)
```

代码的作用是将vpreds裁剪到一个范围内，这个范围是由values - self.config.cliprange\_value和values + self.config.cliprange\_value确定的，其中values是原始的价值函数预测值，self.config.cliprange\_value是裁剪的范围。目的是为了避免value的变化太快。

## Actor Loss

**介绍**

Actor Loss是基于策略梯度的损失函数，用于优化 Actor（策略）。

在深度强化学习中，我们通常有两个主要的组成部分：Actor 和 Critic。Actor 是策略，它决定文本会被怎么样生成。Critic 则是我们的价值函数估计器，它预测我们从当前状态开始，如果遵循当前的策略，能够得到的未来回报。

Actor Loss 是我们用来优化 Actor 的损失函数。它的计算通常基于优势函数，优势函数表示在给定的状态下采取某个行动比遵循当前策略的期望回报要好多少。

在 PPO 中，我们使用一种称为 Importance Sampling 的技术来计算 Actor Loss。我们比较了在旧策略和新策略下行动的概率（Old Logprobs，New Logprobs），然后将这个比值（也就是 Importance Sampling 的权重）与优势函数Advantages相乘，得到了对 Actor Loss 的一个估计。

PPO的Actor loss如下：

，

其中

是新旧策略的比率。 是优势函数，clip 是剪裁函数，它将限制在 范围内， 是一个超参数，通常设置为 0.1 或 0.2。

这个损失函数的目标是最大化策略的期望回报，同时限制新旧策略之间的差异。当新旧策略的比率超出 \[ 范围时，剪裁函数会限制其影响，防止策略更新过大。

**输入输出**

**输入：** Old Logprobs，New Logprobs、Advantages

**输出：** 梯度更新

**代码**

```
# 计算新旧策略下概率的比值
ratio = torch.exp(logprobs - old_logprobs)

# 计算未截断的策略梯度损失
pg_losses = -advantages * ratio

# 计算截断的策略梯度损失
pg_losses2 = -advantages * torch.clamp(ratio, 1.0 - self.config.cliprange, 1.0 + self.config.cliprange)

# 选择两者中较大的作为最终的策略梯度损失
pg_loss = masked_mean(torch.max(pg_losses, pg_losses2), mask)

# 计算因为截断导致策略梯度损失改变的比例
pg_clipfrac = masked_mean(torch.gt(pg_losses2, pg_losses).double(), mask)
```

## Entropy Loss

**介绍**

在信息论与概率统计中，熵(entropy)是表示随机变量不确定性的度量。在强化学习中，策略的熵可以表示为：

一个策略的熵越大，意味着这个策略选择各个动作的概率更加“平均”。在PPO中，为了提高算法的探索能力，我们一般在actor的loss中增加一项策略熵，并乘以一个系数entropy\_coef，使得在优化actor\_loss的同时，让策略的熵尽可能大。一般我们设置entropy\_coef=0.01。

设置这个是因为如果策略总是倾向于选择某些特定的文本生成方式，那么它可能会错过一些其他的文本生成方式带来的更好的奖励。通过增加熵的项，可以使策略在选择词汇时保持一定的随机性，从而有更多的机会去探索那些可能带来更好奖励的文本轨迹。

在我们的Entropy Loss步骤中，只需要Logits就能计算这个损失。

**输入输出**

**输入：** Logits

**输出：** 梯度更新

**代码**

```
entropy = -torch.sum(logits* torch.log(logits + 1e-9), dim=-1).mean()
```

## Policykl

**介绍**

在PPO中，KL散度被用作一种约束，以确保在优化过程中新策略不会偏离旧策略太远。这是为了防止过度优化，因为过度优化可能会导致策略性能的大幅下降。

我们希望在优化目标函数的同时，满足以下的KL散度约束：

其中，δ是一个预设的阈值， 和 分别是旧策略和新策略。

在代码中，每个mini batch都会进行early stop的判定，如果计算出的KL散度大于δ，那么就会停止这一轮的优化，以保证新策略不会偏离旧策略太远。

注意：KL表示KL散度， 表示在状态 下，由参数θ确定的策略产生的动作的概率分布。

**输入输出**

**输入：** Old Logprobs，New Logprobs

**输出：** 是否early stop

**代码**

```
# 计算旧策略和新策略之间的KL散度
policykl = masked_mean(old_logprobs - logprobs, mask) 
# old_logprobs 是旧策略下行为的概率的对数，logprobs 是新策略下的对数概率
# masked_mean 函数计算差异（old_logprobs - logprobs）的平均值，但只考虑mask中对应元素为True的元素

# 检查计算出的KL散度（policykl）是否大于目标KL散度（self.config.target_kl）的1.5倍
if policykl > 1.5 * self.config.target_kl: 
    self.optimizer.zero_grad()  # 如果实际的KL散度超过了目标的1.5倍，那么策略改变过多，这步的梯度也不更新了。
    early_stop = True  # 并设置early_stop标志为True，表示应提前停止优化，以防止策略从旧策略进一步偏离
```

## 资源推荐

最后的最后我再推荐一些实用的强化学习代码库吧

1. [Stable Baselines3](https://link.zhihu.com/?target=https%3A//github.com/DLR-RM/stable-baselines3) ：Stable Baselines3是一个提供各种深度强化学习算法实现的库，包括PPO，DQN，A2C等。所有的算法都采用了相同的API，使得在不同的算法之间进行切换变得非常方便。此外，它还提供了许多实用的功能，如模型保存/加载，向量化的环境，多种预处理选项等。
2. [Spinning Up in Deep RL](https://link.zhihu.com/?target=https%3A//github.com/openai/spinningup) ：这是OpenAI开发的一个库，其中包含了诸如PPO，A2C，SAC等算法的实现。该库的目标是提供一个清晰、简洁、易于理解的深度强化学习算法实现。
3. [Ray's Rllib](https://link.zhihu.com/?target=https%3A//github.com/ray-project/ray/tree/master/rllib) ：Rllib是一个高效的分布式强化学习库，支持PPO，A3C，DQN等算法。它可以很容易地与Ray的其他组件（如Tune和Serve）集成，从而提供了一个全面的深度学习和强化学习的解决方案。

这三个是纯强化学习的代码，如果是和RLHF相关的话，推荐：

1. [lvwerra/trl](https://link.zhihu.com/?target=https%3A//github.com/lvwerra/trl) ：这是一个基于Hugging Face的transformers库的强化学习库，可以用于训练Transformer语言模型。它使用ppo进行训练。
2. [CarperAI/trlx](https://link.zhihu.com/?target=https%3A//github.com/CarperAI/trlx) ：trlX是一个分布式训练框架，专门用于使用强化学习微调大型语言模型，可以使用提供的奖励函数或奖励标签的数据集进行训练。这个框架也支持微调Hugging Face模型。
3. [hpcaitech/ColossalAI Chat](https://link.zhihu.com/?target=https%3A//github.com/hpcaitech/ColossalAI/tree/main/applications/Chat) ： 同上
4. [microsoft/DeepSpeedExamples DeepSpeed-Chat](https://link.zhihu.com/?target=https%3A//github.com/microsoft/DeepSpeedExamples/tree/master/applications/DeepSpeed-Chat) ：同上

编辑于 2023-06-12 19:45・美国[ChatGPT](https://www.zhihu.com/topic/26691895)