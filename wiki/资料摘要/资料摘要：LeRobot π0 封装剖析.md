---
title: 资料摘要：LeRobot π0 封装剖析
type: source
tags: [机器人, LeRobot, π0, VLA, 深度, 教程]
created: 2026-05-19
updated: 2026-05-19
sources:
  - "https://blog.csdn.net/v_JULY_v/article/details/146304377"
confidence: medium
source_url: "https://blog.csdn.net/v_JULY_v/article/details/146304377"
media: article
---

# 资料摘要：LeRobot π0 封装剖析

> v_JULY_v 对 LeRobot 中 π0 策略封装的深度源码剖析。π0 是 Physical Intelligence 开发的 VLA 模型（3B 参数），LeRobot 中为 JAX → PyTorch 的移植版本。

## 核心要点

**LeRobot pi0 vs 官方 openpi 区别：**

| 维度 | openpi (官方) | LeRobot pi0 |
|------|-------------|-------------|
| 框架 | JAX | PyTorch (HuggingFace 移植) |
| 生态 | 独立库 | LeRobot 框架组件 |
| 接口 | 手动配置 | `PI0Policy.from_pretrained("lerobot/pi0")` |
| 优化 | — | KV cache、flex attention、GQA |
| 适配 | 通用 | 含 Aloha 特殊处理、空相机支持 |

**核心架构：**
- 基于 PaliGemma 视觉语言模型 + Gemma 专家模型（18 层、1024 隐藏单元）
- 流匹配（Flow Matching）生成机器人动作序列
- 支持分组查询注意力 (GQA) 优化推理
- 三种注意力实现：eager（兼容）、fa2（快速）、flex（灵活）

**转换工具**：将原始 JAX/Orbax 格式权重转换为 PyTorch 格式，支持三种模型变体（pi0_base、pi0_aloha_sim、pi0_aloha_towel）

## 相关

- [[VLA（视觉-语言-动作模型）]]
- [[ACT（动作分块变换器）]]
- [[资料摘要：π*0.6 真实世界强化学习]]
- [[资料摘要：LeRobot 框架架构剖析]]
- [[资料摘要：LeRobot系列专栏索引]]
- [[Wiki 目录]]
