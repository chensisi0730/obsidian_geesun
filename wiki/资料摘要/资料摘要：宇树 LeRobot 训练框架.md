---
title: 资料摘要：宇树 LeRobot 训练框架
type: source
tags: [机器人, 宇树, LeRobot, 教程, 深度]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://github.com/unitreerobotics/unitree_lerobot"
  - "https://github.com/unitreerobotics/unitree_rl_lab"
confidence: medium
source_url: "https://github.com/unitreerobotics/unitree_lerobot"
media: article
---

# 资料摘要：宇树 LeRobot 训练框架

> 宇树科技（Unitree Robotics）开源的 LeRobot 训练微调框架和 IsaacLab RL 训练框架，支持 G1 人形机器人数据采集、格式转换、策略训练和真机部署全流程。

## unitree_lerobot — LeRobot 训练

基于 LeRobot v2.0，新增 Unitree 机器人数据格式支持：

- **数据采集**：通过 [avp_teleoperate](https://github.com/unitreerobotics/avp_teleoperate) 遥操作采集 JSON 格式数据
- **数据处理**：提供数据集编辑器（PyQt5）裁剪/删除不合格片段
- **数据转换**：`convert_unitree_json_to_lerobot.py` 将 JSON → LeRobot 格式，支持多种机器人类型
- **支持策略训练**：ACT、Diffusion Policy、π0、Pi0.5、GR00T
- **真机评测**：`eval_g1.py` 支持真实机器人和 Sim 环境推理
- **数据回放**：在机器人上重放数据集离线验证

### 支持的机器人类型

`Unitree_Z1_Single`, `Unitree_Z1_Dual`, `Unitree_G1_Dex1`, `Unitree_G1_Dex3`, `Unitree_G1_Brainco`, `Unitree_G1_Inspire`, `Unitree_G1_Dex1_Sim`

## unitree_rl_lab — IsaacLab RL 训练

基于 IsaacLab 的宇树机器人强化学习环境，支持 Go2、H1、G1-29dof：

- 基于 RSL-RL 训练 RL 策略，`./unitree_rl_lab.sh -t --task Unitree-G1-29dof-Velocity`
- **Sim2Sim**：训练后导出 → Mujoco 验证策略效果
- **Sim2Real**：通过 `unitree_sdk2` 编译控制器部署真机
- 提供 URDF/USD 两种机器人模型加载方式

## 相关

- [[LeRobot]]
- [[Isaac Lab]]
- [[GR00T N1.5]]
- [[VLA（视觉-语言-动作模型）]]
- [[遥操作（Teleoperation）]]
- [[Wiki 目录]]
