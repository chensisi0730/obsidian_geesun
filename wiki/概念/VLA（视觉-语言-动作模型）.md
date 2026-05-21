---
title: VLA（视觉-语言-动作模型）
type: concept
tags: [概念, 机器人, 多模态]
aliases: [VLA, Vision-Language-Action, 视觉语言动作模型]
created: 2026-05-02
updated: 2026-05-02
sources:
  - "[[资料摘要：π*0.6 真实世界强化学习]]"
  - "[[资料摘要：LeRobot 框架架构剖析]]"
  - "[[资料摘要：SmolVLA 轻量级 VLA 模型]]"
  - "[[资料摘要：资料摘要：LeRobot π0 封装剖析]]"
  - "[[资料摘要：资料摘要：OpenPI π0 模型家族]]"
related_concepts: [多模态学习, 机器人控制, 模仿学习, 轻量级部署, 流匹配]
confidence: high
---

# VLA（视觉-语言-动作模型）

> 融合视觉、语言和动作的多模态机器人策略模型，能够根据视觉观察和语言指令生成机器人动作序列。

## 是什么

VLA（Vision-Language-Action）是一类端到端的机器人策略模型，将三种模态统一在一个 Transformer 架构中：
- **视觉**：RGB/RGB-D 图像，通过视觉编码器提取特征
- **语言**：自然语言指令，通过语言模型编码
- **动作**：机器人关节位置/速度，通过动作专家解码

## 为什么重要

- **端到端学习**：从原始感知到动作输出，无需手工设计中间表示
- **语言引导**：支持自然语言指令控制，降低编程门槛
- **泛化能力**：多任务预训练后可泛化到新任务

## 工作原理

```mermaid
flowchart LR
    A[视觉观察<br/>RGB Image] --> C[Transformer<br/>VLA Model]
    B[语言指令<br/>"Pick the cup"] --> C
    C --> D[动作序列<br/>Joint Positions]
    D --> E[机器人执行]
```

## 代表模型

| 模型 | 团队 | 特点 |
|------|------|------|
| π0 / π*0.6 | Physical Intelligence | 流匹配 + 真实世界 RL |
| π0-FAST | Physical Intelligence | FAST 动作分词器，训练快 5x |
| π0.5 | Physical Intelligence | 知识隔离，开放世界泛化 |
| GR00T N1.5 | NVIDIA | 通用人形机器人 |
| RT-1 / RT-2 | Google | 机器人 Transformer |
| OpenVLA | 开源社区 | 开源 VLA 基础模型 |
| SmolVLA | 社区 | 消费级 GPU 训练，CPU 部署 |

## 训练范式演进

```mermaid
flowchart LR
    A[预训练<br/>大规模数据] --> B[监督微调<br/>模仿学习]
    B --> C[强化学习<br/>真实世界/仿真]
```

## 与相邻概念的区别

- **ACT**：纯视觉-动作模型，无语言输入
- **Diffusion Policy**：基于扩散的动作生成，无语言
- **VLA**：融合语言，支持指令引导

## 相关

- [[GR00T N1.5]]
- [[ACT（动作分块变换器）]]
- [[流匹配（Flow Matching）]]
- [[资料摘要：π*0.6 真实世界强化学习]]
- [[资料摘要：资料摘要：OpenPI π0 模型家族]]
- [[资料摘要：资料摘要：LeRobot π0 封装剖析]]
- [[OpenARM]]
- [[Wiki 目录]]
