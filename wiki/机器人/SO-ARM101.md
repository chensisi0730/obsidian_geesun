---
title: SO-ARM101
type: entity
tags: [机器人, 硬件, 开源, 教程]
aliases: [SO-ARM100, SO101]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "[[资料摘要：SOArm101 仿真与机械臂策略训练]]"
confidence: high
---

# SO-ARM101

> 低成本、开源的 3D 可打印机械臂套件，设计用于与 LeRobot 库无缝配合。

## 概述

SO-ARM101 是由 The Robot Studio 推出的开源机械臂项目。它继承了 SO-ARM100 的设计理念，以极低的成本提供可访问的机器人操作研究平台。SOArm101 机械臂可连接 USB 串口，在 LeIsaac + Isaac Lab 仿真环境中作为领导臂进行遥操作数据采集。

## 关键特性

- **开源 3D 可打印**：所有结构件可 3D 打印，降低硬件门槛
- **LeRobot 兼容**：原生对接开源机器人学习库 LeRobot
- **USB 串口控制**：通过 `/dev/ttyACM*` 串口连接，简化部署
- **仿真集成**：在 Isaac Lab 中有对应的 `so101_follower.usd` 模型

## 相关

- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[LeIsaac]]
- [[Isaac Lab]]
- [[遥操作（Teleoperation）]]
- [[Wiki 目录]]
