---
title: 资料摘要：UnifoLM-WMA-0 世界模型
type: source
tags: [机器人, 宇树, 世界模型, 深度, VLA]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://github.com/unitreerobotics/unifolm-world-model-action"
confidence: medium
source_url: "https://github.com/unitreerobotics/unifolm-world-model-action"
media: article
---

# 资料摘要：UnifoLM-WMA-0 世界模型

> 宇树科技发布的开源世界模型-动作（WMA）框架，核心是一个能理解物理交互规律的世界模型，兼具仿真引擎和策略增强两大功能。

## 核心要点

- **WMA = World Model + Action**：世界模型 + 动作头的统一架构
- **双重角色**：
  1. **仿真引擎**：作为交互式仿真器运行，生成合成训练数据
  2. **策略增强**：预测未来物理交互过程，优化当前决策
- **真机效果**：右上角小窗口显示世界模型对未来视频的预测，辅助控制指令生成
- **训练策略**：
  1. 在 Open-X 数据集微调视频生成模型作为世界模型
  2. 决策模式后训练
  3. 仿真模式后训练
- **模型版本**：Base（Open-X 微调）和 Dual（宇树开源数据联合微调）
- **基于 DynamiCrafter、Diffusion Policy、ACT、HPT 构建**

## 部署架构

**Client-Server 架构**：服务器端运行世界模型推理，机器人客户端采集观测并发送到服务器，服务器预测视频和动作后返回。

## 开源数据集

5 个宇树真机操作数据集用于训练，含 Z1 和 G1 机器人，已发布在 HuggingFace。

## 相关

- [[VLA（视觉-语言-动作模型）]]
- [[ACT（动作分块变换器）]]
- [[资料摘要：宇树 LeRobot 训练框架]]
- [[Wiki 目录]]
