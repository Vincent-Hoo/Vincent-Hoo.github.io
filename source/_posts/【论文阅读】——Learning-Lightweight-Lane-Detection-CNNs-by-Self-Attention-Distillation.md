---
title: >-
  【论文阅读】——Learning Lightweight Lane Detection CNNs by Self Attention
  Distillation
mathjax: true
date: 2019-08-30 21:42:31
tags:
---

> Title：Learning Lightweight Lane Detection CNNs by Self Attention Distillation
> Author：Yuenan Hou, Zheng Ma, et al.
> Session：ICCV, 2019
> Abstract：作者提出了一种新的蒸馏方式：自注意力机制蒸馏网络，该蒸馏不需要教师网络作为监督信息，而是将自身网络深层的 attention map 作为浅层的监督信息，从而指导浅层网络的学习。



<!--more-->



*preface*

本篇论文将聚焦 Lane Detection task，该任务用于自动驾驶领域，给定一张图片，需要判断出有多少条路径，且路径的位置在哪，本质上是一个图像分割的任务。而为了降低分割的难度，在网络中设计出另外一条 path 来判断图像中是否存在 lane。

<br>

## Self-Attention Distillation(SAD)

类似 attention transfer 论文，提出了三种 attention map 的计算方式

- $G_{sum}(A_m)=\sum_{i=1}^{C_m}|A_m|$
- $G_{sum}^p(A_m)=\sum_{i=1}^{C_m}|A_m|^p$
- $G_{max}^p(A_m)=\max_{i=1,C_m}|A_m|^p$

网络的整体框架如下，AT-GEN 模块将计算到的 attention map 再经过 bilinear upsampling 后，通过一个 spatial softmax。

{% asset_img 1.png 650 650 %}



在原有的分割损失上，加上自注意力蒸馏损失，$M$ 为网络层数。
$$
L_{distill}=\sum_{m=1}^{M-1}L(AT-GEN(A_m),AT-GEN(A_{m+1}))
$$
训练过程为先无蒸馏情况下，预训练网络，然后再加入 SAD，下面为一些可视化结果。

{% asset_img 2.png 650 650 %}



<br>



## 实验结果

下图为 SAD 在 TuSimple 和 BDD100K 两个数据集的结果，SAD 都取得最好的结果。

{% asset_img 3.png 500 500 %}

{% asset_img 4.png 500 500 %}



一些 ablation study 如下，作者比较不同的 path，比如让 block1 去 mimic block4，或者 block2 去 mimic block3等。实验结果表明，浅层网络最好不用 SAD，因为浅层网络挖掘低层特征，不需要深层网络信息的指导。而近邻 mimic 要比跨层 mimic 要好。

{% asset_img 5.png 500 500 %}

作者还尝试了在不同时间点引入 SAD，虽然最后收敛的结果都近似，但是预训练越久，attention map 越能表征信息，对于 SAD 的作用就越大，收敛也就越快。 

{% asset_img 6.png 500 500 %}

<br>