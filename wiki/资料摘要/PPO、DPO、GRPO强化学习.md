---
title: 资料摘要：PPO、DPO、GRPO强化学习
type: source
tags: [LLM强化学习, PPO, DPO, GRPO, 综述, 教程]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "https://zhuanlan.zhihu.com/p/1984387073625593089"
confidence: medium
source_url: "https://zhuanlan.zhihu.com/p/1984387073625593089"
media: article
---

# 资料摘要：PPO、DPO、GRPO强化学习

> 知乎专栏文章用通俗语言和类比全面对比 PPO、DPO、GRPO 三种 LLM 强化学习对齐算法，涵盖原理、流程图、优缺点及学界最新进展。

## 核心要点

- **SFT 的局限**：SFT 只是模仿训练数据分布，无法区分多个"正确答案"的优劣；RL 的作用是注入偏好，对概率分布进行挤压和拉伸
- **PPO = 不计成本的特训班**：需要 4 个模型（Actor、Ref Model、Reward Model、Critic），显存占用极高
- **DPO = 穷哥们的福音**：去掉 Reward Model 和 Critic，只需 2 个模型，但要求高质量偏好对数据（Chosen vs Rejected）
- **GRPO = 内部赛马机制**：去掉 Critic，改为组内相对排名（组内奖励标准化），只需 3 个模型
- **CRITIC 的作用**：逐 token 打分定位"病情时刻"，与 Reward Model（整句打分）互补

## 详细笔记

### PPO 详解

**四个模型配合**：
- **Actor**（待训练）：生成回答
- **Ref Model**（冻结）：KL 散度约束，防止偏离原始模型
- **Reward Model**（冻结）：整句打分（标量）
- **Critic**（待训练）：逐 token 打分，输出 [Batch, SeqLen, 1]

**Critic 存在的意义**（常见疑问）：
- RM 只给整句最终分，但不知道好/坏出现在哪个 token
- Critic 逐 token 预测，与 RM 最终分结合算出每步 Advantage，精确指导更新

**核心公式**（Clip 机制）：
- $r_t = \pi_\theta / \pi_{\theta\_\text{old}}$（比率）
- clip 将 $r_t$ 限制在 $[1-\epsilon, 1+\epsilon]$，防止一步更新过大

### DPO 详解

**核心简化**：
- 去掉 Reward Model 和 Critic，仅需 Actor + Ref Model
- 直接利用好/坏答案的概率比作为隐式奖励

**关键要求**：
- **同分布数据**（On-Policy）：偏好对应由 Actor 自身（或能力相近模型）生成，否则改错无效
- **对比学习本质**：同时看好答案（提高概率）和坏答案（压低概率），防止模型偷懒

**痛点**：对数据质量极度敏感，标注噪声会直接带偏模型

### GRPO 详解

**核心创新**：
- 去掉 Critic，改为组内 G 个采样输出的平均奖励做 baseline
- 奖励：数学/代码用规则验证（快准狠）；主观任务用额外 Reward Model

**公式拆解（三部分）**：
1. 组平均：对 G 个样本的 Loss 取平均
2. Clip 机制（来自 PPO）：限制更新幅度
3. KL 散度（来自 DPO）：约束不偏离 Ref Model

**痛点**：
- 采样成本高：每步需生成 G 个（如 8-16 个）长答案
- 依赖冷启动：SFT 模型质量差时，G 个答案全错则 GRPO 空转
- 奖励设计难：主观任务需额外 RM

### 进阶：GRPO 在 DeepSeek V3.2 中的改进

- **无偏 KL 估计**：加重要性采样比率防止梯度爆炸
- **Off-Policy 序列掩码**：忽略坏样本中偏移太大的序列
- **Keep Routing / Keep Sampling Mask**：确保 MoE 和采样的稳定性

### PO 家族新成员

- **DAPO**（Dynamic Sampling）：解决 GRPO clip 太死板、长文本梯度稀释的问题
- **GSPO**（Group Sequence）：为 MoE 设计，看整条序列而非单 token，防方差爆炸
- **SAPO**（Soft Adaptive，Qwen 团队）：软门控替代硬截断，好答案宽容/坏答案严厉

## 引用与数据

- 论文：[PPO](https://arxiv.org/abs/1707.06347)
- 论文：[DPO](https://arxiv.org/abs/2305.18290)
- 论文：[GRPO (DeepSeekMath)](https://arxiv.org/abs/2402.03300)
- 论文：[DeepSeek-R1](https://arxiv.org/abs/2501.12948)
- 论文：[DeepSeek-V3.2](https://arxiv.org/abs/2504.21110)
- 论文：[Qwen3 Technical Report](https://arxiv.org/abs/2505.xxxxx)

## 相关

- [[PPO（近端策略优化）]]
- [[DPO（直接偏好优化）]]
- [[GRPO（组相对策略优化）]]
- [[RLHF（基于人类反馈的强化学习）]]
- [[资料摘要：GRPO算法详解]]
- [[资料摘要：RLHF中的PPO拆解]]
- [[Wiki 目录]]
