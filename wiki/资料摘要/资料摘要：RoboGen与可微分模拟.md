---
title: 资料摘要：RoboGen与可微分模拟
type: source
tags: [机器人, 生成式模拟, 可微分物理, 数据]
created: 2026-06-09
updated: 2026-06-09
sources:
  - "EP8对话淦创、周衔：RoboGen如何通过生成模型和可微分模拟大规模合成机器人示教数据"
confidence: high
source_url: https://mp.weixin.qq.com/s/EP8
media: article
---

# 资料摘要：RoboGen与可微分模拟

> Robot Data 系列第八期访谈——淦创（UMass 教授、MIT-IBM 研究经理）和周衔（CMU 机器人所博士毕业生）讨论 RoboGen 全自动数据生成流水线，以及可微分物理引擎的原理和在机器人学习中的应用。

## 核心要点

- **RoboGen**：通过生成式模拟自动学习多种机器人技能，最小化人工干预
- **四阶段流水线**：任务提议 → 场景生成 → 训练监督生成 → 技能学习（propose-generate-learn 循环）
- **底层依赖 Genesis**：RoboGen 的场景生成、可微分模拟均基于 Genesis 物理引擎
- **可微分物理引擎**：比 RL 效率高几个数量级，用梯度下降替代搜索
- **柔性物体处理的必要性**：刚体可用 RL，但处理衣物、流体等必须用可微分模块
- **多项支撑工作**：PlastineLab（21 年）、Roboninja、FluidLab、SoftZoo、ThinSellLab 等

## 详细笔记

### RoboGen 流水线

```
任务提议 (LLM + in-context learning)
    ↓
场景生成 (3D assets + 空间配置 + 干扰项)
    ↓
训练监督生成 (LLM 生成奖励函数代码)
    ↓
技能学习 (RL / 轨迹优化 → 策略 / 轨迹)
```

**任务提议**：LLM 基于物体语义信息自主提出新任务（如看到微波炉→提出"加热食物"）

**场景生成**：3D assets（text-to-3D / image-to-3D）按有意义的空间配置摆放 + 干扰项增加多样性。新方向：3D GS 用于 Real2Sim

**训练监督生成**：LLM 将抽象任务（如"向前跑"）转化为代码形式的奖励函数

**技能学习**：复杂任务→RL 策略；简单任务→路径规划轨迹。多条轨迹可通过模仿学习蒸馏为通用策略

### 可微分物理引擎原理

```
传统物理引擎：状态 → 动作 → 新状态 (不可导) → 搜索/RL
可微分物理引擎：状态 → 动作 → 新状态 (每步可导) → 梯度下降优化
```

可微分（Differentiable） = 每一步都用数学公式精确计算梯度，用梯度做优化，结果更好更快。

### 关键研究成果链

```
PlastineLab (2021) ← 与胡渊明 Taichi 合作
    ↓ 从可微分模拟到机器人技能学习
Roboninja (切割各种物体) ← 与 Shuran 组合作  
    ↓ 仿真无限数据 → Policy → Real World Transfer
FluidLab / SoftZoo / ThinSellLab
    ↓
Genesis (统一可微分物理引擎平台)
    ↓
RoboGen (全自动生成式模拟)
```

### 策略 vs 轨迹

| | 策略 (Policy) | 轨迹 (Trajectory) |
|--|-------------|-------------------|
| 定义 | 闭环控制系统，根据状态采取行动 | 固定状态下每一步动作预先确定 |
| 适合 | 复杂任务 | 简单任务 |
| 生成方式 | RL 训练 | 路径规划 |
| 蒸馏 | 多条轨迹→模仿学习→通用策略 | 单条记录 |

## 引用与数据

- RoboGen 发表于 CoRL 2023
- 可微分物理引擎与胡渊明 Taichi 框架合作
- 可微分模拟技能学习效率比 RL 高数个数量级

## 相关

- [[资料摘要：Genesis 物理引擎]]
- [[资料摘要：机器人跑酷与数据来源]]
- [[可微分物理引擎（Differentiable Physics）]]
- [[Genesis]]
- [[Isaac Lab]]
- [[PPO（近端策略优化）]]
- [[Wiki 目录]]
