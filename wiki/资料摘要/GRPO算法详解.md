---
title: 资料摘要：GRPO算法详解
type: source
tags: [LLM强化学习, GRPO, 深度, 教程]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "https://www.zhihu.com/question/10766825126/answer/88583863333"
confidence: medium
source_url: "https://www.zhihu.com/question/10766825126/answer/88583863333"
media: article
---

# 资料摘要：GRPO算法详解

> 知乎回答深入解析 DeepSeek 采用的 GRPO（Group Relative Policy Optimization）算法，从 RL 基础到 PPO 再到 GRPO 递进讲解，涵盖 DeepSeek-R1 的训练流程和数学推导。

## 核心要点

- **GRPO 的核心创新**：移除 PPO 中的 Value Model（Critic），改用同一问题的一组采样输出的平均奖励作为 baseline
- **四大步骤**：生成补全 → 计算优势值（组内归一化）→ 估计 KL 散度 → 计算损失
- **优势计算**：GRPO 使用组内奖励标准化替代 Critic 的价值估计：$A_i = (r_i - \text{mean}(r)) / \text{std}(r)$
- **KL 散度近似**：使用 Schulman et al. (2020) 的近似器 $\hat{D}_{\text{KL}} = \frac{p_{\text{ref}}}{p_\theta} - \log\frac{p_{\text{ref}}}{p_\theta} - 1$ 替代精确 KL 计算
- **DeepSeek-R1 三阶段**：冷启动 SFT → GRPO RL 训练 → 拒绝采样生成新 SFT 数据 → 全场景 RL

## 详细笔记

### RL 基础（资料中的符号体系）

- 策略函数 $\pi(a|s)$：状态 s 下选择动作 a 的条件概率
- 价值函数：$V_\pi(s)$（状态价值）、$Q_\pi(s,a)$（动作价值）
- 优势函数：$A_\pi(s,a) = Q_\pi(s,a) - V_\pi(s)$
- 策略梯度定理：$\nabla J(\theta) = \mathbb{E}[\nabla\log\pi_\theta(a|s) \cdot A(s,a)]$

### PPO 回顾

- 替代目标函数使用重要性采样比率 $r_t(\theta) = \pi_\theta(a_t|s_t) / \pi_{\theta\_\text{old}}(a_t|s_t)$
- Clip 机制限制 $r_t(\theta)$ 在 $[1-\epsilon, 1+\epsilon]$ 内
- PPO 在 LLM 场景需要 4 个模型：Policy、Reference、Reward、Value（Critic）

### GRPO 详解

**模型简化**：仅需 3 个模型（Policy、Reference、Reward），去掉 Value Model

**优势函数**：对同一问题 q 生成 G 个输出 $\{o_1, ..., o_G\}$
- 结果监督：每个 token 的优势相同 $A_i = (r_i - \text{mean}(r)) / \text{std}(r)$
- 过程监督：每个 token 的优势逐级计算

**损失函数**：
- 简化版（无 clip）：$\mathcal{L} = \frac{1}{G}\sum_i \frac{1}{|o_i|} \sum_t [A_i \cdot \frac{p_\theta}{p_{\theta\_\text{old}}} - \beta \cdot \hat{D}_{\text{KL}}]$
- 完整版：含 clipped surrogate objective

**RLHF 统一范式**（资料中提出的框架）：
- 三个组件：Data Source、Reward Function、Algorithm
- SFT / RFT / DPO / PPO / GRPO 均可统一到梯度系数框架下

### DeepSeek-R1 训练流程

1. DeepSeek-V3-Base → GRPO 训练 → DeepSeek-R1-Zero（不善表达但推理强）
2. 冷启动数据 SFT → GRPO → 拒绝采样生成 600k 推理数据
3. 合入通用 SFT 数据重训 → 全场景 RL → DeepSeek-R1

### GRPO vs PPO 对比

| 维度 | PPO | GRPO |
|------|-----|------|
| 模型数 | 4（含 Critic） | 3（无 Critic） |
| Baseline | Critic 网络估计 | 组内平均奖励 |
| 显存 | 高 | 低（砍半） |
| 优势估计 | GAE | 组内归一化 |
| 适用场景 | 通用 | 数学/代码等可验证任务尤佳 |

## 引用与数据

- 论文：[DeepSeek-R1](https://arxiv.org/abs/2501.12948)
- 论文：[DeepSeekMath](https://arxiv.org/abs/2402.03300)（GRPO 原始提出）
- 论文：[PPO](https://arxiv.org/abs/1707.06347)
- 代码：[TRL (HuggingFace)](https://github.com/huggingface/trl)

## 相关

- [[GRPO（组相对策略优化）]]
- [[PPO（近端策略优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：RLHF中的PPO拆解]]
- [[资料摘要：PPO、DPO、GRPO强化学习]]
- [[Wiki 目录]]
