---
title: 资料摘要：SOArm101 仿真与机械臂策略训练
type: source
tags: [机器人, 仿真, 策略训练, 教程]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "https://wiki.seeedstudio.com/cn/simulate_soarm101_by_leisaac/"
confidence: medium
source_url: "https://wiki.seeedstudio.com/cn/simulate_soarm101_by_leisaac/"
media: article
---

# 资料摘要：SOArm101 仿真与机械臂策略训练

> Seeed Studio 官方教程，展示如何使用 LeIsaac 在 Isaac Lab 中对 SOArm101 机械臂进行远程操作和数据收集，并通过 NVIDIA Isaac GR00T N1.5 进行策略微调和部署。

## 核心要点

- **完整流水线**：从仿真环境搭建 → 遥操作数据收集 → 数据集转换 → 策略训练 → 模型部署的端到端流程
- **工具链**：LeIsaac（遥操作桥接）+ Isaac Lab（仿真）+ GR00T N1.5（策略模型）+ SO-ARM101（硬件）
- **架构设计**：推理端点（服务器）与控制端点（客户端）解耦的设计模式
- **硬件要求**：Ubuntu PC + NVIDIA GPU（RTX 3080）+ SOArm101 领导臂（通过 USB 连接）

## 详细笔记

### 环境搭建
- 使用 conda 创建 `leisaac` 环境，Python 3.10
- 需安装 CUDA 11.8、PyTorch 2.5.1、IsaacSim 4.5.0 和 IsaacLab v2.1.0
- 50 系列 GPU 建议使用 isaacsim5.0

### 数据收集
- 通过 USB 连接 SO-ARM101 领导臂到 Ubuntu 电脑
- 使用 `teleop_se3_agent.py` 启动遥操作任务
- 按 `b` 键开始/停止远程操作，按 `r`（失败）或 `n`（成功）重置环境并标记
- 数据集以 HDF5 格式存储

### 数据集转换
- 使用 LeIsaac 的 `isaaclab2lerobot.py` 脚本将 HDF5 数据转换为 LeRobot 兼容格式
- 转换后的数据集存储在 `~/.cache/huggingface/lerobot/`

### 策略训练
- 基于 NVIDIA Isaac GR00T N1.5 进行微调
- 需安装 flash-attn（推荐预编译包）
- 训练命令通过 `gr00t_finetune.py` 启动，支持多 GPU

### 推理部署
- 服务器（推理端）运行 `inference_service.py`
- 客户端（控制端）运行 `policy_inference.py`
- 通过 ZMQ 通信，支持语言指令如 "Pick up the orange and place it on the plate"

### 局限性
- 实验中仅收集 3 组数据，模型未能成功抓取橙子
- 作者指出更多数据可显著提高准确性

## 引用与数据

- 项目：[LeIsaac](https://github.com/LightwheelAI/leisaac)
- 项目：[NVIDIA Isaac Lab](https://developer.nvidia.com/isaac/lab)
- 项目：[SO-ARM101](https://github.com/TheRobotStudio/SO-ARM100)
- 项目：[NVIDIA Isaac GR00T](https://github.com/NVIDIA/Isaac-GR00T)
- 框架：[LeRobot](https://github.com/huggingface/lerobot)

## 相关

- [[SO-ARM101]]
- [[LeIsaac]]
- [[Isaac Lab]]
- [[GR00T N1.5]]
- [[遥操作（Teleoperation）]]
- [[ACT（动作分块变换器）]]
- [[PPO（近端策略优化）]]
- [[GRPO（组相对策略优化）]]
- [[Wiki 目录]]
