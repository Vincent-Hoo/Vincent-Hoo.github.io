---
title: 【论文阅读】—— Wide Residual Networks
date: 2018-12-03 19:17:39
mathjax: true
tags:
---

> Title：Wide Residual Networks
> Authors：Sergey Zagoruyko, Nikos Komodakis
> Session：BMVC, 2016
> Abstract：传统的 ResNet 都是以深度为主，作者从宽度（channel 大小）出发，提出了通过拓宽网络的 channel 数，也能达到和深层 ResNet 一样的效果，且训练更快。

<!--more-->

<br>

*preface*

ResNet 的提出证明了深层网络的强大，但是随着网络不断地加深，有时候为了少量的精度需要把网络层数翻倍。且太深的网络会导致特征的重用减小，即梯度在反向传递的，只会从 skip-connection 中传递，而不会往 residual block 中走，只有很少的 block 能学到有用的信息，且 block 之间的共同信息很少。并且作者认为 ResNet 最强大的地方在于 residual block，而不在于深度，深度只是 ResNet 强大之处的一个补充而已。因此，**作者从“宽度”的角度入手，提出了 WRN，16 层的 WRN 性能就比之前的 ResNet 效果要好**。 


<br>
# WRN

下图的 basic 和 bottleneck 是原始 ResNet 提出的两种 residual block，bottleneck 会在深层的 ResNet 中用到，主要用来减少参数和计算量。注意一点，BN 和 ReLu 在卷积层之前，对应了 ResNet-v2 的 full pre-activation。

{% asset_img 1.png %}

作者基于 basic 的 residual block，引入两个因子：深度因子 $l$ 和 宽度因子 $k$，深度因子表示一个 residual block 里卷积层的层数，宽度因子表示卷积核的数量翻倍的倍数，如原来的卷积层为 $[3 \times 3, 16]$，表示 16 个 $3 \times 3$ 的卷积核，假如 $k = 2$，那么卷积层则为 $[3\times 3, 16 \times 2]$，即增加了卷积核的数量，拓宽了 feature map 的个数，或 channel 数。

可以看出，basic 的 residual block 对应 WRN 中 $l = 2$、$k = 1$，WRN 整体的网络结构如下

{% asset_img 2.png %}

作者再从下面几个方面去探索 WRN 的性能

- residual block 中卷积的类型：$B(M)$ 表示 block 中卷积的类型，如 $B(3,1)$ 表示先一个 $3 \times 3$ 的卷积，接着一个 $1 \times 1$ 的卷积
- 深度因子 $l$：在改变深度因子的时候，保持网络整体参数量不变，即增大深度因子，block 数量就减少。
- 宽度因子 $k$：WRN-n-k 表示网络共有 n 层，宽度因子为 k。
- residual block 加入 dropout：在卷积之后加入 dropout。

<br>

# 实验结果

**block 的卷积类型 & 深度因子 $l$**

- 参数量和准确率都差不多，因此选择 $B(3,3)$
- 不同深度，如 $B(3,3,3)$ 和 $B(3,3,3,3)$ 都不及 $B(3,3)$ 

{% asset_img 3.png %}



**residual block width**

- k 增大，错误率下降，depth 增大，错误率不一定会下降，可以看 WRN-40-8 与 WRN-22-8

{% asset_img 4.png %}


**对比 ResNet**

- WRN 比 ResNet-1001 都要好 0.92%
- 且 WRN 训练要比 ResNet 快很多，尽管 WRN 的参数要比 ResNet 要多

{% asset_img 5.png %}


**dropout的作用**

{% asset_img 6.png %}



<br>

*In Conclusion*

网络的深度和宽度都可以使得网络性能变好，之前的工作大多集中在加深网络层数，网络的宽度也是一个不可小看的角度。




<br>