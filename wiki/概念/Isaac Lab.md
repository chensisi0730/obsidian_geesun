---
title: Isaac Lab
type: concept
tags: [概念, 机器人, 仿真, 教程]
aliases: [Isaac Lab, NVIDIA Isaac Lab, NVIDIA Isaac™ Lab]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
related_concepts: [机器人学习, 仿真环境]
confidence: high
---

# Isaac Lab

> NVIDIA 开源的统一机器人学习框架，构建于 Isaac Sim 之上，旨在加速机器人策略的训练与部署。

## 是什么

Isaac Lab 提供标准化的仿真环境、任务定义和训练接口，让开发者能够在仿真中训练机器人策略并部署到真实机器人。

## 关键特性

- **仿真环境**：基于 Isaac Sim（NVIDIA Omniverse）提供高保真物理仿真
- **任务系统**：支持自定义任务（如 PickOrange），含成功/失败判定
- **相机支持**：仿真环境中集成 RGB 相机用于视觉策略训练
- **多环境并行**：支持 `--num_envs` 参数并行运行多个仿真环境

## 相关

- [[SO-ARM101]]
- [[LeIsaac]]
- [[GR00T N1.5]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
