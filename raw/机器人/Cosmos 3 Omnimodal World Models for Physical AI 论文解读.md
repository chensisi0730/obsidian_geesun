63 人赞同了该文章

目录

项目主页： [huggingface.co/blog/nvi](https://link.zhihu.com/?target=https%3A//huggingface.co/blog/nvidia/cosmos-3-for-physical-ai)  
技术报告： [research.nvidia.com/lab](https://link.zhihu.com/?target=https%3A//research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf)

![](https://pic3.zhimg.com/v2-6f0e4bdcf161b42f3f76dd71f14b48cc_1440w.jpg)

这篇工作的目标是做一个 **[omnimodal world model](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=omnimodal+world+model&zhida_source=entity)** ：同一个模型族同时支持理解、生成、仿真和动作预测，模态覆盖 text、image、video、audio、action。Cosmos 3 的定位是一个 [Physical AI](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=Physical+AI&zhida_source=entity) 基座： [Reasoner](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=Reasoner&zhida_source=entity) 负责看懂世界、做物理/空间/任务推理； [Generator](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=Generator&zhida_source=entity) 负责生成图像、视频、音频和动作，或者在给定动作条件下 rollout future observation。

## 主要创新点

- 从“多个专用模型”变成一个 omnimodal world model：Cosmos 1 / Cosmos 2 更像是把 physical understanding、world generation、controlled generation 分别做成不同能力； [Cosmos 3](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=2&q=Cosmos+3&zhida_source=entity) 的核心变化是把这些能力合到一个模型框架里。Cosmos 3 希望把“ [VLM](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=VLM&zhida_source=entity) 看图、world model 预测视频、policy 再预测动作”这些步骤变成同一模型的不同运行模式：同一个系统既能回答“这个场景是否物理合理”，也能根据 action 预测未来视频，还能从视频反推出 action。
- Cosmos 3 的核心结构是一个 **[two-tower Mixture-of-Transformers](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=two-tower+Mixture-of-Transformers&zhida_source=entity) (MoT)** 。Reasoner 是 [autoregressive tower](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=autoregressive+tower&zhida_source=entity) ，用 causal self-attention 做 next-token prediction，主要负责语言、视觉理解、物理推理和 [grounding](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=grounding&zhida_source=entity) ；Generator 是 diffusion transformer tower，用 full attention 对 noisy image/video/audio/action token 做去噪，负责 [连续模](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E8%BF%9E%E7%BB%AD%E6%A8%A1&zhida_source=entity) 态生成。
- Reasoner-to-Generator，先理解/推理，再生成/仿真：Reasoner 和 Generator 不是平行孤立的两个模型。Reasoner 先通过 [AR](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=AR&zhida_source=entity) 方式分析图像、视频和文本；Generator 再在 Reasoner 的条件下，通过 [diffusion](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=2&q=diffusion&zhida_source=entity) 过程生成未来 observation 或 action。Generator 运行时会激活两个 tower，也就是生成不是纯 diffusion，而是带有 reasoning condition 的 diffusion。
- action 也作为输入/输出模态之一。在 action 用法上，它同时支持三类任务：
- **forward dynamics** ：给定起始图像/视频和 action trajectory，预测未来 observation；
	- **inverse dynamics** ：给定视频变化，反推出 [ego-motion](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=ego-motion&zhida_source=entity) 或机器人动作；
	- **policy generation** ：给定语言和视觉观测，直接生成 action trajectory。

## 模型与方法

![](https://picx.zhimg.com/v2-2deef464e19b91feb9d5eae09a306d79_1440w.jpg)

Cosmos 3 的核心设计是一个 **AR-DM hybrid omnimodal transformer** 。 其中：

- **AR subsequence / Reasoner** ：处理语言 token，以及用 ViT encoder 编码后的 image/video understanding token；训练/推理方式类似 VLM，用 causal self-attention 做 next-token prediction。
- **DM subsequence / Generator** ：处理 VAE 编码后的 image/video latent、audio latent、action token；训练目标是 diffusion / rectified-flow denoising，推理时通过 iterative denoising 生成 image、video、audio、action。
- **[MoT backbone](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=MoT+backbone&zhida_source=entity)** ：每层有两套 [路由参数](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0&zhida_source=entity) ，一套给 Reasoner token，一套给 Generator token；两者不是完全隔离，Generator token 可以通过 [joint attention](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=joint+attention&zhida_source=entity) 看到 Reasoner context。官方模型卡也明确说 Cosmos 3 由 [autoregressive transformer](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=autoregressive+transformer&zhida_source=entity) 和 diffusion transformer 两个互补 tower 组成，文本用 AR 解码，非文本模态通过 [denoising](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=3&q=denoising&zhida_source=entity) 生成。

### Encoders：不同模态先映射到统一 hidden space

Cosmos 3 支持 language、image/video、audio、action。进入 MoT backbone 之前，每种模态先经过自己的 encoder / [tokenizer](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=tokenizer&zhida_source=entity) ，再投影到 transformer hidden dimension。对于 [非语言模态](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E9%9D%9E%E8%AF%AD%E8%A8%80%E6%A8%A1%E6%80%81&zhida_source=entity) ，论文还会加一个 learnable modality-specific [embedding](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=embedding&zhida_source=entity) ，让共享的 [位置编码](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E4%BD%8D%E7%BD%AE%E7%BC%96%E7%A0%81&zhida_source=entity) 和 attention 能区分“这是 vision token、audio token 还是 action token”。

**Language encoder**  
语言部分沿用 VLM/LLM 的 tokenizer。语言 token 只进入 **AR subsequence** ，由 Reasoner tower 处理。 在纯语言生成、VQA、 [captioning](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=captioning&zhida_source=entity) 、embodied reasoning、planning 这类任务里，Cosmos 3 就像一个标准 VLM：语言和 ViT 视觉 token 组成 AR prefix，然后 autoregressively 输出文本。

**Image / Video：两套 [视觉编码器](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E8%A7%86%E8%A7%89%E7%BC%96%E7%A0%81%E5%99%A8&zhida_source=entity)**  
Cosmos 3 对视觉用了两套 encoder，分别服务 understanding 和 generation。  
ViT encoder：用于视觉理解的 image/video 会通过 **ViT encoder** 编成 AR token。这部分对应 Reasoner 的“看懂图像/视频”的能力，比如 captioning、VQA、temporal localization、physical plausibility、robot task reasoning。  
VAE encoder：服务 Generator / generation，用于 image/video generation 的视觉 token 不是 ViT token，而是 **Wan2.2-TI2V-5B 的 video VAE latent** 。

**Audio encoder：** audio generation 使用一个 frozen audio VAE

**Action encoder**  
action 是 Cosmos 3 和普通视频 [生成模型](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E7%94%9F%E6%88%90%E6%A8%A1%E5%9E%8B&zhida_source=entity) 最大的区别之一。Cosmos 3 的 action input 是 per-frame action sequence，不同 [embodiment](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=embodiment&zhida_source=entity) 有不同维度。

Cosmos 3 把不同 embodiment 的动作都拆成三类共享几何组件：

1. **Ego pose** ：agent 主观察 [坐标系](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E5%9D%90%E6%A0%87%E7%B3%BB&zhida_source=entity) 的运动，例如相机/车辆/head camera pose
2. **Effector pose** ： [机械臂](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E6%9C%BA%E6%A2%B0%E8%87%82&zhida_source=entity) 末端、手腕、手等 [执行器](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E6%89%A7%E8%A1%8C%E5%99%A8&zhida_source=entity) pose
3. **Grasp state** ：夹爪开合、手指状态、fingertip position 等 manipulation state

### Token arrangement

Cosmos 3 的 token sequence 被固定拆成两段：\[AR subsequence, DM subsequence\]

AR 部分负责 reasoning / understanding，里面有language tokens和ViT-encoded image/video tokens。这些 token 全部送到 Reasoner tower，用 causal self-attention。  
  
DM 部分负责 generation / denoising，里面有VAE image/video tokens、audio tokens、 [action tokens](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=action+tokens&zhida_source=entity) 。  
DM subsequence 内部也有统一顺序。 AR tokens 永远放在 diffusion tokens 前面； 在 diffusion subsequence 内，每个模态都是 **clean conditioning tokens 在前，noisy target tokens 在后** ； 在 conditioning 和 noisy target 内部，模态顺序是 **vision -> audio -> action** 。  
  
**不同 generation mode 的 token 形式**

**Language mode：** 只有 AR subsequence，Generator tower 不激活。 模型就是一个标准 VLM / [multimodal LLM](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=multimodal+LLM&zhida_source=entity) ，输出文本。

**Text-to-Image：** AR 是 prompt，DM 是 noisy image latent

**Text-to-Video / Text-to-Video + Audio:**AR 是 prompt，DM 是 noisy video latent；如果生成 audio，则 audio noisy token 接在 video 后面

**Image-to-Video / Video-to-Video：** I2V/V2V 引入 clean conditioning frames，第一帧或前几帧是 clean condition，后续 [视频帧](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E8%A7%86%E9%A2%91%E5%B8%A7&zhida_source=entity) 是 noisy target。Generator 用 full attention 在 diffusion subsequence 内同时看 clean prefix 和 noisy future。

**Video Transfer：** video transfer 是 [控制条件](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E6%8E%A7%E5%88%B6%E6%9D%A1%E4%BB%B6&zhida_source=entity) 生成，例如 edge/depth/segmentation/control video -> RGB video。clean control video token 在前，noisy RGB target token 在后。

**Action modes：** 支持三种模式，Forward Dynamics、Inverse Dynamics、Policy / Joint Video-Action Prediction（同时 denoise future video 和 action）

![](https://pic4.zhimg.com/v2-8eb923edf819203cbceb17e0a4cad937_1440w.jpg)

### MoT backbone：双塔、共享注意力上下文

Cosmos 3 的 backbone 是 **Mixture-of-Transformers** ，但这里的 MoT 不是常见 MoE 那种 token routing 到多个专家，而是更像 **[modality](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=2&q=modality&zhida_source=entity) /function-aware dual tower** 。

每一层 transformer decoder 都有两套参数：Reasoner tower parameters: 处理 AR tokens，Generator tower parameters: 处理 DM tokens。Reasoner 保留 VLM 的 language/vision reasoning 能力；Generator 虽然要学 diffusion denoising，但一开始也继承了 VLM 的语义和视觉理解能力。  
  
**Reasoner tower：** Reasoner tower 的 attention 是标准 causal attention，它只能看 AR subsequence 内自己之前的 token。 DM tokens 不会反过来更新 AR tokens。这样可以保持 language generation 的自回归一致性，不会因为 diffusion target token 泄漏而破坏 VLM 的文本能力。

**Generator tower：** Generator tower 的 attention 是 full attention，但 keys/values 同时来自 AR 和 DM。 每个 DM token 可以看完整 AR context，也可以看同一个 diffusion subsequence 里的 clean condition token 和 noisy target token， DM token 之间是 bidirectional full attention， 但 AR token 不看 DM token。  
Reasoner -> Generator: 可以条件化生成  
Generator -> Reasoner: 不反向污染 AR 推理

## 训练流程和数据

大致流程是先训练 **Reasoner** ，目标是让模型具备视觉语言理解、 [空间推理](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E7%A9%BA%E9%97%B4%E6%8E%A8%E7%90%86&zhida_source=entity) 、时间推理和 Physical AI 任务推理能力； 然后用训练好的 Reasoner [权重初始化](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E6%9D%83%E9%87%8D%E5%88%9D%E5%A7%8B%E5%8C%96&zhida_source=entity) **Generator** ，让 Generator 从 Reasoner 继承语义理解和 [世界知识](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E4%B8%96%E7%95%8C%E7%9F%A5%E8%AF%86&zhida_source=entity) ； Generator 再经过 image/video/audio 预训练、引入 action 和 transfer 的 [mid-training](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=mid-training&zhida_source=entity) ，最后针对 T2I、I2V、Robot Policy 各自做 [post-training](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=post-training&zhida_source=entity) 。

Cosmos 3 的训练分为 5 个主阶段：

1. **Reasoner Pre-training** 学 general multimodal understanding：OCR、VQA、captioning、grounding、image/video reasoning、text instruction。
2. **Reasoner SFT** 把 Reasoner 往 Physical AI 任务上专门调：robotics、 [autonomous driving](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=autonomous+driving&zhida_source=entity) 、smart infrastructure、spatial grounding、temporal localization、physical plausibility、action CoT。
3. **Generator Pre-training** 用 Reasoner 权重初始化 Generator；冻结 Reasoner tower，只训练 generation-specific 参数；学习 image/video/audio 的 rectified flow generation。
4. **Generator Mid-training** 在保留 image/video/audio 生成能力的同时，引入 action、video transfer、driving transfer，让模型学习 action-conditioned world modeling、inverse dynamics、policy generation、control-conditioned generation。
5. **Generator Post-training** 分别得到专家模型Nano/Super 是通用 omnimodal model，Policy-DROID 用语言和视觉观测生成机器人动作，Image2Video/Text2Image 是专门 generation checkpoint
![](https://pica.zhimg.com/v2-0df5ddee538201a34384f5e2e8e1e796_1440w.jpg)

### 阶段 1：Reasoner Pre-training

Reasoner 是 Cosmos 3 里负责“看懂世界”的 tower。训练目标是标准 **next-token prediction** ，输入是 image-text、video-text、text-only conversation。模型从 language model + ViT encoder + multimodal projector 开始训练。  
Reasoner [pre-training](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=pre-training&zhida_source=entity) 总共 **22.0M samples** 。其中：

- **19.7M** 来自 Nemotron Nano 2 data collection 的子集；
- **2.3M** 是额外 curated 数据，用来增强 math、video、spatial grounding、instruction-following。

Reasoner pre-training 仍然是 **image-text / OCR / grounding 主导** ，视频占比不大。论文的解释是：pre-training 先建立 general vision-language alignment、reading、spatial grounding；更强的视频和 Physical AI 时序能力放到后面的 SFT 阶段补。

Reasoner pre-training 数据有两级清洗。

第一步是 **semantic deduplication** 。 每个 training example 被看成一个 conversation：可以是 image/video + instruction-response，也可以是纯文本 instruction-response。

- image-text / text-only 用 **[Qwen3-VL-Embedding-8B](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=Qwen3-VL-Embedding-8B&zhida_source=entity)** 取 embedding；
- video-text 用 **Perception Encoder PE-Core-G14-448** 取 embedding；
- 然后做 K-means clustering；
- cluster 内用 [cosine similarity](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=cosine+similarity&zhida_source=entity) 找 near-duplicate；
- similarity > **0.95** 的样本被去掉。 论文报告 multimodal dedup 去掉了 **4.23%** 数据。

第二步是 **AI-judge quality filtering** 。 用 **Gemma-4-31B-it** 当 judge，从三个维度打 1-5 分：

- Faithfulness：回答是否被图像/视频/文本支持；
- Completeness：是否完整回答 instruction；
- Correctness：事实、逻辑、任务答案是否正确。

Reasoner pre-training 使用较宽松 threshold=2，因为目标是保留覆盖面；SFT 使用 threshold=5，因为目标是高质量监督。AI-judge 在 threshold=2 时保留 **78%** ，threshold=5 时保留 **46%** 。

### 阶段 2：Reasoner Supervised Fine-Tuning

Reasoner SFT 的目标不是继续泛化预训练，而是把模型专门推向 Physical AI reasoning。SFT 聚焦三个 domain：

- autonomous vehicle
- robotics
- smart infrastructure

同时增强 general spatial understanding 和 temporal understanding。

SFT 使用 importance-aware sampling：每个 dataset 根据重要性、质量和规模分配固定 sampling budget。为了防止专业化后 general capability 掉太多，作者还混入 filtered high-quality pre-training data，比例是pre-training: SFT = 1: 4，另外还加入 800K lightweight instruction-following samples，用于稳定聊天和 instruction following 能力。

Reasoner SFT 总共 2.2M samples：

- Image-text：1,051,513
- Video-text：1,079,200
- Text-only：40,960
- Total：2,171,673

### 阶段 3：Generator Pre-training

Generator 是 diffusion tower，训练目标是 rectified flow matching。Generator pre-training 只训练 image、video、audio，不引入 action 和 transfer。

数据量：

- Text-to-Image：767M images
- T2V / I2V / V2V：348M video clips，论文正文更精确写 347.7M video clips
- Video+Audio：139M audio-video clips，正文更精确写 138.9M usable audio clips

视觉数据来自：

- 7.8B raw images
- 3B raw source videos
- 过滤后得到 767M images + 347.7M videos+Audio 139M、Action 8M

公开来源包括 OpenImages、COYO-700M、YouTube Video、UMI，私有来源包括 Egocentric、Nexar、AgiBot、HOI，合成来源包括 HiDream-I1、Qwen-Image-2512 和 Qwen3-VL synthetic captions。

视觉数据的清洗pipeline 是：

1. raw data collection + preprocessing：视频先用 **TransNetV2** 做 scene-change detection，把长视频切成时间连续 clip；再用 `ffmpeg cropdetect` 去黑边，统一编码格式。
2. embedding + deduplication ：Qwen3-VL-Embedding-8B 提图像embedding，Cosmos-Embed1-448p 提视频embedding， 从全量里采样 147M images 和 400M video clips，各自用 cuML KMeans 做 20,000 clusters，cluster 内用 cosine similarity 去重
3. categorization + basic filtering ： in-house VLM models 做 semantic tagging 和 quality filtering ， image/video 都分到 47 个 hierarchical categories，
4. structured caption annotation
5. 按 resolution/duration 打成 training shards

### 阶段 4：Generator Mid-training

mid-training 是 Cosmos 3 从“通用 image/video/audio generator”变成“Physical AI world model”的关键阶段。训练loss仍然是 rectified flow。action 继承 vision noise schedule。总 loss 是各模态 [velocity](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=velocity&zhida_source=entity) MSE 的加权和。

它有两个目标：

**Domain specialization** ：增强 robotics、autonomous driving、human activity、 [physics](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=physics&zhida_source=entity) 、warehouse 等 Physical AI 场景，补 pre-training 中少见的 long-tail dynamics、fine-grained manipulation、safety-critical scenarios。

**Multimodal integration** ：引入 action 和 transfer，使模型能：

- action-conditioned future video prediction
- inverse dynamics：从 observed trajectory 推断 action
- joint action + future video generation
- edge/depth/segmentation/world-scenario-map conditioned transfer generation

**mid-training 的数据规模：**

- Image： mid-training image pool 是 15.6M samples
- Video：curated video clips 是 74.7M
- Video+Audio：final audio pool 是 18.8M
- Action： final curated action data 是 8.4M episodes，61.3K hours
- Video Transfer：4M，其中 general transfer 约 3M videos，driving transfer MADS 约 1.1M samples

mid-training 里有 5 类 [synthetic data](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=synthetic+data&zhida_source=entity) ：

| Dataset | Clips | Resolution / FPS | 主要用途 |
| --- | --- | --- | --- |
| [SDG](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=SDG&zhida_source=entity) -PhyxSim | 76,489 | 1920×1080 / 30 | 刚体碰撞、多物体交互、物理状态监督 |
| SDG-RobotSim | 208,022 in Fig. 8/Tab. 22；Appendix C public release 又写 386,270 RGB clips | varies | 机器人碰撞、操作、humanoid motion |
| SDG-DriveSim | 264,000 | 3840×2160 / 24 | [自动驾驶](https://zhida.zhihu.com/search?content_id=276014127&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E9%A9%BE%E9%A9%B6&zhida_source=entity) 长尾场景 |
| SDG-SynHuman | 236,937 | 1920×1080 / 30 | 数字人、多人体、人类运动 |
| SDG-Warehouse | 122,952 | 1920×1080 / 30 | 仓库安全、人-叉车交互、工业场景 |

这五类 synthetic data 的作用分别是：

- **SDG-PhyxSim** ：补真实视频里很难拿到的 dense ground-truth rigid-body dynamics；
- **SDG-RobotSim** ：补机器人 embodiment persistence、contact、long-horizon robot video；
- **SDG-DriveSim** ：补自动驾驶里稀有但关键的 long-tail scenarios；
- **SDG-SynHuman** ：补 human dynamics、camera-motion priors、multi-character interactions；
- **SDG-Warehouse** ：补 industrial warehouse safety、人-叉车交互、空间约束。

论文的 ablation 结论是单一 SDG 会带来 domain bias，但全部混合的 SDG-All 能更稳定地提升多个 Physical AI 维度，所以最终模型把 SDG 和 real high-quality videos 混在 dedicated mid-training stage 里。  
  
action 数据总规模8.4M episodes，61.3K hours，共四大来源：

- Egocentric Motion：1.7M episodes，41.3K hours，67.4%， 每只手有 21-keypoint 3D pose，包括 per-joint position 和 orientation
- Autonomous Vehicle：10.0K hours，16.3%，来自 NVIDIA Hyperion 平台的 in-house driving logs， 覆盖天气、光照、道路条件、纵向/横向 maneuver
- Robotics：5.4K hours，90.4K tasks，516.7K episodes，8.7%。 action 不直接用低层 controller，而是用 state difference 构造 pseudo-actions，避免 PID/controller interface 的 embodiment-specific 噪声
![](https://picx.zhimg.com/v2-abb0448f947e4fb2fa6b4f73ae6b750f_1440w.jpg)

- Camera Motion：4.6K hours，7.5%，从 pre-training video dataset 里挖 ， 用 ViPE 和 DepthAnything3 估计 camera pose，转成 unified action coordinate convention。 去掉 pose estimation 不可靠的 clip，比如 jitter 过大、camera intrinsics 异常 ，最终得到 1.9M clips

### 阶段5：Post-training

**阶段 5A：Text-to-Image Post-training：** 只针对 **Cosmos3-Super-Text2Image** 。 目标是把 Cosmos 3 的 physical world understanding 迁移到高质量图像生成上。  
T2I post-training 是两阶段 SFT。第一阶段是broad T2I specialization。第二阶段是high-quality refinement  
**阶段 5B：Image-to-Video Post-training：** 只针对 Cosmos3-Super-Image2Video。目标是专门增强physical law understanding 、 object permanence 、 scene geometry 、 plausible future frames 、 embodied planning 需要的 future prediction 。训练数据使用1,000 high-quality manually curated videos，额外混入 20% T2I image tokens  
**阶段 5C：Robot Policy Post-training：** Robot Policy post-training 得到 **Cosmos3-Nano-Policy-DROID** 。 目标是验证 Cosmos 3 mid-trained omnimodal world model 能不能进一步变成 robot policy model。post-training 进一步做三件事：

- 加入 proprioception
- 降低推理延迟
- 输出可执行 robot action，用于 closed-loop control

训练数据为 **DROID数据，覆盖76K trajectories，350 hours，86 tasks，564 scenes， 分辨率：360×640，**  
输入的图像为三视角拼接， wrist-view原始 **360×640** ，放在上方；两个 external-view：各 **180×320** ，左右拼在下方。最终形成 **540×640**

## 实验结论

**Reasoning 能力**  
Cosmos3-Super 和 Cosmos3-Nano 分别在 VANTAGE-Bench 的 32B/8B tier 达到领先结果，并在 temporal action reasoning 相关评测中表现强。官方页面也强调它能做 physical concept grounding、vision-language reasoning、robot policy reasoning 等。  
**Generation 能力**  
在 generation 侧，官方宣称 Cosmos 3 在 R-Bench、PAIBench-G、Physics-IQ、RoboLab 等 physical-AI generation benchmarks 上达到 open-source SOTA 或领先结果。官方页面还说它在 Robotics、Smart Space、Driving 平均指标上位居开放模型第一，并在 text-to-image、image-to-video、robot policy 等开放生成任务上排名靠前。  
**Action / robot policy**  
Cosmos 3 的 action policy 代表模型是 `Cosmos3-Nano-Policy-DROID` ，输入语言和视觉观测，输出机器人 action trajectories。HuggingFace 模型卡把它列为 16B 级模型，并说明其用途是从 visual observations 和 language instruction 生成 robot action。  
Cosmos 3 的 action 能力不是单纯 behavioral cloning，而是和 forward dynamics / inverse dynamics 放在同一 omnimodal generation 框架里。也就是说，policy generation 可以受益于同一个模型学到的视觉动态、物理约束和动作-状态转移结构。

编辑于 2026-06-01 18:43・北京[机器人](https://www.zhihu.com/topic/19551273)[如何写好一篇SCI？并能够快速发表?](https://zhuanlan.zhihu.com/p/28659048449)

[我们课题组的老板是属于业界大佬，平常基本见不到人，我只在年度组会有幸见过大佬两次。我导是大佬众多手下中比较年轻也比...](https://zhuanlan.zhihu.com/p/28659048449)

赞同 63