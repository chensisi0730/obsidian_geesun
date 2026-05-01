---
title: GR00T N1.5
type: concept
tags: [概念, 机器人, 策略模型, 教程]
aliases: [Isaac GR00T, GR00T, NVIDIA GR00T N1.5, Isaac GR00T N1.5]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
related_concepts: [机器人学习, 策略训练, 模仿学习]
confidence: high
---

# GR00T N1.5

> NVIDIA 开源的通用人形机器人推理和技能基础模型，支持通过微调适配特定任务。

## 是什么

GR00T N1.5 是基于 Transformer 架构的机器人策略模型，可从遥操作收集的数据中学习动作策略。它采用**推理端点与控制端点解耦**的部署架构：

- **服务器**：运行模型推理，接收观察并输出动作
- **客户端**：获取机器人状态并协调运动控制，通过 ZMQ 与服务器通信

## 训练流程

1. 使用 LeIsaac + Isaac Lab 遥操作收集示范数据
2. 转换为 LeRobot 格式
3. 基于预训练 GR00T N1.5 权重复制微调（fine-tuning）
4. 部署到仿真或真实机器人

## 相关

- [[SO-ARM101]]
- [[LeIsaac]]
- [[Isaac Lab]]
- [[遥操作（Teleoperation）]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
