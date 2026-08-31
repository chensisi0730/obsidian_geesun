[收录于 · PI系列解读](https://www.zhihu.com/column/c_2018034381093097708)

110 人赞同了该文章

原论文： *[π0.7](https://zhida.zhihu.com/search?content_id=273271213&content_type=Article&match_order=1&q=%CF%800.7&zhida_source=entity): a Steerable Generalist Robotic Foundation Model with Emergent Capabilities ，2026.4.16*  
项目页： [pi.website/pi07](https://link.zhihu.com/?target=https%3A//pi.website/pi07)

本文已授权

[@具身智能之心](https://www.zhihu.com/people/9556e569f8a54346907f66b6a94c5ce5)

VLA 的主流路线通常把任务指令、视觉观测和机器人状态塞进模型，然后让动作专家生成一个 action chunk。这个范式已经证明了规模化预训练对机器人控制是有用的，但它仍然有一个很现实的问题：同一句“clean up the kitchen”背后可能对应不同速度、不同质量、不同中间阶段、不同机器人形态下的多种策略。如果训练数据里混入失败轨迹、低质量轨迹、自主 rollout、非机器人视频和不同 embodiment 的数据，而 prompt 仍然只告诉模型“做什么”，模型很难知道这些数据到底应该如何被使用。

π0.7 的核心动机是：机器人基础模型不只需要理解任务目标，还需要在训练和推理时知道“用什么风格、什么中间目标、什么控制模式去做”。因此本文没有把主要贡献放在提出一个全新的网络结构上，而是把重点放在 **多模态上下文条件化（diverse context conditioning）** ：语言任务、子任务指令、subgoal images、 [episode metadata](https://zhida.zhihu.com/search?content_id=273271213&content_type=Article&match_order=1&q=episode+metadata&zhida_source=entity) 、control mode 共同进入 prompt，使模型能够从混合质量、多来源的数据里学到可控策略，并在测试时通过 prompt 被精确 steering。

主要发现如下：

- π0.7 可以在没有任务级后训练的情况下，直接完成多类复杂灵巧任务；在 π\*\_0.6 任务上，它能匹配任务专用 RL specialist，并在 diverse laundry folding 和 box building 上取得更高 throughput
- prompt 组成并不是装饰项：去掉 episode metadata 或去掉 autonomous evaluation episodes 后，π0.7 在 π\*\_0.6 任务上的表现都会下降，尤其 throughput 差距明显
- 在没有目标机器人任务数据的情况下，π0.7 能把衣物折叠等灵巧技能迁移到 [UR5e](https://zhida.zhihu.com/search?content_id=273271213&content_type=Article&match_order=1&q=UR5e&zhida_source=entity)
- 规模化数据是否有用取决于 prompt 是否足够细：带 metadata 的 π0.7 能从更大但平均质量更低的数据中持续获益；没有 metadata 时，更多混合质量数据反而性能下降

接下来，我们将本文内容拆解为以下 4 个部分：

1. 模型结构和方法定位
2. 世界模型和prompt 设计
3. 模型训练与推理流程
4. 实验结果与附录细节

---

### 一、模型结构和方法定位

论文对自己的定位说得比较清楚：π0.7 并不是要提出一个全新的 VLA 架构，而是提出一种让 VLA 利用更多异质数据的方法。以往的机器人基础模型往往依赖高质量人工 demonstration，但现实里的可用数据更复杂：有成功演示、有失败 episode、有自主 rollouts、有 egocentric human video，也有来自 web 的非机器人数据。问题在于，这些数据不能被简单混在一起训练，否则模型可能把低质量行为也当成目标策略。

π0.7 的解决思路是让每条轨迹都带上更丰富的上下文。训练时，模型不只看到“任务是什么”，还看到“当前子任务是什么”“未来子目标图像是什么”“这条轨迹的速度、质量、是否犯错是什么”“动作应该用 joint 还是 end-effector 控制”。这些信息共同组成 context ，让同一批混合数据在训练时被区分开来，在推理时又可以反向用来指定想要的行为模式。

![](https://pic4.zhimg.com/v2-2e455ad0e63eacc9983c758fe917a397_1440w.jpg)

论文先从标准 flow-based VLA 形式开始。训练数据集 包含机器人轨迹，每条轨迹由观测 和动作 组成。观测写作：

其中 表示第 个相机在时刻 的图像， 是相机数量， 是机器人关节配置。动作 可以是 joint commands，也可以是 end-effector commands。

VLA 通常根据最近一段观测历史 预测未来动作 chunk 。其中 是动作 chunk 长度，实际执行时可能只执行较短的 步，以便闭环更新。每个训练样本还带有上下文 ；传统设定下， 往往只是语言指令 ：

在这个基础上，π0.7 的 VLA 训练目标写成（式 1）：

逐项解释：

- 是 VLA 策略模型，参数为 。
- 是从当前时刻开始的一段未来动作 chunk。
- 是最近 个时间窗口内的观测历史。
- 是 prompt/context，π0.7 的关键就在于把它从单一语言指令扩展成多模态、多属性的条件集合。
- 表示在训练数据分布上最大化条件动作 likelihood。

论文也提醒：如果 action expert 使用 [Flow Matching](https://zhida.zhihu.com/search?content_id=273271213&content_type=Article&match_order=1&q=Flow+Matching&zhida_source=entity) ，那么实际优化的是近似 lower bound，而不是闭式 log-likelihood。这个细节很重要，因为 π0.7 的动作专家仍然是 flow-based action expert，但文章的核心不是换掉这个目标，而是改变 的信息量和可控性。

![](https://pic2.zhimg.com/v2-837f1be20d2da185e8aa3de0ac265431_1440w.jpg)

---

### 二、世界模型和prompt 设计

π0.7 的 prompt 可以理解为把“任务目标”和“执行方式”拆开表达。论文主要加入四类上下文：子任务指令、subgoal images、episode metadata、control mode。它们分别解决不同问题。

### 子任务指令：把长程任务拆成当前可执行阶段

整体任务指令 描述最终目标，例如“clean up the kitchen”。但对于长程任务，仅靠整体指令很难告诉动作模型当前应该做哪一步。因此 π0.7 额外加入中间的子任务指令 ，例如“open the fridge door”。在推理时， 可以由高层语言策略生成，也可以由人类通过 coaching 给出。

这使 π0.7 的 prompt 不只是“最终要什么”，还包含“现在该做什么”。从实验看，这一点对后文的长程任务 coaching 尤其关键：模型可以在没有新 action-level demonstration 的情况下，靠人类逐步语言指导完成完全未见过的任务。

### subgoal images：用未来图像补足语言里说不清的执行细节

子任务指令能表达高层意图，但很多执行细节很难用语言说清楚。例如“open the fridge door”没有指定机械臂应该如何接近门把手、抓哪里、最后场景应该变成什么样。π0.7 因此使用多视角 subgoal images：

其中 是第 个相机视角下期望的近未来图像， 是所有视角的 subgoal image 集合。base view 更容易表达环境与物体状态，wrist view 更容易表达手臂和夹爪的局部状态，因此多视角 subgoal 可以提供比纯语言更细的空间约束。

推理时，subgoal images 由轻量 world model 生成。论文将这个 world model 记为 ，训练目标为：

这里：

- 是用于生成 subgoal images 的 world model，参数为 。
- 是当前机器人观测。
- 是当前子任务指令。
- 是 episode metadata。
- 是 ground-truth future subgoal image，论文中由 segment 末尾图像提供，即 。
- 是带高质量分段语言标签的训练片段子集。
- 是 standard Flow Matching loss，用于训练生成模型。

注意这里的目标函数在论文里写成最大化形式，但 本身是 Flow Matching 训练项。解读时需要抓住核心：world model 学的是“给定当前观测、子任务和 metadata，生成对动作模型有用的近未来视觉目标”，而不是推理时 rollout 一整段未来视频。

![](https://pic4.zhimg.com/v2-dc1725b9359e0fab60a37d3c6c401ac1_1440w.jpg)

### episode metadata：让混合质量数据不再互相污染

π0.7 的一个重要设计是给每段训练 episode 标注 metadata 。论文列出的 metadata 包含：

| 字段 | 含义 | 推理时设置 |
| --- | --- | --- |
| Overall speed | episode 长度，以 500 steps 为区间离散化 | 每个任务设置为 episode length 的第 15 百分位 |
| Overall quality | 执行质量，取 1 到 5，5 为最高 | 固定为 5 |
| Mistake | 是否出现错误，例如抓取失败或执行了错误子任务 | 固定为 false |

这套设计的核心价值在 Fig. 18：当训练数据越来越大、但平均质量下降时，带 metadata 的 π0.7 仍能持续提升；没有 metadata 的版本则可能因为混入低质量数据而退化。也就是说，metadata 让模型知道哪些行为是“应该模仿的高质量策略”，哪些只是“曾经发生过的低质量轨迹”。

### control mode：把动作空间也放进 prompt

π0.7 同时训练 joint-level 和 end-effector actions，并使用文本标识符指定控制模式：

其中 是 control mode。推理时，可以根据任务选择 joint control 或 end-effector control。附录 E 进一步比较了 prior models 在 cross-embodiment tasks 上的 joint-space 与 EE control，结论是 EE control 没有显示出明显优势，因此主文 cross-embodiment 实验聚焦 joint-space control。

论文给出的完整 prompt 形态大致如下：

```
<Multi-view observation><Multi-view subgoal>
Task: peel vegetables. Subtask: pick up the
peeler. Speed: 8000. Quality: 5. Mistake:
false. Control Mode: joint.<Proprioception>
```

训练时，π0.7 会对 prompt 组件做 dropout，使测试时可以灵活组合不同条件：visual subgoal images 只加到每个 batch 的 25% 样本中；在有 subgoal images 的样本中，子任务指令 有 30% 概率被 drop；episode metadata 整体有 15% 概率被 drop，且 speed/quality/mistake 各自还有 5% 概率单独 drop；control mode 不做 dropout。

---

### 三、模型训练与推理流程

π0.7 的模型本体继承 π0.6 与 MEM 的路线：VLM backbone 初始化自 [Gemma3 4B VLM](https://zhida.zhihu.com/search?content_id=273271213&content_type=Article&match_order=1&q=Gemma3+4B+VLM&zhida_source=entity) ，使用 MEM-style video history encoder，并接一个约 860M 参数的 action expert。动作专家使用 Flow Matching 预测连续动作，固定处理 50 个 action tokens，对应 50-step action chunk。论文还使用 training-time real-time action chunking（RTC）来模拟推理延迟：训练时模拟 0 到 12 个 timestep 的延迟，对 50Hz 机器人相当于最多 240ms。

在 subgoal images 训练上，π0.7 同时使用真实未来图像和 world model 生成图像。真实图像的 timestep 采样规则是：以 0.25 概率采样 segment 末尾图像（与 world model 目标一致），以 0.75 概率从当前时刻之后 0-4 秒内均匀采样未来图像。此外，作者还用 world model 生成大量 subgoal images，替换真实未来图像构造训练样本，以缓解 train-test mismatch。

![](https://pic4.zhimg.com/v2-41e68eda4d067008d33d0e6ac1e9ead5_1440w.jpg)

运行时，π0.7 根据任务配置不同上下文，不做任务特定后训练。模型总是使用 control mode 和 episode metadata；子任务指令 来自高层语言策略或 human coaching；如果使用 subgoal images，则在语义意图变化或距离上次生成超过 秒时刷新。动作生成使用 5 个 denoising steps 产生 50-step action chunk，并执行其中 步。

论文给出的运行时伪代码如下（Algorithm 1）：

![](https://pic4.zhimg.com/v2-1d8139fcfaa2dd3bd995026235ee87a3_1440w.jpg)

这段伪代码里每个变量的作用是：

- 是初始观测， 是时刻 的观测。
- 是整体任务指令， 是当前子任务指令。
- 是 episode metadata， 是 control mode。
- 是 world model 采样得到的 subgoal image。
- 是最终给 VLA 的上下文。
- 是模型生成的未来动作 chunk， 是当前实际执行动作。
- 秒控制 subgoal 刷新间隔， 控制每次 chunk 推理后实际执行多少步再重新推理。
- “Non-blocking (async)” 表示 world model 生成 subgoal images 不阻塞 VLA 控制循环；VLA 始终使用当前可用的最新 subgoal。

由于 prompt 组件在训练中带 dropout，π0.7 还可以对 prompt 的某些部分使用 classifier-free guidance（CFG）。论文给出的动作 denoising 引导方向为：

其中：

- 表示对动作 的 log-policy 梯度，在 denoising 过程中作为引导方向。
- 是完整上下文。
- 是“unconditional”模式下的上下文，即被 drop 掉对应 prompt 组件后的条件。
- 是 CFG 权重。论文使用中等强度的 。

直观上，这个公式把“完整 prompt 下的动作趋势”与“缺少特定 prompt 条件时的动作趋势”做差，再用 放大差异，从而强化 prompt 中指定的行为模式。论文实际把 CFG 应用在 episode metadata 上，用于在灵巧任务中诱导高质量、高速度、无错误的行为。

附录 B 进一步说明了注意力模式。没有 image goals 时，π0.7 使用与 π0.5 类似的注意力：memory-aware image views 的 embeddings 之间全局双向注意力；训练期 FAST tokens 与 flow actions 彼此不互相 attend。加入 image goals 时，image goals 作为额外 block-causal bidirectional block 放在 text prompt 之后。推理期做 CFG 时，正样本与负样本被打包进同一序列，形成一个双分支 attention tree，两条分支互不 attend，以提高推理效率。

![](https://pic1.zhimg.com/v2-bf99e6c14ff923db9c3057679efaeee8_1440w.jpg)

附录 C 说明了 world model 的训练细节。world model 初始化自 BAGEL，训练数据包括机器人数据、带高质量分段语言标签的 egocentric human video、若干开源 image editing 数据集和 open-source video 数据集。每个训练样本包含子任务指令 、3 个当前相机输入 、3 个目标图像 ，其中 是 segment 的最后时间步。相机输入同时经过 ViT 与 VAE：ViT 负责语义理解，VAE 负责细粒度图像细节；ViT 输入 resize 到 ，VAE 输入（包括目标图像）resize 到 。

---

### 四、实验结果与附录细节

### 直接上手灵巧任务：generalist 能不能逼近 specialist

π0.7 首先被放到一组复杂灵巧任务上，问题很直接：一个通用模型能否不经过任务特定后训练，接近甚至匹配专用 specialist。Fig. 6 的任务包括 espresso making、box building、laundry folding，以及 peanut butter sandwich、shirt inside-out、drive through door、slice zucchini、peel fruits and vegetables、take out trash 等。

![](https://pic3.zhimg.com/v2-1fa2e5b3493fdfb1d398fe594531f10c_1440w.jpg)

论文的结论是：同一个 π0.7 generalist 可以匹配 π\*\_0.6 或 π0.6 的任务专用 post-trained specialist，并且在 diverse laundry folding 和 box building 上 throughput 更高。这里的 throughput 被定义为 successes per hour，并在图中相对 specialist 做 normalized reporting。这个结果的重点不是 π0.7 在所有数字上都“压倒”specialist，而是它没有为每个任务单独微调，却能达到接近专用策略的水平。

Fig. 7 的 ablation 说明 prompt 与训练数据来源本身是关键变量。π0.7（no metadata）去掉 episode metadata，π0.7（no eval data）不使用 autonomous evaluation episodes。完整 π0.7 在所有任务上都优于这两个 ablation，差距在 throughput 上尤其明显。论文据此认为，autonomous evaluation data 与 metadata 的组合，是从 mixed-quality 数据中蒸馏出强行为的重要原因。

![](https://picx.zhimg.com/v2-d2400174809f3e089e02f23d625e4815_1440w.jpg)

论文还单独评测了需要显式记忆的任务，例如 Swap 3 Mugs、Find Object、Scoop Coffee、Window Cleaning。π0.7 在不做任务特定微调的情况下，达到与 MEM 论文中带 memory 的 π0.6 微调 specialist 相近或更好的表现（Fig. 8）。这说明 π0.7 继承 MEM-style history encoder 后，不只是能做瞬时视觉反应，也能利用历史上下文完成需要记忆的操作。

![](https://pic3.zhimg.com/v2-c7b46cb3e4c01653cddfdce946f67d54_1440w.jpg)

### 指令跟随：复杂指代与反数据偏置

π0.7 在 4 个未见厨房和 2 个未见卧室中评测 14 个 instruction following scenarios。每个场景包含 3-6 步开放式指令，指标是所有评测中被正确执行的指令比例。Fig. 9 显示 π0.7 相比 π0.5/π0.6 有明显提升。

![](https://pic2.zhimg.com/v2-6c76ad253a63ab9c44001cca84456b4b_1440w.jpg)

更有意思的是 Fig. 10/11。Fig. 10 设计了复杂指代指令，例如“pick up the object I would use to eat soup”或“pick up the fruit on the largest plate”。π0.7 在复杂指令上优于旧模型，而使用 generated subgoal images 的 π0.7(GC) 进一步提升。

Fig. 11 则是反数据偏置任务：例如数据里通常是把垃圾放进 trash bin、盘子放进 bussing bin，而测试时要求反过来。π0.7 能更好地遵循指令而不是复制数据里的常见模式；在 Reverse Fridge to Microwave 上，π0.7(GC) 的 subgoal images 对成功非常关键。

![](https://pic3.zhimg.com/v2-944fbf1c972a7e8d1a5febfde32fb576_1440w.jpg)

![](https://pic2.zhimg.com/v2-6288107d1715e156fa2a83923485702b_1440w.jpg)

### 跨 embodiment：不是复刻源机器人动作，而是换策略

跨 embodiment 迁移是 π0.7 最值得关注的部分。论文先测试较简单的 object rearrangement/repositioning 任务，例如 Table Setting、Bag In Backpack、Organize Tupperware、Shirt Bagging。对于 Table Setting，数据来自多种机器人，所有模型都有较强迁移；当 embodiment gap 变大时，π0.5 明显退化，π0.6 和 π0.7 仍能保持较强表现；在 Shirt Bagging 这种从小型静态双臂迁移到单臂 UR5e 的任务上，π0.7 明显超过 prior models。

更关键的是，π0.7 不是简单复刻源机器人上的动作。Fig. 13 给出两个例子：源机器人做 bag insertion 时需要一只手撑开袋子，UR5e 因为 reach 更大，π0.7 采用单臂 pick-and-place；源机器人折衬衫时常用倾斜 end-effector 压住布料，而 UR5e 上 π0.7 生成更适合其运动学的垂直抓取。这个现象说明 cross-embodiment transfer 不是动作轨迹拷贝，而是策略重组。

![](https://pic2.zhimg.com/v2-9fac78e2fee022e209621faf4051f609_1440w.jpg)

![](https://picx.zhimg.com/v2-b5db6af8a4ea0b97a22df10739c1f205_1440w.jpg)

附录 F 的人类实验给了一个更强对照。作者招募 10 名经验排名前 2% 的 teleoperators，平均有约 375 小时跨机器人遥操作经验，但没有在 UR5e 上执行过 shirt folding。每人 3 次，共 30 次，无 warm-up。结果是：人类专家平均 90.9% task progress / 80.6% success；π0.7(GC) 为 85.6% task progress / 80% success。这个结果不能说明 π0.7 超过人类，但足以说明它在这个 zero-shot transfer setting 下已经接近强人类基线。

![](https://pic3.zhimg.com/v2-f1a7556dadb25536e59686c50b71583c_1440w.jpg)

### 组合泛化与 coaching：短程任务能直接做，长程任务需要语言分解

![](https://pic4.zhimg.com/v2-a73d02ae7141c1a58b5919e9a513bf1f_1440w.jpg)

论文把 compositional task generalization 分成两类。短程新任务，如 press French press plunger、scoop rice into rice cooker、wipe office supplies、spin articulated objects，π0.7 可以直接通过语言或 generated image goals 完成（Fig. 17），这些任务没有收集专门 robot data。

![](https://pic1.zhimg.com/v2-824e82ba9142cc8d6457d74c84f86cfa_1440w.jpg)

更长程的完全未见任务，如 Loading an Air Fryer、Unloading an Air Fryer、Toasting a Bagel，直接一句话要求模型完成并不够。论文采用 language coaching：人类像教人一样逐步给出“grasp the handle of the air fryer with the left hand”“open the air fryer with the left hand”等指令。π0.7 能通过这些步骤完成任务，而 prior models 往往因为语言跟随能力不足表现很差；加入 generated subgoal images 后，π0.7(GC) 更强（Fig. 15）。

![](https://pic4.zhimg.com/v2-61c11a171c2e4d4e7cd99a6c66619d6d_1440w.jpg)

这些 coaching episodes 还可以训练一个高层语言策略，让它自动给 π0.7 发子任务 prompt。Fig. 16 显示，π0.7（autonomous）在多个未见长程任务上能接近 π0.7（coaching）的表现，不需要额外 teleoperation 或低层动作数据。

![](https://pic4.zhimg.com/v2-689bbdc90bad68b1934545ddc4f52599_1440w.jpg)

### 数据 scaling：metadata 决定“更多混合数据”是增益还是噪声

Fig. 18 是本文最有方法论价值的 ablation。作者以 Laundry（T-Shirts and Shorts）为例，按 fold quality 和 speed 把数据分成 top 30%、top 50%、top 80%、all data 四档，然后分别训练带 metadata 和不带 metadata 的 π0.7。结果显示，不带 metadata 的模型在加入更多混合质量数据时可能变差；带 metadata 的模型则能随着数据变大持续提升，即使平均数据质量下降。

右图进一步控制数据量：去掉 task diversity 最高的 20% 数据，与随机去掉 20% 数据比较。π0.7 和随机去掉 20% 的版本显著好于去掉最高多样性 20% 的版本，说明任务多样性本身确实被模型转化成了未见短程任务上的泛化能力。

![](https://pica.zhimg.com/v2-3624b92e540f285e134b73696905240e_1440w.jpg)

### 推理代价：VLA 快，world model 贵，但异步可缓解

π0.7 和高层策略都基于 Gemma3 4B，VLA 推理使用单张 NVIDIA H100。经过 RTC 之后的优化，最小 π0.7 variant 在 3 个相机输入、5 个 denoising steps、training-time RTC 下可达到 38ms；启用 MEM vision encoder 并加入 subgoal images 时，最坏情况下推理时间为 127ms。  
  
subgoal image 生成更慢：world model 是 14B 级别，序列长度接近 10,000 tokens；作者使用 4-way tensor parallelism（4xH100）、8-bit 大矩阵乘法量化和修改版 SageAttention，25 个 denoising steps（每步包含 text 和 image CFG）生成 subgoal images 需要 1.25 秒。运行时采用异步策略，π0.7 在 world model 生成下一组 subgoal 时继续执行当前动作。

---

### 总结

当数据来源越来越杂、质量越来越不一致、embodiment 越来越多时，模型需要的不只是更多数据，而是能解释这些数据的上下文。π0.7 用 subtask instruction、subgoal images、episode metadata 和 control mode 共同构成 prompt，本质上是在给每条数据加上“该如何被学习”的标签。这也是对PI之前博客中提到的Moravec's Paradox的回应——当现有的文本数据无法很好地指导物理动作，我们要如何做才能让模型学会行动？  
  
π0.7 将 world model 用作 subgoal image provider，让视觉未来目标成为 prompt 的一部分，以此来让VLA更好地学习行动。使用world model正在变成越来越多人的共识（无论是WAM的方式还是其他VLA + world model的方法），如今作为行业龙头的PI也拥抱了世界模型，想必在后续会涌现出更多关于具身世界模型的工作。

---

补充一下，pi07的工作和之前的一篇工作VISTA有点撞车了，笔者有幸和VISTA的作者交流了关于目前VLA的一些看法。感兴趣的朋友可以关注这篇文章，链接中同样有对于VISTA的解读。

编辑于 2026-04-23 12:12・北京[具身智能](https://www.zhihu.com/topic/21535379)[云服务器 38元/年起](https://click.aliyun.com/m/1000413641/?spu=biz%3D0%26ci%3D3741608%26si%3Db9eb868c-d9e9-4594-bf03-2259c4d2a9c3%26ts%3D1780994833%26zid%3D1629)

[

AI 加速季，智惠生产力，QoderWork CN 首月 0 元，Qwen3.7 限时 5 折，秒悟新注送 1 万积分，加入 OPC 赢百万助...

](https://click.aliyun.com/m/1000413641/?spu=biz%3D0%26ci%3D3741608%26si%3Db9eb868c-d9e9-4594-bf03-2259c4d2a9c3%26ts%3D1780994833%26zid%3D1629)