28 人赞同了该文章

## 引言：机器人训练的「数据荒漠」与人类视频的「金矿」

基于真实机器人的模仿学习数据采集推动了机器人操作领域的巨大进步。然而，数据采集过程对机器人硬件的需求限制了数据规模。本文探索了利用人类第一视角视频训练视觉-语言-动作（VLA）模型。使用人类视频的优势不仅在于其数据规模，更重要的是场景和任务的丰富性。通过基于人类视频训练的可预测人类手腕和手部动作的VLA模型，通过 [逆运动学](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E9%80%86%E8%BF%90%E5%8A%A8%E5%AD%A6&zhida_source=entity) （Inverse Kinematics）和 [动作重定向](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%8A%A8%E4%BD%9C%E9%87%8D%E5%AE%9A%E5%90%91&zhida_source=entity) （Retargeting），将人类动作转换为机器人动作。我们利用少量机器人操作演示对模型进行微调得到机器人策略模型，即EgoVLA。本文提出了Isaac Humanoid Manipulation Benchmark仿真基准，设计了多样化的双手操作任务并提供演示数据。我们基于该基准对EgoVLA进行微调与评估，结果表明其性能显著优于基线方法，并通过 [消融实验](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%B6%88%E8%9E%8D%E5%AE%9E%E9%AA%8C&zhida_source=entity) 验证了人类数据的重要性。

