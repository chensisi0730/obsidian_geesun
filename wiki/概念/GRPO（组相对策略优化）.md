---
title: GRPO（组相对策略优化）
type: concept
tags: [概念, LLM强化学习, GRPO, 深度]
aliases: [GRPO, Group Relative Policy Optimization, 组相对策略优化]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：GRPO算法详解]]"
  - "[[资料摘要：PPO、DPO、GRPO强化学习]]"
related_concepts: [PPO, DPO, RLHF, 策略梯度]
confidence: high
---

# GRPO（组相对策略优化）

> DeepSeek 提出的策略优化算法，通过移除 Critic 模型并用组内奖励标准化替代，大幅降低显存占用，是 DeepSeek-R1 和 DeepSeek-Math 的核心训练算法。

## 是什么

GRPO（Group Relative Policy Optimization）由 DeepSeek 在 DeepSeek-Math 论文中首次提出，后用于 DeepSeek-R1 训练。它的核心思路是：**不要绝对裁判，要相对排名**。每步训练对同一问题生成 G 个回答，用组内平均奖励作为 baseline 计算优势，从而省去需要训练的 Critic 模型。

## 为什么重要

- **显著降低显存**：从 PPO 的 4 模型减少到 3 模型（去掉 Critic），显存占用约减半
- **驱动 DeepSeek-R1**：使得 671B 的巨型模型的 RL 训练成为可能
- **基线更真实**：组内平均分是"当下真实水平"，比 Critic 网络的估计更可靠
- **适合推理任务**：数学/代码等有标准答案的任务，GRPO 能敏锐捕捉正确的"开窍"样本

## 工作原理

### 四步流程

1. **生成补全**：对每个 prompt 采样 G 个输出 $\{o_1, ..., o_G\}$
2. **计算奖励**：
   - 理科（Math/Code）：规则验证（对=1分，错=0分），快准狠
   - 文科（Text）：额外 Reward Model 打分
3. **组内优势归一化**：$A_i = (r_i - \text{mean}(r)) / \text{std}(r)$
4. **优化目标**：最大化优势 + KL 散度约束

### 损失函数

$\mathcal{L} = \frac{1}{G}\sum_{i=1}^G \frac{1}{|o_i|}\sum_{t=1}^{|o_i|} \left[ \min\left(\frac{\pi_\theta}{\pi_{\theta\_\text{old}}} A_i,\ \text{clip}(\frac{\pi_\theta}{\pi_{\theta\_\text{old}}}, 1-\epsilon, 1+\epsilon) A_i\right) - \beta \cdot D_{\text{KL}}(\pi_\theta \| \pi_{\text{ref}}) \right]$

**三部分拆解**：
- **PPO 遗产（Clip）**：限制更新幅度，小步快跑
- **DPO 遗产（KL 散度）**：约束不偏离参考模型
- **GRPO 创新（组平均）**：去掉 Critic，组内比较

### DeepSeek V3.2 的改进

- **无偏 KL 估计**：加重要性采样比率，防梯度爆炸
- **Off-Policy 序列掩码**：忽略坏样本中偏移太大的序列
- **Keep Routing**：MoE 训练时锁死路由路径
- **Keep Sampling Mask**：保采样空间一致性

## 与相邻概念的区别

- **PPO**：有 Critic，需要 4 模型，显存贵但数据效率高。GRPO 是其"省显存版"
- **DPO**：不需要 Reward Model，但需要成对偏好数据。GRPO 用组内比较替代成对比较

## 局限性

- **采样成本高**：每步需生成 G 个（8-16）长回答，推理时间拉长
- **依赖冷启动**：SFT 模型太差时 G 个回答全错则 GRPO 空转
- **奖励设计难**：主观任务仍需额外 Reward Model

## 相关

- [[PPO（近端策略优化）]]
- [[DPO（直接偏好优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：GRPO算法详解]]
- [[资料摘要：PPO、DPO、GRPO强化学习]]
- [[Wiki 目录]]
