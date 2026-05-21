## 快速入门指南

本指南适用于那些迫不及待地想要动手操作并将涉及到构建自己项目时会遇到的最常见概念！这包括安装、运行 RL、查找环境、创建新项目等！

Isaac Lab 的强大功能源自几个关键特性，我们将在本指南中简要介绍。

1. **向量化**: 强化学习需要多次尝试一项任务。通过将环境向量化，Isaac Lab 可以加速此过程，即通过并行运行多个相同环境的副本来减少在更新模型权重之前收集数据所需的时间。大部分的代码库致力于定义需要通过此向量化系统触及的环境的部分。
2. **模块化设计**: Isaac Lab 被设计为模块化，这意味着您可以设计您的项目以具有可根据不同需求替换的各种组件。例如，假设您想训练支持特定子集机器人的策略。您可以通过编写一个控制器接口层来设计环境和任务，以形式化我们其中一种 Manager 类（在本例中是 `ActionManager` ）。大部分其余的代码库致力于定义您项目中需要被这个管理系统触及的部分。

要开始，我们首先会安装 Isaac Lab 并运行一个训练脚本。

## 快速安装指南

有许多方式可以 [安装](https://docs.robotsfan.com/isaaclab/source/setup/installation/index.html#isaaclab-installation-root) Isaac Lab，但为了本快速入门指南的目的，我们将跟随使用虚拟环境的 pip 安装路线。

要开始，我们首先定义我们的虚拟环境。

```bash
# create a virtual environment named env_isaaclab with python3.11 and pip
conda create -n env_isaaclab python=3.11
# activate the virtual environment
conda activate env_isaaclab
```

```bash
# create a virtual environment named env_isaaclab with python3.11 and pip
uv venv --python 3.11 --seed env_isaaclab
# activate the virtual environment
source env_isaaclab/bin/activate
```

```batch
# create a virtual environment named env_isaaclab with python3.11
uv venv --python 3.11 env_isaaclab
# activate the virtual environment
env_isaaclab\Scripts\activate
```

接下来，安装一个支持CUDA的PyTorch 2.7.0版本。

> ```bash
> pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
> ```

在安装 Isaac Sim 之前，我们需要确保 pip 已经更新。要更新 pip，请运行

```bash
pip install --upgrade pip
```

```batch
python -m pip install --upgrade pip
```

现在我们可以安装 Isaac Sim 包。

```
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
```

最后，我们可以安装 Isaac Lab。要开始，使用以下命令克隆存储库

```bash
git clone git@github.com:isaac-sim/IsaacLab.git
```

```bash
git clone https://github.com/isaac-sim/IsaacLab.git
```

安装现在只需要简单地导航到存储库，然后使用带有 `--install` 标志的根脚本进行调用！

```bash
./isaaclab.sh --install # or "./isaaclab.sh -i"
```

```bash
isaaclab.bat --install :: or "isaaclab.bat -i"
```

## 使用 Isaac Launchable 快速开始

对于初次学习 Isaac Lab 且没有足够本地计算资源的用户， [Isaac Launchable](https://github.com/isaac-sim/isaac-launchable) 项目是一种无需手动安装即可快速入门的方法。

通过此项目，用户可以纯粹从 Web 浏览器与 Isaac Sim 和 Isaac Lab 交互，一个标签页运行 Visual Studio Code 用于开发和命令执行，另一个标签页提供 Isaac Sim 的流式用户界面。

此方法使用 [NVIDIA Brev](https://brev.nvidia.com/) ，这是一个提供易于配置的按小时付费云计算的平台。Brev Launchables 是预配置的、优化的计算和软件环境。

要立即尝试，请点击下面的按钮。要了解更多关于如何使用此项目或如何创建您自己的 Launchable，请参阅项目仓库 [此处](https://github.com/isaac-sim/isaac-launchable) 。

## 启动训练

通过位于 `isaaclab/scripts/reinforcement_learning` 目录中的相应 `train.py` 和 `play.py` 脚本访问 Isaac Lab 的各个后端。调用这些脚本将需要一个 **任务名称** 和对应的 **入口点** 到 gymnasium API。例如

```bash
python scripts/reinforcement_learning/skrl/train.py --task=Isaac-Ant-v0
```

这将训练 mujoco 蚂蚁 "奔跑" 。您可以使用 `--help` 标志查看您可用的各种启动选项。请特别注意 `--num_envs` 选项和 `--headless` 标志，这两个在尝试开发和调试新环境时非常有用。在此级别指定的选项将自动覆盖代码中可能定义的任何配置等效项（只要这些定义是 `@configclass` 的一部分，请参阅下文）。

## 列出可用环境

上面， `Isaac-Ant-v0` 是任务名称， `skrl``是使用的 RL 框架。 ``Isaac-Ant-v0` 环境已经在 [Gymnasium API](https://gymnasium.farama.org/) 中注册，您可以通过调用 `list_envs.py` 脚本查看入口点是如何定义的，可以在 `isaaclab/scripts/environments/list_envs.py` 中找到。您应该会看到如下条目

```bash
$> python scripts/environments/list_envs.py

+--------------------------------------------------------------------------------------------------------------------------------------------+
|  Available Environments in Isaac Lab
+--------+----------------------+--------------------------------------------+---------------------------------------------------------------+
| S. No. | Task Name            | Entry Point                                | Config
.
.
.
+--------+----------------------+--------------------------------------------+---------------------------------------------------------------+
|   2    | Isaac-Ant-Direct-v0  |  isaaclab_tasks.direct.ant.ant_env:AntEnv  |  isaaclab_tasks.direct.ant.ant_env:AntEnvCfg
+--------+----------------------+--------------------------------------------+---------------------------------------------------------------+
.
.
.
+--------+----------------------+--------------------------------------------+---------------------------------------------------------------+
|   48   | Isaac-Ant-v0         | isaaclab.envs:ManagerBasedRLEnv            |   isaaclab_tasks.manager_based.classic.ant.ant_env_cfg:AntEnvCfg
+--------+----------------------+--------------------------------------------+---------------------------------------------------------------+
```

请注意，有两种不同的 `Ant` 任务，一种是用于 `Direct` 环境，另一种是用于 `ManagerBased` 环境。这是您可以在 Isaac Lab 立即使用的 [两个主要工作流程](https://docs.robotsfan.com/isaaclab/source/overview/core-concepts/task_workflows.html#feature-workflows) 。Direct 工作流程将为您提供最快速通往用于强化学习的工作自定义环境的路径，但 Manager based 工作流程将为您的项目提供更广泛开发所需的模块化。出于本快速入门指南的目的，我们只会专注于 Direct 工作流程。

## 生成您自己的项目

使用 Isaac Lab 开始新项目起初可能会让人望而生畏，但这就是为什么我们提供 [模板生成器](https://docs.robotsfan.com/isaaclab/source/overview/own-project/template.html#template-generator) ，通过命令行快速生成新项目的原因。

```bash
./isaaclab.sh --new
```

这将根据您选择的设置为您创建一个新项目

- **外部 vs 内部**: 确定项目是作为 isaac lab 存储库的一部分构建，还是作为外部扩展加载的。
- **Direct vs Manager**: 直接任务主要包含环境定义中的所有实现细节，而基于 manager 的项目则意味着使用我们各种环境“部件”的模块化定义。
- **框架**: 您可以在这里选择多个选项。这决定了您打算在项目中本地使用的 RL 框架（您想要使用哪些特定算法实现进行训练）。

创建后，导航到安装的项目并运行

```bash
python -m pip install -e source/<given-project-name>
```

来完成安装过程并注册环境。在模板生成器创建的目录中，您将至少找到一个具有类似以下内容的 `__init__.py` 文件

```python
import gymnasium as gym

gym.register(
    id="Template-isaaclabtutorial_env-v0",
    entry_point=f"{__name__}.isaaclabtutorial_env:IsaaclabtutorialEnv",
    disable_env_checker=True,
    kwargs={
        "env_cfg_entry_point": f"{__name__}.isaaclabtutorial_env_cfg:IsaaclabtutorialEnvCfg",
        "skrl_cfg_entry_point": f"{agents.__name__}.skrl_ppo_cfg:PPORunnerCfg",
    },
)
```

这是实际为将来使用注册环境的函数。请注意， `entry_point` 实际上只是环境定义的 python 模块路径。这就是为什么我们需要将项目安装为包: 模块路径 **就是** gymnasium API 的入口点。

## 配置

无论您在 Isaac Lab 中要做什么，您都需要处理\*\*配置\*\* 。所有配置类都可以通过它们的类定义上方的 `@configclass` 装饰器和缺少 `__init__` 函数来识别。例如，考虑下面这个关于 [cartpole 环境](https://docs.robotsfan.com/isaaclab/source/tutorials/03_envs/create_direct_rl_env.html#tutorial-create-direct-rl-env) 的配置类。

```python
@configclass
class CartpoleEnvCfg(DirectRLEnvCfg):
    # env
    decimation = 2
    episode_length_s = 5.0
    action_scale = 100.0  # [N]
    action_space = 1
    observation_space = 4
    state_space = 0

    # simulation
    sim: SimulationCfg = SimulationCfg(dt=1 / 120, render_interval=decimation)

    # robot
    robot_cfg: ArticulationCfg = CARTPOLE_CFG.replace(prim_path="/World/envs/env_.*/Robot")
    cart_dof_name = "slider_to_cart"
    pole_dof_name = "cart_to_pole"

    # scene
    scene: InteractiveSceneCfg = InteractiveSceneCfg(num_envs=4096, env_spacing=4.0, replicate_physics=True)

    # reset
    max_cart_pos = 3.0  # the cart is reset if it exceeds that position [m]
    initial_pole_angle_range = [-0.25, 0.25]  # the range in which the pole angle is sampled from on reset [rad]

    # reward scales
    rew_scale_alive = 1.0
    rew_scale_terminated = -2.0
    rew_scale_pole_pos = -1.0
    rew_scale_cart_vel = -0.01
    rew_scale_pole_vel = -0.005
```

请注意，整个类定义只是一组值字段和其他配置。配置类对于在训练过程中需要关心向量化的任何内容都是必不可少的。 如果您想要能够将环境复制成千上万次，并且异步地管理每个数据，您需要以某种方式 "标记" 哪些场景部分对这个复制过程（向量化）是重要的。 这就是配置类的作用！

在这种情况下，该类定义了整个训练环境的配置！请注意 `InteractiveSceneCfg` 中的 `num_envs` 变量。这实际上会被 `train.py` 脚本内部的 CLI 参数所覆盖。配置提供了一条通往配置层次结构中的任何变量的直接路径，从而轻松修改在启动时由环境“配置”的任何内容。

## 机器人

在 Isaac Lab 中，机器人完全被定义为配置的实例。如果您检查 `source/isaaclab_assets/isaaclab_assets/robots` ，您将看到许多文件，每个文件都包含了有关所讨论机器人的配置。这些单独的文件的目的是更好地定义所有不同机器人的范围，但没有任何阻止您 [向您的项目添加新的机器人](https://docs.robotsfan.com/isaaclab/source/tutorials/01_assets/add_new_robot.html#tutorial-add-new-robot) ，甚至添加到 `isaaclab` 存储库中！例如，考虑以下配置中的 Dofbot

```python
import isaaclab.sim as sim_utils
from isaaclab.actuators import ImplicitActuatorCfg
from isaaclab.assets.articulation import ArticulationCfg
from isaaclab.utils.assets import ISAAC_NUCLEUS_DIR

DOFBOT_CONFIG = ArticulationCfg(
    spawn=sim_utils.UsdFileCfg(
        usd_path=f"{ISAAC_NUCLEUS_DIR}/Robots/Dofbot/dofbot.usd",
        rigid_props=sim_utils.RigidBodyPropertiesCfg(
            disable_gravity=False,
            max_depenetration_velocity=5.0,
        ),
        articulation_props=sim_utils.ArticulationRootPropertiesCfg(
            enabled_self_collisions=True, solver_position_iteration_count=8, solver_velocity_iteration_count=0
        ),
    ),
    init_state=ArticulationCfg.InitialStateCfg(
        joint_pos={
            "joint1": 0.0,
            "joint2": 0.0,
            "joint3": 0.0,
            "joint4": 0.0,
        },
        pos=(0.25, -0.25, 0.0),
    ),
    actuators={
        "front_joints": ImplicitActuatorCfg(
            joint_names_expr=["joint[1-2]"],
            effort_limit_sim=100.0,
            velocity_limit_sim=100.0,
            stiffness=10000.0,
            damping=100.0,
        ),
        "joint3_act": ImplicitActuatorCfg(
            joint_names_expr=["joint3"],
            effort_limit_sim=100.0,
            velocity_limit_sim=100.0,
            stiffness=10000.0,
            damping=100.0,
        ),
        "joint4_act": ImplicitActuatorCfg(
            joint_names_expr=["joint4"],
            effort_limit_sim=100.0,
            velocity_limit_sim=100.0,
            stiffness=10000.0,
            damping=100.0,
        ),
    },
)
```

这完全定义了 dofbot！您可以将此内容复制到一个 `.py` 文件中并将其作为模块导入，以便在自己的 lab sims 中使用 dofbot。您将在定义带有状态的事物的任何配置中看到的一个常见特征是 `InitialStateCfg` 的存在。请记住，配置是指明向量化的信息， `InitialStateCfg` 描述了机器人在每个环境中创建时的关节状态。 `ImplicitActuatorCfg` 使用由关节时间决定的默认执行模型来定义机器人的关节。并不是所有关节都需要被执行，但如果您不打算使用这些未定义的关节，您将会收到警告。如果您不打算使用这些未定义的关节，您通常可以忽略它们。

## Apps 和 Sims

使用仿真意味着启动 Isaac Sim 应用程序以提供仿真上下文。如果您没有运行由标准工作流程定义的任务，则需要负责创建应用程序、管理上下文并通过时间推进仿真。 这是 "第三个工作流程": 一个 **独立** 应用程序，这是我们为框架、演示、基准测试等编写的脚本所谓的事情...

独立工作流程使您可以完全控制应用程序中的一切和仿真上下文。在 [Isaac Sim 文档](https://docs.isaacsim.omniverse.nvidia.com/latest/index.html) 中详细讨论了开发独立应用程序，但有几点值得着重，因为它们可以非常有用。

```python
import argparse

from isaaclab.app import AppLauncher
# add argparse arguments
parser = argparse.ArgumentParser(
    description="This script demonstrates adding a custom robot to an Isaac Lab environment."
)
parser.add_argument("--num_envs", type=int, default=1, help="Number of environments to spawn.")
# append AppLauncher cli args
AppLauncher.add_app_launcher_args(parser)
# parse the arguments
args_cli = parser.parse_args()

# launch omniverse app
app_launcher = AppLauncher(args_cli)
simulation_app = app_launcher.app
```

`AppLauncher` 是任何 Isaac Sim 应用程序的入口点，如 Isaac Lab！ *许多 Isaac Lab 和 Isaac Sim 模块直到应用程序启动之后才能导入！* 这是在上面的代码的倒数第二行执行的，当构造 `AppLauncher` 时。 `app_launcher.app` 是我们访问套件应用程序框架的接口；广泛的中介代码将仿真与扩展管理系统、GUI 等等绑定在一起。在独立工作流程中，这个界面，通常被称为 `simulation_app` 主要用于检查仿真是否正在运行，并在仿真结束后清理。