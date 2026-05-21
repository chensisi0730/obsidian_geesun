---
title: 资料摘要：OpenARM 产品介绍
type: source
tags: [机器人, OpenARM, 开源硬件, 实体]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://www.openarm.cn/"
  - "https://github.com/enactic/openarm"
confidence: high
source_url: "https://www.openarm.cn/"
media: article
---

# 资料摘要：OpenARM 产品介绍

> OpenARM（OpenSource Artificial Robotics Machines）泡泡具身团队推进的开源双臂人形机器人品牌，Gen1 为完全开源的国产人形机械臂，专为物理 AI 研究设计。

## 核心要点

- **完全开源**：CAD 图纸、固件、控制代码、仿真工具全部公开
- **国产供应链**：深度贴合中国关节电机供应链生态，完全国产化知识产权
- **软件自主**：Rust 重构 OpenARM BrainOS 中间件，兼容 Debian RT / Ubuntu RT
- **被 LeRobot 收录**：HuggingFace LeRobot 官方指定数据采集设备
- **整机售价**：双臂系统约 $6,500 USD（DIY 套件或成品可选）

## 技术规格

| 参数 | 值 |
|------|-----|
| 臂展 | 633mm |
| 额定/峰值负载 | 4.1/6 Kg |
| 自由度 | 7 DOF（单臂） |
| 精度 | ±0.5~2mm |
| 控制接口 | CAN-FD |
| 电源 | 24/48V DC |
| 操作系统 | OpenARM OS（DebianRT/Ubuntu + BrainRT + Lerobot） |
| 编程语言 | Python, Rust, C++ |

## 关键特性

- **遥操作**：重力补偿 + 双边力反馈，Leader-Follower 控制
- **仿真支持**：兼容 MuJoCo 和 Isaac Sim
- **高反向驱动性**（Backdrivability）：安全的人机交互
- **灵活适配**：可搭配轮式/腿式底盘

## GitHub 仓库生态

| 仓库 | 说明 |
|------|------|
| [openarm](https://github.com/enactic/openarm) | 主项目，问题/功能请求 |
| [openarm_hardware](https://github.com/enactic/openarm_hardware) | 完整 CAD（STL/STEP/Fusion 360） |
| [openarm_description](https://github.com/enactic/openarm_description) | URDF/Xacro 仿真描述文件 |
| [openarm_can](https://github.com/enactic/openarm_can) | CAN 总线控制库 |
| [openarm_ros2](https://github.com/enactic/openarm_ros2) | ROS2 集成包 |
| [openarm_teleop](https://github.com/enactic/openarm_teleop) | 遥操作包（单边/双边） |
| [openarm_isaac_lab](https://github.com/enactic/openarm_isaac_lab) | Isaac Lab 仿真环境与训练任务 |

## 相关

- [[OpenARM]]
- [[遥操作（Teleoperation）]]
- [[Isaac Lab]]
- [[LeRobot]]
- [[Wiki 目录]]
