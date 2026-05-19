## Unitree RL Lab

## Overview

This project provides a set of reinforcement learning environments for Unitree robots, built on top of [IsaacLab](https://github.com/isaac-sim/IsaacLab).

Currently supports Unitree **Go2**, **H1** and **G1-29dof** robots.

| Isaac Lab | Mujoco | Physical |
| --- | --- | --- |
| [![](https://camo.githubusercontent.com/4b69105d6fca862100cd39919de9b82fe0a3023dcb11d168e476079e67addcfe/68747470733a2f2f6f73732d676c6f62616c2d63646e2e756e69747265652e636f6d2f7374617469632f64383739616461633235303634386335383764333638316539303635386234395f343830783339372e676966)](https://github.com/chensisi0730/unitree_rl_lab/blob/main/g1_sim.gif) | [![](https://camo.githubusercontent.com/c0de53628ab164d4416a93408efd38fd4bc0743c19915b212225795471ebdae8/68747470733a2f2f6f73732d676c6f62616c2d63646e2e756e69747265652e636f6d2f7374617469632f33633838653034356162313234633361623963373631613939636235653731665f343830783339372e676966)](https://github.com/chensisi0730/unitree_rl_lab/blob/main/g1_mujoco.gif) | [![](https://camo.githubusercontent.com/de88ee3d6437e0c0c0985b4ccb9588382ede56b900a9e27f99ac331b0f247e25/68747470733a2f2f6f73732d676c6f62616c2d63646e2e756e69747265652e636f6d2f7374617469632f36633137633663663532656334653236626266616231666266353931616462325f343830783237302e676966)](https://github.com/chensisi0730/unitree_rl_lab/blob/main/g1_real.gif) |

## Installation

- Install Isaac Lab by following the [installation guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/index.html).
- Install the Unitree RL IsaacLab standalone environments.
	- Clone or copy this repository separately from the Isaac Lab installation (i.e. outside the `IsaacLab` directory):
		```
		git clone https://github.com/unitreerobotics/unitree_rl_lab.git
		```
		- Use a python interpreter that has Isaac Lab installed, install the library in editable mode using:
		```
		conda activate env_isaaclab
		./unitree_rl_lab.sh -i
		# restart your shell to activate the environment changes.
		```
- Download unitree robot description files
	*Method 1: Using USD Files*
	- Download unitree usd files from [unitree\_model](https://huggingface.co/datasets/unitreerobotics/unitree_model/tree/main), keeping folder structure
		```
		git clone https://huggingface.co/datasets/unitreerobotics/unitree_model
		```
		- Config `UNITREE_MODEL_DIR` in `source/unitree_rl_lab/unitree_rl_lab/assets/robots/unitree.py`.
		```
		UNITREE_MODEL_DIR = "</home/user/projects/unitree_usd>"
		```
	*Method 2: Using URDF Files \[Recommended\]* Only for Isaacsim >= 5.0
	- Download unitree robot urdf files from [unitree\_ros](https://github.com/unitreerobotics/unitree_ros)
		```
		git clone https://github.com/unitreerobotics/unitree_ros.git
		```
		- Config `UNITREE_ROS_DIR` in `source/unitree_rl_lab/unitree_rl_lab/assets/robots/unitree.py`.
		```
		UNITREE_ROS_DIR = "</home/user/projects/unitree_ros/unitree_ros>"
		```
		- \[Optional\]: change *robot\_cfg.spawn* if you want to use urdf files
- Verify that the environments are correctly installed by:
	- Listing the available tasks:
		```
		./unitree_rl_lab.sh -l # This is a faster version than isaaclab
		```
		- Running a task:
		```
		./unitree_rl_lab.sh -t --task Unitree-G1-29dof-Velocity # support for autocomplete task-name
		# same as
		python scripts/rsl_rl/train.py --headless --task Unitree-G1-29dof-Velocity
		```
		- Inference with a trained agent:
		```
		./unitree_rl_lab.sh -p --task Unitree-G1-29dof-Velocity # support for autocomplete task-name
		# same as
		python scripts/rsl_rl/play.py --task Unitree-G1-29dof-Velocity
		```

## Deploy

After the model training is completed, we need to perform sim2sim on the trained strategy in Mujoco to test the performance of the model. Then deploy sim2real.

### Setup

```
# Install dependencies
sudo apt install -y libyaml-cpp-dev libboost-all-dev libeigen3-dev libspdlog-dev libfmt-dev
# Install unitree_sdk2
git clone git@github.com:unitreerobotics/unitree_sdk2.git
cd unitree_sdk2
mkdir build && cd build
cmake .. -DBUILD_EXAMPLES=OFF # Install on the /usr/local directory
sudo make install
# Compile the robot_controller
cd unitree_rl_lab/deploy/robots/g1_29dof # or other robots
mkdir build && cd build
cmake .. && make
```

### Sim2Sim

Installing the [unitree\_mujoco](https://github.com/unitreerobotics/unitree_mujoco?tab=readme-ov-file#installation).

- Set the `robot` at `/simulate/config.yaml` to g1
- Set `domain_id` to 0
- Set `enable_elastic_hand` to 1
- Set `use_joystck` to 1.
```
# start simulation
cd unitree_mujoco/simulate/build
./unitree_mujoco
# ./unitree_mujoco -i 0 -n eth0 -r g1 -s scene_29dof.xml # alternative
```
```
cd unitree_rl_lab/deploy/robots/g1_29dof/build
./g1_ctrl
# 1. press [L2 + Up] to set the robot to stand up
# 2. Click the mujoco window, and then press 8 to make the robot feet touch the ground.
# 3. Press [R1 + X] to run the policy.
# 4. Click the mujoco window, and then press 9 to disable the elastic band.
```

### Sim2Real

You can use this program to control the robot directly, but make sure the on-borad control program has been closed.

```
./g1_ctrl --network eth0 # eth0 is the network interface name.
```

## Acknowledgements

This repository is built upon the support and contributions of the following open-source projects. Special thanks to:

- [IsaacLab](https://github.com/isaac-sim/IsaacLab): The foundation for training and running codes.
- [mujoco](https://github.com/google-deepmind/mujoco.git): Providing powerful simulation functionalities.
- [robot\_lab](https://github.com/fan-ziqi/robot_lab): Referenced for project structure and parts of the implementation.
- [whole\_body\_tracking](https://github.com/HybridRobotics/whole_body_tracking): Versatile humanoid control framework for motion tracking.