---
title: 资料摘要：ACT算法精讲
type: source
tags: [机器人, 模仿学习, ACT, 教程, 深度]
created: 2026-05-01
updated: 2026-05-01
sources:
  - "https://zhuanlan.zhihu.com/p/677625871"
confidence: medium
source_url: "https://zhuanlan.zhihu.com/p/677625871"
media: article
---

# 资料摘要：ACT算法精讲

> 知乎文章深入讲解 ACT（Action Chunking with Transformers）算法——ALOHA 遥操作平台使用的模仿学习算法，由 ALOHA 团队于 2023 年 4 月提出，针对精细操作任务表现优异。

## 核心要点

- **ACT = Action Chunking with Transformers**：将动作序列分块预测，配合时序集成（Temporal Ensemble）缓解模仿学习中的复合误差（Compounding Error）
- **CVAE 架构**：使用条件变分自编码器处理多模态示教数据，训练时为生成式模型，推理时隐变量置零退化为确定性策略
- **80M 参数量**，单任务模型在 2080Ti（11G）上 5 小时训练完成，推理仅 0.01 秒
- **关键发现**：L1 loss 优于 L2；预测绝对关节角优于相对变化
- 输入含 4 个相机视角（480×640）和关节角，输出为分块后的关节角绝对值

## 详细笔记

### 问题背景：复合误差（Compounding Error）

测试时若 agent 遇到训练集未覆盖的状态，预测产生误差，导致进入更陌生的状态，误差逐步累积。ACT 通过两种手段缓解：

1. **Action Chunking**：每 k 步获取一次输入，预测后续 k 步动作并顺序执行。轨迹长度缩短为 1/k，但固定分块会导致不连续运动
2. **Temporal Ensemble**：每步都预测后 k 步动作，对 k 次重叠预测加权平均（权重指数衰减），使运动更平滑。权重公式 $w_i = \exp(-m \cdot i)$，m 控制融合速度

### 模型结构：CVAE

- **Encoder**：输入 [cls] + 观测关节角 + 示教动作序列（不含图像），输出隐变量 z 的均值和方差。训练后弃用
- **Decoder（即最终 Policy）**：输入观测（关节角 + ResNet18 编码的图像）+ Encoder 输出的 z（推理时 z=0），输出分块后的绝对关节角
- **训练目标**：模仿学习 Loss（分块版行为克隆）+ VAE Loss（L1 Reconstruction + KL 正则）
- **网络**：Encoder 和 Decoder 均使用 Transformer

### 相关背景：ALOHA 平台

- 2 个主动臂（Leader）+ 2 个从动臂（Follower），各 6 DOF + 1 DOF 夹爪
- 4 个相机（2 腕部 + 1 上方 + 1 前方）
- 训练时从动臂关节角 + 图像为观测，主动臂关节角为动作标签

### 超参数（参考）

- Chunk size: k（文中未精确给出，需参考论文）
- Clip range: 类似 PPO 的截断机制
- VAE 隐变量维度等

## 引用与数据

- 论文：[ACT: Action Chunking with Transformers](https://arxiv.org/abs/2304.13705)
- 项目：[ALOHA](https://tonyzhaozh.github.io/aloha/)
- 项目：[Mobile ALOHA](https://mobile-aloha.github.io/)

## 相关

- [[ACT（动作分块变换器）]]
- [[资料摘要：SOArm101 仿真与机械臂策略训练]]
- [[Wiki 目录]]
