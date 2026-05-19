---
title: 资料摘要：LeRobot 官方教程
type: source
tags: [机器人, 教程, LeRobot, 模仿学习]
created: 2026-05-02
updated: 2026-05-02
sources:
  - "https://hugging-face.cn/docs/lerobot/il_robots"
confidence: high
source_url: "https://hugging-face.cn/docs/lerobot/il_robots"
media: article
---

# 资料摘要：LeRobot 官方教程

> Hugging Face 官方教程，讲解如何在真实机器人上进行模仿学习，从数据收集到策略训练再到部署评估的完整流程。

## 核心要点

- **完整流水线**：遥操作 → 数据录制 → 策略训练 → 推理评估
- **支持硬件**：SO101、Koch、Aloha 等主流机器人平台
- **策略模型**：ACT、Diffusion Policy、π0 等
- **数据管理**：自动上传 Hugging Face Hub，支持恢复和检查点

## 详细笔记

### 遥操作流程

```bash
lerobot-teleoperate \
   --robot.type=so101_follower \
   --robot.port=/dev/tty.usbmodem58760431541 \
   --robot.id=my_awesome_follower_arm \
   --teleop.type=so101_leader \
   --teleop.port=/dev/tty.usbmodem58760431551 \
   --teleop.id=my_awesome_leader_arm
```

### 录制数据集

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 1920, height: 1080, fps: 30}}" \
    --dataset.repo_id=${HF_USER}/so101_test \
    --dataset.num_episodes=50 \
    --dataset.single_task="Grab the black cube"
```

### 训练策略

```bash
lerobot-train \
  --dataset.repo_id=${HF_USER}/so101_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --policy.device=cuda \
  --wandb.enable=true
```

### 运行推理

```bash
lerobot-record \
  --robot.type=so100_follower \
  --policy.path=${HF_USER}/my_policy \
  --dataset.repo_id=${HF_USER}/eval_so100
```

### 数据收集技巧

- 至少 50 个回合，每个位置 10 个回合
- 保持摄像头固定，抓取行为一致
- 物体在摄像头可见
- 避免过快添加太多变化

### 键盘控制

| 按键 | 功能 |
|------|------|
| → | 提前结束当前回合 |
| ← | 取消当前回合重新录制 |
| ESC | 停止会话，编码并上传 |

## 引用与数据

- 框架：[LeRobot](https://github.com/huggingface/lerobot)
- 可视化工具：[Rerun](https://rerun.io/)
- 训练监控：[Weights & Biases](https://wandb.ai/)

## 相关

- [[LeRobot]]
- [[Isaac Lab]]
- [[遥操作（Teleoperation）]]
- [[ACT（动作分块变换器）]]
- [[资料摘要：Isaac Lab 训练 LeRobot SO-101 机械臂]]
- [[Wiki 目录]]
