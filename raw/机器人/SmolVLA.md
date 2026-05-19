## 前言

自6月份以来，我一方面各种疯狂招人(全职和实习)，一方面和团队各种赶进度

1. 25年6.4日，正值我司注册十周年，在没动南京那边机器的情况下，到位了piper机械臂和宇树G1 edu  
	之后的6.6日，再到齐VR和吊架  
	6.8日，在新同事们分的其中一个两人小组上，VR已可遥操piper
2. 6.9-6.13，长沙具身团队先后完成通过键盘、手柄、VR、主从臂遥操piper机械臂的工作
3. 6.16  
	$\rightarrow$ 对于人形关于人形的所有配套硬件 基本全部补齐，比如vr外的：双目及深度相机 + 3d打印件，开始正式开发g1 edu  
	$\rightarrow$ 对于机械臂，本周开采数据 il/rl/vla全部开训，当然，会先采收纳杯子的数据，然后训练lerobot ACT、π0，最后部署在piper真机上  
	然后想训练π0的话，便不得不在本地上4090 24G/5090 32G之类的——毕竟训练可以云端 但推理还是得本地，最后不得不买了个带5090的工作站

但毕竟不是每个开发者都舍得买4090 24G/5090 32G之类的，为方便这帮朋友的开发—— *我写文章不一定只是为了我个人或我司，也为天下众人与朋友* ，故开始让我关注那些和π0效果相差不大但所占GPU资源相对较少的

于此便看到了SmolVLA，本文便来解读之

## 第一部分 SmolVLA

### 1.1 引言与相关工作

#### 1.1.1 引言

如原SmolVLA论文所说，尽管基础模型在数字世界取得了显著成就，但其在现实世界中的应用——尤其是在机器人领域——仍然受限

1. 具体来说，机器人策略（Zhao 等, 2023；Chi 等, 2024；Lee 等, 2024；Hansen 等,2022）  
	*ALOHA ACT  
	diffusion policy  
	Behavior generation with latent actions  
	Temporal difference learning for model predictive control*  
	  
	在不同物体类型、位置、环境和任务之间的泛化能力方面仍面临挑战（Xie 等, 2024；Ebert等, 2021）  
	*Decomposing the generalization gap in imitation learning for visual robotic manipulation  
	Bridge data: Boosting generalization of robotic skills with cross-domain datasets*  
	  
	机器人应能够适应新环境和新物体，这要求其具备强大的技能和对世界的常识性理解  
	然而，在这一方向上的进展往往受限于高质量且多样化数据的可获得性
2. 为了解决这一局限性，越来越多的研究开始探索以视觉-语言-动作（VLA）模型为形式的机器人基础模型（Team 等，2024；O’Neill 等，2024；Brohan 等，2023；Kim 等，2024；Black 等，2024；Bjorck 等，2025；Li 等，2024；Huang 等，2024）  
	*Octo  
	Open x-embodiment  
	Rt-2  
	Openvla  
	π0  
	Gr00t n1  
	Vision-language foundation models as effective robot imitators  
	Language is not all you need: Aligning perception with language models*  
	  
	VLA 旨在融合预训练的大型语言模型和视觉-语言模型中蕴含的抽象推理、世界知识和决策能力。这些模型能够接受多模态输入——如视觉观测和自然语言指令——并预测相应的机器人动作
3. 早期结果显示，其在泛化能力方面具有可观的提升（Black等，2024；Brohan 等，2023）  
	*π0  
	Rt-2*  
	  
	VLA模型仍处于早期开发阶段，尚未像LLM和VLM那样成熟或被广泛采用。许多具有影响力的VLA进展仍属专有，许多模型只共享权重，而未公开完整的训练细节和关键方法组件  
	  
	虽然OpenVLA和RT-2-X等开源项目展现了开放VLA系统的可行性，但这些模型仍然庞大、资源消耗高，并依赖昂贵的机器人平台，从而影响了其可获得性

故此，他们推出了 SmolVLA——一个紧凑但高效的 VLA 模型，并同步发布了可复现且高效的训练与推理方案，其主要贡献如下：

