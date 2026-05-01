---
title: LeIsaac
type: entity
tags: [机器人, 工具, 仿真, 教程]
aliases: [leisaac, LightwheelAI LeIsaac]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
confidence: high
---
- [x] 
# LeIsaac

> LightwheelAI 开源的桥接工具，在 Isaac Lab 中通过 LeRobot 兼容的 SO101Leader 提供机械臂遥操作、数据收集与策略部署功能。

## 概述

LeIsaac 连接真实机械臂与 NVIDIA Isaac Lab 仿真环境，实现从遥操作数据采集到策略训练再到仿真部署的端到端工作流。其核心是通过 SO101Leader（基于 LeRobot 协议）控制物理机械臂，同步驱动 Isaac Lab 中的仿真模型。

## 关键特性

- **遥操作**：支持 SE(3) 姿态控制，通过 `teleop_se3_agent.py` 脚本启动
- **数据记录**：收集遥操作数据为 HDF5 格式
- **数据转换**：提供 `isaaclab2lerobot.py` 脚本，将 HDF5 转为 LeRobot 兼容格式
- **策略推理客户端**：在 Isaac Lab 中加载训练好的模型进行仿真部署

## 相关

- [[SO-ARM101]]
- [[Isaac Lab]]
- [[GR00T N1.5]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
