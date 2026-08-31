---
title: 资料摘要：OpenPI π0 模型家族
type: source
tags: [机器人, VLA, Physical Intelligence, π0, 深度]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://bbs.openarm.cn/d/5-openpipi"
confidence: medium
source_url: "https://bbs.openarm.cn/d/5-openpipi"
media: article
---

# 资料摘要：OpenPI π0 模型家族

> OpenARM 社区对 Physical Intelligence（PI）团队 VLA 模型的深度分析，涵盖 π0、π0-FAST、π0.5 三代模型迭代及「Hi Robot」分层推理框架。

## 核心要点

### 模型迭代

| 模型 | 核心技术 | 特点 |
|------|---------|------|
| **π0** | 流匹配（Flow Matching） | 推理速度快，适合实时控制 |
| **π0-FAST** | FAST 动作分词器 + 自回归 | 训练速度比 π0 快 5x，精确跟随复杂指令 |
| **π0.5** | 知识隔离（Knowledge Insulation） | 开放世界强泛化能力 |

### π0 架构三大组件

1. **SigLIP 视觉编码器** — 图像特征提取
2. **PaliGemma 语言模型** — 语言指令理解
3. **Action Expert 动作专家网络** — 动作序列生成（流匹配）

### Hi Robot 分层推理框架

2025年2月26日发布，将 VLA 模型纳入**分层规划与执行过程**，上层处理复杂任务分解，下层执行基础操作。解决 VLA 模型无法处理长时间跨度复杂任务的问题。

## 莫拉维克悖论与机器人奥运会

PI 团队用 π0.6 参与了「机器人奥运会」挑战（5 类 15 个日常任务），核心结论：

- **平均成功率 52%，任务完成度 72%**
- **无机器人预训练的 VLM 基线：任务完成度仅 9%，未成功完成任何任务**
- 大多数任务数据采集耗时 < 9 小时
- 物理限制（夹爪太宽）导致部分任务无法完成

### 关键洞察

> 物理智能无法仅通过语言模型获得——人类不会在网络上发布"如何移动手臂洗锅"，因为每个人都知道，且无法用文字传达。VLA 模型的价值在于从多样化数据中捕获通用物理知识，使下游技能可以用更小数据集学习。

## π0.7 补充（2026.4）

π0.7 是 PI 团队 2026 年 4 月发布的最新工作，核心贡献已在该资料中未覆盖：

- **多模态上下文条件化**：子任务指令 + subgoal images（World Model 生成）+ episode metadata（speed/quality/mistake）+ control mode
- **World Model 生成 Subgoal**：轻量 world model 生成近未来视觉目标，辅助 VLA 动作生成
- **模型参数**：Gemma3 4B VLM + MEM video history encoder + 860M action expert
- **关键实验发现**：metadata 让模型从更大但质量更低的数据中持续获益；零样本完成灵巧操作并匹配 RL specialist
- **跨 embodiment 迁移**：不是动作轨迹拷贝，而是策略重组——UR5e 上 π0.7 生成适合其运动学的操作策略

详见 [[资料摘要：π0.7 多模态上下文条件化 VLA]]。

## 相关

- [[VLA（视觉-语言-动作模型）]]
- [[流匹配（Flow Matching）]]
- [[资料摘要：π*0.6 真实世界强化学习]]
- [[资料摘要：π0.7 多模态上下文条件化 VLA]]
- [[Wiki 目录]]
