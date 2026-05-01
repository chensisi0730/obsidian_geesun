---
title: DPO（直接偏好优化）
type: concept
tags: [概念, LLM强化学习, DPO, 教程]
aliases: [DPO, Direct Preference Optimization, 直接偏好优化]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：PPO、DPO、GRPO强化学习]]"
related_concepts: [PPO, GRPO, RLHF]
confidence: high
---

# DPO（直接偏好优化）

> 斯坦福提出的 LLM 对齐算法，通过直接优化偏好对的概率比来替代 PPO 中的显式奖励建模和强化学习，大幅降低训练成本。

## 是什么

DPO（Direct Preference Optimization）的核心洞察是：Reward Model 可以被隐式表示为策略本身。它不再需要训练独立的 Reward Model 和 Critic，而是直接利用 Chosen（好答案）和 Rejected（坏答案）的成对偏好数据来优化策略。

## 为什么重要

- **大幅降低硬件门槛**：从 PPO 的 4 模型降到 2 模型（仅 Actor + Ref Model），显存减半
- **训练稳定**：无需处理 RL 中的采样和优势估计等复杂环节
- **影响力广**：成为开源社区最常用的对齐算法之一

## 工作原理

### 损失函数

$\mathcal{L} = -\mathbb{E}[\log \sigma(\beta \cdot (\text{score}_{\text{chosen}} - \text{score}_{\text{rejected}}))]$

其中 $\text{score} = \log(\pi_\theta / \pi_{\text{ref}})$，衡量"当前策略相比参考策略有多倾向于生成这个答案"。

**逻辑拆解**：
1. 计算好/坏答案相对于 Ref Model 的概率比
2. 好答案概率比 - 坏答案概率比 = 差距
3. Sigmoid 将差距映射到 [0,1]
4. -log 将概率转化为 Loss

### 关键要求：同分布数据

DPO 数据最好由当前 Actor（或能力相近模型）生成自己的"错题集"。若数据来自过强的模型（如 GPT-5），小模型可能本来就不会生成那些"坏答案"，梯度几乎为零，训练无效。

## 与相邻概念的区别

- **PPO**：需要 Reward Model + Critic，硬件要求高；DPO 更轻量但不支持在线探索
- **GRPO**：需要 Reward Model（至少规则验证），但支持在线采样探索；DPO 纯离线
- **SFT**：仅模仿数据，不区分好坏；DPO 显式拉大好坏答案的差距

## 局限性

- **数据噪声敏感**：偏好标注错误会直接带偏模型
- **无探索能力**：纯离线学习，无法在线探索新策略
- **性能天花板**：在部分基准上效果不如 PPO/GRPO

## 相关

- [[PPO（近端策略优化）]]
- [[GRPO（组相对策略优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：PPO、DPO、GRPO强化学习]]
- [[Wiki 目录]]
