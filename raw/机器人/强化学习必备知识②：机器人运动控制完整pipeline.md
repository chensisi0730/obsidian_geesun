18 人赞同了该文章

> *原文链接： [强化学习必备知识②：机器人运动控制完整pipeline](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/rSiXsXgVNQ2gJLB8-HTg4g)*

![动图封面](https://pic3.zhimg.com/v2-256764bf40c207523e50fe2a065cbcae_b.jpg)

强化学习在机器人运动 [控制领域](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%8E%A7%E5%88%B6%E9%A2%86%E5%9F%9F&zhida_source=entity) 的重要性已是共识，就不再这里过多赘述了。但这套范式对应的完整工程落地逻辑却鲜有梳理。

**这种从“人工设计规则”到“数据驱动学习”的转变，究竟是怎么做到的？**

今天这篇分享，我们依旧回归技术底层，系统梳理一下机器人强化学习运动控制的技术管线 *（以四足机器人为例）* 。

**涵盖基础闭环、 [分层控制](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%88%86%E5%B1%82%E6%8E%A7%E5%88%B6&zhida_source=entity) 、PPO算法、特权信息蒸馏、奖励函数设计、域随机化及GPU并行仿真等核心模块。**

## 01 机器人是如何“学习”的？

要理解机器人强化学习，我们首先要搞懂一个基础框架：

**强化学习闭环。**

举个例子，你可以把强化学习想象成训练一只小狗，你不会告诉小狗“先抬左前腿 30 度，再收缩右后腿肌肉”，而是给它一个指令，当它做对了，你就给一块肉干 **（奖励）** ；做错了，你就轻轻拍它一下 **（惩罚）** 。

在 [机器人领域](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E9%A2%86%E5%9F%9F&zhida_source=entity) ，这个过程被抽象成了四个核心要素：

1. **观测（Observation）** ：智能体传感器采集的局部状态观测，包含关节角度、 [角速度](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%A7%92%E9%80%9F%E5%BA%A6&zhida_source=entity) 、机身倾角、地形高度图、相机视觉等，对应 POMDP 局部观测空间；
2. **动作（Action）** ：策略网络输出的控制指令，具身场景多为 [连续动作空间](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%BF%9E%E7%BB%AD%E5%8A%A8%E4%BD%9C%E7%A9%BA%E9%97%B4&zhida_source=entity) ；
3. **环境（Environment）** ：物理仿真引擎 / 真实 [物理系统](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E7%89%A9%E7%90%86%E7%B3%BB%E7%BB%9F&zhida_source=entity) ，依据机器人动作求解 [动力学方程](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8A%A8%E5%8A%9B%E5%AD%A6%E6%96%B9%E7%A8%8B&zhida_source=entity) ，输出下一时刻观测；
4. **即时奖励（Reward）** ：基于任务指标量化的单步收益，用于指导策略迭代优化。
![](https://pic1.zhimg.com/v2-234c39b9fbce0fc14a6fb9dba5d35afc_1440w.jpg)

▲图1 | 强化学习的基础闭环：智能体（Agent）通过观察环境状态，采取动作，并根据环境反馈的奖励来不断优化自己的策略

智能体核心载体为深度神经网络 [参数化策略](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8F%82%E6%95%B0%E5%8C%96%E7%AD%96%E7%95%A5&zhida_source=entity) ，训练目标为最大化折扣累积回报,为 [折扣因子](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%8A%98%E6%89%A3%E5%9B%A0%E5%AD%90&zhida_source=entity) 。

完整训练过程为循环执行交互采样、梯度更新，通过大量试错迭代收敛至 [最优策略](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%9C%80%E4%BC%98%E7%AD%96%E7%95%A5&zhida_source=entity)

**——这个过程，就是策略（Policy）的训练。**

```
# 单步强化学习基础交互流程
obs = env.reset()  # 初始化观测o_0
for t in range(max_timesteps):
    action = policy_net(obs)  # 策略网络输出动作a_t
    next_obs, reward, done, info = env.step(action)  # 环境交互，获取o_{t+1}, r_t
    buffer.append((obs, action, reward, next_obs, done))  # 存储轨迹样本
    obs = next_obs
    if done:
        obs = env.reset()
# 基于样本池更新策略网络参数θ
update_policy(buffer)
```

▲强化学习最底层的 [单智能体](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8D%95%E6%99%BA%E8%83%BD%E4%BD%93&zhida_source=entity) 环境交互

## 02 机器人强化学习的完整管线

**在实际工程中，仅仅有一个基础闭环是无法适配真实 [机器人动力学](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%8A%A8%E5%8A%9B%E5%AD%A6&zhida_source=entity) 非线性、 [执行器](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%89%A7%E8%A1%8C%E5%99%A8&zhida_source=entity) 延迟、硬件扰动等问题的。**

为了让训练出来的策略既聪明又稳定，研究人员通常会采用 **分层控制的架构。**

![](https://pic1.zhimg.com/v2-3cd1f7db3031876647977b686d100fe4_1440w.jpg)

▲图2 | 机器人强化学习的典型训练管线：高层神经网络输出目标动作，低层控制器负责具体执行，物理引擎提供环境反馈

从这张图中我们可以看到，整个系统通常分为两层：

- **高层：DRL 策略网络（大脑）**

输入传感器观测，输出目标关节位置

**这是通过强化学习训练出来的神经网络。** 它的运行频率通常是 50 Hz（每秒做 50 次决策）。

它根据传感器传来的数据，思考一下，然后给出一个宏观的指令，比如：“所有关节移动到这个新的目标角度”。

- **低层： [PD 控制器](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=PD+%E6%8E%A7%E5%88%B6%E5%99%A8&zhida_source=entity) （小脑/ [脊髓](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%84%8A%E9%AB%93&zhida_source=entity) ）**

如果让神经网络直接控制电机输出多大的 [力矩](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8A%9B%E7%9F%A9&zhida_source=entity) （扭矩），不仅训练难度极高，而且一旦遇到突发干扰，机器人很容易失控摔倒。

因此，主流的做法是让神经网络只输出“目标位置”，然后交给底层的 PD 控制器（比例- [微分控制器](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%BE%AE%E5%88%86%E6%8E%A7%E5%88%B6%E5%99%A8&zhida_source=entity) ）去执行。标准 PD 力矩公式：

为当前关节角度，为关节角速度； [比例增益](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%AF%94%E4%BE%8B%E5%A2%9E%E7%9B%8A&zhida_source=entity) 、 [微分增益](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%BE%AE%E5%88%86%E5%A2%9E%E7%9B%8A&zhida_source=entity) 为控制器固定参数。

PD 控制器运行频率极高（通常在 200 Hz 到 1000 Hz 之间）。它就像机器人的 [脊髓反射](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%84%8A%E9%AB%93%E5%8F%8D%E5%B0%84&zhida_source=entity) 一样，实时对比“当前关节角度”和“目标关节角度”，然后计算出电机需要输出多大的力矩。

**这种“高层算位置，低层算力矩”的模式有三大优势：**

- **降低训练难度** ：神经网络不需要去学习复杂的底层 [物理力学](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E7%89%A9%E7%90%86%E5%8A%9B%E5%AD%A6&zhida_source=entity) ，只管定目标就行。
- **自带柔顺性** ：PD 控制器可以像弹簧一样，吸收足端落地时的冲击力，保护硬件。
- **弥合虚实鸿沟** ：真实电机和仿真模型总有差异，底层的 PD 控制器可以帮神经网络兜底，抹平这些微小的硬件差异。

## 03 PPO 算法：为什么大家都在用它？

分层架构确定训练管线后，需选择稳定的策略优化算法完成网络迭代。

**如果你去看机器人强化学习的论文，十有八九会看到一个名字：PPO（Proximal Policy Optimization， [近端策略优化](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%BF%91%E7%AB%AF%E7%AD%96%E7%95%A5%E4%BC%98%E5%8C%96&zhida_source=entity) ）。**

在 PPO 出现之前，强化学习 [训练机器人](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%AE%AD%E7%BB%83%E6%9C%BA%E5%99%A8%E4%BA%BA&zhida_source=entity) 就像“走钢丝”。如果策略更新的步子迈得太大，好不容易学会的一点点走路技巧可能瞬间崩溃。

![](https://pic2.zhimg.com/v2-e9fe22334dd6d82ffe1083f10abe07f7_1440w.jpg)

▲图3 | PPO 算法的核心网络结构：通过限制新旧策略的差异，保证训练过程的稳定收敛

**PPO 的核心能力在于它的“ [截断机制](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%88%AA%E6%96%AD%E6%9C%BA%E5%88%B6&zhida_source=entity) （Clipping）”。** 约束新旧策略动作 [概率比值](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%A6%82%E7%8E%87%E6%AF%94%E5%80%BC&zhida_source=entity) ，限制单 [次梯度](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%AC%A1%E6%A2%AF%E5%BA%A6&zhida_source=entity) 更新幅度，标准 Clip [目标函数](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E7%9B%AE%E6%A0%87%E5%87%BD%E6%95%B0&zhida_source=entity) ：

其中：：新旧策略概率比；

：GAE 广义优势函数，衡量当前动作相对均值的收益；

：截断阈值，工程常规取值 。

```
初始化策略参数 θ_0，值函数参数 φ_0
for k = 0, 1, 2, ... do
    用当前策略 π_k = π(θ_k) 与环境交互，收集轨迹集合 D_k = {τ_i}
    计算折扣回报 R̂_i = Σ_{k=i}^{H} γ^{k-i} r_k
    基于当前值函数 V_{φ_k} 计算优势估计 Â
    通过最大化PPO-Clip目标更新策略：
        θ_{k+1} = argmax_θ (1/|D_k|) Σ_{τ∈D_k} Σ_i min( r_i(θ)Â_i, clip(r_i(θ), 1-ε, 1+ε)Â_i )
    通过最小化均方误差拟合值函数：
        φ_{k+1} = argmin_φ (1/|D_k|) Σ_{τ∈D_k} Σ_i (V_φ(s_i) - R̂_i)²
end for
```

▲一轮又一轮「采集仿真交互轨迹→计算回报与优势→更新策略网络→更新 [价值网络](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E4%BB%B7%E5%80%BC%E7%BD%91%E7%BB%9C&zhida_source=entity) 」的标准训练流程

**简单来说，PPO 在每次更新神经网络时，都会把“新策略”和“旧策略”做个对比。**

如果它发现新策略相比旧策略变化太剧烈，它就会强行把更新幅度“截断”，限制在一个安全的范围内。

*这就好比教练在教运动员，每次只允许纠正一点点动作细节，绝不允许推翻重来。*

这种“小步快跑、稳扎稳打”的机制，使得 PPO 极其稳定，成为了目前机器人领域最主流的算法。

## 04 特权信息与 Teacher-Student 架构

PPO 解决仿真内训练稳定性，但在仿真环境里，机器人可以说是开了“ [上帝视角](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E4%B8%8A%E5%B8%9D%E8%A7%86%E8%A7%92&zhida_source=entity) ”：

**它可以轻易知道地形的确切高度、地面的 [摩擦系数](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%91%A9%E6%93%A6%E7%B3%BB%E6%95%B0&zhida_source=entity) 、甚至自己重心的精确位置。**

我们把这些真实世界中很难获取的数据，称为 **特权信息** （Privileged Information）。

但在真实世界里，机器人只能靠自己身上的相机和惯性传感器（IMU），不仅视野有限，数据还充满了 [噪点](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%99%AA%E7%82%B9&zhida_source=entity) 。

**怎么解决这个矛盾？**

研究人员发明了 Teacher-Student（师生蒸馏） 架构。

![](https://pica.zhimg.com/v2-39776aadbe28f21b5eb11ecbbcf7163e_1440w.jpg)

▲图4 | 经典的 Teacher-Student 训练架构：先利用特权信息训练一个全知全能的老师，再让只能获取普通传感器数据的学生去模仿老师的动作

整个过程分为两步：

- **阶段 1：教师网络训练**

输入完整特权信息与基础观测，在仿真环境中训练全局最优运动策略，具备完整 [环境动力学](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E7%8E%AF%E5%A2%83%E5%8A%A8%E5%8A%9B%E5%AD%A6&zhida_source=entity) 感知能力。

- **阶段 2：学生网络 [蒸馏训练](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%92%B8%E9%A6%8F%E8%AE%AD%E7%BB%83&zhida_source=entity)**

仅输入真机可用局部观测，以最小化师生策略 KL [散度](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%95%A3%E5%BA%A6&zhida_source=entity) 为蒸馏损失，同时叠加任务奖励损失， [联合优化](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E8%81%94%E5%90%88%E4%BC%98%E5%8C%96&zhida_source=entity) 学生网络参数：

为 PPO 原始强化学习损失，为蒸馏损失 [权重系数](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E6%9D%83%E9%87%8D%E7%B3%BB%E6%95%B0&zhida_source=entity) 。  
  
通过蒸馏，学生网络可仅依靠局部传感器观测，推理等效全局动力学特征，复刻教师稳定运动行为。

算法与蒸馏架构确定后，策略迭代方向完全由 [奖励函数](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=2&q=%E5%A5%96%E5%8A%B1%E5%87%BD%E6%95%B0&zhida_source=entity) 约束，奖励函数是定义机器人运动行为的核心约束模块。

## 05 奖励函数：“调教”机器人

如果说算法是引擎，那么奖励函数（Reward Function）就是方向盘。

**机器人最终走成什么样，全看你奖励什么、惩罚什么。**

以四足机器人为例，早期研究中，工程师们为了让机器人走得像狗，会把狗的动作录下来，强迫机器人去模仿（也就是模仿学习）。

**通常只设定几个最基础的奖励和惩罚：**

其中：

：速度跟踪奖励：如果设定前进速度为 1m/s，机器人达到该速度则给予正向加分。

：能耗惩罚：电机输出力矩越大，惩罚扣分越多，引导机器人实现节能运行。

：动作平滑惩罚：若关节动作抖动剧烈则进行扣分，约束机器人输出平滑动作。

：姿态惩罚：当机身倾斜角度过大时进行扣分。

为各分项权重超参。

就靠这几个简单的规则，神经网络为了拿到最高分，会在成千上万次的试错中，自动涌现出类似动物的“小跑（Trot）”步态。

因为物理规律决定了，对于四条腿的结构来说，小跑就是最省力、最稳定的移动方式。

## 06 跨越虚实鸿沟：域随机化

**无论奖励函数如何精细，仿真模型与真实物理世界之间固有的动力学偏差仍是部署阶段的核心障碍：**

电机有延迟，地面有坑洼， **甚至机器人的某条腿可能比图纸上重了 100 克** 。这种仿真与现实的差异，被称为 Sim-to-Real Gap（虚实鸿沟）。

为了跨越这条鸿沟， **最常用的就是：域随机化（Domain Randomization）。**

![](https://pic1.zhimg.com/v2-a7a3b1c5d0d7f1540903e8e3a179c960_1440w.jpg)

▲图5 | 域随机化技术：在仿真中刻意注入各种随机噪声，强迫策略网络学会适应各种恶劣的物理条件

形式化地，设仿真环境可调 [参数向量](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8F%82%E6%95%B0%E5%90%91%E9%87%8F&zhida_source=entity) 为 ，参数服从 [先验分布](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%85%88%E9%AA%8C%E5%88%86%E5%B8%83&zhida_source=entity) ；

参数维度覆盖机器人刚体质量、 [地面摩擦系数](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%9C%B0%E9%9D%A2%E6%91%A9%E6%93%A6%E7%B3%BB%E6%95%B0&zhida_source=entity) 、电机响应延迟、传感器 [高斯噪声](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E9%AB%98%E6%96%AF%E5%99%AA%E5%A3%B0&zhida_source=entity) 幅值等。

域随机化的优化目标为在 [参数分布](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%8F%82%E6%95%B0%E5%88%86%E5%B8%83&zhida_source=entity) 下最大化期望累积奖励：

实际训练流程中，每个 episode 初始化阶段会对 的各维度独立随机采样，等价于让策略在数千组参数互不相同的仿真 “ [平行宇宙](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%B9%B3%E8%A1%8C%E5%AE%87%E5%AE%99&zhida_source=entity) ” 中迭代试错，例如：

- 把机器人的质量随机增加或减少 20%；
- 把地面的摩擦系数在“冰面”和“砂纸”之间随机切换；
- 给传感器的读数加上各种随机噪点；
- 甚至模拟电机时不时的延迟或轻微卡顿。

在数千组极端恶劣的平行仿真环境中完成训练的神经网络，部署至实体机器人后，现实场景中常见的微 [小扰动](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=%E5%B0%8F%E6%89%B0%E5%8A%A8&zhida_source=entity) 与误差，大多不会对其运行造成明显影响。

## 07 GPU 并行仿真

你可能会问，要经历这么多试错，还要在几千个平行宇宙里训练，这得花多少时间？

**如果放在五年前，可能需要几个星期。但现在，只需要几个小时。**

![](https://pic2.zhimg.com/v2-d363cc020521fe27ad02ac0a19ef39af_1440w.jpg)

▲图6 | 基于 GPU 的大规模并行仿真：可以在单一显卡上同时模拟数千个机器人，将收集数据的速度提升了成百上千倍

以 [NVIDIA Isaac Gym](https://zhida.zhihu.com/search?content_id=277938539&content_type=Article&match_order=1&q=NVIDIA+Isaac+Gym&zhida_source=entity) 为例，整个物理世界和神经网络都被搬到了 GPU 上 **。在一张 RTX 4090 显卡上，可以同时模拟 4000 多个机器人。**

它们在不同的地形上同时奔跑、摔倒、爬起，每秒钟能产生 **近 10 万帧** 的训练数据。这种算力上的大规模并行，让机器人强化学习的迭代速度产生了质的飞跃。

## 一个值得认真对待的范式转变

回顾本文梳理的这套技术体系，有一个值得认真思考的问题： **为什么这套方法在短短几年内就能把四足机器人的运动能力推进得如此之快？**

![](https://pic3.zhimg.com/v2-8a377f80cec3834e33b5f4662344ac0c_1440w.jpg)

▲图7 | 经过强化学习训练的四足机器人，已经能够自主适应雪地、碎石、溪流等复杂的野外非结构化地形

**答案并不只是"算力更强了"或"算法更好了"这么简单。**

更根本的原因在于，研究人员找到了一种让机器人自己发现物理规律的方式，而不是试图把人类对物理世界的理解全部编码进规则里。

**当奖励函数只告诉机器人"走快一点、别费电、别摔倒"，神经网络在数以亿计的仿真步骤中，会自发地涌现出符合力学规律的步态** ——这种步态，和动物在漫长进化中形成的运动模式高度吻合。

当然，这套范式也有它清晰可见的局限性。目前的强化学习策略，大多依赖于精心设计的奖励函数和大量的仿真数据。

换句话说，今天的机器人强化学习，仍然是一种高度任务专用的技术。每一项新技能，背后都对应着一套新的奖励工程和仿真环境搭建。

**这也正是当前研究的核心矛盾所在：如何让机器人从"被精心设计的任务里学会一件事"，走向"在开放世界里持续学习多件事"？**

**这个问题，目前还没有令人满意的答案。** 一些研究者在尝试引入语言模型来自动生成奖励函数，另一些人在探索跨任务的通用表征，还有一些人在重新审视模仿学习与强化学习结合的路径。

无论哪条路最终走通，有一点是确定的： **它的进展速度，已经开始让那些曾经认为"通用机器人还需要几十年"的判断显得保守。**

发布于 2026-06-30 10:10・贵州[四足机器人](https://www.zhihu.com/topic/21768047)[程序员0基础入门大模型的学习路线！](https://zhuanlan.zhihu.com/p/31864213680)

[

0基础入门大模型，transformer、bert这些是要学的，但是 你的第一口不一定从这里咬下去。真的没有必要一上来就把时间精力全部投入到复杂的理论、各种晦涩的数学公式还有编程语言上，这...

](https://zhuanlan.zhihu.com/p/31864213680)

已赞同 18