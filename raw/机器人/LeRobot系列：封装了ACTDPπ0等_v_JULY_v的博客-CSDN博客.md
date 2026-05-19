![](https://i-blog.csdnimg.cn/columns/default/20201014180756757.png?x-oss-process=image/resize,m_fixed,h_224,w_224)

### LeRobot系列：封装了ACT/DP/π0等

已关注

文章平均质量分 97

Hugging Face打造的机器人开源库

关注数： 文章数：6 文章阅读量：58680 文章收藏量：404

### v\_JULY\_v

七月在线创始人，结构之法算法之道blog之博主

展开

专栏收录文章

- [
	摘要：本文介绍了轻量级视觉-语言-动作模型SmolVLA的创新设计与应用。该模型通过优化架构（跳层处理、视觉token压缩、交错注意力机制）和流匹配动作专家，在消费级GPU上实现高效训练与CPU部署。使用不到3万个公开样本进行预训练后，其性能媲美更大规模VLA模型。异步推理架构将感知与动作预测解耦，显著降低延迟。相比传统VLA依赖昂贵硬件，SmolVLA为机器人领域提供了可复现、低成本的解决方案，尤其适合具身智能的实时控制场景。
	原创 2025-06-17 23:31:08 · 6344 阅读 · 1 评论
	](https://blog.csdn.net/v_JULY_v/article/details/148723688)
- [
	近期在抠lerobot源码时，看到其封装了ALOHA ACT、diffusion policy、π0时，我就在想，lerobot其实可以再封装下idp3我甚至考虑是否从我联合带的那十几个具身研究生中选几个同学做下这事，对他们也是很好的历练截止到，25年3.18日晚上，我把lerobot抠的差不多了，然后刚看到傅利叶fork了lerobot，并在fork的fourier-lerobot中，把idp3封装了进去，实在是卷啊..再加之工厂机械臂开发订单之外，我司近期接到的B端。
	原创 2025-03-22 23:58:46 · 5790 阅读 · 3 评论
	](https://blog.csdn.net/v_JULY_v/article/details/146448646)
- [
	本文详细剖析了LeRobot框架中π0模型的实现与优化。π0是一个结合视觉-语言-动作的多模态模型，用于通用机器人控制，核心包括： 架构设计 基于PaliGemma视觉语言模型与Gemma专家模型的融合 采用流匹配技术生成机器人动作序列 支持分组查询注意力(GQA)优化推理效率 关键实现 转换工具：将JAX实现的模型转换为PyTorch格式 配置系统：统一管理输入/输出结构、归一化策略和训练参数 注意力优化：提供三种实现（eager/fa2/flex）适配不同硬件 训练流程：通过噪声插值和向量场预测学习动作
	原创 2025-06-02 00:04:13 · 11346 阅读 · 5 评论
	](https://blog.csdn.net/v_JULY_v/article/details/148370072)
- [
	过去2年多的深入超过此前7年，全靠夜以继日的勤奋，一天当两天用，抠论文 抠代码 和大模型及具身同事讨论，是目前日常而具身库里，idp3 π0 lerobot值得反复研究，故，近期我一直在抠π0及lerobot的源码本文一开始是此文《LeRobot——Hugging Face打造的机器人开源库：包含对顶层script、与底层基础层dataset的源码分析》的第四部分，考虑到为避免该文的篇幅过长，故把该文的第四部分独立出来，成本文该模块包含以下策略该模块主要包含以下组件可能马上就有同学疑问了，那这个模块和π0的
	原创 2025-03-17 00:15:54 · 11703 阅读 · 8 评论
	](https://blog.csdn.net/v_JULY_v/article/details/146304377)
- [
	本文解析了ALOHA团队提出的动作序列预测算法ACT（Action Chunking with Transformers）在LeRobot框架中的实现与应用。该算法通过Transformer架构同时预测未来动作序列（动作块），而非传统单步预测，使机器人行为更加连贯前瞻。文章详细剖析了核心组件： ACTPolicy类作为接口层，提供两种动作选择机制 多模态Transformer架构包含： 可选VAE编码器捕获动作分布 ResNet视觉骨干网络提取图像特征 Transformer编码器处理多模态输入 Trans
	原创 2025-06-01 20:24:51 · 6638 阅读 · 1 评论
	](https://blog.csdn.net/v_JULY_v/article/details/148369919)
- [
	5月6日，Hugging Face的机器人项目负责人雷米·卡德内Remi Cadene宣布推出LeRobot开源代码库，并形容它对于机器人的意义就如同“Transformer架构之于NLP”Remi Cadene在推文中表示，LeRobot之于机器人就像Transformer架构之于NLP——它提供带有预训练检查点的高级AI模型的简洁实现。他们还复现了来自学术界的 31 个数据集和一些模拟环境，无需实体机器人即可开始使用
	原创 2024-06-15 00:47:28 · 16871 阅读 · 10 评论
	](https://blog.csdn.net/v_JULY_v/article/details/139692392)