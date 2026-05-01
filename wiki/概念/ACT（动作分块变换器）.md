---
title: ACT（动作分块变换器）
type: concept
tags: [概念, 机器人, 模仿学习, 教程]
aliases: [ACT, Action Chunking with Transformers, 动作分块变换器]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：ACT算法精讲]]"
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
related_concepts: [模仿学习, 遥操作, CVAE]
confidence: medium
---

# ACT（动作分块变换器）

> 将 Transformer 与动作分块（Action Chunking）结合的模仿学习算法，通过 CVAE 架构处理多模态示教数据，在精细机器人操作任务中表现优异。

## 是什么

ACT 由 ALOHA 团队于 2023 年 4 月提出，其核心是将动作预测从单步变为多步分块（Chunk），每 k 步预测一次后续 k 步的完整动作序列。模型采用 CVAE（条件变分自编码器）架构，训练时学习示教数据的多模态分布，推理时退化为确定性策略。

## 为什么重要

- **缓解复合误差**：Action Chunking + Temporal Ensemble 大幅降低模仿学习中的误差累积问题
- **处理多模态数据**：人类示教数据常有多种完成任务的方式，CVAE 架构天然适合建模这种多峰分布
- **轻量高效**：80M 参数，5 小时训练，0.01 秒推理，适合实际部署
- **作为 baseline**：被 RoboAgent、BridgeData V2 等多篇后续论文引用

## 工作原理

1. **CVAE Encoder**（仅训练时使用）：输入观测关节角 + 示教动作序列，输出隐变量 z 的分布参数
2. **CVAE Decoder（即 Policy）**：输入观测（关节角 + ResNet18 编码的图像）+ 隐变量 z（推理时 z=0），输出分块后的绝对关节角序列
3. **Temporal Ensemble**：对 k 次重叠预测做指数衰减加权平均，使动作平滑

## 与相邻概念的区别

- **行为克隆（BC）**：单步预测，易积累误差。ACT 使用分块预测 + 时序集成
- **DPO/PPO/GRPO**：RL 类算法，需要奖励信号。ACT 是纯模仿学习，无需奖励函数

## 相关

- [[PPO（近端策略优化）]]
- [[资料摘要：ACT算法精讲]]
- [[遥操作（Teleoperation）]]
- [[Wiki 目录]]
