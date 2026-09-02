---
title: 资料摘要：NVIDIA Cosmos 世界模型
type: source
tags: [世界模型, NVIDIA, 机器人, 仿真]
created: 2026-06-09
updated: 2026-06-09
sources:
  - "NVIDIA Cosmos 官方页面"
confidence: high
source_url: https://www.nvidia.com/en-us/ai/cosmos/
media: article
---

# 资料摘要：NVIDIA Cosmos 世界模型

> NVIDIA 官方产品页面介绍的 Cosmos 3——首个具备原生推理、世界和动作生成功能的全能世界基础模型，用于加速物理 AI 开发。

## 核心要点

- **Cosmos 3**：基于 Mixture-of-Transformers 架构的全能模型，支持视觉推理、世界模拟和动作生成
- **四大核心能力**：
  1. **视觉 AI 推理**（VLM 模式）——质检、安防、物流等场景的实时检测与密集字幕
  2. **策略模型构建**（WAM 骨干）——为机器人策略学习加速
  3. **世界模拟**——可控的物理世界模拟器，预测多种方法并收敛到正确行为
  4. **合成视频数据生成**——从文本/图像/视频/声音/动作输入生成无限可能的未来情景
- **开放生态系统**：提供数据处理（Cosmos Curator）、训练加速、评估框架和 Agent Skills

## 详细笔记

### 架构与能力

Cosmos 3 的独特之处在于它是一个 **Omni-Model**（全能模型），单个模型同时具备：

1. **作为 VLM**：理解复杂现实场景中的物体、交互和意图
2. **作为 WAM**（World Action Model）：作为机器人策略学习的骨干网络，将预学习动作适应特定任务
3. **作为世界模拟器**：在闭环中预测和评估多种方法
4. **作为视频生成器**：扩展合成训练数据

### 与机器人学习的关系

Cosmos 3 特别强调其在**机器人策略学习**中的应用——作为 WAM 骨干，可在专门的相机和具身数据上后训练，使策略模型能够大规模适应特定任务。

### 开放工具链

| 工具 | 功能 |
|------|------|
| Cosmos Curator | 快速筛选、注释、去重大量传感器数据 |
| Cosmos Evaluator | 大规模审核和评分成品视频输出 |
| 后训练/优化框架 | 快速构建和部署世界模型 |
| Agent Skills | 将编程智能体转为合成数据专家 |

### 硬件优化

Cosmos 3 针对 NVIDIA Blackwell GB200 / RTX PRO 6000 系列优化，实现训练、合成数据生成、仿真和推理的峰值性能。

### 行业采用

已被机器人、自动驾驶和工业视觉 AI 领域的领先开发者采用。

## 相关

- [[NVIDIA Cosmos]]
- [[Isaac Lab]]
- [[VLA（视觉-语言-动作模型）]]
- [[资料摘要：Genesis 物理引擎]]
- [[Wiki 目录]]
