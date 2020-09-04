---
title: >-
  【论文阅读】——MotioNet, 3D Human Motion Reconstruction from Monocular Video with
  Skeleton Consistency
mathjax: true
date: 2020-08-30 20:04:11
tags:
---

> Title: MotioNet: 3D Human Motion Reconstruction from Monocular Video with Skeleton Consistency
> Author: Mingyi Shi, et al
> Conference: ToG, 2020(transaction of graphics)
> Abstract: 作者提出了一种 data-driven 的 3 维人体运动重构模型，该模型很好地解决了脚贴地、绝对位移等问题。

<!--more-->

*prefix*

3D 人体重构方面主要分为两类，一种是预测 pose，另一种是预测 shape。首先 3D pose estimation 的任务是预测 3D 的关节点，它只能作为 human motion reconstruction 的一个子任务，因为 3D 的关节点并没有关节旋转的信息，比方说，相同的一套 3D 关节点可以对应着无数个真正的姿势，因为关节点的内旋不会导致关节点物理位置的移动，并且由于预测上误差的存在，会导致骨骼的长短发生抖动。

3D 人体重构一般分为两种方法，一种是从 3D joint position 中预测出关节点的旋转；另外一种是 model-based，比如基于著名的 SMPL 人体蒙皮模型，用网络去预测 SMPL 模型的参数。

本文基于的方法是第一种方法，从 2D joint position 中预测关节点旋转、root 移动、骨骼长度、脚部是否接触地面，不基于任何人体模型，完全 data-driven 的。

<br>



#### Methods

这里只讲一下整个网络的推理过程，下面的是网络结构，2D 的关节点作为输入，作者用的是 openpose 的结果，并且会将关节点的 confidence 组成向量输入到网络，注意网络的输入是一串时间 T 内的 keypoint。

{% asset_img 1.png %}

然后分为两个 encoder，如下图，$E_Q$ 采取的是 temporal 的卷积，然后分成三条 branch，分别预测每一帧的 joint rotations、global root position 和 foot contact labels；而 $E_S$ 首先会将 T 个时间的输入整合到一起再采取普通的卷积操作，输出 bone length，因为对于一个人来说，骨骼长度是固定的，并不需要时间的信息，也不会每一帧会有不同的骨骼长度，所以也不需要 temporal 卷积。输出的 bone length 相当于预测了 SMPL 里面的 shape 参数，表征一个人的高矮胖瘦，然后会用预测的 bone 生成一个 T-pose。

{% asset_img 2.png %}

FK(forward kinematics) 模块的输入是 T-pose、joint rotation、root position，$\tilde{s}_{init} \in R^{3J}$ 相当于这个人的初始 pose，J 为关节点数，然后每一帧，将 T-pose 根据 joint rotation $\tilde{q}^t \in R^{4J}$ 进行旋转，关节旋转用四元数表示。

{% asset_img 3.png %}

对于每一帧，J 个关节点遵循从 root 到 leaf 的规则，$\tilde{P}_n^t=\tilde{P}^t_{parent(n)}+R^t_n\tilde{s}_n$，$\tilde{P}_n^t \in R^3$ 是第 n 个关节点在时间 t 的位置，关节 n 的 parent 指的是骨骼关系里的 parent，比如手肘是手腕的 parent， $R_n^t$ 是第 n 个关节点的旋转矩阵，$\tilde{s}_n \in R^3$ 是第 n 个关节点相对其 parent 的offset，手腕的位置等于手肘的位置加上旋转乘以手腕相对手肘的位置偏移

{% asset_img 4.png %}

最后再加上根节点的移动

{% asset_img 5.png %}

上面是整个推理流程，在训练阶段，从网络图看出是有一个判别器的，主要用来判别动作是否真实，但它并不是直接拿 joint rotation 输入到判别器，而是拿相邻 rotation 的差值，相当于求了一个差分。其他的 loss 包括 joint rotation loss、bone length loss、root position loss、foot contact loss

<br>

模型结果可以看下论文主页和油管视频

- https://rubbly.cn/publications/motioNet/
- https://www.youtube.com/watch?time_continue=2&v=8YubchlzvFA&feature=emb_logo

<br>