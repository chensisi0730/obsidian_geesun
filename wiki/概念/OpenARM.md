---
title: OpenARM
type: entity
tags: [概念, 机器人, 开源硬件, OpenARM, 实体]
aliases: [OpenARM Gen1, enactic OpenARM, 泡泡具身, OpenArm, OpenSource Artificial Robotics Machines]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "[[资料摘要：资料摘要：OpenARM 产品介绍]]"
related_concepts: [遥操作, 模仿学习, Isaac Lab, LeRobot, SO-ARM101]
confidence: high
---

# OpenARM

> OpenARM（OpenSource Artificial Robotics Machines）泡泡具身团队推出的完全开源国产人形机械臂品牌，Gen1 为 7-DOF 双臂系统，专为物理 AI 研究和接触密集型操作任务设计，双臂系统约 $6,500 USD。

## 是什么

OpenARM 是一个完全开源的人形机械臂平台，包含完整的 CAD 图纸、固件、控制代码和仿真工具。其核心设计理念是国产供应链友好、高反向驱动性（安全人机交互）和开放生态。

## 硬件规格

- **自由度**：7 DOF 每臂（双臂共 14 DOF）
- **臂展**：633mm
- **负载**：额定 4.1 Kg / 峰值 6 Kg
- **精度**：±0.5~2mm
- **控制接口**：CAN-FD
- **电源**：24/48V DC
- **关节**：可反向驱动，安全交互

## 软件生态

| 组件 | 说明 |
|------|------|
| **OpenARM BrainOS** | Rust 重构的中间件，兼容 Debian RT / Ubuntu RT |
| **openarm_teleop** | 遥操作包（单边 Leader-Follower / 双边力反馈） |
| **openarm_isaac_lab** | Isaac Lab 仿真环境与训练任务 |
| **openarm_ros2** | ROS2 集成 |
| **openarm_can** | CAN 总线底层控制库 |
| **openarm_description** | URDF/Xacro 仿真描述 |

## 与 SO-ARM100/101 的关系

- SO-ARM100/101 是 [TheRobotStudio](https://github.com/TheRobotStudio/SO-ARM100) 的开源机械臂方案，同样为 6-DOF + 夹爪
- OpenARM 为 7-DOF，负载更高（4.1 vs 1.5 Kg），定位为工业级开源方案
- 两套方案都兼容 LeRobot，用户可用同一套软件栈训练
- OpenARM 已被 [LeRobot](https://github.com/huggingface/lerobot) 收录为官方指定数据采集设备

## 相关

- [[遥操作（Teleoperation）]]
- [[Isaac Lab]]
- [[LeRobot]]
- [[ACT（动作分块变换器）]]
- [[资料摘要：资料摘要：OpenARM 产品介绍]]
- [[资料摘要：资料摘要：OpenPI π0 模型家族]]
- [[Wiki 目录]]