![](https://i-blog.csdnimg.cn/direct/5f2fb683b1ce4614b3f59c1d5018240d.png)

- **轻量级架构**  
	专为在消费级 GPU上训练及在 CPU 上部署而优化，其关键的设计选择包括：  
	(i) 跳过 VLM 的部分层级  
	(ii) 使用极少量的视觉 token  
	(iii) 利用小型预训练 VLM  
	(iv) 将自注意力层与更轻量的交叉注意力层交错排列
- **在社区驱动的数据集上进行预训练**  
	SmolVLA 采用端到端训练，所用数据集全部来自公开的、社区贡献的数据，总共不到 3 万个样本。即便使用的数据量比以往方法少一个数量级，依然展现出强大的性能
- 异步推理  
	作者引入了一套优化的异步推理架构， **将动作执行与观测处理及动作预测解耦** ，从而降低延迟，实现快速且高效的资源利用推理。且在模拟环境和现实场景下对 SmolVLA 进行了多任务评估。有趣的是，尽管体量显著更小，性能却能够媲美甚至超越体量更大的 VLA 模型

#### 1.1.2 相关工作

首先，视觉-语言模型（VLMs）。视觉-语言模型旨在处理视觉和文本两种模态——最常见的方式是同时接受图像和文本作为输入，并在视觉上下文的基础上生成文本

1. 视觉语言模型（VLMs）的发展受到大语言模型（LLMs）成功的推动，许多方法在预训练的LLM基础上构建，并采用类似的训练范式  
	通常，VLMs（Alayrac等，2022；Laurençon等，2024；Lin等，2023a）  
	*Flamingo  
	What matters when building vision-language models?  
	Pali: A jointly-scaled multilingual language-image model*  
	是通过将预训练的视觉编码器（Radford等，2021；Zhai等，2023；Fini等，2024）  
	*Learning transferable visual models from natural language supervision  
	Sigmoid loss for language image pre-training  
	Multimodal autoregressive pre-training of large vision encoders*  
	与预训练的LLM（AI@Meta，2024；Jiang等，2023；Wang等，2024）  
	*Llama 3.2 model card  
	Mistral 7b  
	Qwen2.5-vl technical report*  
	集成而构建的  
	  
	训练过程通常分为多个多模态阶段  
	首先在大规模的图像-文本标注数据集（Schuhmann等，2022；Byeon等，2022）  
	*Laion coco: 600m synthetic captions from laion2b-en  
	Coyo-700m: Image-text pair dataset*  
	和交错的视觉-语言语料库（Laurençon等，2023；Zhu等，2023）上进行预训练  
	*OBELICS: An open web-scale filtered dataset of interleaved image-text documents*  
	  
	随后在指令微调数据集（Liu等，2023b；Tong等，2024；Laurençon等，2024）  
	*Improved baselines with visual instruction tuning  
	InstructBLIP: Towards general-purpose vision-language models with instruction tuning  
	What matters when building vision-language models?*  
	上进行有监督微调  
	  
	其他研究表明，不依赖预训练视觉编码器也具有一定优势（Bavishi等，2023；Shukor等，2025；Diao等，2025, 2024）；  
	*Introducing our multimodal models  
	Scaling laws for native multimodal models  
	Unveiling encoder-free vision-language models  
	EVEv2: Improved baselines for encoder-free vision-language models*  
	  
	同时，也有研究致力于开发更统一的架构，将图像和文本统一表示为离散token，使单一模型能够处理多模态token序列（Wang等，2022；Shukor等，2023b；Team，2024；Lin等，2024）  
	*OFa: Unifying architectures, tasks, and modalities through a simple sequence-to-sequence learning framework  
	UniVal: Unified model for image, video, audio and language tasks  
	Chameleon: Mixed-modal early-fusion foundation models  
	MoMA: Efficient early-fusion pre-training with mixture of modality-aware experts*
2. 提升效率也成为VLM研究的核心议题之一  
	部分研究通过使用更小且多样化的数据集（Liu等，2023b；Dai等，2023；Bai等，2025；Zhu等，2024；Tong等，2024）  
	*Improved baselines with visual instruction tuning  
	InstructBLIP: Towards general-purpose vision-language models with instruction tuning  
	Qwen2.5-vl technical report  
	Kangaroo: A powerful video-language model supporting long-context video input  
	InstructBLIP: Towards general-purpose vision-language models with instruction tuning*  
	  
	训练更小规模的模型（Marafioti等，2025；Korrapati，2024；Yao等，2024）  
	*SmolVLM: Redefining small and efficient multimodal models  
	Moondream  
	MiniCPM-V: A GPT-4V level MLLM on your phone*  
	  
	或仅调整预训练单模态模型的一小部分参数（Shukor等，2023a；Vallaeys等，2024；Mañas等，2023；Koh等，2023；Tsimpoukelli等，2021；Li等，2023）  
	*Skipping computations in multimodal LLMs  
	Improved baselines for data-efficient perceptual augmentation of LLMs  
	MAPL: Parameter-efficient adaptation of unimodal pre-trained models for vision-language few-shot prompting  
	Grounding language models to images for multimodal inputs and outputs  
	Multimodal few-shot learning with frozen language models  
	Kangaroo: A powerful video-language model supporting long-context video input*  
	来降低训练成本  
	  
	虽然大多数VLM研究聚焦于图像和文本模态，但最新研究已证明，类似技术也可扩展用于整合视频和音频等额外模态（Wang等，2025；Liu等，2024；Zhang等，2025；Kong等，2024）  
	*InternVideo2.5: Empowering video MLLMs with long and rich context modeling  
	VideoLLaMA 3: Frontier multimodal foundation models for image and video understanding  
	Audio Flamingo: A novel audio language model with few-shot learning and dialogue abilities*

其次，对于视觉-语言-动作模型（VLA）

1. 机器人研究中一个日益受到关注的领域是开发通用策略——能够执行广泛任务、在不同环境和机器人结构间泛化的模型  
	该方向中一个突出的策略是利用VLA，这类模型能够处理  
	（i）以自然语言给出的任务指令  
	（ii）视觉观测（例如来自摄像头流的图像）  
	以及（iii）本体感觉输入，以输出控制动作  
	  
	早期的工作如Octo和RT-1在大规模机器人演示数据集上从零训练基于transformer 的模型
2. 为提升性能和泛化能力，RT-2利用了预训练的视觉-语言模型（VLM），并在机器人特定数据上进一步训练。在推动开放性和可复现性的努力中，OpenVLA发布了一个拥有70 亿参数、在公开数据上训练的VLA，用于生成离散动作token
3. 由于动作token 化对连续控制存在限制，π0和DexVLA提出使用基于扩散的解码器来生成连续动作。在这一方向上，Black 等（2024）和Wen等（2025）都提出了适配一个预训练VLM——RDT-1B，并引入了一个大型扩散组件（称为动作专家），直接在机器人演示上进行训练  
	  
	最近，Pertsch 等（2025）提出了一种完全自回归的方法，采用了一种新颖的动作tokenizer，相较于传统分箱方法有所改进，但仍然存在推理速度慢（自回归）的缺点
4. 为提升VLA 的效率，TinyVLA从零开始在多模态数据上训练了一个轻量级、参数量低于10 亿的模型，并在机器人数据集上进行了微调，尽管缺乏大规模机器人数据的预训练限制了其更广泛的泛化能力

SmolVLA 与上述大多数工作目标相似，致力于开发和发布在训练和推理效率方面都表现优异的开源模型

### 1.2 SmolVLA：小巧、高效且功能强大

总体而言，SmolVLA 是一个轻量级的 VLA，由紧凑型预训练视觉语言模型（VLM）和通过流匹配训练的动作专家组成

- 给定多张图像和描述任务的语言指令，该模型输出一组动作。在社区收集的数据集上，模型首先通过模仿学习进行预训练，随后在真实世界和仿真环境中进行评估  
	预训练数据涵盖多样化的任务和行为，使模型能够学习具备泛化能力的物理技能，并实现跨场景迁移
- 在推理阶段，他们引入了异步执行栈，将动作执行与感知和预测解耦，从而实现更快、更具响应性的控制

#### 1.2.1 模型架构：SmolVLM-2(SigLIP和SmolLM2) + 流匹配动作专家

SmolVLA 由两个主要组件组成：

1. 一个用于感知的预训练VLM
2. 一个被训练用于执行动作的动作专家

这两个组件是相互连接的，因为VLM 处理状态输入以生成特征，这些特征会对动作专家进行调节，而动作专家产生的动作又会改变输入到VLM 的状态

具体来说，VLM 处理传感-运动状态，包括来自多个RGB 摄像头的图像，以及描述任务的语言指令。随后，VLM 输出的特征会被直接输入到动作专家，动作专家输出最终的连续动作

第一，对于视觉-语言模型（VLM）而言

作者利用预训练的视觉-语言模型作为机器人环境感知的主要骨干。视觉-语言模型在多样的多模态数据上进行预训练，能够捕捉丰富的世界知识  
  
为了保证高效性和可访问性，作者选择了SmolVLM-2（Marafioti等，2025），这是一款针对多图像和视频输入优化的高效模型

SmolVLM-2 **由SigLIP和SmolLM2组成** ，具体而言，其依赖SigLIP（Zhai等，2023）对视觉特征进行编码，为SmolLM2语言解码器（Allal等，2025）提供输入  
  
在SmolVLA中

1. VLM组件通过视觉编码器处理图像序列，并通过token洗牌技术减少token数量以提高效率
2. 语言指令被分词为文本token
3. 传感运动状态通过 **线性层** 投射为单一token，以匹配语言模型的token维度

最后，视觉、语言和状态token被拼接后传递给语言解码器。解码器层获得的结果随后用于为动作专家提供条件

第二，对于状态、动作与特征投影器

作者在 SmolVLA 的多个位置使用线性投影层

具体来说，使用线性投影层来：

1. (i) 将状态投影以匹配 VLM 的维度
2. (ii) 将动作投影以匹配动作专家的维度
3. (iii) 适配 VLM 特征，使其与动作专家的维度对齐

第三，视觉token减少

尽管高分辨率图像对于VLM性能至关重要，但它们会显著增加推理成本

- 为保证效率，SmolVLM-2采用了图像切片训练（Lin等，2023b），这是一种流行的方法，除了处理全局图像外，还会处理同一图像的多个裁剪区域
- 然而，为了获得更快的推理速度，作者并未采用切片方法，仅使用全局图像，并结合像素洗牌操作，将每帧的视觉token数量限制为64个

第四，通过跳层实现更快的推理速度

为了获得更快的推理时间，作者在VLM 中跳过了一些计算

1. 已有研究（Shukorand Cord, 2024；Tang et al., 2023）表明，在不显著降低性能的情况下，可以跳过预训练模型中的某些层  
	最近，（El-Nouby et al., 2024；Bolya et al., 2025；Rajasegaran et al., 2025）进一步证明，下游任务的最佳特征并不一定来自VLM 的最后一层
2. 因此，他们的动作专家并非只使用最后一层特征，而是可以访问直到指定层N 的所有特征。实际应用中，他们发现将N 设置为总层数的一半(N = L/2) 可以在速度和性能之间取得良好平衡，有效地将LLM 和动作专家的计算成本减半

第五，类似π0的流匹配动作专家

动作专家 $\mathbf{v}_{\theta}$ 被训练用于从VLM 特征中预测一个动作片段 $\mathbf{A}_{t}=\left(a_{t}, \ldots, a_{t+n}\right)$

- 与以往的工作一致，他们实现的 $\mathbf{v}_{\theta}$ 依赖于transformer 架构（Vaswani, 2017）
- 与以往的VLA 架构不同，他们交错使用了交叉注意力层和自注意力层，因此将条件流匹配Transformer（Esser 等，2024；Liu, 2022；Lipman 等，2022）用作 $\mathbf{v}_{\theta}$

动作专家的训练目标定义如下：

$$
\mathcal{L}^{\tau}(\theta)=\mathbb{E}_{p\left(\mathbf{A}_{t} \mid \mathbf{o}_{t}\right), q\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)}\left[\left\|\mathbf{v}_{\theta}\left(\mathbf{A}_{t}^{\tau}, \mathbf{o}_{t}\right)-\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)\right\|^{2}\right]
$$

