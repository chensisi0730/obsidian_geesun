---
title: PPO（近端策略优化）
type: concept
tags: [概念, LLM强化学习, PPO, 深度, 教程]
aliases: [PPO, Proximal Policy Optimization, 近端策略优化]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：GRPO算法详解]]"
  - "[[资料摘要：RLHF中的PPO拆解]]"
  - "[[资料摘要：PPO、DPO、GRPO强化学习]]"
related_concepts: [GRPO, DPO, RLHF, 策略梯度]
confidence: high
---

# PPO（近端策略优化）

> OpenAI 提出的策略优化算法，通过 Clip 机制限制策略更新幅度，确保训练稳定性，是 RLHF 中应用最广泛的强化学习算法。

## 是什么

PPO（Proximal Policy Optimization）通过引入"裁剪"机制控制新旧策略的差异，避免更新过大导致性能崩溃。在 LLM 对齐场景中，PPO 通常需要同时维护 4 个模型：Actor（待训练策略）、Reference Model（冻结参考）、Reward Model（冻结打分）、Critic（逐 token 价值估计）。

## 为什么重要

- **RLHF 第三阶段核心**：ChatGPT 使用 PPO 将 Reward Model 的偏好信号转化为策略更新
- **稳定性保障**：Clip 机制使得强化学习在大规模语言模型上也能稳定训练
- **细粒度更新**：Critic 逐 token 打分的粒度使模型能精确定位和修正问题行为

## 工作原理

### 十步流程

1. **Rollout**：从 prompt 库采样，Actor 生成 response
2. **Evaluate**：Reward Model 给整句打分
3. **Old Policy Sampling**：记录旧策略 LogProbs 和 Critic Values
4. **KL Penalty**：KL 散度作为每 token 奖励惩罚项，防止偏离 Ref Model
5. **GAE（广义优势估计）**：多步 TD 误差加权和计算 Advantage
6. **New Policy Sampling**：新策略下计算新 LogProbs
7. **Critic Loss**：MSE 训练 Critic 更准确
8. **Actor Loss**：$-\mathbb{E}[\min(r\cdot A,\ \text{clip}(r, 1-\epsilon, 1+\epsilon)\cdot A)]$
9. **Entropy Loss**：保持策略探索性
10. **Policy KL**：若 KL 超标则 early stop

### 核心公式

**优势估计**（GAE）：$A_t = \sum_{l=0}^\infty (\gamma\lambda)^l \delta_{t+l}$，其中 $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$

**Actor Loss（裁剪版）**：$\mathcal{L} = -\mathbb{E}[\min(r_t(\theta)A_t,\ \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)A_t)]$

## Critic vs Reward Model 的分工

- **Reward Model**：整句读完才给最终分（标量）。像期末考试的卷面总分
- **Critic**：每生成一个 token 都预测预期回报。像课堂上的随堂测验，能精确定位"从哪一步开始出问题"

## 与相邻概念的区别

- **GRPO**：去掉 Critic，用组内平均替代价值函数。更省显存，适合数学/代码等可验证任务
- **DPO**：去掉 Reward Model 和 Critic，直接优化偏好对概率。更轻量，但对数据质量要求极高
- **TRPO**：PPO 的前身，使用 KL 约束而非 Clip，计算更复杂

## 相关

- [[GRPO（组相对策略优化）]]
- [[DPO（直接偏好优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：GRPO算法详解]]
- [[资料摘要：RLHF中的PPO拆解]]
- [[资料摘要：PPO、DPO、GRPO强化学习]]
- [[Wiki 目录]]
