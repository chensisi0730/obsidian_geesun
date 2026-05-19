开始使用

教程

数据集

策略

奖励模型

[SARM](https://hugging-face.cn/docs/lerobot/sarm)

推理

模拟

机器人处理器

机器人

遥操作员

[电话](https://hugging-face.cn/docs/lerobot/phone_teleop)

支持的硬件

[PyTorch 加速器](https://hugging-face.cn/docs/lerobot/torch_accelerators)

资源

关于

加入 Hugging Face 社区

并获得增强的文档体验

在模型、数据集和 Spaces 上进行协作

通过加速推理获得更快的示例

切换文档主题

开始使用

## 在真实机器人上进行模仿学习

本教程将讲解如何训练神经网络来自主控制真实机器人。

**你将学到：**

1. 如何记录和可视化你的数据集。
2. 如何使用你的数据训练策略并为评估做准备。
3. 如何评估你的策略并可视化结果。

通过遵循这些步骤，你将能够以高成功率复现任务，例如抓取乐高积木并将其放入箱子中，正如下面的视频所示。

本教程不针对特定机器人：我们将为你提供适用于任何受支持平台的命令和 API 代码片段。

在数据收集期间，你将使用“遥操作”设备，例如主臂或键盘来遥操作机器人并记录其运动轨迹。

一旦收集了足够的轨迹，你将训练一个神经网络来模仿这些轨迹，并部署训练好的模型，使你的机器人能够自主执行任务。

如果你在任何环节遇到问题，请加入我们的 [Discord 社区](https://discord.com/invite/s3KuuzsPFb) 寻求帮助。

## 设置和校准

如果你还没有设置和校准你的机器人和遥操作设备，请按照机器人特定的教程进行操作。

## 遥操作

在本例中，我们将演示如何遥操作 SO101 机器人。对于每个命令，我们还提供相应的 API 示例。

请注意，与机器人关联的 `id` 用于存储校准文件。在使用相同的设置进行遥操作、记录和评估时，使用相同的 `id` 非常重要。

命令

API 示例

```
lerobot-teleoperate \
   --robot.type=so101_follower \
   --robot.port=/dev/tty.usbmodem58760431541 \
   --robot.id=my_awesome_follower_arm \
   --teleop.type=so101_leader \
   --teleop.port=/dev/tty.usbmodem58760431551 \
   --teleop.id=my_awesome_leader_arm
```

遥操作命令将自动

1. 识别任何缺失的校准并启动校准程序。
2. 连接机器人和遥操作设备并开始遥操作。

## 摄像头

要将摄像头添加到你的设置中，请遵循此 [指南](https://hugging-face.cn/docs/lerobot/cameras#setup-cameras) 。

## 带摄像头遥操作

使用 `rerun` ，你可以再次进行遥操作，同时可视化摄像头馈送和关节位置。在此示例中，我们使用的是 Koch 机械臂。

命令

API 示例

```
lerobot-teleoperate \
    --robot.type=koch_follower \
    --robot.port=/dev/tty.usbmodem58760431541 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 1920, height: 1080, fps: 30}}" \
    --teleop.type=koch_leader \
    --teleop.port=/dev/tty.usbmodem58760431551 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

## 录制数据集

一旦您熟悉了远程操作，您就可以记录您的第一个数据集。

我们使用 Hugging Face Hub 功能来上传你的数据集。如果你之前没有使用过 Hub，请确保你可以使用具有写入权限的令牌通过 CLI 登录，该令牌可以在 [Hugging Face 设置](https://hugging-face.cn/settings/tokens) 中生成。

通过运行此命令将您的令牌添加到 CLI

```
huggingface-cli login --token ${HUGGINGFACE_TOKEN} --add-to-git-credential
```

然后将您的 Hugging Face 存储库名称存储在一个变量中

```
HF_USER=$(hf auth whoami | head -n 1)
echo $HF_USER
```

现在你可以记录一个数据集了。要记录 5 个回合并将你的数据集上传到 Hub，请为你的机器人调整下面的代码并执行命令或 API 示例。

命令

API 示例

```
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem585A0076841 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ front: {type: opencv, index_or_path: 0, width: 1920, height: 1080, fps: 30}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/tty.usbmodem58760431551 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true \
    --dataset.repo_id=${HF_USER}/record-test \
    --dataset.num_episodes=5 \
    --dataset.single_task="Grab the black cube"
```

#### 数据集上传

在本地，你的数据集存储在以下文件夹： `~/.cache/huggingface/lerobot/{repo-id}` 。数据记录结束后，你的数据集将上传到你的 Hugging Face 页面（例如： `https://hugging-face.cn/datasets/${HF_USER}/so101_test` ），你可以通过运行以下命令获取：

```
echo https://hugging-face.cn/datasets/${HF_USER}/so101_test
```

您的数据集将自动标记为 `LeRobot` ，以便社区轻松找到它，您还可以添加自定义标签（例如，在此示例中为 `tutorial` ）。

您可以通过搜索 `LeRobot` [标签](https://hugging-face.cn/datasets?other=LeRobot) 在 Hub 上查找其他 LeRobot 数据集。

你也可以手动将本地数据集推送到 Hub，运行：

```
huggingface-cli upload ${HF_USER}/record-test ~/.cache/huggingface/lerobot/{repo-id} --repo-type dataset
```

#### 记录函数

`record` 函数提供了一套工具，用于在机器人操作期间捕获和管理数据

##### 1\. 数据存储

- 数据使用 `LeRobotDataset` 格式存储，并在记录期间保存在磁盘上。
- 默认情况下，数据集将在记录后推送到你的 Hugging Face 页面。
	- 要禁用上传，请使用 `--dataset.push_to_hub=False` 。

##### 2\. 检查点和恢复

- 检查点在记录过程中自动创建。
- 如果发生问题，你可以通过重新运行相同的命令并添加 `--resume=true` 来恢复。恢复记录时， `--dataset.num_episodes` 必须设置为 **要记录的额外回合数** ，而不是数据集的目标总回合数！
- 要从头开始记录，请 **手动删除** 数据集目录。

##### 3\. 记录参数

使用命令行参数设置数据记录的流程

- `--dataset.episode_time_s=60` 每个数据记录回合的时长（默认： **60 秒** ）。
- `--dataset.reset_time_s=60` 每个回合后重置环境的时长（默认： **60 秒** ）。
- `--dataset.num_episodes=50` 要记录的总回合数（默认： **50** ）。

##### 4\. 记录期间的键盘控制

使用键盘快捷键控制数据记录流程

- 按 **右箭头 (`→`)** ：提前结束当前回合或重置时间并进入下一回合。
- 按 **左箭头 (`←`)** ：取消当前回合并重新录制。
- 按 **Escape (`ESC`)** ：立即停止会话，编码视频并上传数据集。

#### 数据收集技巧

一旦你对数据记录感到满意，就可以创建一个更大的数据集进行训练。一个好的起始任务是在不同位置抓取物体并将其放入箱子。我们建议记录至少 50 个回合，每个位置 10 个回合。保持摄像头固定，并在整个录制过程中保持一致的抓取行为。还要确保你操作的物体在摄像头可见。一个好的经验法则是，你只需要看着摄像头图像就能自己完成任务。

在接下来的章节中，你将训练你的神经网络。在实现可靠的抓取性能后，你可以开始在数据收集期间引入更多变化，例如额外的抓取位置、不同的抓取技术以及改变摄像头位置。

避免过快地添加太多变化，因为它可能会阻碍您的结果。

如果您想深入了解这个重要主题，您可以查看我们撰写的关于什么是一个好的数据集的 [博客文章](https://hugging-face.cn/blog/lerobot-datasets#what-makes-a-good-dataset) 。

#### 故障排除：

- 在 Linux 上，如果左、右箭头键和 Escape 键在数据记录期间没有反应，请确保你已设置 `$DISPLAY` 环境变量。请参阅 [pynput 限制](https://pynput.readthedocs.io/en/latest/limitations.html#linux) 。

## 可视化数据集

如果你使用 `--control.push_to_hub=true` 将数据集上传到 Hub，你可以通过复制粘贴 Hub 提供的 repo id 来 [在线可视化你的数据集](https://hugging-face.cn/spaces/lerobot/visualize_dataset) ：

```
echo ${HF_USER}/so101_test
```

## 重放回合

一个有用的功能是 `replay` 函数，它允许你重放你记录的任何回合或来自任何可用数据集的回合。此函数可帮助你测试机器人动作的可重复性，并评估其在同一型号机器人之间的可转移性。

你可以使用以下命令或 API 示例在你的机器人上重放第一个回合：

命令

API 示例

```
lerobot-replay \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem58760431541 \
    --robot.id=my_awesome_follower_arm \
    --dataset.repo_id=${HF_USER}/record-test \
    --dataset.episode=0 # choose the episode you want to replay
```

你的机器人应该能够复制你记录的动作。例如，请查看 [此视频](https://x.com/RemiCadene/status/1793654950905680090) ，其中我们在来自 [Trossen Robotics](https://www.trossenrobotics.com/) 的 Aloha 机器人上使用了 `replay` 。

## 训练策略

要训练一个策略来控制你的机器人，请使用 [`lerobot-train`](https://github.com/huggingface/lerobot/blob/main/src/lerobot/scripts/lerobot_train.py) 脚本。需要几个参数。这是一个命令示例：

```
lerobot-train \
  --dataset.repo_id=${HF_USER}/so101_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=${HF_USER}/my_policy
```

让我们来解释一下这个命令：

1. 我们通过 `--dataset.repo_id=${HF_USER}/so101_test` 作为参数提供了数据集。
2. 我们通过 `policy.type=act` 提供了策略。这将加载来自 [`configuration_act.py`](https://github.com/huggingface/lerobot/blob/main/src/lerobot/policies/act/configuration_act.py) 的配置。重要的是，该策略将自动适应你的机器人（例如 `laptop` 和 `phone` ）在数据集中保存的电机状态、电机动作和摄像头的数量。
3. 我们提供了 `policy.device=cuda` ，因为我们是在 Nvidia GPU 上训练，但你也可以使用 `policy.device=mps` 在 Apple 芯片上进行训练。
4. 我们提供了 `wandb.enable=true` 以便使用 [Weights and Biases](https://docs.wandb.ai/quickstart) 来可视化训练图。这是可选的，但如果你使用它，请确保你已通过运行 `wandb login` 登录。

训练应花费数小时。你将在 `outputs/train/act_so101_test/checkpoints` 中找到检查点。

要从检查点恢复训练，下面是一个从 `act_so101_test` 策略的 `last` 检查点恢复的命令示例：

```
lerobot-train \
  --config_path=outputs/train/act_so101_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

如果你不想在训练后将模型推送到 Hub，请使用 `--policy.push_to_hub=false` 。

此外，你还可以提供额外的 `tags` 或为你的模型指定 `license` ，或通过添加 `--policy.private=true --policy.tags=\[ppo,rl\] --policy.license=mit` 使模型存储库成为 `private` 。

#### 使用 Google Colab 进行训练

如果你的本地计算机没有强大的 GPU，你可以利用 Google Colab 来训练你的模型，方法是遵循 [ACT 训练笔记本](https://hugging-face.cn/docs/lerobot/notebooks#training-act) 。

#### 上传策略检查点

训练完成后，使用以下命令上传最新的检查点：

```
huggingface-cli upload ${HF_USER}/act_so101_test \
  outputs/train/act_so101_test/checkpoints/last/pretrained_model
```

你也可以使用以下命令上传中间检查点：

```
CKPT=010000
huggingface-cli upload ${HF_USER}/act_so101_test${CKPT} \
  outputs/train/act_so101_test/checkpoints/${CKPT}/pretrained_model
```

## 运行推理并评估你的策略

你可以使用 [`lerobot-record`](https://github.com/huggingface/lerobot/blob/main/src/lerobot/scripts/lerobot_record.py) 脚本，以策略检查点作为输入，来运行推理并评估你的策略。例如，运行此命令或 API 示例来运行推理并记录 10 个评估回合：

命令

API 示例

```
lerobot-record  \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.cameras="{ up: {type: opencv, index_or_path: /dev/video10, width: 640, height: 480, fps: 30}, side: {type: intelrealsense, serial_number_or_name: 233522074606, width: 640, height: 480, fps: 30}}" \
  --robot.id=my_awesome_follower_arm \
  --display_data=false \
  --dataset.repo_id=${HF_USER}/eval_so100 \
  --dataset.single_task="Put lego brick into the transparent box" \
  # <- Teleop optional if you want to teleoperate in between episodes \
  # --teleop.type=so100_leader \
  # --teleop.port=/dev/ttyACM0 \
  # --teleop.id=my_awesome_leader_arm \
  --policy.path=${HF_USER}/my_policy
```

正如你所见，这与之前用于记录训练数据集的命令几乎相同。有两处变化：

1. 增加了一个 `--control.policy.path` 参数，它指示了你的策略检查点的路径（例如 `outputs/train/eval_act_so101_test/checkpoints/last/pretrained_model` ）。如果你将模型检查点上传到 Hub，你也可以使用模型存储库（例如 `${HF_USER}/act_so101_test` ）。
2. 数据集的名称以 `eval` 开头，以反映你正在运行推理（例如 `${HF_USER}/eval_act_so101_test` ）。
[在 GitHub 上更新](https://github.com/huggingface/lerobot/blob/main/docs/source/il_robots.mdx)