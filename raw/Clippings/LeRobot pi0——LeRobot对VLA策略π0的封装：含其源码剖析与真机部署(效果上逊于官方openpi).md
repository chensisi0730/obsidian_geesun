## 前言

本文一开始是属于此文《LeRobot源码剖析——对机器人各个动作策略的统一封装：包含ALOHA ACT、Diffusion Policy、VLA模型π0》的第三部分，但

1. “π0和本博客” 的影响力实在太大了，影响力大的其中一个表现是：光私我加「 *七月具身：π0复现微调交流群* 」的朋友 在短短两个月 便已高达近100人，而群中大家对π0的各种疑问，使得我想把π0解读的更深入、更细致
2. 也为了避免《LeRobot源码剖析》一文的篇幅过长

故独立成本文

另，有一点值得大家特别注意下，即关于π0有两个版本的代码

- 一个是本文介绍的LeRobot对π0策略的封装，即 **LeRobot pi0**
- 一个是π0所在公司官方发布的 **openpi**  
	*$\rightarrow$ 关于 **π0的介绍** 请参见此文：《 [π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)](https://blog.csdn.net/v_JULY_v/article/details/143472442 "π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)") 》  
	$\rightarrow$ 关于 **openpi** 代码的解析，参见此文：《 [π0源码(openpi)剖析——从π0模型架构的实现：如何基于PaLI-Gemma和扩散策略去噪生成动作，到基于C/S架构下的模型训练与部署](https://blog.csdn.net/v_JULY_v/article/details/146068251 "π0源码(openpi)剖析——从π0模型架构的实现：如何基于PaLI-Gemma和扩散策略去噪生成动作，到基于C/S架构下的模型训练与部署") 》*

> 顺带，我司『七月在线』在南京团队之外，长沙团队也火速建立起来了「 *包括机器， *当然， 对* 于机器 ，我没有去动南京、武汉的机器，长沙陆陆续续上新设备，很快，会再有宇树G1* 」
> 
> ---
> 
> 而为尽快 让近期新招的新同事们，尽快具备对应的能力，我给新同事们 定了两个目标，即老同事不做太多辅助的情况下，六月份之内
> 
> 1. 通过协作机械臂，完成叠衣服的任务，对于该任务，会先尝试π0、dexvla
> 2. 通过宇树G1 edu，完成搬箱子的任务
> 
> 且我们长沙具身团队目前还在持续扩人(之后是上海、武汉再分别扩员)，目前已有来自华科、中南的，有意来我司全职或实习的，欢迎私我

## 第一部分 封装的pi0：涉及配置、模型训练/推理、attention优化等

该模块主要包含以下组件

![](https://i-blog.csdnimg.cn/direct/997d8f4e13204ff3b22408624659691f.png)

1. **转换工具 (conversion\_scripts/)**  
	包含将 pi0 模型转换为 HuggingFace 格式的脚本  
	提供了与 JAX 实现进行对比的工具  
	包含性能基准测试脚本
2. **配置系统 (configuration\_pi0.py)**  
	定义了 \`PI0Config\` 类，继承自 \`PreTrainedConfig\`  
	配置了模型的输入/输出结构、归一化映射、图像预处理参数  
	支持特定的机器人配置，例如针对 Aloha 机器人的适配  
	包含训练相关的参数设置，如学习率、权重衰减等
3. **注意力机制优化 (flex\_attention.py)**  
	提供了基于 PyTorch 的灵活注意力机制实现  
	针对 PyTorch 2.5.0 及以上版本的优化  
	支持分组查询注意力(GQA)以提高效率
4. **核心模型实现 (modeling\_pi0.py)**  
	实现了 \`PI0Policy\` 类，封装了训练和推理功能  
	实现了 \`PI0FlowMatching\` 类，这是核心的流匹配模型  
	包含对机器人电机角度的转换处理，尤其是针对 Aloha 机器人的特殊处理
5. **paligemma\_with\_expert.py**

可能马上就有同学疑问了，那这个模块和π0的官方实现库—— ***π0官方库的实现详见此文《 *****[π0源码剖析——从π0模型架构的实现(如何基于PaLI-Gemma和扩散策略去噪生成动作)，到基于C/S架构下的模型训练与部署](https://blog.csdn.net/v_JULY_v/article/details/146068251 "π0源码剖析——从π0模型架构的实现(如何基于PaLI-Gemma和扩散策略去噪生成动作)，到基于C/S架构下的模型训练与部署")***** 》的分析*** ，有何区别或不同呢？

1. 实现语言和框架差异  
	openpi: 使用 JAX 框架实现，这是一个为高性能数值计算设计的库  
	lerobot/pi0: 使用 PyTorch 框架实现，是 JAX 版本的移植版本  
	  
	包括从代码注释中也可以明确看到："Designed by Physical Intelligence. Ported from Jax by Hugging Face"，表明 lerobot 中的实现是由 Hugging Face 团队将原始 JAX 代码移植到 PyTorch
2. 集成与生态系统  
	openpi: 作为独立库存在，专注于 π0 模型本身  
	lerobot/pi0: 集成到更大的 LeRobot 框架中，遵循 LeRobot 的设计模式和接口标准  
	*例如，lerobot/pi0 实现中的 \`PI0Policy\` 类继承自 LeRobot 的 \`PreTrainedPolicy\` 接口，这使它能够与整个 LeRobot 框架的数据处理、训练和评估流程无缝集成*  
	  
	当然了，π0官方库本身也提供了类似「将Libero数据集转换为LeRobot数据集」的脚本
3. 多模态模型整合与加速模型推理  
	openpi: 可能需要手动配置与外部模型的交互  
	lerobot/pi0 中实现了一个特殊的 \`PaliGemmaWithExpertModel\` 类，用于整合 PaliGemma 多模态模型与 Gemma 专家模型  
	  
	*且lerobot 实现包含了针对 PyTorch 的优化，如灵活注意力机制 (\`flex\_attention.py\`)，用于加速模型推理——* *实现了KV cache*  
	*支持不同的注意力实现方式 (eager、fa2、flex)，可以根据硬件和性能需求进行选择*
4. 权重转换机制  
	lerobot/pi0 包含专门的转换脚本 (\`conversion\_scripts/convert\_pi0\_to\_hf\_lerobot.py\`)，用于将原始 JAX 模型权重转换为 PyTorch 格式  
	这显示 lerobot 的实现是基于原始模型的移植，而不是独立实现
5. 特有的适配性扩展  
	lerobot/pi0 添加了一些针对特定机器人硬件的适配功能，这些在原始 openpi 实现中可能不存在或实现方式不同：  
	*Aloha 机器人适配: 通过 \`adapt\_to\_pi\_aloha\` 参数配置，提供了专门处理 Aloha 机器人关节角度和夹爪位置的转换函数  
	空相机支持: 通过 \`empty\_cameras\` 参数支持额外的空相机输入，用于模拟缺失的摄像头视角*
6. 接口更简洁、使用更简单  
	lerobot 版本提供了更简洁的接口，例如：
	```cobol
	# 使用预训练模型
	policy = Pi0Policy.from_pretrained("lerobot/pi0")
	 
	# 微调模型
	python lerobot/scripts/train.py \
	--policy.path=lerobot/pi0 \
	--dataset.repo_id=danaaubakirova/koch_test
	```

总之，lerobot/common/policies/pi0 本质上是 openpi 官方 JAX 实现的 PyTorch 移植版本，由 Hugging Face 团队开发，专门适配 LeRobot 框架。这个移植版本保持了原始算法的核心功能，同时添加了适配性扩展和针对pytorch的优化，使其能够更好地适应 LeRobot 生态系统和更广泛的机器人硬件

两者最根本的区别在于实现语言（JAX vs. PyTorch），和集成框架（独立库 vs. LeRobot 框架组件）

### 1.1 转换conversion\_scripts：把JAX 实现的 π0 转换为 PyTorch 格式

在conversion\_scripts目录中，主要有以下4个文件：

1. benchmark.py
2. compare\_with\_jax.py
3. conversion\_utils.py
4. convert\_pi0\_to\_hf\_lerobot.py

conversion\_scripts 模块的主要目的是将 Physical Intelligence 公司开发的原始 JAX 实现的 π0 模型转换为 PyTorch 格式，以便在 LeRobot 框架中使用

从代码中可以确认

- 脚本支持将三种不同的模型变体转换为 PyTorch 格式：  
	\`pi0\_base\`: 基础模型  
	\`pi0\_aloha\_sim\`: 适用于 ALOHA 仿真环境的模型，包含空相机支持  
	\`pi0\_aloha\_towel\`: 适用于 ALOHA 真实机器人的模型，支持特殊的关节角度转换
- 原始 JAX π0 模型和转换后的 PyTorch 实现都使用了 Gemma 模型作为动作专家，而不是简单的 MLP 结构。这一点在 conversion\_utils.py 中的 \`get\_gemma\_config()\` 函数中得到了体现，该函数配置了一个 18 层、1024 隐藏单元的 Gemma 模型

#### 1.1.1 核心实现convert\_pi0\_to\_hf\_lerobot.py：将JAX格式的π0模型权重转换为PyTorch格式

这是核心转换脚本，负责将原始 JAX/Orbax 格式的 π0 模型权重转换为 PyTorch/HuggingFace 格式。主要功能包括：

转换流程

1. 从 Orbax 检查点加载 JAX 格式的模型权重
2. 提取 PaliGemma 视觉编码器和语言模型的权重
3. 提取 Gemma 动作专家模型的权重
4. 提取线性投影层的权重
5. 重新映射权重以匹配 PyTorch 模型的结构
6. 根据目标模型类型（pi0\_base、pi0\_aloha\_sim、pi0\_aloha\_towel）应用不同的配置
7. 保存为 HuggingFace 兼容格式

核心转换工作在\`slice\_paligemma\_state\_dict\`和\`slice\_gemma\_state\_dict\`函数中完成。这些函数执行精细的参数映射，处理各种Transformer组件（注意力层、MLP、层归一化等）的权重和偏置。每个函数都需要处理大量的张量重塑、转置和重组操作，以保持模型架构的语义等价性。例如，注意力层的查询、键和值投影矩阵需要特别注意，因为JAX和PyTorch的张量排列约定不同

##### 1.1.1.1 slice\_initial\_orbax\_checkpoint

脚本首先通过Orbax检查点管理器从OCDBT（Orbax CheckPoint Directory-Based Tree）格式加载原始模型参数。它使用\`slice\_initial\_orbax\_checkpoint\`函数将嵌套的参数树结构扁平化，并分离出PaliGemma参数和投影参数

##### 1.1.1.2 slice\_paligemma\_state\_dict

\`slice\_paligemma\_state\_dict\`函数处理视觉编码器（基于SigLIP）、多模态投影器和语言模型（Gemma）的前半部分，同时将专家模型的参数分离出来

1. 函数首先处理参数命名约定的变体，通过检查状态字典中是否存在\`"/value"\`后缀来确定参数存储格式
	```python
	def slice_paligemma_state_dict(state_dict, config):  # 定义函数，用于将JAX格式的PaliGemma参数转换为PyTorch格式
	    suffix = "/value" if "img/embedding/kernel/value" in state_dict else ""  # 确定参数键值的后缀，根据参数存储格式不同而变化
	 
	    # fmt: off  # 关闭代码格式化，保持原格式
	    # patch embeddings  # 处理图像补丁嵌入层参数
	    state_dict["paligemma.vision_tower.vision_model.embeddings.patch_embedding.weight"] = state_dict.pop(f"img/embedding/kernel{suffix}").transpose(   # 提取并转换补丁嵌入权重，调整维度顺序
	        3, 2, 0, 1
	    )
	    state_dict["paligemma.vision_tower.vision_model.embeddings.patch_embedding.bias"] = state_dict.pop(f"img/embedding/bias{suffix}")                  # 提取补丁嵌入偏置
	    # 处理位置嵌入参数
	    state_dict["paligemma.vision_tower.vision_model.embeddings.position_embedding.weight"] = state_dict.pop(f"img/pos_embedding{suffix}").reshape(    # 提取位置嵌入权重并重塑形状
	        -1, config.vision_config.hidden_size
	    )
	```
2. 随后进行三个主要阶段的处理  
	第一阶段处理视觉编码器部分。它首先转换补丁嵌入（patch embeddings）和位置嵌入（positional embeddings），调整张量形状和维度顺序以匹配PyTorch模型的期望格式
	```cobol
	# 提取视觉层参数，基础模型中有27层
	# 提取第一个层归一化的缩放参数
	encoderblock_layernorm0_scale = state_dict.pop(f"img/Transformer/encoderblock/LayerNorm_0/scale{suffix}") 
	# 提取第一个层归一化的偏置参数
	encoderblock_layernorm0_bias = state_dict.pop(f"img/Transformer/encoderblock/LayerNorm_0/bias{suffix}")  
	# 提取第二个层归一化的缩放参数
	encoderblock_layernorm1_scale = state_dict.pop(f"img/Transformer/encoderblock/LayerNorm_1/scale{suffix}")  
	# 提取第二个层归一化的偏置参数
	encoderblock_layernorm1_bias = state_dict.pop(f"img/Transformer/encoderblock/LayerNorm_1/bias{suffix}")  
	# 提取MLP第一个全连接层的权重
	encoderblock_mlp_dense0_kernel= state_dict.pop(f"img/Transformer/encoderblock/MlpBlock_0/Dense_0/kernel{suffix}")  
	# 提取MLP第一个全连接层的偏置
	encoderblock_mlp_dense0_bias= state_dict.pop(f"img/Transformer/encoderblock/MlpBlock_0/Dense_0/bias{suffix}")  
	# 提取MLP第二个全连接层的权重
	encoderblock_mlp_dense1_kernel= state_dict.pop(f"img/Transformer/encoderblock/MlpBlock_0/Dense_1/kernel{suffix}")
	# 提取MLP第二个全连接层的偏置  
	encoderblock_mlp_dense1_bias= state_dict.pop(f"img/Transformer/encoderblock/MlpBlock_0/Dense_1/bias{suffix}")  
	# 提取注意力机制中键投影的权重
	encoderblock_attention_0_key_kernel = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/key/kernel{suffix}")  
	# 提取注意力机制中键投影的偏置
	encoderblock_attention_0_key_bias = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/key/bias{suffix}")  
	# 提取注意力机制中值投影的权重
	encoderblock_attention_0_value_kernel = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/value/kernel{suffix}")  
	 # 提取注意力机制中值投影的偏置
	encoderblock_attention_0_value_bias = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/value/bias{suffix}") 
	# 提取注意力机制中查询投影的权重
	encoderblock_attention_0_query_kernel = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/query/kernel{suffix}")  
	# 提取注意力机制中查询投影的偏置
	encoderblock_attention_0_query_bias = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/query/bias{suffix}")  
	# 提取注意力机制中输出投影的权重
	encoderblock_attention_0_out_kernel = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/out/kernel{suffix}")  
	# 提取注意力机制中输出投影的偏置
	encoderblock_attention_0_out_bias = state_dict.pop(f"img/Transformer/encoderblock/MultiHeadDotProductAttention_0/out/bias{suffix}")
	```
	然后，函数提取全部27层视觉Transformer的参数，包括层归一化（layernorm）、多层感知机（MLP）和多头注意力机制（attention）的权重和偏置。对于每个注意力子层，它都需要进行精确的形状转换和转置操作，确保查询（query）、键（key）、值（value）和输出投影（output projection）矩阵都被正确映射
	```cobol
	# 遍历所有视觉层（共27层）
	for i in range(config.vision_config.num_hidden_layers):  
	    # 设置第i层的第一个层归一化权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.layer_norm1.weight"] = encoderblock_layernorm0_scale[i].transpose()  
	    # 设置第i层的第一个层归一化偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.layer_norm1.bias"] = encoderblock_layernorm0_bias[i]  
	    # 设置第i层的第二个层归一化权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.layer_norm2.weight"] = encoderblock_layernorm1_scale[i].transpose()  
	    # 设置第i层的第二个层归一化偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.layer_norm2.bias"] = encoderblock_layernorm1_bias[i]  
	    # 设置第i层MLP的第一个全连接层权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.mlp.fc1.weight"] = encoderblock_mlp_dense0_kernel[i].transpose()  
	    # 设置第i层MLP的第一个全连接层偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.mlp.fc1.bias"] = encoderblock_mlp_dense0_bias[i]  
	    # 设置第i层MLP的第二个全连接层权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.mlp.fc2.weight"] = encoderblock_mlp_dense1_kernel[i].transpose()  
	    # 设置第i层MLP的第二个全连接层偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.mlp.fc2.bias"] = encoderblock_mlp_dense1_bias[i]  
	     # 设置第i层注意力的键投影权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.k_proj.weight"] = encoderblock_attention_0_key_kernel[i].reshape(-1, config.vision_config.hidden_size).transpose() 
	    # 设置第i层注意力的键投影偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.k_proj.bias"] = encoderblock_attention_0_key_bias[i].reshape(-1, config.vision_config.hidden_size).reshape(-1)  
	     # 设置第i层注意力的值投影权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.v_proj.weight"] = encoderblock_attention_0_value_kernel[i].reshape(-1, config.vision_config.hidden_size).transpose() 
	     # 设置第i层注意力的值投影偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.v_proj.bias"] = encoderblock_attention_0_value_bias[i].reshape(-1, config.vision_config.hidden_size).reshape(-1) 
	    # 设置第i层注意力的查询投影权重
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.q_proj.weight"] = encoderblock_attention_0_query_kernel[i].reshape(-1, config.vision_config.hidden_size).transpose()  
	    # 设置第i层注意力的查询投影偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.q_proj.bias"] = encoderblock_attention_0_query_bias[i].reshape(-1, config.vision_config.hidden_size).reshape(-1)
	    # 设置第i层注意力的输出投影权重  
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.out_proj.weight"] = encoderblock_attention_0_out_kernel[i].reshape(-1, config.vision_config.hidden_size).transpose()  
	    # 设置第i层注意力的输出投影偏置
	    state_dict[f"paligemma.vision_tower.vision_model.encoder.layers.{i}.self_attn.out_proj.bias"] = encoderblock_attention_0_out_bias[i].reshape(-1, config.vision_config.hidden_size).reshape(-1)  
	# 设置视觉模型最终层归一化的权重
	state_dict["paligemma.vision_tower.vision_model.post_layernorm.weight"] = state_dict.pop(f"img/Transformer/encoder_norm/scale{suffix}").transpose()  
	# 设置视觉模型最终层归一化的偏置
	state_dict["paligemma.vision_tower.vision_model.post_layernorm.bias"] = state_dict.pop(f"img/Transformer/encoder_norm/bias{suffix}")
	```
3. 第二阶段处理多模态投影器和词嵌入，这是连接视觉和语言模型的关键桥梁。投影器参数需要转置以适应框架间的张量排列差异
	```python
	# multimodal projector  # 处理多模态投影器参数
	# 设置多模态投影器线性层的权重
	state_dict['paligemma.multi_modal_projector.linear.weight'] = state_dict.pop(f"img/head/kernel{suffix}").transpose()  
	# 设置多模态投影器线性层的偏置
	state_dict['paligemma.multi_modal_projector.linear.bias'] = state_dict.pop(f"img/head/bias{suffix}")
	```
4. 第三阶段转换语言模型（Gemma）部分，处理18层Transformer结构。这一部分特别复杂，因为JAX中的einsum表示和PyTorch的线性层表示有很大不同。代码通过复杂的转置和重塑操作将注意力计算的矩阵调整为正确的形状和排列
	```python
	# 处理文本解码器(Gemma)部分
	# 提取词嵌入向量
	embedding_vector = state_dict.pop(f"llm/embedder/input_embedding{suffix}")  
	# 设置语言模型词嵌入层的权重
	state_dict["paligemma.language_model.model.embed_tokens.weight"] = embedding_vector  
	# 提取einsum注意力和MLP表示，Gemma-2B中有18层
	# 提取注意力向量einsum参数
	llm_attention_attn_vec_einsum = state_dict.pop(f"llm/layers/attn/attn_vec_einsum/w{suffix}")  
	# 提取键值einsum参数
	llm_attention_kv_einsum = state_dict.pop(f"llm/layers/attn/kv_einsum/w{suffix}")  
	# 提取查询einsum参数
	llm_attention_q_einsum = state_dict.pop(f"llm/layers/attn/q_einsum/w{suffix}")  
	# 提取MLP门控einsum参数
	llm_mlp_gating_einsum = state_dict.pop(f"llm/layers/mlp/gating_einsum{suffix}")  
	# 提取MLP线性层参数
	llm_mlp_linear = state_dict.pop(f"llm/layers/mlp/linear{suffix}")  
	# TODO verify correctness of layer norm loading  # 待办：验证层归一化加载的正确性
	# 提取注意力前的层归一化参数
	llm_input_layernorm = state_dict.pop(f"llm/layers/pre_attention_norm/scale{suffix}")  
	# 提取前馈网络前的层归一化参数
	llm_post_attention_layernorm = state_dict.pop(f"llm/layers/pre_ffw_norm/scale{suffix}")
	```
	特别是对查询投影的处理需要进行三次转置和一次重塑，将(8, 2048, 256)的原始形状转换为PyTorch模型中期望的(2048, 2048)形状
	```cobol
	# 遍历文本模型的所有层（共18层）
	for i in range(config.text_config.num_hidden_layers):  
	    # 查询einsum参数形状为(8, 2048, 256)
	    # 重塑查询投影权重
	    q_proj_weight_reshaped = llm_attention_q_einsum[i].transpose(0, 2, 1).reshape(config.text_config.num_attention_heads * config.text_config.head_dim, config.text_config.hidden_size)  
	    # 设置第i层查询投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.self_attn.q_proj.weight"] = q_proj_weight_reshaped  
	    # 重塑键投影权重
	    k_proj_weight_reshaped = llm_attention_kv_einsum[i, 0, 0].transpose()  
	    # 设置第i层键投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.self_attn.k_proj.weight"] = k_proj_weight_reshaped  
	    # 重塑值投影权重
	    v_proj_weight_reshaped = llm_attention_kv_einsum[i, 1, 0].transpose()
	    # 设置第i层值投影权重  
	    state_dict[f"paligemma.language_model.model.layers.{i}.self_attn.v_proj.weight"] = v_proj_weight_reshaped  
	    # 输出投影处理
	    # 重塑输出投影权重
	    o_proj_weight_reshaped = llm_attention_attn_vec_einsum[i].transpose(2, 0, 1).reshape(config.text_config.num_attention_heads * config.text_config.head_dim, config.text_config.hidden_size)  
	    # 设置第i层输出投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.self_attn.o_proj.weight"] = o_proj_weight_reshaped  
	    # mlp layers  # 处理MLP层参数
	    # 获取门控投影权重
	    gate_proj_weight = llm_mlp_gating_einsum[i, 0]  
	    # 设置第i层MLP门控投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.mlp.gate_proj.weight"] = gate_proj_weight.transpose()  
	    # 获取上投影权重
	    up_proj_weight = llm_mlp_gating_einsum[i, 1]  
	    # 设置第i层MLP上投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.mlp.up_proj.weight"] = up_proj_weight.transpose()  
	    # 设置第i层MLP下投影权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.mlp.down_proj.weight"] = llm_mlp_linear[i].transpose()  
	    # 设置第i层输入层归一化权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.input_layernorm.weight"] = llm_input_layernorm[i]  
	    # 设置第i层注意力后层归一化权重
	    state_dict[f"paligemma.language_model.model.layers.{i}.post_attention_layernorm.weight"] = llm_post_attention_layernorm[i]  
	# 设置语言模型最终归一化层权重
	state_dict["paligemma.language_model.model.norm.weight"] = state_dict.pop(f"llm/final_norm/scale{suffix}")  
	 # 设置语言模型输出头权重（与词嵌入共享权重）
	state_dict["paligemma.language_model.lm_head.weight"] = embedding_vector # weights are tied.
	```
	MLP中的门控投影（gate\_proj）、上投影（up\_proj）和下投影（down\_proj）权重也需要类似的处理
5. 最后，函数将参数分为主模型参数和专家模型参数，返回两个分离的状态字典。这种分离允许后续代码分别处理PaliGemma主体和Gemma专家组件，支持PI0模型的混合架构设计
	```python
	# 恢复代码格式化
	# 初始化专家模型参数字典
	expert_dict = {}  
	# 初始化最终状态字典
	final_state_dict = {}  
	# 遍历状态字典中的所有键值对
	for key, value in state_dict.items():  
	    # 如果键不在以下列表中（不是专家模型参数）
	    if key not in [  
	        f"llm/final_norm_1/scale{suffix}",
	        f"llm/layers/attn/attn_vec_einsum_1/w{suffix}",
	        f"llm/layers/attn/kv_einsum_1/w{suffix}",
	        f"llm/layers/attn/q_einsum_1/w{suffix}",
	        f"llm/layers/mlp_1/gating_einsum{suffix}",
	        f"llm/layers/mlp_1/linear{suffix}",
	        f"llm/layers/pre_attention_norm_1/scale{suffix}",
	        f"llm/layers/pre_ffw_norm_1/scale{suffix}",
	    ]:
	        # 将值转换为PyTorch张量并添加到最终状态字典
	        final_state_dict[key] = torch.from_numpy(value)  
	    else:
	        # 将专家模型参数添加到专家字典
	        expert_dict[key] = value  
	# 返回最终状态字典和专家字典
	return final_state_dict, expert_dict
	```

##### 1.1.1.3 slice\_gemma\_state\_dict

而\`slice\_gemma\_state\_dict\`函数专门处理Gemma专家模型部分。对于27层视觉编码器和18层语言模型，脚本中的循环分别为每层精确地重映射参数

##### 1.1.1.4 convert\_pi0\_checkpoint

最后，\`convert\_pi0\_checkpoint\`函数整合了所有过程：加载参数、处理投影层权重、处理PaliGemma和Gemma权重、创建适当的模型配置、实例化PI0Policy模型、加载状态字典、转换为指定精度，并保存模型使其与Hugging Face的\`from\_pretrained\`方法兼容

脚本根据检查点路径自动检测是基础模型还是特定于Aloha机器人的变体，并相应地调整配置参数。此外，它支持不同的精度格式（float32、bfloat16、float16），以适应各种硬件和部署场景

#### 1.1.2 conversion\_utils.py：为转换提供关键的配置函数

这是一个辅助工具模块，为转换过程提供了关键的配置函数。具体功能包括：

1. \`get\_paligemma\_config()\`: 创建标准的 PaliGemma 配置对象，设置了图像尺寸、补丁大小以及各种模型参数，如隐藏层大小、注意力头数量等，确保 PyTorch 版本的配置与原始 JAX 模型匹配
2. \`get\_gemma\_config()\`: 创建 Gemma 动作专家模型的配置对象，指定了隐藏层大小(1024)、层数(18)、注意力头数量(8)等参数

具体而言

1. \`get\_paligemma\_config\`函数创建了PaliGemma多模态模型的完整配置，它同时包含视觉和文本处理能力  
	函数首先设置基本的标记配置（如填充标记、开始标记和结束标记的ID），然后定义视觉处理相关参数
	```python
	# 定义函数获取PaliGemma配置，参数precision指定模型精度
	def get_paligemma_config(precision: str):  
	 
	     # 初始化基本配置字典
	    config = { 
	        "image_token_index": None,  # 图像标记索引，初始设为None
	        "pad_token_id": 0,          # 填充标记ID为0
	        "bos_token_id": 2,          # 序列开始标记ID为2
	        "eos_token_id": 1,          # 序列结束标记ID为1
	    }
	```
	视觉部分使用224×224像素的图像输入和14×14像素的补丁大小，产生256个图像标记
	```cobol
	image_size = 224      # 设置图像大小为224像素（边长）
	patch_size = 14       # 设置图像patch大小为14像素（边长）
	# 计算图像patch数量：总像素除以每个patch的像素
	num_image_tokens = (image_size**2) // (patch_size**2)
	```
	函数为文本处理部分配置了一个18层的Transformer架构，每层有8个注意力头但只有1个键值头（表示使用了分组查询注意力机制，这是Gemma模型的特点），隐藏层维度为2048
	```python
	# 设置图像token索引值
	config["image_token_index"] = 257152      
	text_config = {                       # 定义文本处理部分（语言模型）的配置
	    "vocab_size": 257152,             # 词汇表大小
	    "num_hidden_layers": 18,          # 隐藏层数量
	    "num_key_value_heads": 1,         # 键值头数量（用于分组查询注意力）
	    "head_dim": 256,                  # 每个注意力头的维度
	    "torch_dtype": precision,         # 使用传入的精度参数
	    "hidden_size": 2048,              # 隐藏层大小
	    "hidden_activation": "gelu_pytorch_tanh",      # 隐藏层激活函数
	    "num_attention_heads": 8,                      # 注意力头数量
	    "intermediate_size": 16384,                    # 前馈网络中间层大小
	    "is_encoder_decoder": False,                   # 不是编码器-解码器架构
	}
	```
	视觉编码器被配置为27层，具有16个注意力头，隐藏层维度为1152
	```python
	# 定义视觉处理部分的配置
	vision_config = {      
	    "torch_dtype": precision,      # 使用传入的精度参数
	    "image_size": image_size,      # 图像大小
	    "patch_size": patch_size,      # patch大小
	    "num_image_tokens": num_image_tokens,  # 图像token数量
	    "hidden_size": 1152,                   # 视觉模型隐藏层大小
	    "intermediate_size": 4304,             # 视觉模型中间层大小
	    "num_hidden_layers": 27,               # 视觉模型隐藏层数量
	    "num_attention_heads": 16,             # 视觉模型注意力头数量
	    "projector_hidden_act": "gelu_fast",   # 投影器隐藏层激活函数
	    "vision_use_head": False,              # 不使用视觉头
	}
	```
	这些精心选择的参数确保了模型能够有效处理图像信息并与文本进行融合
	```cobol
	# 创建最终PaliGemma配置对象
	final_config = PaliGemmaConfig(text_config=text_config, vision_config=vision_config, **config)  
	return final_config          # 返回配置对象
	```
2. 相比之下，\`get\_gemma\_config\`函数创建了Gemma专家模型的配置，它共享许多与PaliGemma文本部分相同的结构特征，但隐藏层大小减半至1024，中间层大小也从16384减少到4096
	```python
	# 定义函数获取Gemma配置，参数precision指定模型精度
	def get_gemma_config(precision: str):  
	    # 初始化基本配置字典
	    config = { 
	        "image_token_index": None,      # 图像标记索引，初始设为None
	        "pad_token_id": 0,              # 填充标记ID为0 
	        "bos_token_id": 2,              # 序列开始标记ID为2
	        "eos_token_id": 1,              # 序列结束标记ID为1
	    }
	 
	    # 设置图像标记索引值
	    config["image_token_index"] = 257152  
	 
	     # 定义文本处理模型的配置
	    text_config = { 
	        "vocab_size": 257152,          # 词汇表大小
	        "num_hidden_layers": 18,       # 隐藏层数量
	        "num_key_value_heads": 1,      # 键值头数量（用于分组查询注意力）
	        "head_dim": 256,               # 每个注意力头的维度
	        "torch_dtype": precision,      # 使用传入的精度参数
	        "hidden_size": 1024,           # 隐藏层大小（注意比PaliGemma的文本部分小一半）
	        "hidden_activation": "gelu_pytorch_tanh",      # 隐藏层激活函数
	        "num_attention_heads": 8,                      # 注意力头数量
	        "intermediate_size": 4096,                     # 前馈网络中间层大小（比PaliGemma小很多）
	        "is_encoder_decoder": False,                   # 不是编码器-解码器架构
	    }
	```
	这种设计使Gemma专家模型更加轻量，同时保持足够的表达能力来补充PaliGemma的处理能力
	```cobol
	final_config = GemmaConfig()          # 创建空的Gemma配置对象
	final_config.update(text_config)      # 使用text_config更新配置对象
	return final_config                   # 返回配置对象
	```

两个配置函数都接受精度参数（如float32、bfloat16或float16），使模型能够适应不同的硬件和内存需求

### 1.2 配置configuration\_pi0.py：PI0配置类class PI0Config

\`PI0Config\`类是LeRobot项目中π0 策略模型的核心配置组件。作为一个使用Python的\`dataclass\`装饰器实现的配置类，它提供了一套全面的参数集，用于定义模型的输入/输出结构、预处理步骤、微调选项以及训练设置

这个类通过\`@PreTrainedConfig.register\_subclass("pi0")\`装饰器注册为可序列化的预训练配置，使其能与LeRobot的模型加载和保存机制无缝集成

#### 1.2.1 三个主要参数组：输入/输出、归一化、图像预处理

配置类定义了三个主要参数组

1. 首先是输入/输出结构参数，包括观察步数(\`n\_obs\_steps\`)、处理块大小(\`chunk\_size\`)和动作步数(\`n\_action\_steps\`)
	```cobol
	# 定义PI0配置类，继承自PreTrainedConfig
	class PI0Config(PreTrainedConfig): 
	    # Input / output structure.       # 输入/输出结构配置
	    n_obs_steps: int = 1              # 观察步数，默认为1步
	    chunk_size: int = 50              # 处理块的大小，默认为50
	    n_action_steps: int = 50          # 动作步数，默认为50
	```
2. 它还指定了不同输入类型的归一化方式，视觉输入使用恒等映射，而状态和动作数据则进行均值-标准差归一化
	```python
	# 定义归一化映射字典
	normalization_mapping: dict[str, NormalizationMode] = field(  
	    # 使用lambda函数作为默认值工厂
	    default_factory=lambda: {  
	        "VISUAL": NormalizationMode.IDENTITY,      # 视觉数据使用恒等映射（不归一化）
	        "STATE": NormalizationMode.MEAN_STD,       # 状态数据使用均值-标准差归一化
	        "ACTION": NormalizationMode.MEAN_STD,      # 动作数据使用均值-标准差归一化
	    }
	)
	```
3. 图像预处理部分配置将所有输入图像调整为224×224像素大小，并支持添加空摄像机视图，这在Aloha仿真环境中用于补充顶部摄像头的视角
	```cobol
	# 图像预处理配置
	resize_imgs_with_padding: tuple[int, int] = (224, 224)  # 调整图像大小并填充至224x224像素
	# 添加空白图像
	# 用于pi0_aloha_sim，它添加了除顶部相机外的左右手腕空白相机
	empty_cameras: int = 0  # 空白相机数量，默认为0
	```

#### 1.2.2 机器人控制的参数：空间转换等

此配置还包含了特定于机器人控制的参数

- \`adapt\_to\_pi\_aloha\`参数启用从标准Aloha空间到PI内部运行时使用的空间的转换
- 而\`use\_delta\_joint\_actions\_aloha\`则控制是否使用相对于当前状态的关节差值，这对于精确的机器人控制至关重要
```python
# 将关节和夹持器值从标准Aloha空间转换为

 # pi内部运行时使用的空间，该空间用于训练基础模型

 adapt_to_pi_aloha: bool = False   # 是否适应PI Aloha格式，默认为False

 # 在传递给模型之前，将关节维度转换为相对于当前状态的增量

 # 夹持器维度将保持绝对值，# 是否使用Aloha的关节动作增量，默认为False

 use_delta_joint_actions_aloha: bool = False  

 # 分词器配置

 tokenizer_max_length: int = 48    # 分词器最大长度，默认为48

 # 投影器配置

 proj_width: int = 1024            # 投影宽度，默认为1024

# 解码配置

 num_steps: int = 10               # 解码步数，默认为10
```

#### 1.2.3 模型的注意力机制、微调和训练设置

模型的注意力机制、微调和训练设置也有详细配置

\`attention\_implementation\`参数支持多种注意力计算实现（"eager"、"fa2"或"flex"），而\`freeze\_vision\_encoder\`和\`train\_expert\_only\`参数允许选择性地冻结模型组件以进行高效的微调

```cobol
# 注意力机制工具配置

# 是否使用缓存，默认为True

use_cache: bool = True  

# 注意力实现方式，默认为"eager"，也可以是"fa2"或"flex"

attention_implementation: str = "eager"  

# 微调设置

freeze_vision_encoder: bool = True       # 是否冻结视觉编码器，默认为True

train_expert_only: bool = False          # 是否仅训练专家部分，默认为False

train_state_proj: bool = True            # 是否训练状态投影，默认为True
```

训练优化器和学习率调度器的默认设置基于AdamW优化器，并使用余弦衰减与预热的学习率调度策略，这是现代大型预训练模型的常见选择

此外，该类的\`\_\_post\_init\_\_\`方法执行重要的输入验证，确保配置的一致性，例如检查动作步数不超过处理块大小，并验证当前只支持单个观察步骤。它还通过显式的\`NotImplementedError\`标记

### 1.3 paligemma\_with\_expert.py：将PaliGemma与Gemma集成在一起

paligemma\_with\_expert.py是PI0架构的核心模型类，它巧妙地将PaliGemma视觉-语言模型与Gemma专家语言模型集成在一起，形成了一个强大的多模态推理系统。该类继承自Hugging Face的\`PreTrainedModel\`，使其能够与Transformers生态系统无缝集成

#### 1.3.1 对旋转位置编码RoPE的简单实现

这个文件首先定义了一个\`apply\_rope\`函数，用于应用旋转位置编码RoPE到输入张量，这是一种在注意力计算中直接编码位置信息的技术  
与传统的绝对位置编码不同，RoPE通过在复数域中进行旋转变换，在保持向量内积不变的同时编码相对位置信息  
*原理讲解详见此文《 *[一文通透位置编码：从标准位置编码、旋转位置编码RoPE到ALiBi、LLaMA 2 Long(含NTK-aware简介)](https://blog.csdn.net/v_JULY_v/article/details/134085503 "一文通透位置编码：从标准位置编码、旋转位置编码RoPE到ALiBi、LLaMA 2 Long(含NTK-aware简介)")* 》*

1. 该函数首先计算输入张量\`x\`最后维度的一半(\`d\_half\`)，因为RoPE基于二维旋转，对嵌入向量的每对元素进行操作
	```python
	# 定义旋转位置编码(RoPE)函数，接收输入张量、位置张量和最大波长参数
	def apply_rope(x, positions, max_wavelength=10_000):  
	    """
	    Applies RoPE positions [B, L] to x [B, L, H, D].
	    """  
	    # 将RoPE位置编码应用于输入张量，B是批次大小，L是序列长度，H是头数，D是头维度
	 
	    # 计算头维度的一半，因为RoPE处理时会将每个向量分成两半
	2
	```
2. 然后，它获取设备和数据类型信息，并将输入转换为float32以确保计算精度
	```cobol
	dtype = x.dtype              # 获取输入张量的数据类型
	x = x.to(torch.float32)      # 将输入张量转换为float32类型以确保计算精度
	```
3. 接下来，函数计算频率指数\`freq\_exponents\`，它是通过将\`2.0/D\`（其中D是嵌入维度）乘以一个从0到\`d\_half-1\`的序列得到的。这些指数用于创建时间尺度\`timescale\`，形成一个几何级数\`max\_wavelength\*\*freq\_exponents\`
	```cobol
	# 计算频率指数，不同维度使用不同频率的旋转
	freq_exponents = (2.0 / x.shape[-1]) * torch.arange(d_half, dtype=torch.float32, device=device)              
	# 计算时间尺度，形成几何级数，低维度旋转慢，高维度旋转快
	timescale = max_wavelength**freq_exponents   
	# 计算旋转弧度，位置值除以相应的时间尺度
	radians = positions[..., None].to(torch.float32) / timescale[None, None, :].to(torch.float32)  
	# 扩展弧度张量维度以便于后续计算
	radians = radians[..., None, :]
	```
	核心计算步骤是通过将位置值除以相应的时间尺度来获得弧度值\`radians\`。这种方式使得 **不同维度的嵌入以不同的频率旋转，低维度旋转缓慢，高维度旋转迅速，从而在不同尺度上捕获位置信息**
4. 然后，函数计算这些弧度的正弦和余弦值
	```perl
	sin = torch.sin(radians)      # 计算弧度的正弦值
	cos = torch.cos(radians)      # 计算弧度的余弦值
	```
5. 最后，函数将嵌入向量沿最后一个维度分为两半，并分别应用旋转变换：
	```cobol
	# 将输入张量沿最后一个维度分成两半
	x1, x2 = x.split(d_half, dim=-1)  
	# 创建与输入张量相同形状的空张量来存储结果
	res = torch.empty_like(x)  
	# 应用旋转变换的第一部分：前半部分 = x1*cos - x2*sin
	res[..., :d_half] = x1 * cos - x2 * sin  
	# 应用旋转变换的第二部分：后半部分 = x2*cos + x1*sin
	res[..., d_half:] = x2 * cos + x1 * sin
	```
	\- 前半部分：\`x1 \* cos - x2 \* sin\`  
	\- 后半部分：\`x2 \* cos + x1 \* sin\`  
	*这个过程实际上是在二维空间中对向量对执行旋转，旋转角度与位置成正比。这种方法的巧妙之处在于，它使得注意力机制能够自然地感知相对位置（即两个token之间的距离），而不仅仅是绝对位置，这对模型理解序列中的长距离依赖关系和结构关系至关重要*

然后定义了两个主要类：\`PaliGemmaWithExpertConfig\`和\`PaliGemmaWithExpertModel\`，接下来，分别介绍这两个类的实现

#### 1.3.2 PaliGemmaWithExpertConfig：管理和配置PaliGemmaWithExpertModel

\`PaliGemmaWithExpertConfig\`类是为\`PaliGemmaWithExpertModel\`定义配置的类，它继承自Hugging Face的\`PretrainedConfig\`

该类的作用是管理和配置一个复合模型，该模型由PaliGemma（一个视觉-语言模型）和Gemma专家模型组合而成

这个配置类声明了\`model\_type\`为"PaliGemmaWithExpertModel"，并通过\`sub\_configs\`字典定义了两个子配置类型：

1. paligemma\_config
2. gemma\_expert\_config

它们都使用\`AutoConfig\`作为基类。这种结构使模型能够独立配置两个组件，同时保持它们在一个统一的框架内

```coffeescript
# 定义PaliGemma与专家模型的组合配置类，继承自预训练配置基类

class PaliGemmaWithExpertConfig(PretrainedConfig):  

    # 设置模型类型标识符

    model_type = "PaliGemmaWithExpertModel"  

 

    # 定义子配置映射，指定使用AutoConfig处理两个子模型

    sub_configs = {"paligemma_config": AutoConfig, "gemma_expert_config": AutoConfig}
```

**构造函数接受多个参数** ，其中三个关键控制参数决定了模型的行为方式：

1. \`freeze\_vision\_encoder\`，默认为True，控制是否冻结视觉编码器参数
2. \`train\_expert\_only\`，默认为True，决定是否只训练专家模型部分
3. \`attention\_implementation\`，默认为"eager"，指定使用哪种注意力机制实现（可选值为"eager"、"fa2"或"flex"）
	```python
	def __init__(
	    self,
	    paligemma_config: dict | None = None,      # PaliGemma模型的配置字典，可选
	    gemma_expert_config: dict | None = None,   # Gemma专家模型的配置字典，可选
	    freeze_vision_encoder: bool = True,        # 是否冻结视觉编码器，默认为True
	    train_expert_only: bool = True,            # 是否仅训练专家模型部分，默认为True
	    attention_implementation: str = "eager",   # 注意力机制的实现方式，默认为"eager"
	    **kwargs,                                  # 额外的关键字参数
	):
	    # 保存是否冻结视觉编码器的设置
	    self.freeze_vision_encoder = freeze_vision_encoder  
	    # 保存是否仅训练专家模型的设置
	    self.train_expert_only = train_expert_only  
	    # 保存注意力实现方式的设置
	    self.attention_implementation = attention_implementation
	```

此外，对于该构造函数

- 如果没有提供\`paligemma\_config\`，构造函数会创建一个默认配置，这个配置指定了PaliGemma模型的详细参数，包括  
	词汇表大小（257152）、隐藏层维度（2048）
	```cobol
	if paligemma_config is None:       # 如果没有提供PaliGemma配置
	    # Default config from Pi0      # 使用PI0的默认配置
	    # 从映射中获取PaliGemma配置类并实例化
	    self.paligemma_config = CONFIG_MAPPING["paligemma"](  
	        transformers_version="4.48.1",      # Transformers库版本
	        _vocab_size=257152,                 # 词汇表大小
	        bos_token_id=2,                     # 开始标记ID
	        eos_token_id=1,                     # 结束标记ID
	        hidden_size=2048,                   # 隐藏层大小
	        image_token_index=257152,           # 图像标记索引
	        model_type="paligemma",             # 模型类型
	        pad_token_id=0,                     # 填充标记ID
	        projection_dim=2048,                # 投影维度
	```
	文本配置（如注意力头数量、隐藏层数）
	```csharp
	# 文本配置
	text_config={  
	    # 隐藏层激活函数
	    "hidden_activation": "gelu_pytorch_tanh",  
	    "hidden_size": 2048,          # 隐藏层大小
	    "intermediate_size": 16384,   # 中间层大小
	    "model_type": "gemma",        # 文本模型类型为gemma
	    "num_attention_heads": 8,     # 注意力头数量
	    "num_hidden_layers": 18,      # 隐藏层数量
	    "num_image_tokens": 256,      # 图像token数量
	    "num_key_value_heads": 1,     # 键值头数量(分组注意力)
	    "torch_dtype": "float32",     # PyTorch数据类型
	    "vocab_size": 257152,         # 词汇表大小
	},
	```
	和视觉配置（如SigLIP视觉模型的参数）
	```python
	# 视觉配置
	vision_config={  
	    "hidden_size": 1152,              # 隐藏层大小
	    "intermediate_size": 4304,        # 中间层大小
	    "model_type": "siglip_vision_model",  # 视觉模型类型为SigLIP
	    "num_attention_heads": 16,        # 注意力头数量
	    "num_hidden_layers": 27,          # 隐藏层数量
	    "num_image_tokens": 256,          # 图像标记数量
	    "patch_size": 14,                 # 图像块大小
	    "projection_dim": 2048,           # 投影维度
	    "projector_hidden_act": "gelu_fast",  # 投影器隐藏层激活函数
	    "torch_dtype": "float32",          # PyTorch数据类型
	    "vision_use_head": False,          # 是否使用视觉头
	},
	```
- 同样，如果没有提供\`gemma\_expert\_config\`，也会创建一个默认的Gemma专家模型配置，配置中包含注意力头参数、隐藏层参数、激活函数选择等关键设置
	```cobol
	if gemma_expert_config is None:     # 如果没有提供Gemma专家配置
	    # Default config from Pi0       # 使用PI0的默认配置
	    self.gemma_expert_config = CONFIG_MAPPING["gemma"](  # 从映射中获取Gemma配置类并实例化
	        attention_bias=False,       # 是否使用注意力偏置
	        attention_dropout=0.0,      # 注意力dropout率
	        bos_token_id=2,             # 开始tokenID
	        eos_token_id=1,             # 结束token ID
	        head_dim=256,               # 注意力头维度
	        hidden_act="gelu_pytorch_tanh",          # 隐藏层激活函数
	        hidden_activation="gelu_pytorch_tanh",   # 隐藏层激活函数(冗余)
	        hidden_size=1024,           # 隐藏层大小
	        initializer_range=0.02,     # 初始化范围
	        intermediate_size=4096,     # 中间层大小
	        max_position_embeddings=8192,            # 最大位置嵌入数
	        model_type="gemma",                      # 模型类型
	        num_attention_heads=8,                   # 注意力头数量
	        num_hidden_layers=18,                    # 隐藏层数量
	        num_key_value_heads=1,                   # 键值头数量(分组注意力)
	        pad_token_id=0,              # 填充标记ID
	        rms_norm_eps=1e-06,          # RMS归一化的epsilon值
	        rope_theta=10000.0,          # RoPE位置编码的theta参数
	        torch_dtype="float32",               # PyTorch数据类型
	        transformers_version="4.48.1",       # Transformers库版本
	        use_cache=True,                      # 是否使用缓存
	        vocab_size=257152,                   # 词汇表大小
	    )
	```

**最后，在\`\_\_post\_init\_\_\`方法中** ，配置类执行两项重要的验证：

1. 首先检查\`train\_expert\_only\`和\`freeze\_vision\_encoder\`的设置是否兼容（如果只训练专家模型，则视觉编码器必须被冻结）
2. 其次验证\`attention\_implementation\`参数值是否有效。这些验证确保模型配置的一致性，防止训练过程中可能出现的问题

通过这种详细的配置机制，\`PaliGemmaWithExpertModel\`能够灵活地适应不同的训练和推理需求，同时保持设置的一致性和有效性

#### 1.3.3 PaliGemmaWithExpertModel：分别初始化VLM PaliGemma、Gemma 300M

\`PaliGemmaWithExpertModel\`是一个结合了PaliGemma视觉-语言模型和Gemma专家语言模型的架构

1. 在初始化阶段，模型实例化了PaliGemma和Gemma两个子模型\`PaliGemmaForConditionalGeneration\`处理视觉和初始语言理解  
	以及\`GemmaForCausalLM\`作为专家模型处理后续的推理和生成任务
	```ruby
	# 定义PaliGemma与专家模型的组合类，继承自PreTrainedModel
	class PaliGemmaWithExpertModel(PreTrainedModel):  
	    config_class = PaliGemmaWithExpertConfig  # 指定配置类
	 
	    # 初始化方法，接收配置参数
	    def __init__(self, config: PaliGemmaWithExpertConfig):  
	        super().__init__(config=config)  # 调用父类初始化方法
	        self.config = config  # 保存配置
	 
	        # 实例化PaliGemma模型
	        self.paligemma = PaliGemmaForConditionalGeneration(config=config.paligemma_config)  
	 
	        # 实例化Gemma专家模型
	        self.gemma_expert = GemmaForCausalLM(config=config.gemma_expert_config)
	```
	并移除了不需要的Gemma词嵌入层（因为输入嵌入已由PaliGemma处理）
	```rust
	# 移除未使用的词嵌入层，设置为None，因为输入嵌入已由PaliGemma处理
	self.gemma_expert.model.embed_tokens = None
	```
	通过\`to\_bfloat16\_like\_physical\_intelligence\`方法，模型将关键组件转换为bfloat16格式，提高计算效率并减少内存占用，同时与原始Physical Intelligence实现保持一致
	```cobol
	self.to_bfloat16_like_physical_intelligence()  # 将模型转换为bfloat16格式
	self.set_requires_grad()          # 设置各部分是否参与梯度更新
	```
2. 该模型实现了灵活的训练控制机制  
	\`set\_requires\_grad\`
	```python
	def set_requires_grad(self):  # 设置模型各部分是否需要梯度
	    if self.config.freeze_vision_encoder:      # 如果配置为冻结视觉编码器
	        self.paligemma.vision_tower.eval()     # 将视觉塔设置为评估模式
	        for params in self.paligemma.vision_tower.parameters():  # 遍历视觉塔的所有参数
	            params.requires_grad = False       # 设置不需要梯度
	    if self.config.train_expert_only:          # 如果配置为只训练专家模型
	        self.paligemma.eval()                  # 将整个PaliGemma设置为评估模式
	        for params in self.paligemma.parameters():      # 遍历PaliGemma的所有参数
	            params.requires_grad = False                # 设置不需要梯度
	```
	和重写的\`train\`方法
	```python
	def train(self, mode: bool = True):  # 重写train方法，控制训练模式
	    super().train(mode)              # 调用父类的train方法
	    if self.config.freeze_vision_encoder:       # 如果配置为冻结视觉编码器
	        self.paligemma.vision_tower.eval()      # 即使在训练模式下，也将视觉塔设为评估模式
	    if self.config.train_expert_only:           # 如果配置为只训练专家模型
	        self.paligemma.eval()                   # 即使在训练模式下，也将PaliGemma设为评估模式
	```
	确保即使在训练模式下，冻结的组件（如视觉编码器或整个PaliGemma模型）也保持在评估状态  
	这种设计使得用户可以根据任务需求和计算资源灵活地选择微调策略，比如仅训练Gemma专家部分而保持视觉-语言基础模型不变
3. 模型提供了两个关键的嵌入辅助方法：  
	\`embed\_image\`将图像转换为特征表示
	```ruby
	def embed_image(self, image: torch.Tensor):          # 图像嵌入方法
	    return self.paligemma.get_image_features(image)  # 使用PaliGemma获取图像特征
	```
	\`embed\_language\_tokens\`将语言token转换为嵌入表示
	```ruby
	# 语言token嵌入方法
	def embed_language_tokens(self, tokens: torch.Tensor):  
	    # 使用PaliGemma语言模型的嵌入层处理token
	    return self.paligemma.language_model.model.embed_tokens(tokens)
	```
	这些方法为下一节「 *3.4.3 PI0FlowMatching类的实现：嵌入处理、训练、推理(迭代去噪生成最终动作)* 」中的\`PI0FlowMatching\`类的\`embed\_prefix\`功能提供了底层支持
4. \`forward\`方法是一个精心设计的复杂函数，它实现了PaliGemma和Gemma Expert两个模型的联合前向计算过程。正如代码中的TODO注释所示，这确实是一个"巨大的前向函数"，但其复杂性是有必要的，因为它实现了两个独立模型在层级上的深度集成  
	  
	该函数首先准备两个模型列表\`models\`，并从输入嵌入中获取批次大小
	```python
	# 待办：将这个巨大的前向传播方法拆分为模块或函数
	def forward(
	    self,
	    attention_mask: Optional[torch.Tensor] = None,        # 注意力掩码
	    position_ids: Optional[torch.LongTensor] = None,      # 位置ID
	    # 过去的键值对缓存
	    past_key_values: Optional[Union[List[torch.FloatTensor], Cache]] = None,  
	    # 输入嵌入列表[前缀嵌入, 后缀嵌入]
	    inputs_embeds: List[torch.FloatTensor] = None,  
	    use_cache: Optional[bool] = None,              # 是否使用缓存
	    fill_kv_cache: Optional[bool] = None,          # 是否填充键值缓存
	):
	    # 定义模型列表，包含PaliGemma语言模型和Gemma专家模型
	    models = [self.paligemma.language_model.model, self.gemma_expert.model]
	```
	随后，它执行了一个关键的层循环，遍历PaliGemma文本配置中指定的层数。在每一层，函数对两个模型的输入应用相同的处理步骤：层归一化（input\_layernorm）、计算查询/键/值投影
	```cobol
	# RMSNorm  # RMS归一化处理
	num_layers = self.paligemma.config.text_config.num_hidden_layers  # 获取层数
	head_dim = self.paligemma.config.text_config.head_dim  # 获取注意力头维度
	for layer_idx in range(num_layers):  # 遍历每一层
	    query_states = []        # 初始化查询状态列表
	    key_states = []          # 初始化键状态列表
	    value_states = []        # 初始化值状态列表
	    # 遍历输入嵌入
	    for i, hidden_states in enumerate(inputs_embeds):  
	        if hidden_states is None:  # 如果隐藏状态为None
	            continue  # 继续下一次循环
	        # 获取当前模型的当前层
	        layer = models[i].layers[layer_idx]  
	        # 应用输入层归一化
	        hidden_states = layer.input_layernorm(hidden_states)  
	        # 获取输入形状（除去最后一维）
	        input_shape = hidden_states.shape[:-1]  
	        # 构建隐藏形状，适合多头注意力
	        hidden_shape = (*input_shape, -1, layer.self_attn.head_dim)  
	        # 转换为bfloat16类型
	        hidden_states = hidden_states.to(dtype=torch.bfloat16)  
	        query_state = layer.self_attn.q_proj(hidden_states).view(hidden_shape)      # 计算查询状态并重塑
	        key_state = layer.self_attn.k_proj(hidden_states).view(hidden_shape)      # 计算键状态并重塑
	        value_state = layer.self_attn.v_proj(hidden_states).view(hidden_shape)      # 计算值状态并重塑
	        query_states.append(query_state)          # 添加到查询状态列表
	        key_states.append(key_state)              # 添加到键状态列表
	        value_states.append(value_state)          # 添加到值状态列表
	    # B:批次大小，L:序列长度，H:头数，D:头维度
	    # concatenate on the number of embeddings/tokens  # 在嵌入/标记数量维度上连接
	    query_states = torch.cat(query_states, dim=1)  # 连接所有查询状态
	    key_states = torch.cat(key_states, dim=1)      # 连接所有键状态
	    value_states = torch.cat(value_states, dim=1)  # 连接所有值状态
	```
	然后连接并应用旋转位置编码（RoPE）
	```cobol
	query_states = apply_rope(query_states, position_ids)  # 应用RoPE位置编码到查询状态
	key_states = apply_rope(key_states, position_ids)      # 应用RoPE位置编码到键状态
	```
	代码中包含了高效的键值缓存机制，这对推理性能至关重要  
	当设置\`use\_cache=True\`时，函数会根据\`fill\_kv\_cache\`参数决定是填充新的缓存还是追加到现有缓存。这允许模型在自回归生成过程中重复使用之前计算的键值对，大大减少了计算量
	```cobol
	if use_cache and past_key_values is None:  # 如果使用缓存且过去的键值对为None
	    past_key_values = {}  # 初始化为空字典
	if use_cache:      # 如果使用缓存
	    if fill_kv_cache:          # 如果需要填充键值缓存
	        past_key_values[layer_idx] = {      # 存储当前层的键值对
	            "key_states": key_states,       # 存储键状态
	            "value_states": value_states,   # 存储值状态
	        }
	    else:          # 如果不填充缓存，则使用已有缓存
	        # # 待办：这里可以进行一些优化
	        # 连接过去和当前的键状态
	        key_states = torch.cat([past_key_values[layer_idx]["key_states"], key_states], dim=1)        
	        # 连接过去和当前的值状态
	        value_states = torch.cat(  
	            [past_key_values[layer_idx]["value_states"], value_states], dim=1
	        )
	```
	经过RoPE处理后通过选择的注意力实现（由\`get\_attention\_interface\`方法确定，以确定"eager"、"fa2"或"flex"）计算注意力输出
	```cobol
	attention_interface = self.get_attention_interface()  # 获取注意力接口
	att_output = attention_interface(      # 计算注意力输出
	    attention_mask, batch_size, head_dim, query_states, key_states, value_states
	)
	att_output = att_output.to(dtype=torch.bfloat16)      # 转换为bfloat16类型
	```
	---
	*插入解释一下这个get\_attention\_interface方法*
	```cobol
	def get_attention_interface(self):
	    if self.config.attention_implementation == "fa2":
	        // fa2对应flash_attention_forward
	        attention_interface = self.flash_attention_forward
	    elif self.config.attention_implementation == "flex":
	        // flex对应于pi0/paligemma_with_expert.py的开头的引入：from lerobot.common.policies.pi0.flex_attention import flex_attention_forward
	        attention_interface = flex_attention_forward
	    else:
	        // 对应下面马上要介绍的eager_attention_forward
	        attention_interface = self.eager_attention_forward
	    return attention_interface
	```
	*其中的flex\_attention\_forward下下文的3.5节，至于eager\_attention\_forward下面马上要介绍*
	---
	计算得到的注意力输出被分割并通过输出投影、残差连接和前馈网络（MLP）处理
	```cobol
	# att_output的第一部分是前缀（直到序列长度）
	outputs_embeds = []        # 初始化输出嵌入列表
	start = 0                  # 初始化起始索引
	for i, hidden_states in enumerate(inputs_embeds):  # 遍历输入嵌入
	    layer = models[i].layers[layer_idx]            # 获取当前模型的当前层
	    if hidden_states is not None:                  # 如果隐藏状态不为None
	        end = start + hidden_states.shape[1]       # 计算结束索引
	        # 如果数据类型不匹配
	        if att_output.dtype != layer.self_attn.o_proj.weight.dtype:  
	            # 转换数据类型
	            att_output = att_output.to(layer.self_attn.o_proj.weight.dtype)  
	        # 应用输出投影
	        out_emb = layer.self_attn.o_proj(att_output[:, start:end])  
	        # 待办：第一个dropout（默认为0.0）
	        # 第一个残差连接
	        out_emb += hidden_states          
	        # 克隆第一个残差后的结果   
	        after_first_residual = out_emb.clone()  
	        # 应用注意力后的层归一化
	        out_emb = layer.post_attention_layernorm(out_emb)  
	        # 应用多层感知机
	        out_emb = layer.mlp(out_emb)      
	        # 待办：第二个dropout（默认为0.0）
	        # 添加第二个残差连接
	        out_emb += after_first_residual      
	        # 添加到输出嵌入列表
	        outputs_embeds.append(out_emb)       
	        start = end  # 更新起始索引
	    else:  # 如果隐藏状态为None
	        outputs_embeds.append(None)  # 添加None到输出嵌入列表
	inputs_embeds = outputs_embeds  # 更新输入嵌入为输出嵌入，准备下一层处理
	```
	最后应用最终的层归一化
	```python
	# 最终归一化
	outputs_embeds = []               # 初始化最终输出嵌入列表
	# 遍历输入嵌入
	for i, hidden_states in enumerate(inputs_embeds):  
	     # 如果隐藏状态不为None
	    if hidden_states is not None: 
	        out_emb = models[i].norm(hidden_states)  # 应用最终层归一化
	        outputs_embeds.append(out_emb)           # 添加到输出嵌入列表
	    else:  
	        outputs_embeds.append(None)              # 添加None到输出嵌入列表
	# 返回输出嵌入和过去的键值对
	return outputs_embeds, past_key_values
	```
5. \`eager\_attention\_forward\`方法实现了标准的多头注意力机制，支持分组查询注意力(Grouped Query Attention，允许多个查询头共享相同的键值头，这是Gemma架构的特点)优化
	```cobol
	def eager_attention_forward(
	    self, attention_mask, batch_size, head_dim, query_states, key_states, value_states
	):
	    num_att_heads = self.config.paligemma_config.text_config.num_attention_heads
	    num_key_value_heads = self.config.paligemma_config.text_config.num_key_value_heads
	    num_key_value_groups = num_att_heads // num_key_value_heads
	```
	它将查询、键和值向量进行矩阵乘法操作，应用注意力掩码，执行softmax归一化，并计算最终的注意力输出

### 1.4 modeling\_pi0.py：含模型训练、模型推理(迭代去噪生成动作)

根据本博客的此文《 [π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)](https://blog.csdn.net/v_JULY_v/article/details/143472442 "π0——用于通用机器人控制的VLA模型：一套框架控制7种机械臂(基于PaliGemma和流匹配的3B模型)") 》

![](https://i-blog.csdnimg.cn/direct/98c9b28e5045450aa85251ee10da30c3.png)

可知pi0 模型采用了一个复杂的架构，主要由以下部分组成：

```
┌──────────────────────────────┐

│               actions        │

│               ▲              │

│              ┌┴─────┐        │

│  kv cache    │Gemma │        │

│  ┌──────────►│Expert│        │

│  │           │      │        │

│ ┌┴────────┐  │x 10  │        │

│ │         │  └▲──▲──┘        │

│ │PaliGemma│   │  │           │

│ │         │   │  robot state │

│ │         │   noise          │

│ └▲──▲─────┘                  │

│  │  │                        │

│  │  image(s)                 │

│  language tokens             │

└──────────────────────────────┘
```

该模块依赖于：

1. PyTorch 作为基础深度学习框架
2. Transformers 库中的 PaliGemma 和 Gemma 模型
3. LeRobot 框架中的数据处理和规范化工具

#### 1.4.1 库的导入与几个辅助函数的实现

具体而言，该代码首先导入了必要的库，包括PyTorch和其自定义的模块，如\`PaliGemmaWithExpertModel\`。文件顶部的文档字符串提供了模型的概述、论文链接、安装说明以及使用示例

代码中定义了几个辅助函数：\`create\_sinusoidal\_pos\_embedding\`用于生成正弦余弦位置编码向量；\`sample\_beta\`用于生成Beta分布样本；\`make\_att\_2d\_masks\`用于创建二维注意力掩码；\`resize\_with\_pad\`用于调整图像大小并进行填充；\`pad\_vector\`用于向量填充；\`normalize\`和\`unnormalize\`用于值的标准化与还原；以及一系列用于机器人抓取器转换的函数

#### 1.4.2 PI0Policy类的实现：将「PI0FlowMatching模型」集成到LeRobot框架中进行训练和推理

\`PI0Policy\`是一个包装类，用于将下一节的「 **PI0FlowMatching** 模型」集成到LeRobot框架中进行训练和推理

![](https://i-blog.csdnimg.cn/direct/e107d4b7c3d643869d1738de499f1934.png)

相当于PI0Policy类侧重高层抽象与环境的交互，而PI0FlowMatching侧重底层算法底线，当使用模型时，用户主要通过PI0Policy与系统交互，而不需要直接接触PI0FlowMatching的复杂实现细节

该类继承自\`PreTrainedPolicy\`，提供了一个统一的接口来处理多模态输入（图像、机器人状态、语言指令）并生成机器人动作序列

1. 在初始化阶段，\`PI0Policy\`接收一个配置对象和可选的数据集统计信息，设置了输入输出的归一化处理器，初始化了PaliGemma语言分词器和PI0FlowMatching模型核心。它还创建了一个动作队列，用于高效地管理预测的动作序列
2. 该类的\`select\_action\`方法是其核心推理接口，它实现了一个智能的队列机制：当动作队列为空时，它会处理完整的输入批次（包括准备图像、状态和语言指令），然后使用模型一次性生成多步动作序列并填充队列；在每次调用时，它只返回队列中的下一个动作，从而提高执行效率。这种设计特别适合于需要连续动作控制的机器人环境
3. 在训练过程中，\`forward\`方法负责计算损失函数
	```cobol
	def forward(self, batch: dict[str, Tensor], noise=None, time=None) -> tuple[Tensor, dict[str, Tensor]]:
	        # 执行完整的训练前向传播并计算损失
	        if self.config.adapt_to_pi_aloha:      # 如果配置为适配PI-Aloha模型
	            batch[OBS_ROBOT] = self._pi_aloha_decode_state(batch[OBS_ROBOT])  # 对机器人状态观测进行PI-Aloha解码转换
	            batch[ACTION] = self._pi_aloha_encode_actions_inv(batch[ACTION])  # 对动作进行PI-Aloha编码的逆变换
	```
	它首先对输入进行归一化处理，准备好所有模态数据
	```cobol
	batch = self.normalize_inputs(batch)       # 对输入数据进行归一化处理
	batch = self.normalize_targets(batch)      # 对目标数据进行归一化处理
	images, img_masks = self.prepare_images(batch)    # 准备并处理图像输入及其掩码
	state = self.prepare_state(batch)          # 准备机器人状态数据
	lang_tokens, lang_masks = self.prepare_language(batch)  # 准备语言指令的标记和掩码
	actions = self.prepare_action(batch)              # 准备动作数据
	actions_is_pad = batch.get("actions_id_pad")      # 获取动作填充标识（如果存在）
	```
	然后调用模型的前向传播函数计算每个步骤和每个电机的损失
	```cobol
	loss_dict = {}  # 初始化损失追踪字典，用于记录损失计算过程
	losses = self.model.forward(images, img_masks, lang_tokens, lang_masks, state, actions, noise, time)      # 调用核心PI0FlowMatching模型计算损失
	loss_dict["losses_after_forward"] = losses.clone()  # 记录模型前向传播后的原始损失
	```
	该方法还实现了智能的损失处理，包括对填充区域的剔除和统计跟踪。
4. 该类还包含几个专门的预处理方法：\`prepare\_images\`方法对图像进行调整大小、填充和归一化，以适应SigLIP视觉模型的要求；\`prepare\_language\`方法对文本指令进行分词处理；\`prepare\_state\`和\`prepare\_action\`方法对状态和动作向量进行填充
5. 特别值得注意的是适配Aloha系统的方法（\`\_pi\_aloha\_decode\_state\`、\`\_pi\_aloha\_encode\_actions\`等），这些方法通过翻转特定关节和转换抓取器位置，实现了与Aloha系统的兼容，展示了该模型在不同机器人平台间的适应性

#### 1.4.3 PI0FlowMatching类的实现：嵌入处理、训练、推理(迭代去噪生成最终动作)

\`PI0FlowMatching\`类是π0模型的核心实现，这是一个先进的视觉-语言-动作流模型，专为通用机器人控制而设计。该模型通过融合视觉输入、语言指令和机器人状态来生成精确的机器人动作序列

该类采用了流匹配（Flow Matching）技术，这是一种类似于扩散模型的方法，但具有更高效的训练和采样特性

在初始化阶段，它创建了一个PaliGemmaWithExpertModel实例（将PaliGemma视觉-语言模型与Gemma专家模型结合），并设置了处理状态、动作和时间信息的投影层

类的核心功能分为嵌入处理、训练流程和推理流程三个主要方面

**首先是嵌入处理，分为embed\_prefix和embed\_suffix**

\`embed\_prefix\`方法处理模型的前缀输入：图像和语言输入，使用PaliGemma模型将图像嵌入到特征空间，并对语言token进行嵌入，同时创建适当的注意力掩码以允许图像和语言token之间的全面注意力交互

1. 首先，该方法通过迭代输入的图像列表，将每个图像传递给\`paligemma\_with\_expert.embed\_image\`函数，生成图像嵌入。这些嵌入随后被转换为bfloat16数据类型，以优化内存使用和计算效率  
	  
	接着，方法应用了一个重要的归一化步骤，将图像嵌入乘以嵌入维度的平方根，这是Transformer架构中常用的缩放技术，有助于稳定训练过程和梯度流动  
	对于每个图像，方法还创建了相应的掩码，来标记哪些位置包含有效的图像内容，这些掩码将在后续的注意力计算中使用
2. 对于语言输入，该方法使用\`paligemma\_with\_expert.embed\_language\_tokens\`函数将文本标记转换为嵌入表示，并同样应用了归一化，乘以嵌入维度的平方根。语言嵌入和相应的掩码也被添加到累积列表中
3. 在处理完所有输入后，方法创建了注意力掩码（\`att\_masks\`）来控制不同输入组件之间的交互。值得注意的是，图像标记之间以及图像和语言标记之间被设置为完全可以相互关注（值为0），这允许模型充分融合视觉和语言信息。最后，方法将所有嵌入和掩码沿着序列维度（dim=1）连接起来，并对注意力掩码进行适当的扩展，以匹配批次大小
4. 返回的三元组包含连接后的嵌入、填充掩码和注意力掩码，这些将作为PaliGemma模型的输入，使其能够处理多模态信息并生成上下文丰富的表示，进而用于后续的机器人动作生成。代码中的TODO注释也表明了未来可能的优化方向，如预分配内存和移除循环以提高性能

\`embed\_suffix\`方法负责处理模型的"后缀"输入——即机器人状态、带噪声的动作和时间步信息，将时间步使用正弦-余弦位置编码表示，并通过一个两层MLP网络融合动作和时间信息

与\`embed\_prefix\`方法处理视觉和语言输入不同，这个方法专注于为Gemma专家模型准备必要的状态和动作表示

1. 首先，方法通过线性投影层\`state\_proj\`对机器人状态进行编码，将其转换为bfloat16数据类型以保持计算效率，并添加一个额外的维度使其成为一个单独的标记。对应的掩码被设置为全1，表示这是有效数据。注意力掩码值被设为1，这意味着前缀元素（图像和语言标记）不应关注这个状态标记，从而创建了信息流的单向边界
2. 接下来，方法处理时间步信息，使用正弦-余弦位置编码进行嵌入。这种编码技术特别适合表示连续的时间值，通过在不同时间尺度上（从4e-3到4.0的周期范围）使用正弦和余弦函数，创建了一个能够有效区分不同时间点的表示
3. 方法还对带噪声的动作应用了线性投影\`action\_in\_proj\`  
	然后，它巧妙地将时间嵌入扩展为与动作嵌入相同的形状，并在特征维度上连接它们  
	这个组合后的表示经过一个小型的多层感知机（MLP）处理：首先通过\`action\_time\_mlp\_in\`线性层，然后应用SiLU激活函数（也称为Swish），最后通过\`action\_time\_mlp\_out\`线性层。这一过程有效地融合了动作和时间信息，创建了上下文感知的表示
4. 在注意力掩码的设置上，方法采用了精心设计的模式：第一个动作标记被设为1，表示前缀元素不应关注它；而剩余的动作标记被设为0，允许完全的交叉注意力  
	  
	*这种设计确保了模型中信息的适当流动——状态和初始动作标记作为上下文独立的起始点，而后续的动作标记则能够关注和利用所有可用信息*
5. 最后，方法将所有嵌入和掩码连接起来，并对注意力掩码进行适当的扩展和格式化，以便在后续的Transformer处理中使用  
	  
	*这种结构化的表示方式是流匹配算法成功运行的关键，使模型能够从噪声动作平滑过渡到目标动作*

**其次是训练过程**

\`forward\`方法是PI0流匹配模型的核心训练流程，它实现了从多模态输入生成机器人动作的完整前向传播路径，并计算训练损失。这个方法基于流匹配（Flow Matching）技术，这是一种类似于扩散模型但更适合连续动作空间的生成方法

1. 首先，该方法确保有可用的噪声和时间参数
	```cobol
	def forward(
	        self, images, img_masks, lang_tokens, lang_masks, state, actions, noise=None, time=None
	    ) -> Tensor:
	        """Do a full training forward pass and compute the loss (batch_size x num_steps x num_motors)"""
	```
	如果未提供，它会分别调用\`sample\_noise\`生成标准正态分布噪声和\`sample\_time\`从Beta分布采样时间步（范围在0.001到0.999之间）
	```cobol
	# 如果没有提供噪声，则生成与动作形状相同的标准正态分布噪声
	if noise is None:
	    noise = self.sample_noise(actions.shape, actions.device)
	# 如果没有提供时间步，则从Beta分布采样时间（范围在0.001到0.999之间）
	if time is None:
	    time = self.sample_time(actions.shape[0], actions.device)
	```
2. 然后，它执行一个关键的线性插值操作：\`x\_t = time\_expanded \* noise + (1 - time\_expanded) \* actions\`，这创建了目标动作的噪声版本，其中时间接近1时更接近纯噪声，接近0时更接近真实动作
	```cobol
	# 扩展时间维度以便与动作形状匹配，用于后续广播操作
	        time_expanded = time[:, None, None]
	 
	0
	        x_t = time_expanded * noise + (1 - time_expanded) * actions
	```
	同时计算\`u\_t = noise - actions\`，表示从真实动作到噪声的向量场方向
	```cobol
	# 计算从真实动作到噪声的向量场方向，这是模型需要学习预测的目标
	u_t = noise - actions
	```
	如此文《 [π0源码剖析——从π0模型架构的实现(如何基于PaLI-Gemma和扩散策略去噪生成动作)，到基于C/S架构下的模型训练与部署](https://blog.csdn.net/v_JULY_v/article/details/146068251 "π0源码剖析——从π0模型架构的实现(如何基于PaLI-Gemma和扩散策略去噪生成动作)，到基于C/S架构下的模型训练与部署") 》中「1.2.4.3 损失函数compute\_loss：训练模型去噪的准确率」一节所说的：  
	$\rightarrow$ *创建带噪动作序列* ***x\_t*** *，相当于* ***x\_t是噪声化的动作*** *，随着时间从0到1，* *原始动作* *逐渐添加* ***真实噪声*** *，* *变为 **纯噪声***  
	******$\rightarrow$*** 而 代表所加的真实噪声*** *，便是咱们所要* ***预测的所添加的噪声*** *的ground truth  
	故* ***所添加的噪声*** *即 =* ***加满噪声的动作*** *\-* *原始动作*
3. 接下来，方法分别调用\`embed\_prefix\`和\`embed\_suffix\`处理输入组件：
	![](https://i-blog.csdnimg.cn/direct/98c9b28e5045450aa85251ee10da30c3.png)
	前者处理图像和语言token
	```php
	# 处理图像和语言输入，生成前缀嵌入表示和对应的掩码
	prefix_embs, prefix_pad_masks, prefix_att_masks = self.embed_prefix(
	    images, img_masks, lang_tokens, lang_masks
	)
	```
	后者处理机器人状态和噪声化的动作
	```perl
	# 处理机器人状态和噪声化动作，生成后缀嵌入表示和对应的掩码
	suffix_embs, suffix_pad_masks, suffix_att_masks = self.embed_suffix(state, x_t, time)
	```
	这两个函数返回的嵌入和掩码被连接起来
	```cobol
	# 沿序列维度连接前缀和后缀的填充掩码
	pad_masks = torch.cat([prefix_pad_masks, suffix_pad_masks], dim=1)
	# 沿序列维度连接前缀和后缀的注意力掩码
	att_masks = torch.cat([prefix_att_masks, suffix_att_masks], dim=1)
	```
	并使用\`make\_att\_2d\_masks\`函数创建二维注意力掩码，控制不同输入元素之间的信息流动
	```cobol
	# 创建二维注意力掩码，控制不同输入元素之间的信息流
	att_2d_masks = make_att_2d_masks(pad_masks, att_masks)
	```
	位置ID通过累积求和填充掩码并减1来生成
	```cobol
	# 通过累积求和填充掩码并减1来计算位置ID，用于位置编码
	position_ids = torch.cumsum(pad_masks, dim=1) - 1
	```
4. 随后，方法将准备好的输入传递给\`paligemma\_with\_expert\`模型进行处理，获取后缀输出（主要是动作表示）
	```cobol
	# 将准备好的输入传递给PaliGemma和Gemma专家模型，获取输出表示
	(_, suffix_out), _ = self.paligemma_with_expert.forward(
	    attention_mask=att_2d_masks,
	    position_ids=position_ids,
	    past_key_values=None,
	    inputs_embeds=[prefix_embs, suffix_embs],
	    use_cache=False,
	    fill_kv_cache=False,
	)
	```
	这个输出被裁剪为仅保留对应于动作步骤的部分
	```php
	# 从输出中提取最后n_action_steps个标记，对应于动作表示
	suffix_out = suffix_out[:, -self.config.n_action_steps :]
	```
	转换为float32数据类型
	```cobol
	# 将输出转换为float32数据类型，保持精度一致性
	suffix_out = suffix_out.to(dtype=torch.float32)
	```
	并通过\`action\_out\_proj\`投影到动作空间，得到预测的向量场\`v\_t\`
	```php
	# 通过线性投影将后缀输出转换为动作向量场预测
	v_t = self.action_out_proj(suffix_out)
	```
5. 最后，方法计算预测向量场\`v\_t\`与真实向量场\`u\_t\`之间的均方误差作为损失函数
	```cpp
	# 计算预测向量场v_t与真实向量场u_t之间的均方误差损失
	losses = F.mse_loss(u_t, v_t, reduction="none")
	# 返回逐元素损失张量，供调用者进一步处理
	return losses
	```
	这种训练方式使模型学习从任意噪声状态到目标动作的向量场，在推理时可以通过从随机噪声开始，沿着这个向量场逐步前进来生成平滑、精确的动作序列

**最后是推理：依次sample\_actions、denoise\_step**

**首先，\`sample\_actions\`** 方法是PI0流匹配模型的核心推理函数，负责根据视觉、语言指令和机器人状态生成一系列动作

与训练时的\`forward\`方法不同，这个方法实现了从随机噪声到有意义的动作序列的生成过程，采用了类似于扩散模型的逐步降噪技术

1. 首先，该方法获取批次大小和设备信息
	```ruby
	def sample_actions(self, images, img_masks, lang_tokens, lang_masks, state, noise=None) -> Tensor:
	        # 执行完整的推理前向传播并计算动作（批次大小 x 步骤数 x 电机数）
	# 获取批次大小（从状态tensor的第一维）
	        device = state.device       # 获取当前设备（CPU或GPU）
	```
	如果未提供噪声，则生成形状为(批次大小, 动作步数, 最大动作维度)的标准正态分布噪声作为起始点
	```python
	if noise is None:        # 如果没有提供噪声
	    actions_shape = (bsize, self.config.n_action_steps, self.config.max_action_dim)      # 创建噪声形状：(批次大小, 动作步数, 最大动作维度)
	    noise = self.sample_noise(actions_shape, device)      # 采样标准正态分布噪声
	```
	接着，它调用\`embed\_prefix\`处理图像和语言输入，创建嵌入表示和对应的掩码
	```php
	# 处理图像和语言输入，生成前缀嵌入及相关掩码
	prefix_embs, prefix_pad_masks, prefix_att_masks = self.embed_prefix(  
	    images, img_masks, lang_tokens, lang_masks
	)
	```
	并通过\`make\_att\_2d\_masks\`函数将其转换为二维注意力掩码，同时计算位置ID
	```cobol
	# 为前缀创建二维注意力掩码
	prefix_att_2d_masks = make_att_2d_masks(prefix_pad_masks, prefix_att_masks)  
	# 计算前缀位置ID（累积和减1）
	prefix_position_ids = torch.cumsum(prefix_pad_masks, dim=1) - 1
	```
2. 一个关键的优化是计算并缓存前缀（图像和语言）输入的键值对  
	这是通过调用\`paligemma\_with\_expert.forward\`并设置\`use\_cache=True\`和\`fill\_kv\_cache=True\`实现的
	```cobol
	# 计算图像和语言的键值缓存，提高推理效率
	_, past_key_values = self.paligemma_with_expert.forward(
	    attention_mask=prefix_att_2d_masks,      # 设置注意力掩码
	    position_ids=prefix_position_ids,        # 设置位置ID
	    past_key_values=None,                    # 初始没有过去的键值对
	    inputs_embeds=[prefix_embs, None],       # 只传入前缀嵌入（图像和语言）
	    use_cache=self.config.use_cache,         # 使用缓存机制
	    fill_kv_cache=True,                      # 填充键值缓存
	)
	```
	由于前缀输入在整个推理过程中保持不变，这种缓存机制避免了重复计算，显著提高了效率
3. 然后，方法设置欧拉法数值积分的 **时间步长\`dt\`（负值，因为时间从1倒数到0）** ，初始化噪声状态\`x\_t\`，并将时间设置为1.0（表示起始的纯噪声状态）
	```cobol
	0
	        dt = -1.0 / self.config.num_steps      
	 
	        # 转换为tensor
	        dt = torch.tensor(dt, dtype=torch.float32, device=device)
	```
	接下来进入主要的降噪循环，直到时间接近或达到0：  
	1\. 将当前时间扩展为与批次大小匹配的张量
	```cobol
	x_t = noise  # 初始化噪声状态为纯噪声
	time = torch.tensor(1.0, dtype=torch.float32, device=device)  # 设置初始时间为1.0（表示纯噪声状态）
	while time >= -dt / 2:                  # 降噪循环，直到时间接近或达到0
	    expanded_time = time.expand(bsize)  # 扩展时间为批次大小匹配的tensor
	```
	2\. 调用\` **denoise\_step** \`方法预测当前状态和时间下的向量场\`v\_t\`——即预测噪声
	```crystal
	v_t = self.denoise_step(    # 执行一步降噪，预测向量场
	    state,                  # 机器人状态
	    prefix_pad_masks,       # 前缀填充掩码
	    past_key_values,        # 键值缓存
	    x_t,                    # 当前噪声状态
	    expanded_time,          # 当前时间步
	)
	```
	3\. 执行欧拉步骤更新\`x\_t\`（通过公式\`x\_t += dt \* v\_t\`）  
	注意，本质就是对去噪，而便是预测的噪声，是时间步长——如上面说过的「 *时间步长\`dt\`为负值（因为是从t=1向t=0方向演化），生成初始随机噪声作为起点，且时间上约定："t=1是噪声，t=0是目标分布"* 」
	```cobol
	# 欧拉步骤，更新噪声状态（沿向量场方向移动）
	x_t += dt * v_t
	```
	这种欧拉积分实际上是在求解概率流ODE——Ordinary Differential Equation，从噪声分布逐步转换到目标动作分布。通过迭代调用\`denoise\_step\`，模型能够逐渐去除噪声，显现出与输入条件（图像、语言和状态）相符的有意义动作序列  
	4\. 更新时间（\`time += dt\`）
	```cobol
	time += dt      # 更新时间（向0移动）
	```
	最后返回去噪后的动作序列
	```csharp
	return x_t      # 返回最终去噪后的动作序列
	```

**其次，\`denoise\_step\`** 方法是PI0流匹配模型中的核心推理组件，负责在流匹配过程中执行单个降噪步骤。该方法接收机器人状态、前缀填充掩码、键值缓存、当前噪声状态和时间步作为输入，并返回向量场预测——return v\_t，指导噪声朝着目标动作转变

1. 首先，方法调用\`embed\_suffix\`函数处理机器人状态、噪声动作和时间步信息，生成相应的嵌入表示和掩码。这些表示包含了状态和噪声动作在当前时间点的完整上下文
	```ruby
	def denoise_step(
	        self,
	        state,
	        prefix_pad_masks,
	        past_key_values,
	        x_t,
	        timestep,
	    ):
	        # 在给定的时间步对噪声\`x_t\`应用一个降噪步骤
	        # 处理状态、噪声动作和时间步，生成后缀嵌入及相关掩码
	        suffix_embs, suffix_pad_masks, suffix_att_masks = self.embed_suffix(state, x_t, timestep)
	```
2. 接下来，方法构建复杂的注意力掩码系统，以实现前缀（已缓存的图像和语言表示）和后缀（状态和动作）之间的适当交互。它计算后缀序列长度、批次大小和前缀长度，然后扩展前缀掩码维度以匹配所需的注意力掩码形状
	```cobol
	suffix_len = suffix_pad_masks.shape[1]      # 获取后缀序列的长度
	batch_size = prefix_pad_masks.shape[0]      # 获取批次大小
	prefix_len = prefix_pad_masks.shape[1]      # 获取前缀序列的长度
	prefix_pad_2d_masks = prefix_pad_masks[:, None, :].expand(batch_size, suffix_len, prefix_len)              # 将前缀掩码扩展为三维形状，适合注意力计算
	```
	同时，它使用\`make\_att\_2d\_masks\`函数为后缀创建二维注意力掩码，并将两个掩码沿第三维连接，形成完整的注意力掩码
	```cobol
	suffix_att_2d_masks = make_att_2d_masks(suffix_pad_masks, suffix_att_masks)  # 为后缀创建二维注意力掩码
	full_att_2d_masks = torch.cat([prefix_pad_2d_masks, suffix_att_2d_masks], dim=2)  # 沿第三维连接前缀和后缀掩码，形成完整注意力掩码
	```
3. 一个关键的处理步骤是位置ID的计算，它先计算前缀偏移量（通过对前缀掩码求和），然后加上后缀填充掩码的累积和并减1
	```cobol
	# 计算前缀偏移量（每个样本有效前缀标记的数量）
	prefix_offsets = torch.sum(prefix_pad_masks, dim=-1)[:, None]  
	# 计算位置ID，确保前缀和后缀的位置编码连续
	position_ids = prefix_offsets + torch.cumsum(suffix_pad_masks, dim=1) - 1
	```
	这确保了位置编码的连续性，使模型能够正确处理序列位置信息
4. 然后，方法调用\`paligemma\_with\_expert.forward\`  
	但与训练阶段不同的是，这里只传入后缀嵌入（前缀部分已通过\`past\_key\_values\`缓存），这大大提高了推理效率
	```cobol
	# 调用PaliGemma和Gemma专家模型的前向传播
	outputs_embeds, _ = self.paligemma_with_expert.forward(  
	    attention_mask=full_att_2d_masks,      # 传入完整注意力掩码
	    position_ids=position_ids,  # 传入位置ID
	    past_key_values=past_key_values,       # 传入缓存的键值对（来自前缀处理）
	    inputs_embeds=[None, suffix_embs],     # 只传入后缀嵌入（前缀已缓存）
	    use_cache=self.config.use_cache,       # 是否使用缓存机制
	    fill_kv_cache=False,          # 不填充新的键值缓存（使用现有缓存）
	)
	```
	方法设置\`fill\_kv\_cache=False\`，表示使用现有缓存而非创建新缓存
5. 最后，方法提取后缀输出，特别是与动作步骤对应的部分，将其转换为float32数据类型（保持计算精度）
	```cobol
	# 提取后缀输出（对应于Gemma专家模型输出）
	suffix_out = outputs_embeds[1]  
	# 只保留最后n_action_steps个标记的输出（对应动作部分）
	suffix_out = suffix_out[:, -self.config.n_action_steps :]  
	# 转换为float32数据类型以保持计算精度
	suffix_out = suffix_out.to(dtype=torch.float32)
	```
	并通过\`action\_out\_proj\`投影到动作空间，得到向量场预测\`v\_t\`
	```php
	# 通过线性投影将输出转换为动作空间中的向量场预测
	v_t = self.action_out_proj(suffix_out)  
	# 返回预测的向量场（指导噪声如何移动到目标点）
	return v_t
	```

这个方法体现了流匹配算法的精髓——它不是直接预测动作，而是预测动作空间中的向量场，指导噪声状态如何逐步转变为有意义的动作。在\`sample\_actions\`方法的循环中，这个函数被反复调用，通过欧拉积分逐步将随机噪声转化为精确、平滑且符合条件的机器人动作序列

### 1.5 flex\_attention.py：实现了分组查询注意力

#### 1.5.1 对分组查询注意力(GQA)的回顾

\`flex\_attention\_forward\`函数实现了PyTorch 2.5之后引入的FlexAttention机制，这是一种高效的注意力计算方案，专为大型语言模型设计，特别是使用分组查询注意力(GQA)的模型

> 关于GQA的介绍，详见此文《 [https://blog.csdn.net/v\_JULY\_v/article/details/134228287](http://一文通透各种注意力：从多头注意力MHA到分组查询注意力GQA、多查询注意力MQA "https://blog.csdn.net/v_JULY_v/article/details/134228287") 》
> 
> ![](https://i-blog.csdnimg.cn/direct/f5e8a732263a4655a4c598442262d824.png)

在PI0架构中，这是三种可选的注意力实现之一（其他两种为"eager"和"fa2"），提供了优化的内存使用和计算效率

#### 1.5.2 每个键值KV头服务于8个查询Q头——相当于value头数/key头数是query头数的1/8

函数开始时记录输入张量的原始数据类型，然后设置分组查询注意力的参数：8个注意力头但只有1个键值头，每个键值KV头服务于8个查询Q头—— *相当于value头数/key头数是query头数的1/8* ， *这种配置是Gemma模型的特点，能在保持表达能力的同时显著减少内存占用和计算量*

```cobol
original_dtype = query_states.dtype      # 保存查询状态的原始数据类型

num_att_heads = 8              # 设置注意力头数量为8

num_key_value_heads = 1        # 设置键值头数量为1（分组查询注意力的特点）

num_key_value_groups = num_att_heads // num_key_value_heads  # 计算每个键值头对应的查询头组数
```

接下来，函数对键状态和值状态执行精心设计的扩展操作，使单个键值头能够被多个查询头共享。这通过添加维度、扩展和重塑键值张量来实现，确保它们与查询头的数量匹配

1. 比如先对K做添加、扩展、重塑
	```cobol
	# 在键状态张量中添加一个维度，用于后续展开
	key_states = key_states[:, :, :, None, :]  
	# 扩展键状态张量以匹配所有查询头
	key_states = key_states.expand(  
	    # 扩展为[批次大小, 序列长度, 键值头数, 每组查询头数, 头维度]
	    batch_size, key_states.shape[1], num_key_value_heads, num_key_value_groups, head_dim  
	)
	# 重塑键状态张量以便于计算
	key_states = key_states.reshape(  
	    # 重塑为[批次大小, 序列长度, 总注意力头数, 头维度]
	)
	    batch_size, key_states.shape[1], num_key_value_heads * num_key_value_groups, head_dim
	```
2. 然后再对V做添加、扩展、重塑
	```cobol
	# 在值状态张量中添加一个维度，用于后续展开
	value_states = value_states[:, :, :, None, :]  
	# 扩展值状态张量以匹配所有查询头
	value_states = value_states.expand(  
	    # 扩展为[批次大小, 序列长度, 键值头数, 每组查询头数, 头维度]
	    batch_size, value_states.shape[1], num_key_value_heads, num_key_value_groups, head_dim  
	)
	 # 重塑值状态张量以便于计算
	value_states = value_states.reshape( 
	    # 重塑为[批次大小, 序列长度, 总注意力头数, 头维度]
	    batch_size, value_states.shape[1], num_key_value_heads * num_key_value_groups, head_dim  
	)
	```
3. 最后做转置
	```cobol
	# 转置查询状态张量，将头维度移到前面 [批次大小, 注意力头数, 序列长度, 头维度]
	query_states = query_states.transpose(1, 2)  
	# 转置键状态张量，将头维度移到前面 [批次大小, 注意力头数, 序列长度, 头维度]
	key_states = key_states.transpose(1, 2)  
	# 转置值状态张量，将头维度移到前面 [批次大小, 注意力头数, 序列长度, 头维度]
	value_states = value_states.transpose(1, 2)
	```

为了保证计算精度，函数将所有状态转换为float32类型

```cobol
# 将查询状态转换为float32类型以提高计算精度

query_states = query_states.to(torch.float32)  

# 将键状态转换为float32类型以提高计算精度

key_states = key_states.to(torch.float32)  

# 将值状态转换为float32类型以提高计算精度

value_states = value_states.to(torch.float32)
```

然后处理因果掩码（causal mask）。掩码确保每个位置只能关注当前及之前的位置，这对自回归生成至关重要

```cobol
# 将输入的注意力掩码赋值给因果掩码变量

causal_mask = attention_mask  

# 如果因果掩码不为空

if causal_mask is not None:  

    # 调整掩码形状以匹配注意力头和序列长度

    causal_mask = causal_mask[:, None, :, : key_states.shape[2]]  

    # 如果掩码的注意力头维度为1，但查询状态有多个头

    if causal_mask.shape[1] == 1 and query_states.shape[1] > 1:  

        # 扩展掩码以匹配查询状态的注意力头数

        causal_mask = causal_mask.expand(-1, query_states.shape[1], -1, -1)
```

#### 1.5.2 针对FlexAttention的优化，函数实现的一个巧妙的块处理系统

针对FlexAttention的优化，函数实现了一个巧妙的块处理系统：

1. 通过\`precomputed\_mask\_factory\`创建掩码访问函数，将序列长度向上取整为128（块大小）的倍数，并添加适当的填充
	```cobol
	# 定义预计算掩码工厂函数
	def precomputed_mask_factory(precomputed_mask: torch.Tensor) -> _mask_mod_signature:  
	    # 内部定义掩码修改函数，接收批次、头、查询索引和键值索引
	    def mask_mod(b, h, q_idx, kv_idx):  
	        # 危险区域：如果索引超出形状，会在设备端触发断言
	        # 返回指定位置的掩码值
	        return precomputed_mask[b][h][q_idx][kv_idx]  
	    return mask_mod  # 返回掩码修改函数
	# 获取因果掩码的形状参数
	b_mask, h_mask, q_len, kv_len = causal_mask.shape  
	# 设置块大小为128，用于优化计算
	block_size = 128  
	# 将查询长度向上取整到块大小的倍数
	q_len_rounded = _round_up_to_multiple(q_len, block_size)  
	# 将键值长度向上取整到块大小的倍数
	kv_len_rounded = _round_up_to_multiple(kv_len, block_size)  
	# 关键：我们需要在这里扩展，否则会得到CUDA索引错误
	# 计算查询维度需要的填充量
	pad_q = q_len_rounded - q_len  
	# 计算键值维度需要的填充量
	pad_k = kv_len_rounded - kv_len  
	# 对因果掩码进行填充，使其大小符合块大小要求
	padded_causal_mask = F.pad(causal_mask, (0, pad_k, 0, pad_q), value=0.0)  
	# 创建填充掩码的修改函数
	mask_mod_fn_orig = precomputed_mask_factory(padded_causal_mask)
	```
2. 代码中最关键的部分是对掩码的处理和块掩码的创建  
	首先通过\`create\_mask\`生成完整的4D掩码
	```cobol
	# 创建4D掩码
	=
	        # 使用原始掩码修改函数
	        mod_fn=mask_mod_fn_orig,  
	        B=b_mask,                    # 批次大小
	        H=h_mask,                    # 头数量
	        Q_LEN=q_len_rounded,         # 查询长度（已取整）
	        KV_LEN=kv_len_rounded,       # 键值长度（已取整）
	        device=causal_mask.device,   # 设备与因果掩码相同
	        _compile=False,              # 不使用编译
	    )
	```
	然后通过\`create\_block\_mask\`将其转换为更高效的块式表示
	```cobol
	# 为4D掩码创建掩码修改函数
	mask_mod_fn_padded = precomputed_mask_factory(mask_4d)  
	block_mask = create_block_mask(       # 创建块掩码
	    mask_mod=mask_mod_fn_padded,      # 使用填充后的掩码修改函数
	    B=b_mask,                         # 批次大小
	    H=h_mask,                         # 头数
	    Q_LEN=q_len_rounded,              # 向上取整后的查询长度
	    KV_LEN=kv_len_rounded,            # 向上取整后的键值长度
	    BLOCK_SIZE=block_size,            # 块大小
	    device=causal_mask.device,        # 使用与因果掩码相同的设备
	    _compile=False,                   # 不编译
	)
	```
	这些块构造函数接受\`mask\_mod\`函数作为输入，该函数提供了安全访问掩码值的方法，特别注意了越界访问可能导致的设备端断言错误
3. 最后，函数调用\`flex\_attention\`内核，该内核在底层实现了高效的注意力计算
	```cobol
	# 掩码在内核中应用，理想情况下比score_mod更高效
	# 调用FlexAttention函数计算注意力输出和权重
	attn_output, attention_weights = flex_attention(  
	    query_states,            # 查询状态
	    key_states,              # 键状态
	    value_states,            # 值状态
	    block_mask=block_mask,   # 块掩码
	    # 启用分组查询注意力(GQA)，因为我们已经对查询/键状态进行了相应的形状调整
	    enable_gqa=True, 
	    # 设置缩放因子，默认为head_dim的平方根的倒数
	    scale=head_dim**-0.5 if scaling is None else scaling,  
	    # 返回对数和指数值
	    return_lse=True,  
	)
	```
4. 结果被转换回原始数据类型，转置并重塑为期望的输出格式\[批次大小, 序列长度, 嵌入维度\]
	```cobol
	# 将注意力输出转换回原始数据类型
	attn_output = attn_output.to(dtype=original_dtype)  
	# [B, Q_LEN, H, head_dim]，转置注意力输出并确保内存连续
	attn_output = attn_output.transpose(1, 2).contiguous()  
	# 重塑注意力输出的形状
	attn_output = attn_output.reshape(  
	    batch_size,      # 批次大小
	    -1,              # 自动计算第二维大小
	    # 合并头数和头维度
	    attn_output.shape[2] * attn_output.shape[3],  # merges [H, head_dim]  
	)
	# 返回注意力输出
	return attn_output
	```

// 待更