![](https://picx.zhimg.com/v2-8053e127a107f5d45a78a2c2c0c8a6a1_1440w.jpg)

论文链接：  
[arxiv.org/abs/2507.1244](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2507.12440v1)  
项目链接：  
[rchalyang.github.io/Ego](https://link.zhihu.com/?target=https%3A//rchalyang.github.io/EgoVLA/)

## 人类第一视角视频数据的价值

与依赖 [仿真模拟](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E4%BB%BF%E7%9C%9F%E6%A8%A1%E6%8B%9F&zhida_source=entity) 的方法相比，基于真实机器人数据进行监督学习，不仅能避免Sim2Real的domain gap，还能提升任务复杂度。为了高效收集复杂的机器人操作数据，研究者提出了多种工具，包括支持关节映射的远程操作系统、 [外骨骼装置](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%A4%96%E9%AA%A8%E9%AA%BC%E8%A3%85%E7%BD%AE&zhida_source=entity) 以及VR设备。然而，这些工具对实体机器人和专业操作员的需求，从根本上限制了可采集数据的规模。  
那么，能否从人类视频中学习操作技能？如果我们把人类视为一种“特殊的机器人”，那么全球80亿人正作为“机器人”在各类环境中持续运作。若能利用人类视频数据 [训练机器人](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E8%AE%AD%E7%BB%83%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity) 策略，不仅能大幅增加训练数据的规模，更重要的是能显著提升任务的多样性与场景的丰富性。这使得机器人能够在当前硬件难以适配的场景中训练，甚至完成对远程操作而言极具挑战性的任务。  
人类动作与机器人动作空间的差异可能并不大，且可通过少量 [几何变换](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%87%A0%E4%BD%95%E5%8F%98%E6%8D%A2&zhida_source=entity) 进行近似。我们提出不依赖机器人数据而是基于人类第一视角视频数据训练视觉-语言-动作模型（EgoVLA）。具体而言，给定少量视觉观察帧、语言指令和当前手部姿态作为输入，该VLA模型将预测未来几个时间步的人类动作。其动作空间包含人类手腕和手部关节角度，该空间可通过逆运动学将手腕位置转换为 [末端执行器](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%9C%AB%E7%AB%AF%E6%89%A7%E8%A1%8C%E5%99%A8&zhida_source=entity) 位置，并通过重定向将人类手部关节转换为机器人手部关节，从而映射到机器人动作空间。因此，EgoVLA本质上已是一种机器人策略，仅输入为人类手部图像，且动作输出仍存在误差。我们可通过 [遥操作](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E9%81%A5%E6%93%8D%E4%BD%9C&zhida_source=entity) 采集少量机器人演示数据对VLA进一步微调来修正这一问题。通过这种方式，我们无需大规模机器人数据即可完成训练。

## 从人类第一视角视频中学习操作技能

![](https://pic1.zhimg.com/v2-3060a19a33988bd827a2250964075012_1440w.jpg)

### 人类第一视角操作数据集构建

![](https://pic3.zhimg.com/v2-9e458cb739fa28866591f480c9659888_1440w.jpg)

我们构建了一个大规模的人类第一视角操作数据集，该数据集包含人类第一视角的RGB观测数据、手腕姿态、手部姿态和相机姿态。我们的数据集整合了四种不同的序列，其相对比例如图3所示：

- [HOI4D](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=HOI4D&zhida_source=entity) 包含4,000个视频，记录单手操作（如拾取放置、重新定向、关节物体交互）；
- [HOT3D](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=HOT3D&zhida_source=entity) 提供833分钟与33个刚体物体交互的视频，包含精确的3D手部和相机姿态标注；
- [HoloAssist](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=HoloAssist&zhida_source=entity) 提供166小时复杂任务（如电池更换、家具组装、机器安装）的录制数据，尽管其手部姿态标注噪声较大，但捕获了丰富的双手交互场景。为避免HoloAssist因标签噪声过大而被过度代表，我们对其数据进行了1/10的均匀采样，以平衡任务和 [数据源](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%BA%90&zhida_source=entity) ；
- [TACO](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=TACO&zhida_source=entity) 包含2,317个运动序列，覆盖151种工具-动作-物体 [三元组](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E4%B8%89%E5%85%83%E7%BB%84&zhida_source=entity) 。

### EgoVLA模型训练

我们在VLM模型的基础上构建了EgoVLA，以利用其强大的视觉和语义推理能力。具体而言，我们使用NVILA-2B作为主干网络，原因是其具备强大的视觉语言理解能力和紧凑的规模，能够同时实现 [意图推断](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%84%8F%E5%9B%BE%E6%8E%A8%E6%96%AD&zhida_source=entity) 和高效微调。如图2所示，EgoVLA以当前和历史的人类第一视角视觉观察信息、语言指令、动作查询token以及人类本体状态作为输入。这些输入由视觉语言模型主干进行编码，并通过动作头进一步处理，以预测 [未来人类](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%9C%AA%E6%9D%A5%E4%BA%BA%E7%B1%BB&zhida_source=entity) 或机器人的动作。  
视觉观测包含六帧RGB图像：当前观测帧及前五帧，以0.2秒间隔 [采样](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=2&q=%E9%87%87%E6%A0%B7&zhida_source=entity) ，覆盖1秒历史信息。每帧图像分辨率为384×384。语言指令则描述期望行为。人类 [本体感觉](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%9C%AC%E4%BD%93%E6%84%9F%E8%A7%89&zhida_source=entity) 状态包含手腕平移/旋转及手部姿态参数。这些信息通过MLP处理后传递至 [动作预测头](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%8A%A8%E4%BD%9C%E9%A2%84%E6%B5%8B%E5%A4%B4&zhida_source=entity) （action head）。每个预测动作包含手腕姿态（ [相机坐标系](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E7%9B%B8%E6%9C%BA%E5%9D%90%E6%A0%87%E7%B3%BB&zhida_source=entity) 下的3D平移与rot6D表示法的旋转）及手部关节角度——后者通过 [MANO手模型](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=MANO%E6%89%8B%E6%A8%A1%E5%9E%8B&zhida_source=entity) 的前15个主元分量（PCA）表示。EgoVLA模型被训练用于回归相机坐标系下的未来手腕姿态及手部关节参数。目标函数为：

其中，和分别为手腕平移和手部关节角度回归的L2损失，为rot6D手腕旋转损失。, , 为 [权重系数](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%9D%83%E9%87%8D%E7%B3%BB%E6%95%B0&zhida_source=entity) 。

**动作头与动作查询token**

动作头是一个包含6个编码器层的300M规模变换器，每个编码器层的隐藏层大小为1536。它以人类（或机器人）本体感知状态和与动作查询token对应的latent embedding作为输入，在1秒时间内（30赫兹下预测30个未来步骤）为双手预测动作序列 。我们使用词汇表中最后 个词ID作为动作查询token。

**训练细节**

我们首先在人类第一视角操作数据集上对Ego-VLA进行20个 [epoch](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=epoch&zhida_source=entity) 的预训练，随后在机器人演示数据上进行115个epoch的后续训练（其中第100个epoch后降低学习率）。训练过程中，整个模型（包括视觉编码器）均进行微调。

### 将EgoVLA迁移至人形机器人

![](https://pic3.zhimg.com/v2-86502489a5c6ac089c178e7029f2425a_1440w.jpg)

人类与 [人形机器人](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=2&q=%E4%BA%BA%E5%BD%A2%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity) 共享基于双臂和双手的相似操作框架。然而，由于相机位姿、手部形态和视觉外观的差异，直接将EgoVLA迁移至人形机器人具有挑战性。为实现部署，我们利用图4中的统一动作空间，通过少量机器人演示数据对EgoVLA进行微调。

**将机器人数据重定向至人类表征**

为了在机器人数据上进行微调，我们首先将机器人的动作空间与人类表征对齐。对于末端执行器位姿，通过3D变换对齐机器人和人类的坐标系。手部配置的对齐更为复杂：我们通过最小化预测指尖位置与观测指尖位置之间的差异，估计最逼近机器人手部驱动的MANO参数，即最小化 [目标函数](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=2&q=%E7%9B%AE%E6%A0%87%E5%87%BD%E6%95%B0&zhida_source=entity) ：  
其中 为MANO手部参数，是通过MANO [前向运动学](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%89%8D%E5%90%91%E8%BF%90%E5%8A%A8%E5%AD%A6&zhida_source=entity) 计算的指尖位置， 为观测到的机器人指尖位置。这种统一动作空间使EgoVLA能够在机器人演示数据上直接进行微调，无需额外的架构改动或重新初始化。

**将人类手部映射到机器人手部**

在推理阶段，EgoVLA预测的腕部和手部姿势会被映射到机器人的执行器上，如图4（底部行）所示。首先通过 [3D变换](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=2&q=3D%E5%8F%98%E6%8D%A2&zhida_source=entity) 将腕部姿势转换为机器人末端执行器的位姿，并通过逆运动学（IK）求解对应的手臂关节角度。对于手部驱动，我们利用MANO模型从预测的MANO参数计算3D手部关键点。随后，一个轻量级 [多层感知机](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%A4%9A%E5%B1%82%E6%84%9F%E7%9F%A5%E6%9C%BA&zhida_source=entity) （MLP）根据3D手部关键点预测机器人手部的关节控制指令。该MLP通过在机器人演示数据上进行训练得到，这些演示数据中的手部驱动被重新映射为人类手部表征。这种映射平均指尖 [位置误差](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E4%BD%8D%E7%BD%AE%E8%AF%AF%E5%B7%AE&zhida_source=entity) 仅为 米。

## Isaac人形机器人操作基准测试

![](https://pic4.zhimg.com/v2-d99c0263700b47e55d1b93fef31ea141_1440w.jpg)

除了数据匮乏问题，基于学习的 [机器人学](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%AD%A6&zhida_source=entity) 领域面临的一大挑战是缺乏可扩展、 [鲁棒](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E9%B2%81%E6%A3%92&zhida_source=entity) 且可复现的评估方法。真实世界评估往往成本高昂、耗时耗力，且在安全性和可复现性方面存在挑战。近期研究表明，基于模拟的评估与真实世界性能高度相关，可作为可靠的替代方案。为了实现人形机器人操作的一致性基准测试，我们推出了基于NVIDIA Isaac Lab构建的Isaac人形机器人操作基准测试。该 [基准测试](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=4&q=%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95&zhida_source=entity) 并非为直接的模拟 - 真实世界迁移设计，而是借鉴LIBERO和SIMPLER的思路，将模拟环境作为可控、可复现的测试平台，用于评估操作策略。我们的模拟平台以 [Uniree H1人形机器人](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=Uniree+H1%E4%BA%BA%E5%BD%A2%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity) 为核心，配备两只Inspire灵巧手，包含12项操作任务，涵盖短时域 [原子](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E5%8E%9F%E5%AD%90&zhida_source=entity) 动作（推箱、翻转杯子、倒球、关抽屉、开抽屉、开笔记本电脑、堆叠罐子）和长时域多阶段技能（分类罐子、插入罐子、卸载罐子、插入并卸载罐子、将罐子堆叠入抽屉），如图5所示。

### 观测与动作空间

本基准提供了机器人关节位置、末端执行器位姿、接触力以及第一视角RGB-D视觉输入作为观测信息。虽然EgoVLA仅使用第一视角视觉、末端执行器 [位姿](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=5&q=%E4%BD%8D%E5%A7%BF&zhida_source=entity) 、手部关节驱动和任务描述，但我们还为未来研究预留了其他模态。机器人通过末端执行器控制实现手臂操作，通过PD关节控制实现手部操作。每只手具有12个自由度（6个主动自由度，6个拟人自由度），最终的36维动作空间融合了手臂逆运动学与手部直接驱动。控制频率为30Hz。

## 实验效果

### 人类操作建模

![](https://picx.zhimg.com/v2-0e5f03904e6fec733167f652971c7c31_1440w.jpg)

在直接评估机器人操作性能之前，我们首先考察EgoVLA模型在人类第一视角操作数据集（Human Ego-Centric Manipulation Dataset）上训练后对人类手部运动的建模效果。人类手腕平移的平均 [预测误差](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E9%A2%84%E6%B5%8B%E8%AF%AF%E5%B7%AE&zhida_source=entity) 约为8厘米；当投影到 [二维图像](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E4%BA%8C%E7%BB%B4%E5%9B%BE%E5%83%8F&zhida_source=entity) 平面时，归一化误差约为0.13，与HOI-forecast中报告的结果相当。为进一步验证EgoVLA是否能捕捉环境上下文并遵循语言指令，我们从HOI4D评估集中采样案例，在保持视觉输入不变的前提下修改原始语言提示。如图6第三列所示，当指令从“把它放进抽屉”变为“把它从抽屉里拿出来”时，预测的手部轨迹从“移入抽屉”转变为“移向柜体表面”；类似地，第二列中指令从“把某物放进去”改为“打开并关闭柜门”后，轨迹从“将物体放入保险箱”转变为“与保险箱门交互”。这些结果表明，EgoVLA不仅能准确建模人类运动，还能学习动作背后的潜在语义意图。

### 人形机器人性能评估

我们使用两项指标评估不同模型的操作性能：成功率（Success Rate）（衡量任务整体完成情况）和进度率（Progress Rate, PSR）（定义为长时程任务中完成的子任务数占总子任务数的比例）。  
基线模型：我们将EgoVLA与两种基线模型对比：（1）EgoVLA-NoPretrain（在机器人演示数据上微调预训练视觉语言模型，但未经过人类视频预训练）；（2）ACT（为每个任务单独训练一个专用Transformer模型）。  
评估设置：我们在两种场景下 [评估模型](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E8%AF%84%E4%BC%B0%E6%A8%A1%E5%9E%8B&zhida_source=entity) 性能：

- 已见背景（Seen）：视觉背景与训练阶段完全一致；
- 未见背景（Unseen）：视觉背景为全新未见的场景。 为提升鲁棒性，每个测试轨迹开始时物体位置均随机化（具体背景分割与物体随机化范围详见补充材料）。

评估次数：

- 对于“已见背景”，每个模型每项任务评估27次轨迹（3种背景各9个片段）；
- 对于“未见背景”，每个模型每项任务评估66次轨迹（22种未见背景各3个片段）。

### 人类第一视角数据预训练提升In-Domain性能

![](https://pic3.zhimg.com/v2-c7214d57485bd6a4e13503d48740ccf2_1440w.jpg)

我们首先在可见视觉背景下评估模型。如表1和表2所示，EgoVLA在短周期和长周期任务上均持续优于EgoVLA-NoPretrain基线模型。在需要精细操作的任务（如堆叠罐头、分类罐头、插入并卸载罐头、翻转杯子）中，性能提升尤为显著。我们认为，这得益于人类预训练阶段——在此阶段，EgoVLA学习了不依赖具身形态（人类或机器人）的通用操作技能（涉及“手”的操作）。相比之下，EgoVLA-NoPretrain仅从机器人演示中学习，缺乏此类可迁移的先验知识。此外，EgoVLA与EgoVLA-NoPretrain的性能差距在长周期任务中更为明显，前者成功率高出约20%。与专用ACT基线模型相比，通用模型（EgoVLA和EgoVLA-NoPretrain）在短周期和长周期任务上的表现均显著更优。这很可能是因为专用模型需要从零开始同时学习底层操作和长周期规划，而通用模型能够跨任务复用共享的底层技能。

### 人类第一视角数据预训练增强Out-of-Domain泛化能力

![](https://pic4.zhimg.com/v2-90ee250149b345f0aba502523ae5edcd_1440w.jpg)

如表1所示，EgoVLA在未见视觉背景下的短周期任务中保持了较强的泛化能力，平均成功率仅小幅下降，而EgoVLA-NoPretrain的下降幅度达23%。对于长周期任务，如表2所示，EgoVLA在未见背景下的成功率约为30%。尽管成功率较可见环境有所下降，但失败主要发生在任务执行的最终阶段而非早期子目标阶段，这表明进步率保持相似。这证明，在以人类为中心的视频上进行预训练，显著提升了EgoVLA对新颖环境的跨域泛化能力。

## 总结与展望

在本研究中，我们提出了EgoVLA——一种基于第一人称视角人类视频训练的视觉语言动作模型，用于实现灵巧操作。EgoVLA的开发流程包括：首先在大规模人类第一人称视角操作数据集上预训练视觉语言模型，然后在少量机器人演示数据上进行微调。为实现跨实体迁移（即不同形态主体的动作适配），我们引入了统一动作空间，对齐人类与机器人手部的表征。实验结果表明，人类视频预训练使EgoVLA能够学习通用型操作策略，在有限机器人数据条件下，依然能在多样化任务上实现优异性能，并展现出强大的泛化能力。  
我们的预训练框架需要人类手部和腕部姿势标注数据，这可能限制数据的可用性。然而， [高保真](https://zhida.zhihu.com/search?content_id=260581462&content_type=Article&match_order=1&q=%E9%AB%98%E4%BF%9D%E7%9C%9F&zhida_source=entity) AR/VR设备（如Quest 3、Vision Pro、Aria Glasses）的可及性日益提升，预计将缓解这一限制。此外，尽管EgoVLA通过统一动作空间进行了预训练，但若不经过一定量机器人数据的进一步微调，无法直接用于操作任务。未来工作可探索通过更多与具身无关的预训练，提升模型的零样本迁移能力。

还没有人送礼物，鼓励一下作者吧

发布于 2025-07-20 14:41・上海[机器人](https://www.zhihu.com/topic/19551273)

赞同 28