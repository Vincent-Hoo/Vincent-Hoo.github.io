---
title: 3D Human Pose and Shape Reconstruction Review
mathjax: true
date: 2020-09-09 15:04:27
tags:
---

该文章总结了一些三维人体姿态估计和重构的文章，分别为：
- HMR：End-to-end Recovery of Human Shape and Pose
- SPIN：Learning to Reconstruct 3D Human Pose and Shape via Model-fitting in the Loop
- VIBE：Video Inference for Human Body Pose and Shape Estimation

<!--more-->

## End-to-end Recovery of Human Shape and Pose

> Authors: Angjoo Kanazawa
> Conference: CVPR 2018

该模型的输入为一张图片，或者说是一个 bounding box，bbox 中心为人，输出是重构的三维人体。图片首先经过一个 encoder 提取特征，encoder 用的是在分类任务上预训练好的 ResNet50，特征输入到 regressor 里面经过几轮的迭代最后预测一系列参数，包括 SMPL 模型的参数 $\theta$ 、$\beta$ 以及 weak-perspective camera model 的参数 scale、rotation、translation。

输入的图片均有二维关节点的 annotation，从 regressor 得到的参数中生成三维人体 mesh（$M(\theta,\beta)\in R^{3\times N}$，N=6890），然后从 mesh 中通过线性回归得到三维关节点 $X(\theta, \beta)\in R^{3 \times P}$，P 为关节点个数，相机参数 $s\in R, t\in R^2$，从 3 维关节点经过投影生成 2 维的关节点，R 为 global orient，$\Pi$ 为 orthographic projection。
$$
\hat{x}=s \Pi(RX(\theta, \beta))+t
$$
由于每张图片都有 2 维关节点，通过这种方式，约束生成的 3 维关节点经过投影后与 2 维关节点相似，从而训练网络。由于 SMPL 的关节点和 2 维不同数据集的关节点数有一些不同，这里作者做的是将不同数据集融合，保证 SMPL 的点在 2 维关节点的 annotation 中有。
$$
L_{reproj}=\sum_i ||v_i(x_i-\hat{x}_i)||_1
$$
除了每张图片都有 2D supervision 外，有一些图片会有 3D 的 annotation，一般 3D 的 annotation 会以 3D keypoint 的形式给出，用 MoSh 工具可以从 3D keypoint 中得到 SMPL 的参数 $\theta$ 和 $\beta$，3D supervision 的 loss 如下，joint loss 加上参数 loss
$$
L_{3D} = ||X_i-\hat{X}_i||_2^2 + ||[\beta_i, \theta_i] - [\hat{\beta}_i, \hat{\theta}_i]||_2^2
$$
除了一般的 reprojection loss 外，以前的文章都会加入一些人为 的先验，来约束 SMPL 的 $\theta$ 参数来避免一些不自然的动作或者关节点扭动，这里作者采取一种 data-driven 的先验方式，通过训练一个判别器，让网络自己去判断回归出来的 SMPL 参数是否像正常人，首先需要一个 3D 人体的数据库（每个人体的 SMPL 参数都有），然后判别器的网络结构方面，采取的是公用前面的 feature extraction，后面每个关节点逐一判别。

训练网络总的 loss 如下
$$
L=\lambda(L_{reproj}+1L_{3D})+L_{adv}
$$
{% asset_img 1.png %}



训练用到的 2D 数据集有：LSP、LSP-extended、MPII、MS COCO，3D 数据集有：Human3.6M、MPI-INF-3DHP。由于 SMPL 的 23 个关节点和用到的数据集不是很 match，因此用一个 regressor 从生成的 mesh 中回归出 Human3.6M 的14 个点，同样的方法加上 MS COCO 脸上的 5 个点，共 19 个点用于 reprojection loss。

<br>

## Learning to Reconstruct 3D Human Pose and Shape via Model-fitting in the Loop

>Authors: Nikos Kolotouros
>Conference: ICCV 2019

作者结合了 regression-based 和 optimization-based 两种方法，整体的思路就是先用 HMR 的网络回归出来一个初步的人体模型（下图右），然后再用这个人体去初始化 smplify 模块对人体模型进行优化，并用优化后的人体模型作为 GT 去监督回归网络的训练。

{% asset_img 2.png %}

回归网络和 HMR 几乎一样，唯一不同是 HMR 用欧拉角来表征每个关节点的旋转，而 SPIN 用一个 6 维的向量来表征三维旋转。得到的初步三维人体模型再用 smplify 进一步优化。

>SMPLify tries to fit the SMPL model to a set of 2D keypoints using an optimization-based approach.

优化的目标函数包含 reprojection loss 还有一些 pose 和 shape 的 prior，比如惩罚肘部和膝盖不自然的旋转，实现上 smplify 分为两步，第一步是固定 initial pose 和 shape，优化 camera translation 和 body orientation，简单说就是调整人体的整体转向；第二步是优化 pose 和 shape。通过 iterative fitting，得到优化后的人体模型 