- 其中， $\mathbf{o}_{t}$ 表示从第N 层VLM 中提取的观测 ![o_{t}](https://latex.csdn.net/eq?o_%7Bt%7D) 的VLM 特征，且 $\mathbf{A}_{t}^{\tau}=\tau \mathbf{A}_{t}+(1-\tau) \epsilon$ ，其中 $\epsilon \sim \mathcal{N}(0, \mathbf{I})$
- 特别地， $\mathbf{v}_{\theta}$ 被训练用于从VLM 特征和带噪动作 $\mathbf{A}_{t}^{\tau}$ 输出向量场 $\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)=\epsilon-\mathbf{A}_{t}$  
	按照Black 等人（2024）——即π0的做法，作者从Beta 分布中采样τ
- 为了提升推理效率，作者对 $\mathbf{v}_{\theta}$ 使用了0.75 × d 的较小隐藏层规模，其中 ![d](https://latex.csdn.net/eq?d) 为VLM的隐藏维度

> 怎么理解上面这一大段呢？没事，不急，也莫慌，我在 **解读π0的这篇文章** 《 [π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)](https://blog.csdn.net/v_JULY_v/article/details/143472442 "π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)") 》中其实已经解释的很明白了
> 
> ![](https://i-blog.csdnimg.cn/direct/bccb74b5666f49bea6136210bf808205.png)
> 
> ---
> 
> 在训练过程中，使用条件流匹配损失Conditional Flow Matching\[28,32\]对这些动作token进行监督「 ***前者* $\mathbf{v}_{\theta}\left(\mathbf{A}_{t}^{\tau}, \mathbf{o}_{t}\right)$ *为学习网络 相当于预测的噪声，后者 $\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)$ 为学习目标 相当于添加的真实噪声，即训练前者去逼近后者*** 」
> 
> $$
> L^{\tau}(\theta)=\mathbb{E}_{p\left(\mathbf{A}_{t} \mid \mathbf{o}_{t}\right), q\left(\mathbf{A}_{\tau}^{\tau} \mid \mathbf{A}_{t}\right)}\left\|\mathbf{v}_{\theta}\left(\mathbf{A}_{t}^{\tau}, \mathbf{o}_{t}\right)-\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)\right\|^{2}
> $$
> 
> 其中下标 ![t](https://latex.csdn.net/eq?t) 表示机器人时间步，上标 $\tau \in[0,1]$ 表示流匹配时间步
> 
> - 最近在高分辨率图像\[14\]和视频\[38\]合成方面的研究表明，当流匹配与简单的线性高斯(或最优传输)概率路径\[28\]结合时，可以实现强大的经验性能  
> 	  
> 	具体由下述表达式给出  
> 	$q\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)=\mathcal{N}\left(\tau \mathbf{A}_{t},(1-\tau) \mathbf{I}\right)$
> - 在实践中  
> 	**第一步** ，一般都是网络先通过随机采样符合正太分布的噪声 $\epsilon \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$ 进行训练，计算“带噪声的动作”  
> 	$\mathbf{A}_{t}^{\tau}=\tau \mathbf{A}_{t}+(1-\tau) \epsilon$  
> 	*In practice, the networkis trained by sampling random noise ϵ ∼N(0, I), computingthe “noisy actions” Aτt = τAt + (1 −τ)ϵ*  
> 	  
> 	*相当于先加噪，类似 *[此文](https://blog.csdn.net/v_JULY_v/article/details/136318383 "此文")* 中「5.2.1 通过示意图对比：ϵ-prediction、v-prediciton与rectified flow」最后对rectified flow的阐述： 或*  
> 	相当于从动作分布 $\mathbf{A}_{t}$ 到噪声分布 $\mathbf{A}_{t}^{\tau}$  
> 	*之后，* *再* *去噪*  
> 	  
> 	故，便有 **第二步** ：然后 **训练网络输出 $\mathbf{v}_{\theta}\left(\mathbf{A}_{t}^{\tau}, \mathbf{o}_{t}\right)$ 「** *此为动作块的向量场表示， $\mathbf{v}_{\theta}$ 代表预测的噪声* **」 ，以匹配去噪向量场 *$\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)$* (** $\mathbf{u}$ 代表添加的真实噪声**)** 「 *即training the network outputs vθ (Aτt, ot) to match the denoising vector field u(Aτt |At) = ϵ −At.*」  
> 	  
> 	所以才有上面提到的损失函数  
> 	$L^{\tau}(\theta)=\mathbb{E}_{p\left(\mathbf{A}_{t} \mid \mathbf{o}_{t}\right), q\left(\mathbf{A}_{\tau}^{\tau} \mid \mathbf{A}_{t}\right)}\left\|\mathbf{v}_{\theta}\left(\mathbf{A}_{t}^{\tau}, \mathbf{o}_{t}\right)-\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)\right\|^{2}$  
> 	  
> 	*啥意思呢？  
> 	相当于得到了所添加的真实噪声之后，便可以通过该公式 $\mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)=\epsilon-\mathbf{A}_{t}$ ，计算得到 $\mathbf{A}_{t} =\epsilon - \mathbf{u}\left(\mathbf{A}_{t}^{\tau} \mid \mathbf{A}_{t}\right)$*

