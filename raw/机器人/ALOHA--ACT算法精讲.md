---
title: "ALOHA--ACT算法精讲"
source: "https://zhuanlan.zhihu.com/p/677625871"
author:
  - "[[曹玥]]"
published:
created: 2026-05-01
description: "关于斯坦福最新研究：Mobile ALOHA的论文精讲，请阅读我的 另一篇文章 本文将会对Mobile ALOHA和前作ALOHA中使用的ACT算法做详细介绍。 ACT： Action Chunking with Transformers 是ALOHA团队在2023年4月份提出的…"
tags:
  - "clippings"
---
[收录于 · 具身智能论文阅读笔记](https://www.zhihu.com/column/c_1679174816513609728)

木牛流码 等 182 人赞同了该文章

关于斯坦福最新研究： [Mobile ALOHA](https://zhida.zhihu.com/search?content_id=238713718&content_type=Article&match_order=1&q=Mobile+ALOHA&zhida_source=entity) 的论文精讲，请阅读我的 [另一篇文章](https://zhuanlan.zhihu.com/p/676285367)

本文将会对Mobile ALOHA和前作ALOHA中使用的 [ACT算法](https://zhida.zhihu.com/search?content_id=238713718&content_type=Article&match_order=1&q=ACT%E7%AE%97%E6%B3%95&zhida_source=entity) 做详细介绍。  
  
**ACT： Action Chunking with Transformers**  
是ALOHA团队在2023年4月份提出的一种模仿学习算法，针对精细操作任务表现十分优异。  
ACT后续已作为baseline算法在多篇论文，如RoboAgent，BridgeData V2中被引用。  
  
首先回顾一下ALOHA遥操作平台：

![](https://pic1.zhimg.com/v2-b365fde3e47c28b04113e1a64a94fabe_1440w.jpg)

示教过程中有两个主动臂（leader），两个从动臂（follower），每个机械臂都是6个自由度，两个从动臂上分别装有一个对称的二指夹爪。

整个平台包含四个相机，两个在从动臂的腕部，一个在上方，一个在前方，分别朝向操作台的中心。  
  
因此，在收集到的示教数据中，分别包含：4个视角的图像（480\*640），从动臂的关节角（（6DOF臂+1DOF夹爪）\*2）和主动臂的关节角（（6DOF臂+1DOF夹爪）\*2）

在训练过程中，从动臂的关节角和图像一起组成的输入给模型的观测值，而主动臂的关节角则作为了动作标签。  
  
算法的具体实现细节如下：

## 1\. Action Chunking 和 Temporal Ensemble

对于模仿学习，compounding error的出现是导致任务失败的主要原因。

![](https://pica.zhimg.com/v2-11f60c749f39c6f760313aca9112d96a_1440w.jpg)

Compounding error是指，在测试时，若agent遇到训练集里从未见过的输入，那么很有可能给出具有一定误差的预测值，这可能会导致agent进入更加没有见过的状态，进而一步一步的产生越来越大的预测误差。

为了减小compounding error，ACT使用了action chunking。  

Action chunking是神经科学中的一个概念：独立的动作会被组合到一起并作为一个单元被执行，这样会使得动作被存储和执行的效率更高。我们甚至可以把将电池插入凹槽里这个任务视作一个action chunk。  
  
但是在ACT算法中，作者一开始将chunk size设置为固定的k步。每k步，agent获取一次输入，预测后k步的动作，然后按照顺序执行这些动作。

这样做，可以直接将任务轨迹的长度缩短到1/k（只需要做1/k次预测）。

策略模型也就从 $\pi_{\theta} \left(\right. a_{t} \left|\right. s_{t} \left.\right)$ 变成了 $\pi_{\theta} \left(\right. a_{t : t + k} \left|\right. s_{i} \left.\right)$

Chunking也可以帮助刻画人类示教行为的非马尔可夫性（人会根据历史信息来完成任务）。

一个单步预测的策略很容易受到时序相关信息的影响，比如示教过程中的一个突然停顿，会让agent很困扰，因为下一步做什么动作会与停顿的时间有很强的关系。  
  
然而，简单的action chunking会出现这样的问题：每k步突然输入一个新的观测值，这种不连续会导致机器人出现不稳定的行为。

为了使机器人的运动轨迹更加平滑，作者提出每一步都让模型预测后面k步的动作，这样会使得模型在不同时间点上预测的动作有所重合。

进一步地，作者提出一种temporal ensemble的方法来组合这些预测值。

![](https://pic3.zhimg.com/v2-013d41b34f66655c56de345a48311906_1440w.jpg)

对于一个时间步的动作，会有过k次预测，使用加权的方式对这k次预测做平均，权重的设计如下： $w_{i} = e x p \left(\right. - m * i \left.\right)$ ，（w\_0指的是最久的预测）

这是一个以i为变量，单调递减的函数，预测发生的越久，其权重越大，m控制融合新信息的速度，m越小，融合的越快

## 2\. 模型结构

人类示教数据常带有很多噪声：人会用不同的轨迹完成同一个任务，也会对精度要求不高的任务，做出随机性很强的行为。

对此，作者将action chunking policy设置为一个生成式模型，以 [CVAE](https://zhida.zhihu.com/search?content_id=238713718&content_type=Article&match_order=1&q=CVAE&zhida_source=entity) （conditional variational autoencoder）的形式来训练模型，根据输入观测值，生成预测动作。

![](https://pica.zhimg.com/v2-1b6d8c8485480c3c5444e87d684f2b00_1440w.jpg)

**CVAE encoder：**

输入：\[cls\]前缀 + 观测值中的关节角和示教数据中对应的动作序列+position embedding（为了训的更快，这里没有用图像输入）

输出：隐变量z分布的均值和方差

encoder仅用来训练decoder，在测试时，只使用decoder作为policy，z被设置为零。

**CVAE decoder：**

输入：观测值，包括关节角和图像 + encoder输出的隐变量z，对于图像使用 [ResNet18](https://zhida.zhihu.com/search?content_id=238713718&content_type=Article&match_order=1&q=ResNet18&zhida_source=entity) image encoder

输出：预测的动作序列：关节角的绝对值

**整体的训练目标：**

包含两部分：

1. 模仿学习loss：与经典的模仿学习类似，只是action变为action chunk
![](https://pic1.zhimg.com/v2-a3428424cfa2b5bb845b75e5a4b59904_1440w.jpg)

2\. 标准的VAE loss：L1 reconstruction loss和将encoder的输出规范到均值为0，方差为I的高斯分布的正则项

**网络结构：**

对于CVAE的encoder和decoder都使用 [transformer](https://zhida.zhihu.com/search?content_id=238713718&content_type=Article&match_order=1&q=transformer&zhida_source=entity) 。  

最终模型有80M的参数量，使用一个11G的2080Ti GPU可以用5个小时训练出一个单任务模型，同样的机器上模型的推理时间为0.01秒。  
  
在ablation study中，作者发现使用L1 loss作为VAE的reconstruction loss比L2要好；预测绝对关节角比相对变化要好。

## 伪代码

![](https://pic2.zhimg.com/v2-04dbc3ae317719e042fd0cc8e98b8c45_1440w.jpg)

![](https://pic2.zhimg.com/v2-3ac39f52549ff9619c016474f9ddf687_1440w.jpg)

## 模型架构细节图

![](https://pic2.zhimg.com/v2-bd78c073040c8aa891182d24eb1e842f_1440w.jpg)

![](https://pic1.zhimg.com/v2-31cbf1cd0640ec243e4e34c3f6314acc_1440w.jpg)

## 超参数

![](https://pica.zhimg.com/v2-fc27e47e7b0f32394290dac9a5a66c10_1440w.jpg)

编辑于 2024-01-13 14:16・广东[模仿学习](https://www.zhihu.com/topic/23689603)[机器人](https://www.zhihu.com/topic/19551273)[具身智能](https://www.zhihu.com/topic/21535379)[Kamvas 22(Gen 3)数位屏新品上市，90Hz高刷加持](https://store.huion.cn/product/shuweiping/Kamvas-22-Gen-3?spu=biz%3D0%26ci%3D3692314%26si%3D591092d0-58df-4d5b-9be4-04e4a35951c0%26ts%3D1777553053%26zid%3D1629)

[

21.5英寸屏幕，兼具大尺寸显示与高分辨率，拥有90Hz刷新率，画面通透沉浸的同时绘画更流畅；五种色彩模式辅以△...

](https://store.huion.cn/product/shuweiping/Kamvas-22-Gen-3?spu=biz%3D0%26ci%3D3692314%26si%3D591092d0-58df-4d5b-9be4-04e4a35951c0%26ts%3D1777553053%26zid%3D1629)