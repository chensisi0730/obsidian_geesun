[收录于 · 大模型和Agent](https://www.zhihu.com/column/c_1893457013951922494)

88 人赞同了该文章

## 引言

[LeRobot](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=LeRobot&zhida_source=entity) 是由 [Hugging Face](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=Hugging+Face&zhida_source=entity) 开发的一个基于 [PyTorch](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=PyTorch&zhida_source=entity) 的实时机器人学习框架，旨在降低机器人技术的入门门槛，让更多人能够参与并从共享数据集和预训练模型中受益。本文将深入剖析LeRobot框架的架构、支持的策略、仿真平台和硬件、数据采集和训练过程，以及数据格式，帮助读者快速入门这一强大的机器人学习工具。

## 1\. LeRobot框架架构

### 1.1 框架概述

LeRobot是一个专注于实际机器人应用的机器人学习框架，它提供了一系列预训练模型、数据集和工具，特别关注模仿学习和强化学习方法。框架的目标是降低机器人技术的入门门槛，使研究人员和开发者能够更容易地开发和部署机器人应用。 (README.md:55-59)

### 1.2 系统架构

LeRobot的系统架构由几个相互连接的子系统组成，这些子系统协同工作，支持机器人学习。核心基础设施支持机器人学习算法的策略系统、训练数据的数据集管理、模拟的环境接口以及物理硬件交互的机器人控制。示例目录展示了这些系统如何一起使用。

![](https://pic1.zhimg.com/v2-de08aa3fc4d9ab9ec0be3dbbec80f372_1440w.jpg)

lerobot系统架构

### 1.3 代码架构

LeRobot的代码结构清晰，便于开发者理解和扩展：

```
.
├── examples             # 示例和教程，从这里开始学习LeRobot
│   └── advanced         # 包含更高级的示例
├── lerobot
│   ├── configs          # 包含可以在命令行中覆盖的所有选项的配置类
│   ├── common           # 包含核心功能类和工具
│   │   ├── datasets     # 各种人类演示数据集：aloha, pusht, xarm
│   │   ├── envs         # 各种模拟环境：aloha, pusht, xarm
│   │   ├── policies     # 各种策略实现：act, diffusion, tdmpc等
│   │   ├── robot_devices # 硬件接口：dynamixel电机，opencv相机，koch机器人
│   │   └── utils        # 各种工具函数
│   └── scripts          # 包含通过命令行执行的函数
│       ├── eval.py      # 加载策略并在环境中评估
│       ├── train.py     # 通过模仿学习和/或强化学习训练策略
│       ├── control_robot.py # 远程操作真实机器人，记录数据，运行策略
│       ├── push_dataset_to_hub.py # 将数据集转换为LeRobot数据集格式并上传到Hugging Face hub
│       └── visualize_dataset.py # 加载数据集并渲染其演示
├── outputs              # 包含脚本执行结果：日志、视频、模型检查点
└── tests                # 包含用于持续集成的pytest工具
```

(README.md:142-159)

这种结构使得开发者可以轻松地找到所需的组件，并理解它们之间的关系。

### 1.4 主要组件

### 1.4.1 策略系统

LeRobot通过统一的工厂接口实现多种最先进的策略架构。 `make_policy()` 工厂函数为创建不同类型的策略提供了统一的接口。训练（ `train.py` ）和评估（ `eval.py` ）脚本通过这个工厂与策略交互。所有策略都实现了 `PreTrainedPolicy` 接口，其中包括动作选择和模型训练的方法。

lerobot策略系统

### 1.4.2 数据集管理

数据集系统处理机器人数据集的加载、处理和可视化，重点关注多模态数据。 `LeRobotDataset` 类是核心组件，提供了加载和管理机器人数据集的功能。数据集可以从Hugging Face Hub或本地存储访问。 `make_dataset()` 工厂根据配置创建数据集实例。该系统处理片段、视频处理和图像转换，通过Rerun或HTML/Flask接口进行可视化。

数据集管理

### 1.4.3 环境接口

LeRobot基于 [Gymnasium API](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=Gymnasium+API&zhida_source=entity) 为多个模拟环境提供了一致的接口。 `make_env()` 工厂根据配置创建不同的环境类型（Aloha、PushT、 [XArm](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=XArm&zhida_source=entity) ）。该系统包括用于预处理观察（ `preprocess_observation()` ）和将环境特征转换为策略特征（ `env_to_policy_features()` ）的工具。工厂生成Gymnasium向量环境，实现并行模拟。

![](https://pica.zhimg.com/v2-0397fbca554e3ed14c03ea60f7dcbe78_1440w.jpg)

环境配置

### 1.4.4 机器人控制

机器人控制系统提供了与物理机器人交互的接口，支持不同的控制模式和硬件类型。 `control_robot.py` 脚本是机器人控制系统的核心组件，支持不同的控制模式（远程操作、记录、回放、校准）和机器人类型（操作器、移动）。该系统通过电机（Dynamixel、Feetech）和相机（OpenCV、RealSense）的抽象与硬件接口。记录模式与 `LeRobotDataset` 系统集成，用于数据收集。

![](https://pic2.zhimg.com/v2-0f3bdcb1791be5b2d5a434ab606130ab_1440w.jpg)

robot control模块

## 2\. LeRobot支持的策略

LeRobot实现了几种最先进的策略架构，以下是对这些策略的详细介绍：

| 策略类型 | 描述 | 使用场景 | 优点 | 缺点 |
| --- | --- | --- | --- | --- |
| ACT (Action Chunking Transformer) | 基于Transformer的动作分块策略，专为双手操作设计 | 需要精确协调的双手操作任务，如组装、操作复杂物体 | 能够学习长期依赖关系，处理复杂序列任务，对时间步长不敏感 | 训练成本高，需要大量数据，推理速度可能较慢 |
| Diffusion (Denoising Diffusion) | 基于扩散模型的视觉运动控制策略 | 需要精确控制的视觉引导任务，如精确抓取和放置 | 生成高质量、多样化的动作，对不确定性有良好建模 | 推理速度较慢，训练过程复杂 |
| TDMPC (Temporal Difference M）PC) | 时间差分模型预测控制 | 需要预测性控制的任务，如动态环境中的导航和操作 | 结合了模型预测控制的规划能力和强化学习的自适应性 | 对模型精度要求高，计算成本较大 |
| VQBeT (Vector Quantized Behavior) | 向量量化行为Transformer | 需要从多样化演示中学习的任务，如多模态行为学习 | 能够从多样化数据中提取离散行为原语，泛化能力强 | 离散表示可能限制某些连续控制任务的精度 |
| PI0 (Vision-Language-Action) | 视觉-语言-动作策略 | 需要语言指导的任务，如遵循自然语言指令的机器人操作 | 能够理解和执行自然语言指令，多模态融合能力强 | 对语言理解的准确性依赖高，需要配对的语言-动作数据 |
| PI0FAST (Fast Action Tokenization) | 快速动作标记化策略 | 需要实时响应的语言引导任务 | 比PI0更快的推理速度，保持语言理解能力 | 可能在复杂指令上精度略低于PI0 |

这些策略可以通过统一的 `make_policy()` 工厂函数创建，使得在不同任务和环境中切换策略变得简单。

## 3\. LeRobot支持的仿真平台和硬件

### 3.1 支持的仿真环境

LeRobot通过Gymnasium接口支持多个模拟环境：

| 环境 | 描述 | 特点 |
| --- | --- | --- |
| Aloha | 双手机器人操作任务 | 专注于双手协调操作，如倒咖啡、开瓶盖等 |
| PushT | 物体推动操作任务 | 专注于推动物体到目标位置的任务 |
| XArm | XArm机器人操作任务 | 基于现实世界XArm机器人的模拟环境 |

这些环境可以作为额外依赖项安装：

```bash
pip install -e ".[aloha, pusht]"
```

(README.md:122-129)

### 3.2 硬件支持

LeRobot支持多种物理机器人硬件，特别是专注于经济实惠且功能强大的机器人平台。以下是主要支持的硬件类型：

**(1)操作器机器人** ：

- [SO100](https://zhida.zhihu.com/search?content_id=257399840&content_type=Article&match_order=1&q=SO100&zhida_source=entity) （基于Koch设计的双臂机器人）
	- XArm（单臂机器人）

**(2)移动机器人** ：

- LeKiwi（移动机器人平台）

**(3)传感器和执行器** ：

- 相机：OpenCV兼容相机、Intel RealSense深度相机
	- 电机：Dynamixel伺服电机、Feetech伺服电机

### 3.3 SO100机器人案例分析

SO100是LeRobot框架中重点支持的一种双臂机器人，基于Koch设计。它是一个经济实惠的双臂机器人平台，特别适合研究和教育用途。

### 3.3.1 SO100硬件架构

SO100机器人由以下主要组件组成：

- 两个机械臂，每个臂有多个自由度
- Dynamixel伺服电机作为关节驱动
- 一个或多个相机用于视觉感知
- 控制电路和电源系统

### 3.3.2 SO100控制流程

控制SO100机器人的典型流程如下：

## 4\. 使用LeRobot进行遥操作和数据采集

### 4.1 遥操作模式

遥操作模式允许用户直接手动控制机器人。这种模式对于测试机器人硬件、实验动作或准备记录会话非常有用。

### 4.1.1 遥操作流程

(参考 control\_robot.py:233-242)

### 4.1.2 遥操作配置

遥操作可以通过 `TeleoperateControlConfig` 类进行配置，该类提供以下选项：

| 参数 | 类型 | 描述 |
| --- | --- | --- |
| fps | int或None | 限制最大帧率。默认无限制。 |
| teleop\_time\_s | float或None | 遥操作持续时间。默认无限。 |
| display\_data | bool | 是否显示相机馈送和数据可视化。 |

### 4.1.3 遥操作命令示例

基本的无限频率遥操作：

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=so100 \
    --control.type=teleoperate
```

(control\_robot.py:29-39)

限制频率的遥操作，模拟记录：

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=so100 \
    --control.type=teleoperate \
    --control.fps=30
```

(control\_robot.py:42-48)

### 4.2 数据记录模式

记录模式将机器人的观察和动作捕获到结构化数据集中。这种模式对于收集用于机器人学习策略的训练数据至关重要。

### 4.2.1 记录流程

(参考 control\_robot.py:246-340)

![](https://pica.zhimg.com/v2-b186748cb6bb1c82d7f0d454d9a782f8_1440w.jpg)

### 4.2.2 使用策略记录

记录可以使用预训练策略控制机器人进行，这对于评估策略性能很有用。使用策略记录时，遥操作被禁用，策略根据观察生成动作。 (control\_utils.py:213-288)

### 4.2.3 记录配置

记录通过 `RecordControlConfig` 类配置，具有以下关键参数：

| 参数 | 类型 | 描述 |
| --- | --- | --- |
| repo\_id | str | 数据集标识符（例如，'username/dataset\_name'） |
| single\_task | str | 记录期间执行的任务描述 |
| fps | int或None | 记录的帧率 |
| warmup\_time\_s | int或float | 开始数据收集前的预热秒数 |
| episode\_time\_s | int或float | 每个片段的数据记录秒数 |
| reset\_time\_s | int或float | 每个片段后重置环境的秒数 |
| num\_episodes | int | 要记录的片段数量 |
| video | bool | 是否将帧编码为数据集中的视频 |
| push\_to\_hub | bool | 是否将数据集上传到Hugging Face Hub |
| policy | PreTrainedConfig或None | 用于评估记录的可选策略配置 |
| resume | bool | 是否在现有数据集上继续记录 |
| num\_image\_writer\_processes | int | 处理帧保存为PNG的子进程数 |
| num\_image\_writer\_threads\_per\_camera | int | 每个相机写入PNG图像的线程数 |

### 4.2.4 记录命令示例

记录单个测试片段：

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=so100 \
    --control.type=record \
    --control.fps=30 \
    --control.single_task="抓取乐高积木并将其放入箱中。" \
    --control.repo_id=username/test_dataset \
    --control.num_episodes=1 \
    --control.push_to_hub=True
```

(control\_robot.py:50-59)

记录用于训练的完整数据集：

```bash
python lerobot/scripts/control_robot.py \
    --robot.type=so100 \
    --control.type=record \
    --control.fps=30 \
    --control.repo_id=username/training_dataset \
    --control.num_episodes=50 \
    --control.warmup_time_s=2 \
    --control.episode_time_s=30 \
    --control.reset_time_s=10
```

(control\_robot.py:82-91)

### 4.3 模拟环境中的数据采集

除了在实际机器人上收集数据外，LeRobot还支持在模拟环境中收集数据，这对于初始开发和测试非常有用。

```bash
python lerobot/scripts/control_sim_robot.py record \
    --robot-path lerobot/configs/robot/your_robot_config.yaml \
    --sim-config lerobot/configs/env/your_sim_config.yaml \
    --fps 30 \
    --repo-id $USER/robot_sim_test \
    --num-episodes 50 \
    --episode-time-s 30
```

(control\_sim\_robot.py:65-72)

## 5\. LeRobot数据格式

### 5.1 LeRobotDataset格式概述

LeRobot使用一种称为 `LeRobotDataset` 的统一数据格式，它设计用于存储和管理机器人学习所需的多模态数据。这种格式可以轻松地从Hugging Face Hub或本地文件夹加载。

![](https://pic1.zhimg.com/v2-8883481dd22df88eedbef90226ffc78c_1440w.jpg)

lerobot数据格式

### 5.2 数据集结构

`LeRobotDataset` 的文件结构组织如下：

```
.
├── data
│   ├── chunk-000
│   │   ├── episode_000000.parquet
│   │   ├── episode_000001.parquet
│   │   └── ...
│   ├── chunk-001
│   │   ├── episode_001000.parquet
│   │   └── ...
│   └── ...
├── meta
│   ├── episodes.jsonl
│   ├── info.json
│   ├── stats.json
│   └── tasks.jsonl
└── videos
    ├── chunk-000
    │   ├── observation.images.camera1  # 'camera1' 是示例名称
    │   │   ├── episode_000000.mp4
    │   │   └── ...
    │   ├── observation.images.camera2
    │   │   └── ...
    ├── chunk-001
    └── ...
```

(lerobot\_dataset.py:400-438)

### 5.3 数据集组件

`LeRobotDataset` 包含以下主要组件：

**(1)HF Dataset** ：基于Arrow/Parquet的数据集，包含以下特征：

- 观察图像（VideoFrame格式或路径）
	- 状态观察（例如关节位置）
	- 动作（例如关节目标位置）
	- 片段索引和帧索引
	- 时间戳
	- 任务描述

**(2)元数据 (meta)** ：

- `episodes.jsonl`: 包含每个片段的元信息，如起始/结束帧索引、在哪个chunk等。
	- `info.json`: 数据集的整体信息，包括特征、fps、机器人信息等。
	- `stats.json`: 数据集中每个数值特征的统计信息（最大值、均值、最小值、标准差）。
	- `tasks.jsonl`: 如果数据集包含多个任务，这里会列出任务描述和对应的片段。

**(3)视频 (videos)** (可选)：如果 `info.json` 中指定使用视频存储图像，则原始视频文件存储在此处，按chunk和相机名称组织。

## 结论

LeRobot框架通过其模块化的设计、丰富的预训练模型和数据集支持，以及对真实世界机器人应用的关注，显著降低了机器人学习的门槛。无论是进行学术研究还是开发实际应用，LeRobot都提供了一个强大而灵活的平台。希望本篇深度剖析能帮助您快速上手并有效利用LeRobot进行机器人学习的探索与创新。

还没有人送礼物，鼓励一下作者吧

发布于 2025-05-07 00:31・湖北[Kamvas 22(Gen 3)数位屏新品上市，90Hz高刷加持](https://store.huion.cn/product/shuweiping/Kamvas-22-Gen-3?spu=biz%3D0%26ci%3D3692314%26si%3D9d429c4a-f23a-4b47-8bd4-1a483f700474%26ts%3D1777712681%26zid%3D1629)

[

21.5英寸屏幕，兼具大尺寸显示与高分辨率，拥有90Hz刷新率，画面通透沉浸的同时绘画更流畅；五种色彩模式辅以△...

](https://store.huion.cn/product/shuweiping/Kamvas-22-Gen-3?spu=biz%3D0%26ci%3D3692314%26si%3D9d429c4a-f23a-4b47-8bd4-1a483f700474%26ts%3D1777712681%26zid%3D1629)