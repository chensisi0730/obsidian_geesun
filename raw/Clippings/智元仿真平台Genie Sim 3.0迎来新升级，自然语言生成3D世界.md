2 人赞同了该文章

真实世界的数据，让机器人开始真正“看见”和“经历”这个世界。但当数据问题被部分解决之后，一个更深层的问题随之浮现： **仅仅依靠现实世界，是否足够支撑 [具身智能](https://zhida.zhihu.com/search?content_id=272734689&content_type=Article&match_order=1&q=%E5%85%B7%E8%BA%AB%E6%99%BA%E8%83%BD&zhida_source=entity) 的快速进化？** 场景构建难以泛化，评测标准缺乏统一，每一次算法迭代，仍伴随着繁重的工程投入。

**如果现实世界不够快，我们就“生成一个世界”，** 这正是 Genie Sim 诞生的初衷，以构建一个与真实世界等价、甚至更高效的“训练和验证环境”。

今天， **Genie Sim3.0 一站式仿真开发平台** 迎来新升级。通过环境生成、场景泛化、数据采集到模型评测的全流程仿真，显著加速模型训练验证，提升开发者与研究者的研发效率，推动具身智能的创新应用。

从构建到数据，从场景到评测——我们为具身智能提供一套完整可复用的开源底座。

**01/**

**Genie Sim World：语言造世界，环境构建触手可及**

**在传统范式中，仿真环境是被“搭建”的。而在 [Genie Sim 3.0](https://zhida.zhihu.com/search?content_id=272734689&content_type=Article&match_order=1&q=Genie+Sim+3.0&zhida_source=entity) 中，环境第一次成为被“生成”的对象。** 自然语言，即世界的接口。只需一句话或一张图，即可生成可交互、可漫游、可训练的三维世界，实现“输入即场景”的即时生成体验。

- **图文生境：** 无需建模、采集或硬件，仅文本或图片输入，用户即可零门槛生成海量场景。通过多模态大模型，用户指令一改、场景即换，无限泛化。
- **极速生成：** 空间世界模型单次推理即可完成构建，生成速度从“小时级”提升至“分钟级”，实时仿真、动态交互即开即用。
- **虚实一致：** RGB、深度、激光雷达等多模态数据原生同步输出，实现仿真数据与真实世界的浑然一体。
![动图](https://pic2.zhimg.com/v2-fe4f14ed44ff93b0020b89d78f660265_b.webp)

**02/**

**Genie Sim Benchmark：多维度、全方位覆盖机器人算法核心能力的仿真评测基准**

针对机器人算法核心的五大能力——语言指令理解、空间关系认知、原子技能操作、环境扰动适应、零样本跨域迁移，Genie Sim Benchmark分别设计了 **五大任务套件** ，支持Genie Operator系列、π系列、GR00T系列等主流基座模型，多维度系统性评估模型在复杂场景下的综合表现。

- **Instruction 指令跟随：** 检验模型对形状、大小、颜色、逻辑等自然语言指令的理解能力，检验语言与行为的对齐深度。
- **Spatial 空间理解：** 通过相对位置抓取、排序、叠放等任务，评估智能体在几何与语义交织中的空间智能。
- **Manipulation 操作执行：** 衡量多场景下的多样化原子操作技能效果，并通过分层难度设计，检验长程任务中组合运用技能的执行水平。
- **Robust 扰动适应：** 通过光照变化、背景替换、指令泛化、相机噪声、末端切换等十余类实际作业工况中可能出现的扰动，系统评估模型在物理世界中的适应边界与鲁棒性。
- **Sim2Real 训以致用：** 包含一系列零样本真机迁移的评测任务，通过纯仿真数据训练的模型同样可以部署在真机上达到较高的任务成功率，验证模型的跨域迁移能力。
![动图封面](https://picx.zhimg.com/v2-7013a55d12c843b730c4b341c92c8a5f_b.jpg)

![动图封面](https://pica.zhimg.com/v2-bd0825102e63dbbdb618fed355bbf72e_b.jpg)

![动图封面](https://pic3.zhimg.com/v2-738a725352f3c033c3336d403907c874_b.jpg)

![动图封面](https://pica.zhimg.com/v2-0ab70601eb495d1a310a28f9102e532a_b.jpg)

![](https://pic4.zhimg.com/v2-173d445ec6d3163f6d15b0895c0302b5_1440w.jpg)

Genie Sim Benchmark提供π系列和GR00T系列等开源基座模型在各个benchmark任务套件下的一键训练和评测功能，支持多种末端控制方式，快速绘制模型全景能力画像。

![](https://pic2.zhimg.com/v2-7a9d6e2edb2e9fb6270e08cf7e0630a3_1440w.jpg)

Genie Sim Benchmark模型评分

![](https://pica.zhimg.com/v2-55e6c75599c6002ae017c78b1a791936_1440w.jpg)

Genie Sim Benchmark模型评分

![](https://pic1.zhimg.com/v2-9e3f770255b57b1f485b7d4b847fc534_1440w.jpg)

Genie Sim Benchmark模型评分

使用Genie Sim Benchmark仿真数据训练的模型可以实现零样本迁移到真实世界，并且相同模型在仿真环境与真实世界的评测差异<10%，模型验证无需真机部署，显著提升算法迭代效率。

![](https://picx.zhimg.com/v2-60543a0b25bce4f32d5b596d5fdba2ab_1440w.jpg)

Genie Sim-Sim2Real实验对比

**03/**

**Genie Sim x [RLinf](https://zhida.zhihu.com/search?content_id=272734689&content_type=Article&match_order=1&q=RLinf&zhida_source=entity) ：**

**全面支持RLinf框架，开启具身智能“强化”新时代**

Genie Sim x RLinf 开源方案，提供一套“部署简单、迭代高效”的强化学习工具链。完美补齐 [VLA 模型](https://zhida.zhihu.com/search?content_id=272734689&content_type=Article&match_order=1&q=VLA+%E6%A8%A1%E5%9E%8B&zhida_source=entity) 短板，用低成本的 RL 后训练，打通从"泛化理解"到"精准微操"的最后一公里。

- **双引擎联合+并行仿真：** 物理与渲染引擎解耦，实现1,000Hz高精度物理模拟与高保真实时视觉观测；通过大规模并行计算，显著提升吞吐量，加速模型收敛。
- **开源任务 + 评测闭环：** 强化学习方案支持对Genie Sim开源任务在线微调，通过交互式学习解决模型瓶颈；同时依托Genie Sim评测框架，在评估表现的同时提供可靠奖励，驱动模型迭代。
- **通用标准Gym接口：** 极简链路，生态无忧。提供标准API，无缝适配RLinf及社区其他算法环境，降低使用门槛，便于二次开发。

Genie Sim x RLinf 为强化学习打通了从高效训练到闭环评测的路径：并行仿真大幅提升采样效率，标准接口降低开发门槛，让模型在仿真中加速迭代。

![动图封面](https://picx.zhimg.com/v2-bea11eddb4db80d251a692b92ec8269d_b.jpg)

Genie Sim x RLinf 并行训练

从海量仿真数据的开源共享，到大语言模型驱动的场景泛化，再到多维度评测体系的系统构建—— **Genie Sim3.0 将“场景—数据—评测”融为一体，大规模仿真资产同步可在 [觅蜂商城](https://zhida.zhihu.com/search?content_id=272734689&content_type=Article&match_order=1&q=%E8%A7%85%E8%9C%82%E5%95%86%E5%9F%8E&zhida_source=entity) 获取，让开发者不再受困于环境搭建与数据采集的繁琐投入。**

具身智能的进化，需要一座连接虚拟与现实的高效桥梁，它发生在无数次快速构建的场景里，发生在源源不断生成的数据中，也发生在每一次精准、自动化的评测迭代里。智元 Genie Sim 3.0 所做的，是为机器人从仿真走向真实提供 **高效引擎** 。我们希望这不仅是一个仿真平台，更是一个 **加速器** ，一个让机器人从 “缓慢研发”，走向 “快速落地” 的加速器。

我们相信，这套开源平台的开放与共享，将加速模型能力高效进化、迈向真实世界，成为通用机器人生态演进的重要一步。

发布于 2026-04-08 14:36・上海