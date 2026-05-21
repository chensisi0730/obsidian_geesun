---
title: 资料摘要：Isaac Lab 快速入门指南
type: source
tags: [机器人, 仿真, 教程, Isaac Lab, 基础]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://docs.robotsfan.com/isaaclab/"
confidence: high
source_url: "https://docs.robotsfan.com/isaaclab/"
media: article
---

# 资料摘要：Isaac Lab 快速入门指南

> Isaac Lab 官方快速入门指南的中文翻译，覆盖安装配置、Direct/Manager 两种工作流、项目模板生成、配置系统和机器人定义。

## 核心要点

### 安装（Isaac Sim 5.1.0 + Isaac Lab）

```bash
conda create -n env_isaaclab python=3.11
conda activate env_isaaclab
pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
git clone https://github.com/isaac-sim/IsaacLab.git
./isaaclab.sh --install
```

### 两大工作流

| 工作流 | 适用场景 | 特点 |
|--------|---------|------|
| **DirectRLEnv** | 快速原型 | 所有逻辑在一份文件中，最快出结果 |
| **ManagerBasedRLEnv** | 复杂项目 | 模块化设计（ActionManager、SceneManager 等），灵活替换组件 |

### 项目生成

```bash
./isaaclab.sh --new
```

生成器选项：
- **内部 vs 外部**：作为 Isaac Lab 的一部分或独立扩展
- **Direct vs Manager**：直接任务 vs 基于 manager 的项目
- **框架**：选择 RL 框架（skrl、RSL-RL 等）

### 配置系统（@configclass）

所有配置通过 `@configclass` 装饰器定义，例如：

```python
@configclass
class CartpoleEnvCfg(DirectRLEnvCfg):
    decimation = 2
    episode_length_s = 5.0
    action_scale = 100.0
    action_space = 1
    observation_space = 4
    state_space = 0
```

配置可通过 CLI 参数（如 `--num_envs`）覆盖。

### 机器人定义

机器人通过 `ArticulationCfg` 配置定义，包括：
- **spawn**：USD 文件路径 + 物理属性
- **init_state**：初始关节位置/姿态
- **actuators**：执行器配置（`ImplicitActuatorCfg` 使用默认执行模型）

### 独立应用模式

独立脚本通过 `AppLauncher` 启动 Isaac Sim 应用：

```python
from isaaclab.app import AppLauncher
parser.add_argument("--num_envs", type=int, default=1)
app_launcher = AppLauncher(args_cli)
simulation_app = app_launcher.app
```

**注意事项**：AppLauncher 启动前不能导入 Isaac Sim / Isaac Lab 模块。

## 相关

- [[Isaac Lab]]
- [[资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
