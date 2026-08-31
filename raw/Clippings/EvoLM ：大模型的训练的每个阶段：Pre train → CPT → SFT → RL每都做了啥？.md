[收录于 · 论文解读](https://www.zhihu.com/column/c_1901627998102361614)

427 人赞同了该文章

目录

### 背景

来自NeurIPS 2025 oral的一篇文章，用 **100+ 个从头训的 1B/4B 模型** ，严格控制变量，把 **Pre train → CPT → SFT → RL** 每一步的收益、饱和点、遗忘效应、资源配比讲清除

![](https://pic3.zhimg.com/v2-b72b43824d80f1471870655bb53334fa_1440w.jpg)

EvoLM 概览：一个用于研究语言模型训练动态的透明模型套件，涵盖预训练、持续预训练（CPT）、监督微调（SFT）和强化学习（RL）。该框架在领域内（如数学）和领域外（如代码、逻辑）设置中评估上游（语言建模）和下游（问题解决）性能，从而实现对设计权\<br>衡和扩展行为的系统分析

### 待解决的问题

- 每一步到底在贡献什么
- 预训练要训到多少才不浪费
- CPT 能不能省
- SFT 训多久会过 [拟合](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E6%8B%9F%E5%90%88&zhida_source=entity)
- RL 到底对模型能力是不是有效提升
- SFT 与 RL 数据怎么分效果最好

## EvoLM实验设置

所有模型基于 **[LLaMA-2](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=LLaMA-2&zhida_source=entity) 架构** ，规模为 **1B / 4B** ，训练分为 **四个连续标准化阶段** ：

**预训练（Pretrain）**

- 语料： [FineWeb-Edu](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=FineWeb-Edu&zhida_source=entity)
- Token 范围：20 倍参数量（Chinchilla 最优）～ 320B

目标：研究 **轻度过训练** 与 **严重过训练** 对性能的影响。

**持续预训练（CPT）**

- 语料：FineMath（ [数学领域](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E6%95%B0%E5%AD%A6%E9%A2%86%E5%9F%9F&zhida_source=entity) ）
- Token 范围：2B～42B 策略：加入 **通用预训练 [数据回放](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%9B%9E%E6%94%BE&zhida_source=entity)** ，缓解灾难性遗忘。

**监督微调（SFT）**

- 数据：基于 GSM8K / MATH 扩充的数学问答（MetaMathQA、 [OpenMathInstruct2](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=OpenMathInstruct2&zhida_source=entity) 、NuminaMath）
- 处理：按 **模型一致性过滤低质量样本** 。

**强化学习（RL）**

- 算法：PPO
- 奖励：二元可验证奖励（正确/错误）
- 数据：与 SFT **同源但不重叠** 的独立样本集。

**评估** ：

- 上游（Upstream）：语言建模（零样本完形）
- 下游（Downstream）：域内数学（ID）、代码/逻辑/常识（和领域外分布，OOD）
- 指标：Pass@k / Maj@k / RM@k / ORM(使用结果奖励模型— [Skywork-Reward-Llama-3.1-8B-v0.2](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=Skywork-Reward-Llama-3.1-8B-v0.2&zhida_source=entity) 生成) / Correct Ratio

## 预训练 Pretrain，过量预训练会损害后训练

### 上游收益递减

随着预训练 token 上升，模型上游（语言建模）准确率持续提升，但在 **80B~160B** 后收益快速塌缩。

- 1B 模型：20B→80B 显著提升；80B→160B 几乎不再涨
- 4B 模型：饱和期间时间晚一点，320B之后不再涨
![](https://pic1.zhimg.com/v2-c363d1500dbb847484722811b03375f0_1440w.jpg)

Upstream 任务性能与模型预训练 token 数的关系{0.5B, 1B, 4B}-{10BT,20BT, 40BT, 80BT, 160BT, 320BT}

### 过度预训练伤泛化

![](https://pic4.zhimg.com/v2-49c9dced722bd3e34ad5b2a802b9b4ff_1440w.jpg)

SFT / SFT+RL 在 **80B 预训练** 即达饱和。 超过 **160B** 后：

- OOD 性能明显下降
- ORM 分数降低

> **Takeaway1**:过度的通用领域预训练并不总能提升特定领域的后训练，甚至可能在某些下游任务上导致性能退化（在我们的研究中，饱和现象发生在约 80 倍至 160 倍模型规模处）。说明 **过量预训练会损害后训练,损害 OOD 泛化** 。

## 持续预训练 CPT,被低估的阶段

CPT 是连接通用基座与领域能力的 **桥梁** ，也是 RL 能否生效的 **前提** 。

### 灾难性遗忘与回放

![](https://pic3.zhimg.com/v2-49d7871091b536ab81f7039ced769ea8_1440w.jpg)

纯领域 CPT 会导致 **通用能力急剧遗忘** 。 混入 **5% 通用回放数据** 可完美缓解。 最优配比： **8B 通用 + 42B 领域** 。

### CPT 充足，RL 才会强

![](https://pic1.zhimg.com/v2-2b93b09355c5007bd7fd2be8260d8282_1440w.jpg)

- 无 CPT：RL < SFT
- 有 CPT：RL ≫ SFT
- ID/OOD 同步上升，32B~42B 饱和

**Takeaway**

- 领域特定的后训练需要充足的领域特定CPT数据支持： **缺乏此类数据时，SFT性能仍不理想，RL甚至可能损害该性能**
- 随着领域特定CPT数据的增加，领域内下游性能稳步提升，且SFT模型能够从强化学习微调中获得更大收益
- 在拥有充足的领域特定 CPT 数据时， **在领域内任务上进行后训练不仅能提升领域内性能，还能有效泛化到 OOD 任务**
![](https://pica.zhimg.com/v2-e85e61b9f995361c72c3ba6608b8bf4e_1440w.jpg)

预训练模型在 GSM8K-Platinum 上的性能（Pass@1 准确率）1B-160BT使用不同配置进行持续预训练，然后使用 100K SFT 样本\<br>进行 1 轮微调

- 无 CPT：6.04%
- 纯领域 CPT：19.27%
- 回放最优：21.01%

## 监督微调 SFT不是越多越好

### SFT 轮次的影响

将 SFT 样本固定为 100K，对基础模型进行 {1, 2, 4, 8, 16, 32} 轮微调。

![](https://pic4.zhimg.com/v2-f195b435ea7437f4edd75819d5ca94bb_1440w.jpg)

- ID性能：8 轮饱和
- OOD：2~4 轮见顶，之后下降

> 过度的 SFT，尤其是过大的训练轮数，可能限制进一步的 RL 改进空间。

### SFT 样本量的影响

将SFT 样本数量从 50K 变化到 400K，同时将轮数固定为 1 以最小化记忆效应。

![](https://picx.zhimg.com/v2-e787140e4da21afebf3f604a147dff55_1440w.jpg)

- ID性能：样本越多越好
- OOD：波动甚至下降

> 样本越大，RL 越难带来增益。过多会牺牲泛化，并压缩 RL 上限。ID 性能随样本数量增加而单调提升，证实了额外的 SFT 计算量能够持续提升域内任务的性能。然而，OOD 指标出现波动，甚至可能随数据集增大而下降。与增加训练轮数类似，随着模型学习更多 SFT 样本，RL 带来的增量收益逐渐减弱

## 强化学习 RL的作用

### RL 轮次

![](https://pic4.zhimg.com/v2-6c635f86136a9c8f82df3f5f87689271_1440w.jpg)

- 峰值：8~16 轮
- Pass@16：4 轮后下降
- Correct Ratio 持续上升

> **RL 不提升推理边界，只提升已有正确路径的 [置信度](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E7%BD%AE%E4%BF%A1%E5%BA%A6&zhida_source=entity)** 。使用过多轮次或样本的 RL 能够提升 ID 和 OOD 任务上的下游性能，但存 在 [边际效益](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E8%BE%B9%E9%99%85%E6%95%88%E7%9B%8A&zhida_source=entity) 递减

### RL 样本量

![](https://pic1.zhimg.com/v2-a22764876de0a16c83ee0e33bb88dba0_1440w.jpg)

- 饱和：150K~200K
- 350K~400K 出现崩溃：生成长度超限

> 更稳定的方式是 **加轮次，不加样本** 。在 [饱和区间](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E9%A5%B1%E5%92%8C%E5%8C%BA%E9%97%B4&zhida_source=entity) 之外，RL 主要增加采样高质量 [rollout](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=rollout&zhida_source=entity) 的概率，但不一定提升模型的基础推理能力

### SFT/RL 分配

为了进一步研究如何在数据受限场景下配置 SFT 和 RL 的数据分配，从总共 500K 数据集中子采样 100K 样本，评估五种 SFT/RL 划分比例：(10 / 90, 30 / 70, 50 / 50, 70 /30, 90 / 10) K，并进行 4 轮的 SFT 或 RL 训练，选择 100K 是因为这接近 ID 和 OOD性能的饱和区间

![](https://pic3.zhimg.com/v2-9884c2137ce06cface66a81774215200_1440w.jpg)

总数据 100K：

- 重 ID：70K SFT + 30K RL
- 重 OOD：10K SFT + 90K RL

> 在受限的下游数据预算下，将更多样本分配给 SFT 可最大化域内收益，但会牺牲 OOD [泛化能力](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E6%B3%9B%E5%8C%96%E8%83%BD%E5%8A%9B&zhida_source=entity) ；而将更多样本分配给 RL 则能提升 OOD 性能。RL 是“稳定器”不是“创造器”；想要泛化强 → 多给 RL；想要 [垂直精度](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=1&q=%E5%9E%82%E7%9B%B4%E7%B2%BE%E5%BA%A6&zhida_source=entity) → 多给 SFT。

## 两个关键发现

### 中间 checkpoint 完全不可靠

在实际场景中，工程师通常使用完整的学习调度训完所有训练语料的模型，而不是将中间checkpoint作为最终模型，比如：本来要训 160B token，结果只训到 20B、40B 就停了。把这个中间checkpoint拿来当 “训了 20B/40B 的最终模型”

![](https://pic4.zhimg.com/v2-c63ee47dcdb249d9817852d84c04184d_1440w.jpg)

不同预训练配置下上游任务和 MATH（Level 1 和 Level 2）的性能表现。”xBT 完整训练” 指在 xBT 上的完整预训练运行，而”xBT\<br>中间检查点” 指训练过程中在 160B token 处获取的中间检查点，对应于目前为止已见的xBT 数据量。

本文实验中从 160B run 切出 40B checkpoint，性能 **远低于** 专门训 40B 的模型。 原因： **没有完整学习率衰减，优化轨迹不完整** 。

结论：研究training dynamics时 **严禁使用中间 [checkpoint](https://zhida.zhihu.com/search?content_id=274112902&content_type=Article&match_order=5&q=checkpoint&zhida_source=entity) 作为替代品** 。

### ORM评估比PPL困惑度更可靠

![](https://picx.zhimg.com/v2-5e0a6307b8a7bddc2270a9bac8d67001_1440w.jpg)

不同任务中准确率与 ORM 分数的相关性每个子图代表一个数据集，每个点对应一个模型变体。虚线表示线性趋势，Pearson 相关系数标注于各子图标题中

![](https://pic4.zhimg.com/v2-df285f5ad331bdbf0d8d60f1f2066315_1440w.jpg)

准确率与验证集 PPL 在不同任务上的相关性。每个子图代表一个数据集，其中每个点对应一个后训练模型变体。虚线表示线性趋势，Pearson 相关系数标注在每个子图的标题中。

后训练模型 PPL 与下游性能几乎无关（r²≈0）。 ORM 与准确率强相关 **r²=0.62~0.84** 。

> **后训练必须用 ORM 评估，不要再用困惑度** 。与验证损失相比，ORM 分数可作为更可靠的无监督验证指标，有助于预测后训练阶段的下游任务性能

[arxiv.org/pdf/2506.1602](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2506.16029)

编辑于 2026-05-01 22:30・广东[一文告诉你人工智能纯小白学习路线！](https://zhuanlan.zhihu.com/p/31863323446)

[

全文5196字，按照我这个路线坚持完，你会变成一个人工智能的牛人的。它是假定一个没有人工智能基础的程序员学习路线。写在前面：我觉的从deepseek开源以后，会有更多的企业和开发者争...

](https://zhuanlan.zhihu.com/p/31863323446)

赞同 427