---
title: 资料摘要：RLHF中的PPO拆解
type: source
tags: [LLM强化学习, PPO, RLHF, 深度, 教程]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "https://zhuanlan.zhihu.com/p/635757674"
confidence: medium
source_url: "https://zhuanlan.zhihu.com/p/635757674"
media: article
---

# 资料摘要：RLHF中的PPO拆解

> 知乎文章《强化学习从零到RLHF（八）一图拆解RLHF中的PPO》，将 ChatGPT RLHF 第三阶段中 PPO 的训练流程拆解为 10 个步骤，配合 TRL 代码逐段讲解。

## 核心要点

- **RLHF 三阶段**：SFT（行为克隆）→ RM（奖励模型训练）→ PPO（强化学习对齐）
- **PPO 十步拆解**：Rollout → Evaluate → Old Policy Sampling → KL Penalty → GAE → New Policy Sampling → Critic Loss → Actor Loss → Entropy Loss → Policy KL
- **四个模型**：Actor（待训练）、Ref Model（参考）、Reward Model（打分）、Critic（逐 token 价值估计）
- **核心代码示例**：基于 HuggingFace TRL 库的完整 PPO 训练流程

## 详细笔记

### 十步流程

1. **Rollout**：从 prompt 库采样，LM 生成 response（轨迹）
2. **Evaluate**：Reward Model 对完整 response 给出标量奖励
3. **Old Policy Sampling**：计算旧策略下的 LogProbs、Values（Critic 输出）、Ref LogProbs
4. **KL Penalty**：将 KL 散度加入每 token 奖励中（$r_t = -\beta \cdot \text{KL} + \text{Reward}_{\text{末尾}}$）
5. **GAE（广义优势估计）**：多步 TD 误差加权和，平衡偏差和方差
6. **New Policy Sampling**：新策略下计算新的 LogProbs、Values、Logits
7. **Critic Loss**：MSE 最小化 Critic 预测值与实际回报的差距，含裁剪
8. **Actor Loss**：$-\mathbb{E}[\min(r\cdot A, \text{clip}(r)\cdot A)]$
9. **Entropy Loss**：鼓励策略多样性，防止过早收敛
10. **Policy KL**：early stop 判定，若新旧策略 KL > 1.5 × target_kl 则停止

### 关键公式

**GAE**：
- $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$
- $A_t^{\text{GAE}} = \sum_{l=0}^{k} (\gamma\lambda)^l \delta_{t+l}$

**Actor Loss**：
- $r_t(\theta) = \exp(\log\pi_\theta - \log\pi_{\theta\_\text{old}})$
- $\mathcal{L} = -\mathbb{E}[\min(r_t\cdot A_t, \text{clip}(r_t, 1-\epsilon, 1+\epsilon)\cdot A_t)]$

## 引用与数据

- 论文：[PPO](https://arxiv.org/abs/1707.06347)
- 论文：[GAE](https://arxiv.org/abs/1506.02438)
- 库：[TRL (HuggingFace)](https://github.com/huggingface/trl)
- 库：[Stable Baselines3](https://github.com/DLR-RM/stable-baselines3)
- 库：[TRLX (CarperAI)](https://github.com/CarperAI/trlx)

## 相关

- [[PPO（近端策略优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：GRPO算法详解]]
- [[资料摘要：PPO、DPO、GRPO强化学习]]
- [[Wiki 目录]]