SPIN 和上一篇 HMR 不同的点在于，它没有直接用 reprojection loss 来训练回归网络，而是用优化后的人体模型参数来作为回归网络输出的 GT，smplify 后的 SMPL 参数为 $\Theta_{opt}=\{\theta_{opt}, \beta_{opt} \}$，回归网络输出的 SMPL 参数为 $\Theta_{reg}=\{\theta_{reg}, \beta_{reg} \}$；同理 smplify 后的 mesh 为 $M_{opt}$，损失函数包括
$$
L_{3D}=||\Theta_{reg}-\Theta_{opt}||
$$

$$
L_M=||M_{reg} - M_{opt}||
$$

SPIN 的一大特点在于 self-improving，A good initial network estimate $\Theta_{reg}$ will lead the optimization to a better fit $\Theta_{opt}$, while a good fit from the iterative routine will provide even better supervision to the network. 对比上一篇的 HMR，SPIN 完全不需要用到 3D 的数据进行训练，并且对比 HMR 训练判别器告诉回归网络生成的 pose 是否正常，SPIN explicitly 用 smplify 提供更加有效的 pose 信息作为 supervision。

训练集有用到 Human3.6M、MPI-INF-3DHP、LSP 并且混合了其他 2D 数据集如 MPII 和 COCO，2D annotation 方面混合所有数据集的关节点，总数为 24 个点，对于某一张图片上没有标注的点，标记 visibility 为 0，表示不可见。

<br>

## VIBE: Video Inference for Human Body Pose and Shape Estimation

> Authors: Muhammed Kocabas
> Conference: CVPR 2020

整体流程如下，给定一个 T 帧的视频 $V=\{I_t \}_{t=1}^T$，temporal generator 对输入的每一帧图片进行 CNN 提取信息，GRU 处理时序信息，最后 regressor 回归出 SMPL 的参数，对于 $\beta$ 参数，取 T 帧的平均值作为整体的 shape，generator 最后的输出 $\hat\Theta = [(\hat\theta_1,...,\hat\theta_T), \hat\beta]$，与 AMASS（一个 3D 人体模型的库）的真实人体 $\Theta_{real}$ 输出到判别器进行动作的判定。

{% asset_img 3.png %}

**Temporal Generator**

整体的网络结构和 SPIN 基本一样，只是在 CNN 和 Regressor 之间插入了一个 GRU 的模块，SPIN 是单帧处理的，这里加入了 temporal 的信息，因为前面帧的信息对后面帧的处理有帮助，同样用的 6D 旋转表示，生成器的损失项如下，看不同的数据有哪些 annotation 来决定有哪些损失项，2D 损失项是必定有的，reprojected 2D keypoint 和之前的方法一样
$$
L_G=L_{3D} + L_{2D} + L_{SMPL} + L_{adv}
$$

$$
L_{3D} = \sum_{i=1}^T||X_t - \hat{X}_t||_2
$$

$$
L_{2D} = \sum_{i=1}^T||x_t - \hat{x}_t||_2
$$

$$
L_{SMPL} = \sum_{i=1}^T||\theta_t - \hat{\theta}_t||_2 + ||\beta-\hat\beta||_2
$$

**Motion Discriminator**

这里的判别器和 HMR 不同，同样加入了时序的信息，而不是一个单帧动作的判别器，而是一个序列动作的判别器，如下图所示，generator 生成的序列的 SMPL 参数会经过 GRU，然后再经过一个 self-attention 模块最后输入到一个线性层判定动作是否 valid

{% asset_img 4.png %}

训练数据和前两个论文的不同，由于 VIBE 不是单帧处理，而是处理视频数据，因此 2D 数据上用到的是 PoseTrack 和 PennAction，对于 3D keypoint 数据，用到的还是 MPI-INF-3DHP 和 Human3.6M，而 3DPW 和 Human3.6M 提供的 SMPL annotation 也同样用来训练，AMASS 用来训练判别器。



*In Conclusion*

三篇文章都是用回归的方式去学习 SMPL 的参数，也有加入对抗学习来作为先验，VIBE 考虑到了 temporal 的信息，但依旧有很多值得改进的地方。



## Reference

- [HMR paper](https://openaccess.thecvf.com/content_cvpr_2018/papers/Kanazawa_End-to-End_Recovery_of_CVPR_2018_paper.pdf)
- [SPIN paper](https://arxiv.org/pdf/1909.12828.pdf)
- [SPIN Github](https://github.com/nkolot/SPIN)
- [VIBE paper](https://arxiv.org/abs/1912.05656)
- [VIBE Github](https://github.com/mkocabas/VIBE)



<br>