---
title: Genie Sim
type: concept
tags: [概念, 机器人, 仿真, 智元, 工具]
aliases: [Genie Sim, 智元仿真平台, Genie Sim 3.0]
created: 2026-05-20
updated: 2026-05-20
sources:
  - "[[资料摘要：智元 Genie Sim 3.0 仿真平台]]"
related_concepts: [仿真环境, Isaac Lab, Genesis, VLA, RLinf, Sim2Real]
confidence: medium
---

# Genie Sim

> 智元机器人（Zhiyuan Robotics）推出的一站式机器人仿真开发平台，核心特色为自然语言生成3D场景、多维评测基准，以及与 RLinf 集成的强化学习训练流水线。

## 是什么

Genie Sim 是智元团队的仿真平台，聚焦"场景—数据—评测"三大环节。与 Isaac Lab 不同，它的核心差异化在于：

1. **语言生成场景**：不需手动搭建 CAD 场景，直接通过自然语言或图片生成可交互的 3D 世界
2. **内置基准评测**：Genie Sim Benchmark 覆盖五大能力维度，支持主流 VLA 模型一键评测（π 系列、GR00T 系列）
3. **RLinf 集成**：专为 VLA 模型后训练的强化学习流水线

## 关键特性

- **Sim2Real < 10% 差距**：仿真训练的模型迁移到真机评测差异小于 10%
- **1,000Hz 物理模拟**：物理与渲染引擎解耦，支持大规模并行
- **标准 Gym 接口**：与 RLinf 及其他社区算法生态兼容
- **开源模型评分**：提供主流模型在各 benchmark 下的全景能力画像

## 与其他平台的对比

| 维度 | Isaac Lab | Genie Sim | Genesis |
|------|-----------|-----------|---------|
| 开发方 | NVIDIA | 智元机器人 | 开源（19 单位） |
| 场景生成 | 手动搭建（USD） | 自然语言自动生成 | 生成式 AI 自动生成 |
| 评测基准 | 无内置 | 五大维度 Benchmark | 无内置 |
| RL 集成 | RSL-RL / skrl | RLinf | 原生可微分+RL |
| 物理引擎 | PhysX（Omniverse） | 自研双引擎解耦 | 自研通用引擎 |
| 差异化 | 生态完善，工业级 | NL→3D，Sim2Real<10% | 10-80x 加速，可微分 |

## 相关

- [[Isaac Lab]]
- [[Genesis]]
- [[VLA（视觉-语言-动作模型）]]
- [[遥操作（Teleoperation）]]
- [[资料摘要：智元 Genie Sim 3.0 仿真平台]]
- [[资料摘要：7大仿真平台对比]]
- [[Wiki 目录]]
