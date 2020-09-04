---
title: 【论文阅读】—— Regularizing Class-wise Predictions via Self-knowledge Distillation
mathjax: true
date: 2020-06-24 16:18:09
tags:
---

>Title : Regularizing Class-wise Predictions via Self-knowledge Distillation
>Authors : Sukmin Yun, Jongjin Park, et al
>Conference : CVPR 2020
>Abstract : 作者提出了一个简单的自蒸馏策略，通过约束相同类别的样本要输出尽可能相似的结果，得到不错的提升。

<!--more-->

<br>

## Class-wise Self-knowledge Distillation (CS-KD)

在之前的文章里面也有提到，蒸馏更像是对网络的一个正则项，这里的 CS-KD 同样也是作为一个正则的方法，通过约束相同类别的样本之间的输出，从而减少类别内样本输出的方差

{% asset_img 1.png 500 500 %}

上图同样为鸟类别的两张图片，同时输入到一个网络，然后约束他们的输出概率要尽可能地类似，这里相当如引入了一个虚拟教师网络，而这个教师网络就是学生网络本身，并且和一般的 KD 不同点在于，两个网络的输入不同，下面是自蒸馏损失，$\tilde{\theta}$ 是 $\theta$ 的一个 copy，$x^\prime$ 和 $x$ 具有相同标签。

{% asset_img 2.png %}

整体的训练流程如下，对于每一个 batch 的图片，会抽样出另一个 batch，并且拥有相同的标签

{% asset_img 3.png %}

<br>

## Effects of class-wise regularization

作者提出的 CS-KD 能够实现两个目标

- 避免 over-confident 的预测，因为它拿其他样本的输出作为软标签，这比 label smoothing regularization 构造的人工软标签要更加真实。对于误分类的样本，用 CS-KD 训练的网络，会在正确类别上有更加高的概率

{% asset_img 4.png %}

{% asset_img 5.png %}

- 减少类别内输出概率的方差，这也是损失函数最直接的目标

<br>

## Experiments

#### 对比其他的正则化方法

{% asset_img 6.png %}

#### 对比其他自蒸馏的方法

- DDGSD : data-distortion guided self-distillation 是一种一致性正则，约束不同数据增广版本的图片输出一致
- BYOT : be your own teacher 用深层特征指导浅层特征

{% asset_img 7.png %}

#### 和其他方法融合

这里做了和 mixup 还有 kd 融合的结果，均是可以与其他方法相容的

{% asset_img 8.png %}

{% asset_img 9.png %}

<br>