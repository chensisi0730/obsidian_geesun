[收录于 · 硅基进化](https://www.zhihu.com/column/c_1771274012468887552)

74 人赞同了该文章

目录

最近准备系统性地梳理一下 [具身智能](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%85%B7%E8%BA%AB%E6%99%BA%E8%83%BD&zhida_source=entity) （Embodied [AI](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=AI&zhida_source=entity) ）领域的脉络。但面对铺天盖地的论文，如果只是按时间线罗列「谁又刷了什么 [SOTA](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=SOTA&zhida_source=entity) 」，会非常枯燥，也很难抓住问题的本质。

按照我的认知习惯，我打算按专题去写一系列「软综述」——不追求大而全地覆盖所有业界的 [工程实践](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%B7%A5%E7%A8%8B%E5%AE%9E%E8%B7%B5&zhida_source=entity) ，而是重点把关键的问题、逻辑转折和技术里程碑讲清楚，建立一个宏观的问题框架。

这一篇，我们先聊聊当前机器人基础模型的第一条主线： **Vision-Language-Action (VLA)** 。

VLA 的历史，其实就是机器人动作在「大模型范式」和「 [物理控制现实](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%89%A9%E7%90%86%E6%8E%A7%E5%88%B6%E7%8E%B0%E5%AE%9E&zhida_source=entity) 」之间来回折中的历史。读懂了这条线，你就能明白大模型到底是如何一步步学会控制机器人的。

![](https://pic4.zhimg.com/v2-fd355817b5adf764ceafb0d86025aa23_1440w.jpg)

我来，我见，我征服

## 一、为什么机器人需要 VLA？

在讨论 VLA 是什么之前，我们要先弄明白：如果没有它，机器人是怎么工作的？

在传统的机器人学习范式里， [控制策略](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%8E%A7%E5%88%B6%E7%AD%96%E7%95%A5&zhida_source=entity) （Policy）主要解决的是 （从观察到动作）的 [映射](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%98%A0%E5%B0%84&zhida_source=entity) 。如果你给机器人下达一个指令： `把桌上的红色杯子放进抽屉` ，传统的 [系统工程](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%B3%BB%E7%BB%9F%E5%B7%A5%E7%A8%8B&zhida_source=entity) 必须把这个任务拆解成一条漫长且高度模块化的流水线：

1. **[视觉感知](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%A7%86%E8%A7%89%E6%84%9F%E7%9F%A5&zhida_source=entity)** ：识别出红色的杯子和抽屉。
2. **[状态估计](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%8A%B6%E6%80%81%E4%BC%B0%E8%AE%A1&zhida_source=entity)** ：计算杯子的 3D 坐标和姿态。
3. **任务规划** ：决定先抓杯子，再拉抽屉，最后放置。
4. **[运动规划](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%BF%90%E5%8A%A8%E8%A7%84%E5%88%92&zhida_source=entity)** ：计算 [机械臂](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E6%A2%B0%E8%87%82&zhida_source=entity) 在空间中不发生碰撞的运动轨迹。
5. **底层控制** ：将轨迹转化为各个电机的 [力矩](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%8A%9B%E7%9F%A9&zhida_source=entity) 输出，控制夹爪闭合。

这种「流水线」式的架构很容易出现 **[级联误差](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%BA%A7%E8%81%94%E8%AF%AF%E5%B7%AE&zhida_source=entity)** 。只要感知模块识别出的 [边界框](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%BE%B9%E7%95%8C%E6%A1%86&zhida_source=entity) 偏了一厘米，或者 [运动学](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%BF%90%E5%8A%A8%E5%AD%A6&zhida_source=entity) 求解出了微小的偏差，最终的抓取就可能失败。

更重要的是，这种系统是「任务专用」的——换一个蓝色的碗，或者换一个不同形状的抽屉，可能就要重写规则或重新采集数据。

与此同时， [大语言模型（LLM）](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%EF%BC%88LLM%EF%BC%89&zhida_source=entity) /视觉语言模型（VLM）的浪潮几乎已经席卷了人工智能领域的每一个角落。正如过去几年我们亲眼所见，这些「大模型」已经具备了令人惊叹的常识和逻辑能力。如果你拍一张桌子的照片问： `我想喝水，我该拿哪个？` 今天的主流 VLM 已经能轻松回答出： `你应该拿起那个红色的杯子。`

但问题是，VLM 有知识，却不会动。它输出的是文本（Language Token），不是控制电机的电压或关节角度。

在这个背景之下，一个技术问题自然浮现：我们能否用大模型赋能机器人，让它的理解能力变成真实世界里的动作能力？

这就是 VLA 范式诞生的初衷。

## 二、VLA 到底是什么？

VLA，全称 **Vision-Language-Action** 。顾名思义，它试图把视觉语义理解、语言指令理解和机器人动作生成，统一到同一个可扩展的数据与模型框架中。

在 VLA 的视角下，之前的复杂流水线被压缩成了以下简洁的映射：

它直接读取当前的 [图像像素](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%9B%BE%E5%83%8F%E5%83%8F%E7%B4%A0&zhida_source=entity) ，听懂你的自然语言指令，然后输出机器人下一步动作的轨迹或目标位姿。

这个定义其实非常宽泛，为了让我们的讨论不要过于发散，在这篇文章中，我会按照业界主流的提法，给它一个系统边界。

首先，VLA 的本质是一个 **Policy（ [策略模型](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%AD%96%E7%95%A5%E6%A8%A1%E5%9E%8B&zhida_source=entity) ）** ，也就是它名字中 A（动作）所代表的含义。它的所有视觉和语言理解能力，都是为了最终输出那个能在物理世界执行的动作。

其次，在主流语境下的 VLA，主要指的是 **语义增强的无模型（Model-free）策略** ——注意这里「无模型」不是说模型没有 [世界知识](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E4%B8%96%E7%95%8C%E7%9F%A5%E8%AF%86&zhida_source=entity) ，而是说它不显式学习 这样的状态转移模型，也不主要依赖内部 rollout 来选动作。

因此，这篇文章，我们就不涵盖可以做显式自我推演和规划的「 [世界模型](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E4%B8%96%E7%95%8C%E6%A8%A1%E5%9E%8B&zhida_source=entity) 」路线。正如我们在之前的 [Cosmos](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Cosmos&zhida_source=entity) 系列中指出的那样，这两种路线并不对立，但为了理解更加清晰简洁，我们先做一下这样的定义收窄。

另外， **VLA 不是完整的机器人智能** 。仅仅靠一个 VLA，并不能让机器人直接进入开放世界玩耍。在真正落地的 [机器人系统](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E7%B3%BB%E7%BB%9F&zhida_source=entity) 栈中，VLA 处在中间层：

- 在它之上，需要有高层推理（Reasoning）模型来做长视野的任务分解。
- 在它之下，需要有底层的高频控制器（Controller）来保证物理安全性，处理打滑、碰撞等 VLA 反应不过来的微观物理反馈。

明确了「VLA 主要解决的是策略问题」这个前提后，我们就可以正式进入这篇 [技术史](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%8A%80%E6%9C%AF%E5%8F%B2&zhida_source=entity) 的正题了。

## 三、第一块拼图：RT-1 与动作 Token 化

在 RT-1 出现之前，大部分 [端到端](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%AB%AF%E5%88%B0%E7%AB%AF&zhida_source=entity) 的机器人动作预测，被建模为一个连续空间里的回归（Regression）问题。

但 Google 的研究人员换了一个思路：既然 [Transformer](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Transformer&zhida_source=entity) 在离散的文本 Token 序列上取得了如此巨大的成功，我们能不能把机器人的动作也变成序列？

![](https://pica.zhimg.com/v2-e5941fc420a8a1264ffd0ad33852db86_1440w.jpg)

RT-1架构图

如上图所示，RT-1 的基本架构组成如下：

- **[多模态](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%A4%9A%E6%A8%A1%E6%80%81&zhida_source=entity) 输入** ：模型的输入是连续的相机图像（Images）和自然语言指令（Instruction）。
- **[特征提取](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E6%8F%90%E5%8F%96&zhida_source=entity) 与融合 (FiLM + [EfficientNet](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=EfficientNet&zhida_source=entity))** ：图像并没有直接输入 Transformer，而是先经过 EfficientNet 提取视觉特征。这里的关键是引入了 **FiLM (Feature-wise Linear Modulation)** 层，它让自然语言指令在这个阶段就参与到视觉特征的处理中，使得模型能根据指令「有侧重地」观察图像。
- **[特征压缩](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%89%B9%E5%BE%81%E5%8E%8B%E7%BC%A9&zhida_source=entity) (TokenLearner)** ：这是 RT-1 能在真实机器人上跑到 **3 Hz** 频率的关键。 [高维](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E9%AB%98%E7%BB%B4&zhida_source=entity) 的视觉-语言融合特征被 TokenLearner 压缩成少量的紧凑 Token，大幅降低了后续 Transformer 的计算开销（整个模型仅 35M 参数）。
- **[序列建模](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%BA%8F%E5%88%97%E5%BB%BA%E6%A8%A1&zhida_source=entity) (Transformer)** ：Transformer 接收这些压缩后的 Token，进行自回归建模。
- **离散化动作输出 (Action)** ：模型的输出并不是连续的力矩或坐标，而是被 [离散化](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=2&q=%E7%A6%BB%E6%95%A3%E5%8C%96&zhida_source=entity) （Discretized）的 Action Token，分别对应机械臂（Arm）、底盘（Base）和模式（Mode）的控制指令。

如果只记住一个关键点的话，那我觉得就是 **动作离散化** 。RT-1 将连续的物理动作（例如机械臂末端在 6 自由度空间中的移动和夹爪开合）切分成了一个个均匀的格子（Bins）。

例如：

- 原本连续的移动 ，被硬编码映射为 `Token 158`
- 夹爪闭合的操作，被映射为 `Token 255`

通过这种离散化，动作被强行翻译成了 Transformer 能够理解和输出的 Action Token。机器人的控制问题，被成功改写成了类似 [机器翻译](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E7%BF%BB%E8%AF%91&zhida_source=entity) 的 **[自回归序列](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%87%AA%E5%9B%9E%E5%BD%92%E5%BA%8F%E5%88%97&zhida_source=entity) 建模问题** 。这是 [VLA 路线](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=VLA+%E8%B7%AF%E7%BA%BF&zhida_source=entity) 的第一块基石。

但需要注意的是，这时的 Action Token 并不等同于 Language Token，只是被离散化了的一种符号。RT-1也并没有利用预训练VLM的先验世界知识，它是从头训练的。只不过在架构上，它已经是一个具有VLA雏形的、基于Transformer 的 Policy 模型了。

## 四、第二块拼图：RT-2 与真正的 VLA

如果说 RT-1 证明了动作可以被 Token 化，那么 RT-2 才真正宣告了 VLA 概念的成立。

在推进机器人大模型时，业界遇到了一个致命的瓶颈： **真实的机器人轨迹数据太少了。** 相比于互联网上海量的文本和图像，让真实的机器人在实验室里抓杯子攒下来的数据，简直是九牛一毛。但与此同时，视觉语言模型（VLM）已经通过阅读整个互联网的图文，学到了极其丰富的世界知识。

RT-2 试图回答的核心问题就是：能不能通过少量机器人动作数据，把 VLM 从互联网图文中学到的语义知识迁移到 [机器人控制](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E6%8E%A7%E5%88%B6&zhida_source=entity) 中，让机器人在未见过的物体、概念和指令组合上表现出更强泛化？

为了实现这个目标，RT-2 做了一个巧妙（甚至有些暴力）的设计：它把机器人的 [离散动作](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%A6%BB%E6%95%A3%E5%8A%A8%E4%BD%9C&zhida_source=entity) 写成 tokenizer 可以处理的文本字符串，让动作进入 VLM 的语言生成接口。

我们来做一个直观的对比。对于同一张图像，原本的 VLM 输出的是自然语言描述：

> `"The object is a dinosaur."`

而 RT-2 被训练后，输出的则是一串伪装成文本的数字：

> `"1 128 091 241 005 101 127 217"`

这串数字对人类来说毫无意义，但RT-2通过互联网图文数据与机器人动作数据的联合微调，让它成功进入了语言模型的 Token 空间。大模型就像生成一句话一样，自回归地生成了这串代表 [空间坐标](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%A9%BA%E9%97%B4%E5%9D%90%E6%A0%87&zhida_source=entity) 和夹爪状态的动作指令。

![](https://pic2.zhimg.com/v2-3f22e85b7d2c69cf951e0f7b03a1c459_1440w.jpg)

RT-2架构图

RT-2的架构如上图所示：

- 左侧展示了互联网图文数据与机器人动作数据的联合微调
- 中间揭示了 VLM 如何像生成文字一样输出动作 Token，并通过反词元化（De-Tokenize）还原为物理控制量
- 右侧则展现了这种范式在真实物理世界中涌现出的强大语义推理与复杂任务执行能力。

RT-2 的核心贡献，是让机器人把互联网的语义知识转化为了物理动作能力。

想象一下，你对机器人下达指令： `"Pick up the extinct animal"` （捡起那个灭绝的动物）。 在传统流程里，这几乎是不可能完成的任务，因为机器人根本不知道什么是「灭绝的动物」。但 RT-2 能够做到，它的内在 [逻辑链条](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E9%80%BB%E8%BE%91%E9%93%BE%E6%9D%A1&zhida_source=entity) 是：

1. **依靠 VLM 强大的语义推理：** 识别出桌上的绿色塑料玩具是一只恐龙，并知道恐龙是灭绝的动物。
2. **依靠动作 Token 化的能力：** 将对恐龙玩具的空间定位，无缝翻译成 `"1 128..."` 这样的抓取动作序列。

## 五、第三块拼图：RT-X 与机器人的「FLAN 时刻」

RT-2 成功地将动作伪装成「外语」，接入了 VLM 的大脑。但即便模型再聪明，没有足够多、足够杂的机器人实操数据，它依然无法泛化到未见过的场景和不同的机器人本体上。

这就引出了 VLA 发展史上的第三个关键问题：机器人的数据，能不能像 [自然语言处理（NLP）](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E5%A4%84%E7%90%86%EF%BC%88NLP%EF%BC%89&zhida_source=entity) 的数据一样，把所有任务混在一起训练？

在 NLP 领域，有一个著名的被称为 FLAN 的工作：研究人员发现，如果把各种不同的 NLP 任务（翻译、问答、摘要等）转换成统一的指令微调（Instruction Tuning）格式，并混合在一起训练，语言模型就会在 **未见过的任务上** 展现出惊人的零样本（ [Zero-shot](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Zero-shot&zhida_source=entity) ）泛化能力。

RT-X 和它背后的 Open X-Embodiment (OXE) 数据集，就是想在 [机器人领域](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E9%A2%86%E5%9F%9F&zhida_source=entity) 复刻这个奇迹。

![](https://pica.zhimg.com/v2-33f60262af0aaebf54002c19df09ca1e_1440w.jpg)

Open X-Embodiment 数据集概览：来自全球 21 个机构的 34 个实验室，贡献了涵盖 22 种不同机器人本体、100 万条轨迹、527 种技能的海量实操数据。

但机器人的数据混训，比 NLP 难得多。文本在不同任务中都是 [ASCII 码](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=ASCII+%E7%A0%81&zhida_source=entity) 或 Unicode，而机器人的数据之间存在着巨大的 **Embodiment Gap（本体差异）** ：

- **[传感器](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E4%BC%A0%E6%84%9F%E5%99%A8&zhida_source=entity) 差异** ：有的机器人用单目相机，有的用深度相机（ [RGB-D](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=RGB-D&zhida_source=entity) ），安装视角也千差万别。
- **硬件与 [URDF](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=URDF&zhida_source=entity) （统一机器人描述格式）差异** ：不同机械臂的运动学连杆长度、关节数量完全不同。
- **[执行器](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%89%A7%E8%A1%8C%E5%99%A8&zhida_source=entity) 差异** ：有的末端是两指平行夹爪，有的是吸盘，还有的是多指灵巧手。
- **控制频率差异** ：有的是 10Hz 控制，有的是 50Hz 控制。

RT-X 的核心贡献在于，它证明了跨本体（Cross-Embodiment）的数据可以混训，「他山之石，可以攻玉」。

为了解决本体差异，RT-X 设计了一套通用的 Observation / Action / Language Schema，把来自全球几十个实验室、不同形态机器人的海量 [数据对齐](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%AF%B9%E9%BD%90&zhida_source=entity) 到一个标准的 VLA 训练框架下，建立尽可能共享的接口。

![](https://pic4.zhimg.com/v2-1d6ac7e92793ff7bbbec26fff0eb8f77_1440w.jpg)

RT-1-X 与 RT-2-X 架构及跨本体执行机制。面对异构的机器人数据，模型统一接收「图像+指令」输入，并输出离散的动作 Token。右侧展示了 VLA 解决 Embodiment Gap 的精妙之处：同一个模型输出的 Action Token，在部署时可以根据不同的本体硬件，被灵活解码为不同频率（10Hz/3Hz/5Hz）和不同控制逻辑（速度/位置增量）的底层物理指令。

结果非常振奋人心：当把 22 种不同机器人的数据混在一起训练出一个统一的 RT-X 模型后，相比于各家实验室自己用 [小数据](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%B0%8F%E6%95%B0%E6%8D%AE&zhida_source=entity) 训练的专用 Policy，统一模型在各个原始任务上的成功率平均提升了 50%。

Open X-Embodiment 把机器人圈子从「各家闭门造车」阶段推向了「多机器人共享经验」的 [大航海时代](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%A4%A7%E8%88%AA%E6%B5%B7%E6%97%B6%E4%BB%A3&zhida_source=entity) 。

## 六、第四块拼图：OpenVLA 与开源社区

尽管 RT-2 和 RT-X 证明了 VLA 的可行性，但它们并不完全开放。对于广大科研工作者、学生和 [开发者](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E8%80%85&zhida_source=entity) 来说，无法下载权重，也无法根据自己的机器人进行微调，这极大限制了生态的发展。

OpenVLA 成为了VLA [技术路线](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%8A%80%E6%9C%AF%E8%B7%AF%E7%BA%BF&zhida_source=entity) 上的一个关键生态节点，它提供了一个社区可以直接拿来魔改的开源基座。

![](https://pica.zhimg.com/v2-202eb130821e9ad57e83b2a78d9062da_1440w.jpg)

OpenVLA 架构与开源生态全景：它吸收了开源的 Open X-Embodiment 海量数据（左），依托开源的 Llama 2 7B 与 ViT 作为基座（中），不仅复现了以语言模型输出物理动作（ 等）的能力（右），更重要的是，它提供了完整的代码和权重，让普通实验室也能通过参数高效微调（PEFT），将 VLA 部署到自己的机器人上（下）。

OpenVLA 的 [架构设计](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1&zhida_source=entity) 非常务实，它是各种开源 SOTA 组件的集大成者：

1. **[视觉编码器](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%A7%86%E8%A7%89%E7%BC%96%E7%A0%81%E5%99%A8&zhida_source=entity)** ：采用了 `DINOv2` （擅长理解几何和空间关系）加上 `SigLIP` （擅长语义对齐）的融合方案。
2. **语言基座** ：使用了开源的 `Llama 2` （7B 参数级），保证了强大的指令遵循和逻辑推理能力。
3. **训练数据** ：直接吸收了 Open X-Embodiment 提供的大规模混训数据。
4. **动作头（Action Head）** ：沿用了类似 RT-2 的自回归（Autoregressive, AR）离散 Action Token 方案。

得益于 [Prismatic VLM](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Prismatic+VLM&zhida_source=entity) 的高效训练框架，相比从零训练 VLA，社区可以通过 LoRA / PEFT 等方式，以低得多的成本把 OpenVLA 微调到自己的机器人平台上。

## 七、第五块拼图：动作头之争

之前我们介绍，在VLA发展的早期，为了迎合大语言模型的架构，我们把动作离散化成了 Token。但这个设计在后续的实践中，也暴露出了一些局限性。

比如，当我们真正把这样的 [VLA 模型](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=VLA+%E6%A8%A1%E5%9E%8B&zhida_source=entity) 放到实体机器人上执行高频控制时，问题出现了：

- **太慢了** ：动作是逐个 Token 自回归生成的。生成一个 6 自由度加夹爪的动作，模型需要连续推理 7 次。这让整个控制回路的延迟居高不下。
- **太糙了** ：因为动作被离散化成了 Bins（比如分成了 256 份），这种粗颗粒度的控制也许勉强够用来抓取一个杯子，但如果你想让机器人做穿针引线这样的精细动作，这种离散的动作输出可能就力不从心了。

归根结底，真实世界的物理控制，本质上是一个高频、连续且容错率极低的过程。

既然把动作当成离散文字去生成的路子有所局限，那我们能不能重新让 VLA 生成连续的物理轨迹？

这个问题可以说引出了一系列的优化方案。我们不能完整介绍所有工作，只挑选一些关键的、且容易被理解的思路。

为了解决生成速度和连续性的问题，一个普遍的做法是：不要一次只生成一个动作，而是采用 **Action Chunking（动作块）** ——指的是一次预测未来一段动作序列，而不是只预测下一步动作。这个 Chunk 可以被离散 token 化，也可以直接表示为连续浮点轨迹。在后来的很多工作中，比较常见的是连续 Action Chunk。

围绕着「如何生成这个连续的 Action Chunk」，学术界提出了很多不同的方案。我们不能完整介绍所有工作，只挑选一些关键的、且容易被理解的思路：

| 范式代表 | 动作形式 | 直觉 | 优缺点 |
| --- | --- | --- | --- |
| AR (自回归) ：RT-2 / OpenVLA | 离散 Token | 打字机：像敲键盘一样，离散且断续。 | 优点：天然契合 [LLM 架构](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=LLM+%E6%9E%B6%E6%9E%84&zhida_source=entity) 。缺点：推理极慢，动作粗糙，容易发生灾难性 [累积误差](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%B4%AF%E7%A7%AF%E8%AF%AF%E5%B7%AE&zhida_source=entity) 。 |
| Diffusion (扩散) ：Octo / GR00T | 连续 Action Chunk | 捏泥巴：从一团完全无序的噪声开始，经过多步揉捏（去噪），逐渐塑形成一段平滑的动作轨迹。 | 优点：能捕获极度复杂的多模态动作分布，轨迹平滑。缺点：多步去噪（哪怕用了 [DDIM](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=DDIM&zhida_source=entity) 等加速算法）依然有一定的延迟。 |
| Flow Matching (流匹配) ：\\pi0 / \\pi0.5 | 连续 Action Chunk | 顺水推舟：直接学习一条从「噪声状态」流向「专家动作」的速度矢量场，一步步顺流而下。 | 优点：数学上基于连续时间 [向量场](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%90%91%E9%87%8F%E5%9C%BA&zhida_source=entity) / ODE 视角，训练目标更直接。相比传统 diffusion，常被认为采样路径更简洁、可用较少步数生成连续动作。 |
| Regression (回归) ：OpenVLA-OFT | 连续 Action Chunk | 盖章：绕过复杂的生成过程，直接用网络并行回归出未来的动作值。 | 优点：速度极快。缺点：容易产生均值化动作，对多峰动作分布建模较弱。 |

所谓动作头，本质上是 [神经网络](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C&zhida_source=entity) 的「输出接口」。

如果你熟悉大语言模型（LLM），就会知道 LLM 的最后一层通常是一个 Language Modeling (LM) Head。它负责把模型内部高维的 [隐向量](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E9%9A%90%E5%90%91%E9%87%8F&zhida_source=entity) （Hidden States）映射成词表上的 [概率分布](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%A6%82%E7%8E%87%E5%88%86%E5%B8%83&zhida_source=entity) ，决定下一个词输出什么。

同理，在 VLA 中， **Action Head** 负责把模型融合了图像和指令的隐向量 ，映射到真实的机器人动作空间 中。动作头的演进，实际上就是动作表达方式的演进：

- **早期的 AR 动作头（如 RT-1/RT-2）：** 本质是一个分类器。它把动作空间强行切分，将隐向量映射成离散的 Token。比如模型算出一个 ，动作头会输出：「这个向量对应的是第 158 号位移 Token」。
- **后期的连续动作头（如 Diffusion / Flow Matching Head）：** 本质是一个生成器。它接收隐向量 作为条件（Condition），直接输出一段平滑的、连续的物理轨迹（Action Chunk） 。

> PS：有的架构里，负责生成动作的这部分不只是神经网络末端的一个函数层，可能是一个更复杂的模块或子网络，所以可能也会被称作 Action Expert，领会意思即可。

## 八、第六块拼图：从 VLA 到机器人系统栈

到这里我们会发现，VLA 的争论已经不再只是「用不用大模型」，而是变成了一些更工程化的问题：

1. 语义理解和动作生成要不要解耦？
2. 连续动作应该由简单回归、 [扩散模型](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%89%A9%E6%95%A3%E6%A8%A1%E5%9E%8B&zhida_source=entity) ，还是 Flow 来生成？
3. VLA 在真实机器人系统里，应该和高层规划、底层控制器、世界模型如何分工？

于是，迈入到更加落地的阶段， [VLA路线](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=VLA%E8%B7%AF%E7%BA%BF&zhida_source=entity) 的发展也呈现出了一些工程化的趋势： **训练端基础 [模型化](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E5%8C%96&zhida_source=entity) ，部署端系统分层化。**

### 系列

以 Physical Intelligence 发布的 为代表，VLA 从一个什么都干的单体模型，走向了 **「语义大脑 + 运动专家」** 的解耦架构。

![](https://pic2.zhimg.com/v2-c69db6cc081b2ad2c6ef1f96920ed7f3_1440w.jpg)

0的解耦架构：中央的蓝框表明，庞大的预训练 VLM 骨干网络（大脑）退居幕后，专注视觉与语言的语义理解；而独立的 Action Expert（小脑）则接收语义特征，通过流匹配（Flow Matching）生成高频、平滑的连续动作轨迹。配合左侧海量的跨本体数据与高质量灵巧操作数据，展现出强大的零样本泛化与高效微调能力。

在 的设计中：

- **VLM Backbone（视觉语言大脑）** ：负责理解「杯子在哪里、任务是什么」，这部分不需要跑得极快。
- **Action Expert（动作专家）** ：负责通过 Flow Matching，将大脑传来的语义意图，转化为连续平滑的 Action Chunk。

这种解耦，实际上也是真实的 [机器人工程](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%B7%A5%E7%A8%8B&zhida_source=entity) 中系统性的妥协与融合。

在 的基础上， 解决的核心痛点是 **数据规模与异构性的极限扩展** 。真实世界中的物理数据是极其混乱的——有带高 [质量力](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E8%B4%A8%E9%87%8F%E5%8A%9B&zhida_source=entity) 矩反馈的 [遥操作](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E9%81%A5%E6%93%8D%E4%BD%9C&zhida_source=entity) 数据，有只包含视频和粗略文本描述的野外数据，还有来自不同传感器频率、不同机身视角的长尾数据。

证明了架构在面对极端异构数据时的 [吸收能力](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%90%B8%E6%94%B6%E8%83%BD%E5%8A%9B&zhida_source=entity) ，不光能吃精粮，也能吃粗粮。通过更大规模的联合训练（Co-training），模型不再局限于桌面级的抓取或折叠衣物，而是开始涌现出对「开放世界」中未见物体、复杂光影和非结构化环境的强大泛化能力。

而到了 阶段，则又引入了一个关键的概念： **Steerable Generalist Model（可引导的通用模型）** 。

这里的 steerable，不是说用户可以像调控制器参数一样精确指定每个动力学细节，而是说模型可以通过语言、 [元数据](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=%E5%85%83%E6%95%B0%E6%8D%AE&zhida_source=entity) 、控制模式、视觉子目标等条件，显式改变执行策略和行为风格。

比如，你可以通过自然语言指令附加约束：「 **非常缓慢、极其轻柔地** 把杯子拿起来」；或者「 **用力** 擦掉桌子上的污渍」。模型内部的 Action Expert 能够听懂这种对动作风格（Style/Dynamics）的微调，并在生成的连续轨迹（Action Chunk）中实时体现出速度、力度的变化。

### GR00T 系列

如果你去看 NVIDIA 针对人形机器人推出的 **GR00T** 路线，这种「系统分层化」的趋势也很明显。

![](https://pic4.zhimg.com/v2-07061f3d8e6b38a65c227a630eb6924f_1440w.jpg)

NVIDIA GR00T N1 的双系统（Dual-system）架构：它将系统解耦为 System 2（基于 VLM 的慢思考大脑，负责语义理解）和 System 1（基于 DiT 的快直觉小脑，负责动作生成）。通过引入极其细致的机器人本体状态（Robot State），并利用扩散（Diffusion）过程进行连续动作去噪，GR00T 实现了「大模型范式」与「物理动作」的结合。

上图是GR00T N1的基本架构：

1. **System 2（慢思考）** ：VLM 或是拥有长逻辑链的 [Reasoning 模型](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Reasoning+%E6%A8%A1%E5%9E%8B&zhida_source=entity) ，负责拆解任务、理解场景。
2. **System 1（快直觉）** ：基于 Diffusion Transformer 的动作模型，负责将System 2 的语义表征和机器人状态转化为连续动作。
3. **Runtime（底层控制与仿真兜底）** ：这是最容易被 AI 圈忽略，却在机器人圈至关重要的部分。VLA 给出的只是目标，真实的物理世界充满了摩擦、碰撞和重力扰动。这些高频电机力矩计算，必须交给 **Whole-body Controller (WBC，全身控制器)** ，或是通过 [Isaac Lab](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=Isaac+Lab&zhida_source=entity) 等仿真环境利用强化学习（RL）训练出来的底层稳态策略（Gait Policy）来执行。

而 GR00T 系列后续的演进路线，则超脱了「单一策略模型」的范畴，走向了系统生态化的全面构建。

- **Cosmos（物理世界模型与合成底座）** ：我们之前的文章介绍过这个系列。简而言之，Cosmos 提供了世界模型的基座，它不仅能生成符合直觉的视频，更重要的是能在隐空间中预测物理状态的演变。在这个生态里，系统能在虚拟空间里进行物理推理，并为模型合成出高质量的训练数据，缓解真实数据缺乏的瓶颈。
- **[GRAIL](https://zhida.zhihu.com/search?content_id=277961922&content_type=Article&match_order=1&q=GRAIL&zhida_source=entity) （自动化虚拟数据管线）** ：有了 Cosmos 提供的物理底座，GRAIL 则充当了面向人形机器人的「数据流水线」。它能针对极其复杂的移动抓取（Loco-manipulation）任务，自动化地在虚拟环境中生成各种任务场景与动作轨迹，最终将这些成功经验蒸馏给 GR00T 的动作模型。
- **Isaac Lab 与强化学习（Sim-to-Real 的物理兜底）** ：在高性能仿真环境中建模，以此来缩小虚实差距（Sim-to-Real Gap）。在处理需要极高频稳定的移动控制时，上层的 VLA 必须与底层通过强化学习训练出的高鲁棒性步态策略（Gait Policy）紧密结合。VLA 决定手去抓什么，而底层的 RL 控制器负责维持机器人的动态稳定性，确保它在执行动作时不会摔倒。

## 九、总结

回顾一下我们这篇文章所采用的VLA发展视角：

1. RT-1：动作 token 化
2. RT-2：动作进入 VLM 的语言 token 空间
3. RT-X / OXE：跨机器人数据混训
4. OpenVLA：开源 VLA 基座
5. Octo / OpenVLA-OFT / ：发现真实控制不能只靠 token，于是动作生成开始走向 Diffusion、Flow 和连续的 Action Chunk
6. / / GR00T：把 VLA 放进推理（Reasoning）、世界模型（World Model）、控制器（Controller）组成的机器人系统栈

还没有人送礼物，鼓励一下作者吧

编辑于 2026-07-03 15:46・北京[具身智能](https://www.zhihu.com/topic/21535379)[一文告诉你人工智能纯小白学习路线！](https://zhuanlan.zhihu.com/p/31863323446)

[

全文5196字，按照我这个路线坚持完，你会变成一个人工智能的牛人的。它是假定一个没有人工智能基础的程序员学习路线。写在前面：我觉的从deepseek开源以后，会有更多的企业和开发者争...

](https://zhuanlan.zhihu.com/p/31863323446)

赞同 74