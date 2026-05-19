---
title: 资料摘要：LeIsaac EnvHub 集成
type: source
tags: [机器人, Isaac Lab, LeRobot, 仿真, 教程, 深度]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://huggingface.co/docs/lerobot/envhub_leisaac"
  - "https://huggingface.co/docs/lerobot/envhub"
confidence: medium
source_url: "https://huggingface.co/docs/lerobot/envhub_leisaac"
media: article
---

# 资料摘要：LeIsaac EnvHub 集成

> LeRobot 官方文档，介绍 LeIsaac 仿真环境通过 EnvHub 机制集成到 LeRobot 框架中，实现一行代码加载 SO101 各类操作任务环境，支持遥操作数据采集、策略训练全流程。

## 核心要点

- **EnvHub 机制**：LeRobot 允许从 HuggingFace Hub 以 `make_env("user/repo:env.py")` 一行代码加载仿真环境，无需本地安装
- **LeIsaac 已发布 4 个 SO101 操作任务**：PickOrange、LiftCube、CleanToyTable、FoldCloth（含单臂和双臂）
- 支持物理 Leader 设备遥操作仿真环境（通过 `so101_leader`），复刻了完整的遥操作→数据采集→策略训练闭环
- **云仿真**：支持 NVIDIA Brev 云实例，无需本地 GPU 即可运行

## 已支持的环境列表

| 任务 | 环境 ID | 单/双臂 |
|------|---------|---------|
| 捡橙子 | LeIsaac-SO101-PickOrange-v0 | 单臂 |
| 举立方体 | LeIsaac-SO101-LiftCube-v0 | 单臂 |
| 清理玩具桌 | LeIsaac-SO101-CleanToyTable-v0 | 单臂/双臂 |
| 叠布 | LeIsaac-SO101-FoldCloth-BiArm-v0 | 双臂 |

每个任务有 `Direct` 版本（直接环境，支持 `check_success`）。

## 安装与使用

```
conda create -n leisaac_envhub python=3.11
conda activate leisaac_envhub
conda install -c "nvidia/label/cuda-12.8.1" cuda-toolkit
pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
pip install 'leisaac[isaaclab] @ git+https://github.com/LightwheelAI/leisaac.git#subdirectory=source/leisaac' --extra-index-url https://pypi.nvidia.com
pip install lerobot==0.4.1
pip install numpy==1.26.0
```

一行加载环境：`make_env("LightwheelAI/leisaac_env:envs/so101_pick_orange.py", n_envs=1, trust_remote_code=True)`

### 遥操作示例

连接物理 SO101 Leader 即可通过 Leader 控制器驱动仿真机械臂，完整代码见原始文档。

## EnvHub 机制

EnvHub 是 LeRobot 的环境分发机制，核心规则：
- 仓库需要包含 `env.py`，暴露 `make_env(n_envs, use_async_envs)` 函数
- 返回 `gym.vector.VectorEnv`、单 `gym.Env` 或 `{suite: {task_id: VectorEnv}}` 字典
- 支持 Git 版本锁定（`@commit`）、自定义文件路径（`:file.py`）

## 相关

- [[Isaac Lab]]
- [[LeRobot]]
- [[遥操作（Teleoperation）]]
- [[资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂]]
- [[资料摘要：LeRobot 官方教程]]
- [[Wiki 目录]]
