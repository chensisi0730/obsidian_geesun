---
title: 资料摘要：智元 Genie Sim 3.0 仿真平台
type: source
tags: [机器人, 仿真, 智元, Genie Sim, 评测, 资讯, 工具]
created: 2026-05-20
updated: 2026-05-20
sources:
  - "https://zhuanlan.zhihu.com/p/272734689"
confidence: medium
source_url: "https://zhuanlan.zhihu.com/p/272734689"
media: article
---

# 资料摘要：智元 Genie Sim 3.0 仿真平台

> 智元机器人（Zhiyuan Robotics）发布 Genie Sim 3.0 一站式仿真开发平台，核心亮点为自然语言生成3D世界、五大维度评测基准（Benchmark）以及 RLinf 强化学习集成。

## 核心要点

### Genie Sim World — 语言造世界

- **图文生境**：无需建模、采集或硬件，文本/图片输入即可零门槛生成海量场景
- **极速生成**：空间世界模型单次推理即可完成构建，生成速度从小时级→分钟级
- **虚实一致**：RGB、深度、激光雷达等多模态数据原生同步输出

### Genie Sim Benchmark — 五大评测套件

| 套件 | 评测维度 |
|------|---------|
| Instruction | 指令跟随：形状/大小/颜色/逻辑理解 |
| Spatial | 空间理解：相对位置抓取/排序/叠放 |
| Manipulation | 操作执行：多场景原子操作技能 |
| Robust | 扰动适应：光照/背景/噪声/末端切换等 10+ 类 |
| Sim2Real | 训以致用：零样本真机迁移评测 |

**关键数据**：仿真训练的模型在仿真与真实世界评测差异 **< 10%**，VLM 基线（无机器人预训练）无法完成任何任务。

### Genie Sim × RLinf — 强化学习集成

- 双引擎联合：物理与渲染引擎解耦，1,000Hz 高精度物理模拟
- 大规模并行仿真，标准 Gym 接口
- 补足 VLA 模型短板：用低成本 RL 后训练实现"泛化理解→精准微操"

## 相关

- [[Isaac Lab]]
- [[VLA（视觉-语言-动作模型）]]
- [[遥操作（Teleoperation）]]
- [[OpenARM]]
- [[Wiki 目录]]
