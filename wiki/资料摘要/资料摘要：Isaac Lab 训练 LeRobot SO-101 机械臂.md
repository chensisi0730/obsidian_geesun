---
title: 资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂
type: source
tags: [机器人, 仿真, 教程, Isaac Lab, LeRobot]
created: 2026-05-02
updated: 2026-05-02
sources:
  - "https://cloud.tencent.com/developer/article/2557331"
confidence: high
source_url: "https://cloud.tencent.com/developer/article/2557331"
media: article
---

# 资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂

> 腾讯云开发者教程，展示如何使用 Isaac Lab 2.2.0 + Isaac Sim 5.0 训练 SO-101 机械臂完成"抓举积木"任务的完整流程。

## 核心要点

- **环境配置**：Ubuntu 24.04 + Isaac Sim 5.0 + Isaac Lab 2.2.0
- **项目**：使用 [isaac_so_arm101](https://github.com/MuammerBay/isaac_so_arm101) 开源项目
- **任务**：SO-ARM100-Lift-Cube（抓举积木）
- **训练框架**：RSL-RL（Robotic Systems Lab Reinforcement Learning）
- **训练时长**：12000 回合，RTX 4060 Ti 16GB 约 4 小时
- **模型存储**：每 50 回合保存一个 `model_<nnnnn>.pt` 文件

## 详细笔记

### 系统要求

| 组件 | 版本 |
|------|------|
| 操作系统 | Ubuntu 24.04 |
| Isaac Sim | 5.0 |
| Isaac Lab | 2.2.0 |
| Python | 3.10 |

### Isaac Lab 安装步骤

```bash
# 1. 创建 conda 环境
conda create -n isaaclab python=3.10
conda activate isaaclab

# 2. 安装 Isaac Sim 5.0（通过 pip）
pip install isaacsim --extra-index-url https://pypi.nvidia.com

# 3. 克隆 Isaac Lab
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
git checkout v2.2.0

# 4. 安装依赖
./isaaclab.sh --install

# 5. 验证安装
./isaaclab.sh -p scripts/list_envs.py
```

### isaac_so_arm101 项目使用

```bash
# 克隆项目（与 IsaacLab 同级目录）
git clone https://github.com/MuammerBay/isaac_so_arm101
cd isaac_so_arm101

# 安装 SO-100 扩展
python -m pip install -e source/SO_100

# 查看可用任务
python scripts/list_envs.py
```

### 可用任务列表

| 任务 | 用途 |
|------|------|
| SO-ARM100-Lift-Cube-v0 | 训练模型 |
| SO-ARM100-Lift-Cube-Play-v0 | 执行演示 |
| SO-ARM100-Reach-v0 | 训练模型 |
| SO-ARM100-Reach-Play-v0 | 执行演示 |

### 训练流程

```bash
# 训练模型（headless 模式提高效率）
python scripts/rsl_rl/train.py --task SO-ARM100-Lift-Cube-v0 --headless --max_iterations 12000
```

**训练输出位置**：`isaac_so_arm101/logs/rsl_rl/so_arm100_lift/<日期时间>/`

### 执行模拟

```bash
# 使用训练好的模型执行任务
python scripts/rsl_rl/play.py --task SO-ARM100-Lift-Cube-Play-v0 \
    --checkpoint logs/rsl_rl/so_arm100_lift/<模型目录>/model_11999.pt
```

### 测试脚本

| 脚本 | 说明 |
|------|------|
| `zero_agent.py` | 无动作指令，机械臂静止 |
| `random_agent.py` | 随机动作指令，机械臂抖动 |

## 引用与数据

- 项目：[isaac_so_arm101](https://github.com/MuammerBay/isaac_so_arm101)
- 框架：[NVIDIA Isaac Lab](https://github.com/isaac-sim/IsaacLab)
- 框架：[LeRobot](https://github.com/huggingface/lerobot)
- 硬件：SO-101 机械臂（SO-ARM100 系列）

## 相关

- [[Isaac Lab]]
- [[GR00T N1.5]]
- [[遥操作（Teleoperation）]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