第六，交错的交叉和因果自注意力层

动作专家 $\mathbf{v}_{\theta}$ 在VLM 特征的条件下生成动作片段，SmolVLA 中VLM 与动作专家之间的交互由注意力机制促进

- 与以往仅依赖自注意力（SA）（Black 等，2024）或交叉注意力（CA）（Bjorck 等，2025）的工作不同， **作者采用了一种交错的方法，其中每个块包含一个CA 或SA 层**  
	这一设计选择也有别于标准的VLM 架构，后者的每个解码器块通常同时包含SA 和CA 层（Laurençon 等，2023；Alayrac 等，2022；Chen 等，2022）
- 在动作专家的前向传播过程中，动作与VLM 特征之间的交互通过注意力机制进行，将tokens 投影为query、key 和value（Vaswani，2017）  
	在作者的设置中， **交叉注意力CA 层对VLM 的key 和value 进行交叉注意** ，而SA 层则允许vθ 中的动作token 彼此关注
	![](https://i-blog.csdnimg.cn/direct/377124cae7884f41b0a5561be1de239f.png)
	作者在SA 层中采用因果注意力掩码，确保每个动作token 只能关注片段内的过去token，防止未来动作依赖  
	实证结果表明，交错使用CA 和SA 层能够带来更高的成功率和更快的推理速度

此外，自注意力机制有助于生成更平滑的动作片段 $\mathbf{A}$ ，这一点在真实机器人评估中尤为明显

#### 1.2.2 社区收集的预训练数据

在机器人领域，用于大规模预训练的数据量仍然远远小于近期视觉和语言领域取得突破所依赖的数据规模。例如，自然语言基础模型能够从独特的文本界面以及互联网海量数据中获益，而机器人数据集的整合与扩展却显得复杂，原因包括：

1. (i) 各数据集之间的差异
2. 以及 (ii) 数据收集高度依赖人类专家的远程操作

此外，机器人形态、传感器、驱动方式、控制频率和数据格式的高度异质性导致了“数据孤岛”（Bjorck 等，2025）现象——分散的机器人数据集难以实现有效整合

在这种背景下，低端机器人平台的出现以及标准化机器人库的普及，直接缓解了数据异质性问题

1. 与遵循标准化协议的学术数据集不同，社区数据集天然涵盖了多样的机器人实体、控制方案、摄像机视角和任务类型
2. 同时，社区数据集通过包含噪声演示、异质环境和多样的物体交互，真实反映了现实世界的复杂性，因此作为预训练数据具有重要价值

在本研究中，作者从 Hugging Face 获得的社区数据集中筛选出481个子集，筛选标准包括机器人实体类型、回合数量、整体数据质量以及帧覆盖率（见表1）

- 利用VLM进行任务标注  
	依赖社区贡献的数据集会带来标准化方面的挑战。作者观察到任务标注中存在大量噪声——即针对特定数据集，机器人预期行为的自然语言描述  
	更为关键的是，不同数据集包含了诸如task desc等含糊的占位符，或是过于模糊的指令（如Hold或Up），甚至完全缺失操作说明  
	  
	为了提升标注质量，作者 **采用了现成的VLM（Qwen2.5-VL-3B-Instruct）自动生成简明的任务描述**  
	针对每个数据集，作者抽取了具有代表性的帧，并将其与原始指令一同提供  
	模型被提示生成一句简短、以行动为导向的句子，用于总结该行为  
	*完整的提示内容详见附录A.1*
- 摄像机视角归一化  
	使用社区数据集时，另一个挑战在于摄像机命名规范高度多样。例如，数据集中对images.laptop 的引用，可能表示顶部、侧面或手腕安装视角，具体取决于数据集本身  
	  
	作者发现这种不一致性在预训练阶段具有负面影响，而一致的摄像机排序对于该数据场景下的训练非常有益  
	为了解决这一标准化难题，他们手动将每个摄像机映射到标准化的视角类型——优先选择顶部、手腕和侧面视角——并分别将其重命名为 OBS\_IMAGE\_1、OBS\_IMAGE\_2 和 OBS\_IMAGE\_3  
	*对于包含更多视角的数据集，原有顺序得以保留，但训练过程中未使用的视角会被舍弃。未来可以通过 VLMs 自动化此流程，或提出/采用标准化的数据采集规范*

#### 1.2.3 异步推理：动作预测与动作执行解耦

**现代视觉运动控制策略** （Zhao 等，2023；Chi 等，2023；Black 等，2024）输出动作片段序列 $\pi\left(o_{t}\right)=\mathbf{A}_{t}$ ，其中 $\mathbf{A}_{t}=\left(a_{t}, a_{t+1}, \ldots, a_{t+n}\right)$ 是一个长度为n（远大于1）的低层指令序列，这些指令被加入到动作队列中，源自环境观测 ![o_{t}](https://latex.csdn.net/eq?o_%7Bt%7D)

1. 通常，机器人会执行完整的动作片段 $\mathbf{A}_{t}$ ，然后才将新的观测 ![o_{t+n}](https://latex.csdn.net/eq?o_%7Bt+n%7D) 传递给策略 $\pi$ 以预测下一个片段。这导致在每隔 ![n](https://latex.csdn.net/eq?n) 个时间步采集一次观测之间，推理过程是开环的  
	  
	包括Zhao 等（2023）和Chi 等（2023）在内的工作采用了不同的策略，其中机器人控制器交替进行片段预测 $\mathbf{A}_{t} \leftarrow \pi\left(o_{t}\right)$ 和片段消耗 $a_{t} \leftarrow \operatorname{PopFront}\left(\mathbf{A}_{t}\right)$ ，在每一个时间步 ![t](https://latex.csdn.net/eq?t) 计算新的动作片段，并在重叠区间内聚合预测的片段  
	  
	尽管这种方法能够自适应——每个时间步都会处理每一次观测 ![o_{t}](https://latex.csdn.net/eq?o_%7Bt%7D) ——但它依赖于持续运行推理，这在资源受限的场景（如边缘部署）中可能是难以承受的
2. 一种资源消耗较低的方法是，在预测新一批动作之前，完全执行完chunk  $\text { A }$  ，他们将这种策略称为同步（sync）推理  
	此外，同步推理能够在每个时间步高效分配计算资源，从而在控制时减少平均计算负担。相比之下，这种方式本质上会阻碍机器人系统的响应性，因为机器人在计算  $\text { A }$  时处于空闲状态，从而引入了不可见的延迟

**为了将动作块预测与动作执行解耦**

- 如果直接评估机器人系统由于采用开环控制而缺乏自适应性，以及运行时存在的延迟
- 为此，作者开发了一种异步（async）推理栈（算法1） ![](https://i-blog.csdnimg.cn/direct/f76d8a90873b425cb215e57a7c2b2796.png) 其中RobotClient会将观测 ![o_{t}](https://latex.csdn.net/eq?o_%7Bt%7D) 发送到PolicyServer，在推理完成后接收动作块 $\mathbf{A}_{t}$ —— *见下图图2 请注意，策略可以在远程服务器上运行，并且可能配备 GPU*
	![](https://i-blog.csdnimg.cn/direct/375c9ae3f670419c95a8b39ce83632c4.png)

在此过程中，作者通过在控制循环仍在消费先前可用队列时触发块预测，并在有新队列可用时将其与新到达的队列聚合，从而避免了执行延迟

1. 反过来，异步推理通过提高观测被处理以进行块预测的频率，进一步收紧了动作预测与动作执行之间的闭环
2. 关键的是，将动作预测与动作执行解耦还可以直接让远程策略服务器分配更多计算资源，通过网络向机器人客户端发送动作，这在如低功耗机器人等资源受限场景下可能非常有效

**对于实现细节，如下图的伪代码所示**

![](https://i-blog.csdnimg.cn/direct/be1505ce13e4407ca58fb0b3d364f6e7.png)

1. (i) 通过更频繁地捕捉观测数据，收紧了控制回路，直接消除了运行时的空闲间隙
2. (ii) 允许直接在比自主机器人平台通常所配备的更强大的计算资源上运行推理

在ROBOT CLIENT 端，作者通过从一个现成的队列中消费动作，直到队列中剩余动作的数量满足阈值条件 $\left|\mathbf{A}_{t}\right| / n<g$ ，从而实现了(i)。当该条件被触发时，会捕获并发送环境的新观测到（可能是远程的）POLICY-SERVER

1. 为了避免冗余的服务器调用和运行时的异常行为，观测会在关节空间中进行比较，近似重复的观测会被丢弃。如果两个观测在关节空间中的距离低于预设阈值 $\epsilon \in \mathbb{R}_{+}$ ，则被认为是近似重复的
2. 需要注意的是，当机器人客户端可用的队列最终变为空时，无论观测是否相似，都会处理最近的一次观测

有趣的是，异步推理的行为可以通过分析方法进行研究

![](https://i-blog.csdnimg.cn/direct/375c9ae3f670419c95a8b39ce83632c4.png)

1. 首先，设 $\ell$ 为一个随机变量，表示在发送观测值 ![o](https://latex.csdn.net/eq?o) 后接收动作块A 所需的时间，即  
	(i) ROBOTCLIENT 与POLICYSERVER 之间发送观测值o 的时间 $t_{C \rightarrow S}$  
	(ii)POLICYSERVER 上的推理延迟 $\ell_{S}$  
	以及(iii) POLICYSERVER 与ROBOTCLIENT 之间发送A 的时间 $t_{S \rightarrow C}$  
	  
	假设三者相互独立，则 $\mathbb{E}[\ell]=\mathbb{E}\left[t_{C \rightarrow S}\right]+\mathbb{E}\left[\ell_{S}\right]+\mathbb{E}\left[t_{S \rightarrow C}\right]$ ，进一步可简化为 $\mathbb{E}[\ell] \simeq \mathbb{E}\left[\ell_{S}\right]$ ，假设通信时间(i) 在两个方向上相等，且(ii) 相较于推理延迟可以忽略不计
2. 其次，设 $\Delta t$ 为环境的控制周期。在真实世界中，帧率为每秒30 帧，则 $\Delta t=33 \mathrm{~ms}$  
	因此，在运行时避免队列耗尽——即因等待新块而空闲——的条件为 $g \geq \frac{\mathbb{E}\left[\ell_{S}\right] / \Delta t}{n}$  
	在这里，队列阈值 ![g](https://latex.csdn.net/eq?g) 对于ROBOTCLIENT 可用动作的充足性起着关键作用

图3(A)展示了在三种具有代表性的g值下，动作块 $\left|\mathbf{A}_{t}\right|$ 随时间演变的过程，详细说明了以下关键情形：

// 待更