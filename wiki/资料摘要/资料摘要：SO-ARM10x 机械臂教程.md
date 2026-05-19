---
title: 资料摘要：SO-ARM10x 机械臂教程
type: source
tags: [机器人, 教程, 硬件, LeRobot]
created: 2026-05-02
updated: 2026-05-02
sources:
  - "https://wiki.seeedstudio.com/cn/lerobot_so10xarm/"
confidence: high
source_url: "https://wiki.seeedstudio.com/cn/lerobot_so10xarm/"
media: article
---

# 资料摘要：SO-ARM10x 机械臂教程

> Seeed Studio 官方教程，详细讲解 SO-ARM100/101 机械臂的 3D 打印、组装、校准和 LeRobot 集成流程。

## 核心要点

- **硬件平台**：SO-ARM100（旧版）和 SO-ARM101（新版，布线优化）
- **电机配置**：Leader 臂使用不同减速比电机（1:191、1:345、1:147）
- **电源要求**：标准版 5V，Pro 版 Leader 5V / Follower 12V
- **环境支持**：Ubuntu 22.04 (X86) 和 Jetson Orin (Jetpack 6.0/6.1)

## 详细笔记

### SO-ARM101 vs SO-ARM100

| 特性 | SO-ARM100 | SO-ARM101 |
|------|-----------|-----------|
| 布线 | 第3关节可能断线 | 优化布线，不限制关节活动 |
| Leader 齿轮比 | 统一 1:345 | 优化：L1/L3 用 1:191，L4-L6 用 1:147 |
| 主从跟随 | 不支持 | Leader 可实时跟随 Follower |

### 电机配置表

| 舵机型号 | 减速比 | 对应关节 |
|----------|--------|----------|
| ST-3215-C044 | 1:191 | L1, L3 |
| ST-3215-C001 | 1:345 | L2 |
| ST-3215-C046 | 1:147 | L4, L5, L6 |
| ST-3215-C001/C018/C047 | 1:345 | F1-F6 |

### 安装步骤

```bash
# 1. 创建环境
conda create -y -n lerobot python=3.10 && conda activate lerobot

# 2. 克隆仓库
git clone https://github.com/Seeed-Projects/lerobot.git ~/lerobot

# 3. 安装依赖
conda install ffmpeg -c conda-forge
cd ~/lerobot && pip install -e ".[feetech]"

# 4. 查找端口
lerobot-find-port

# 5. 校准舵机
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/ttyACM0
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=/dev/ttyACM1
```

### 3D 打印参数

- 材料：PLA+
- 喷嘴：0.4mm（层高 0.2mm）或 0.6mm（层高 0.4mm）
- 填充：15%
- 支撑：处处需要，忽略 <45° 倾斜面

### 故障排除

```bash
# Linux USB 权限
sudo chmod 666 /dev/ttyACM0
sudo chmod 666 /dev/ttyACM1
```

## 引用与数据

- 项目：[SO-ARM100](https://github.com/TheRobotStudio/SO-ARM100)
- 框架：[LeRobot (Seeed fork)](https://github.com/Seeed-Projects/lerobot)
- 硬件：[Seeed Studio](https://www.seeedstudio.com/)

## 相关

- [[LeRobot]]
- [[遥操作（Teleoperation）]]
- [[资料摘要：LeRobot 官方教程]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
