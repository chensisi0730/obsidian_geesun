---
title: Isaac Lab
type: concept
tags: [概念, 机器人, 仿真, 教程]
aliases: [Isaac Lab, NVIDIA Isaac Lab, NVIDIA Isaac™ Lab]
created: 2026-05-01
updated: 2026-05-02
sources:
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
  - "[[资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂]]"
  - "[[资料摘要：资料摘要：LeIsaac EnvHub 集成]]"
  - "[[资料摘要：资料摘要：Isaac Lab 快速入门指南]]"
  - "[[资料摘要：资料摘要：7大仿真平台对比]]"
  - "[[资料摘要：资料摘要：Genesis 物理引擎]]"
  - "[[资料摘要：资料摘要：宇树 RL Lab（Unitree IsaacLab 训练框架）]]"
related_concepts: [机器人学习, 仿真环境, RSL-RL, EnvHub, Genesis, NVIDIA Cosmos]
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
- **两大工作流**：DirectRLEnv（快速原型）和 ManagerBasedRLEnv（模块化复杂项目）
- **项目模板生成器**：`./isaaclab.sh --new` 快速生成项目骨架
- **配置系统**：通过 `@configclass` 装饰器统一管理训练参数，支持 CLI 覆盖

## 与相邻概念的区别

- **[[Genesis]]**：新一代可微分物理引擎，比 Isaac 快 10-80 倍，原生支持生成式 AI 和可微分模拟
- **[[Genie Sim]]**：智元机器人仿真平台，NL→3D 场景生成和内置 Benchmark
- **[[NVIDIA Cosmos]]**：NVIDIA 世界模型平台，与 Isaac Lab 互补——Isaac 做仿真，Cosmos 做智能推理和生成

## 相关

- [[SO-ARM101]]
- [[LeIsaac]]
- [[GR00T N1.5]]
- [[Genesis]]
- [[NVIDIA Cosmos]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[资料摘要：资料摘要：LeIsaac EnvHub 集成]]
- [[资料摘要：资料摘要：Isaac Lab 快速入门指南]]
- [[资料摘要：资料摘要：7大仿真平台对比]]
- [[资料摘要：资料摘要：Genesis 物理引擎]]
- [[资料摘要：资料摘要：宇树 RL Lab（Unitree IsaacLab 训练框架）]]
- [[OpenARM]]
- [[Wiki 目录]]
