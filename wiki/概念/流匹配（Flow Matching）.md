---
title: 流匹配（Flow Matching）
type: concept
tags: [概念, 机器人, 生成模型, 基础]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "[[资料摘要：SmolVLA 轻量级 VLA 模型]]"
  - "[[资料摘要：LeRobot π0 封装剖析]]"
confidence: medium
related_concepts:
  - Diffusion Policy
  - ACT
  - VLA
---

# 流匹配（Flow Matching）

流匹配是一种生成式建模方法，通过学习从噪声分布到数据分布的平滑概率路径（probability flow），生成连续数据（如机器人动作序列）。与扩散模型（Diffusion Model）同属基于得分的生成模型家族，但流匹配直接学习向量场（vector field），而非得分函数。

## 核心思想

- **概率路径**：定义从简单分布（高斯噪声）到目标数据分布的连续变换
- **向量场学习**：学习一个时间相关的向量场 $v_t(x)$，描述每个时间步 $t$ 的"流动方向"
- **ODE 采样**：训练后通过求解常微分方程 $\frac{dx}{dt} = v_t(x)$ 从噪声生成数据

## 与扩散模型的对比

| 维度 | 扩散模型 | 流匹配 |
|------|---------|--------|
| 前向过程 | 逐步加噪到高斯 | 线性插值到高斯 |
| 学习目标 | 得分函数 $\nabla \log p_t(x)$ | 向量场 $v_t(x)$ |
| 采样步数 | 通常较多（50-1000） | 可少至几步 |
| 轨迹 | 随机（SDE） | 确定性（ODE） |

## 在机器人领域的应用

- **π0**：Physical Intelligence 的 VLA 模型使用流匹配生成动作序列，3B 参数
- **SmolVLA**：轻量级 VLA 使用流匹配动作专家（Flow Matching Action Expert）
- 优势：生成动作更平滑、采样步数少、适合实时控制

## 相关

- [[VLA（视觉-语言-动作模型）]]
- [[ACT（动作分块变换器）]]
- [[Wiki 目录]]
