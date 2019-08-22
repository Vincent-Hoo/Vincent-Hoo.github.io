---
title: 【论文阅读】—— Deep Residual Learning for Image Recognition
date: 2018-11-30 14:51:01
mathjax: true
tags:
---

> Title：Deep Residual Learning for Image Recognition
> Authors：Kaiming He et al
> Session：CVPR, 2016
> Abstract：这就是传说中的ResNet，作者提出了残差学习(residual learning)，解决了深层网络的退化问题(degradation)，并且破天荒地将网络层数加深到 152 层，并且揽获 ILSVRC 2015 的所有比赛冠军（分类，检测，定位）。



<!-- more -->

<br>

*preface*

深度神经网络已经在前几年 ILSVRC 中展示了它的威力，像 VGG 和 GoogLeNet 都将网络层数提升到了 20 层左右，但是作者发现随着神经网络的加深，网络会出现退化问题(degradation problem)，即训练误差不降反升，从下图可以看出，56 层的网络训练误差比 20 层的网络还高，这不是由于过拟合所致，2015 年已经有很多防过拟合的方法出现，如 batch normalization，dropout，各种初始化方式；且如果是过拟合的问题，那么训练误差理应更低，但是从下图可以看出，深层的网络训练误差和测试误差都比浅层网络要高，所以退化问题不是过拟合所致。

{% asset_img degradation.png %}

<br>

# Residual Learning

网络的退化问题表明，不是所有的网络都是容易优化的，起码深层网络比浅层网络难以优化和收敛。

> Identity mapping(function)：恒等映射（函数）是指输出等于输入的映射，即 $y = x$。

假如我们将上面例子的浅层网络（20层网络）通过在其后面叠加恒等函数（不是真的叠加，而是让后面的层去学习一个恒等函数，从而得到恒等函数的效果），我们就可以得到一个深层网络，理论上这个深层网络和浅层网络是等同效果的（如果能学习到恒等函数的话），深层网络可以看成是浅层网络的一个 counterpart，因此深层网络的错误率不应该比浅层网络要高。但是上面的实验也表明了，深层网络有退化问题，这个浅层网络的 counterpart 的错误率要比浅层网络本身要高。

因此作者提出猜想：**或许非线性层比较难学习出一个恒等函数**，因此连浅层网络的错误率都无法逼近，更不用说降低错误率了。

那么如何才能更加有效地学习恒等函数呢？作者提出了残差学习，只要能解决这个问题，就相当于能够解决网络的退化问题。

我们假设网络某几层非线性层（不一定要整个网络）需要学习的 mapping 为 $H(x)$，即潜在真实的 mapping 就为 $H(x)$。与其让这几层非线性层学习整个 $H(x)$，我们让它学习 $F(x)$，$F(x) = H(x) - x$，$F(x)$ 称为原映射 $H(x)$ 的残差；作者假设残差的学习要比原映射容易，一个极端的例子是：假设我们要学习的真实映射 $H(x)$ 就是一个恒等函数，即 $H(x) = x$，那么我们只需要学习让这几层非线性层输出 $F(x) = 0$ 即可；但是如果不是 residual learning，即上面提到的情况，让这几层非线性层直接学习 $g(x) = x$，这要比让这些非线性层学习 $g(x) = 0$ 要难，$g(x)$ 为这几层线性层学习到的映射。

<br>

# Why Resnets work?

考虑一下下面的这个网络，输出为：

$$a^{[l+2]} = Relu(z^{[l+2]} + a^{[l]}) = Relu(w^{[l+2]}a^{[l+1]} + b^{[l+2]} +a^{[l]})$$

假设由于正则或 weight decay 使得 $w^{[l+2]}a^{[l+1]} + b^{[l+2]} = 0$，那么 $a^{[l+2]} = a^{[l]}$，即一个恒等函数。

这表明 residual block 很容易会学到恒等函数，因此在下面的 big NN 后面添加这些 residual block，并不会 hurt 原来 NN 的能力，也就是说添加 residual block 至少不会使得性能下降，反而还会学到一些有用的信息使得性能上升。

而 plain network 当加深的时候，网络甚至连恒等函数都学不到，这也就是为什么网络加深，反而性能下降的原因。而 residual block 解决退化问题的最主要原因是，这些 residual block 能够保证不会 hurt performance，有时候还会 help performance。

{% asset_img 7.png %}

假设我们现在将网络的所有层都改为 residual block，即把上面例子的 Big NN 改成 residual block 的形式，首先这样改动，并不会改变这个 Big NN 的性能，因为假设几层非线性层能够逼近任意函数 $H(x)$，那么它肯定能够逼近 $H(x) - x$。因此基于 residual block 的 big NN 会学到和没有 residual block 的 big NN 一样的函数。在这基础上，根据上面的证明，在它后面加很多个 residual block，也不会 hurt performance，所以就可以解决退化问题了。



<br>

# Identity Mapping by Shortcuts

作者基于上面的理论和猜想，提出了 residual block，如下图

{% asset_img residual_block.png %}

作者称这种首尾相连的线为 shortcut 或者 skip-connection，它们是用来实现 Identity Mapping 的，一个 block 的方程如下，非线性层只需学习残差即可

$$(1)：y = F(x, \{ W_i\}) +x$$

这种 shortcut 连接并没有引入任何的参数和很多的计算量（除了 element-wise 加法），但是却能解决退化问题。

当 $F(x, \{  W_i\})$ 的输出与 $x$ 的维度不匹配的时候，我们还需要将输入 $x$ 进行 projection。

$$(2)：y = F(x, \{  W_i\}) +W_s x$$ 

公式 (1) 称为 Identity Mapping，而公式 (2) 称为 projection shortcut，一般上除非维度不匹配，否则都用 Identity Mapping 即可，没有太多必要引入额外的参数。

<br>

# ResNet结构

下图是 VGG-19、plain network 以及 residual network 的网络图。plain network 和 residual network 都去掉了全连接层，而改用了 global pooling，residual network 中的实线连接为 Identity mapping，而虚线连接为 projection shortcut。

在具体实现上，projection shortcut 有两种选择

1. 直接通过 zero padding，使得输入和输出的维度相同。
2. 通过上面的公式 (2) 用 $1\times 1$ 的卷积核将输入映射到输出。

{% asset_img 1.png %}



作者共实现了层数分别为18、34、50、101 和 152 的 resnet，网络结构如下图，通过不断地堆叠 residual block，注意一点，卷积操作都是采取 SAME 卷积，feature map 维度减小是在每一个 conv i_x 的开头通过设置 stride = 2 将其缩小一半，具体看上图的虚线连接的 block。

从下表可以看出随着层数的增多，resnet 的计算量并没有提升很多，而参数也是要比 VGG 等网络要少。

{% asset_img 2.png %}

作者在构建 50 层以上的 resnet 的时候，采用了 GoogLeNet 一样的策略，用一个 $1 \times 1$ 的卷积核作为 bottleneck，来减少参数和计算量。

{% asset_img bottleneck.png %}

<br>

# 实验结果

**实验设置：**

- 训练阶段：预处理时像 VGG 一样随机采样 [256, 480] 的最短边，然后随机 crop 出 $224 \times 224$ 的图像以及水平翻转，然后做mean substracted。 
- 测试阶段：预测时候使用 AlexNet 的10-crop测试法，最好的结果是跟从 VGG 中的全卷积后的 multi-scale 评估，scale为 {224, 256, 384, 480, 640}。



## Plain Network vs Residual Network

- 深层的 plain 网络的误差要比浅层的 plain 网络误差要高，表明出现退化问题。
- 作者分析了深层 plain 网络的前向激活和反向梯度值，都没有发生 vanish 的现象，证明不是梯度消失所导致。
- residual network 为了和 plain network作对比，用的是 Identity mapping 和 zero padding for dismatch dimension，这是为了不引入额外的参数。
- 对比 plain network，resnet-34 的错误率要比 resnet-18 要低，意味着退化问题可以用 residual learning 解决。
- 当网络不是很深的时候，如 18 层，resnet 和 plain network 差不多，但是 resnet 收敛要更快。

{% asset_img 3.png %}
{% asset_img 4.png %}



## Identity Mapping vs Projection Shortcuts

skip-connection 的三种设置

- A. zero padding for increased dimension；parameter-free Identity mapping for other shortcuts.
- B. projection shortcuts for increased dimension；parameter-free Identity mapping for other shortcuts.
- C. all shortcuts are projection. （输入输出维度相同的也用一个 $1 \times 1$ 的卷积核去处理）

结果表明 B 要比 A 稍微好一点，C 比 B 稍微好一点，但整体差不多，说明解决退化问题，projection shortcut 并不是必须。

{% asset_img 5.png %}



## Comparision with other models

- 深度越深的 resnet 准确率越高，且没有出现退化问题。
- 实验表明，resnet 要比其他模型要好，且参数和计算量都要少。

{% asset_img 6.png %}

作者还把 resnet 应用到了 CIFAR-10 分类上，且使用了 1000 层的resnet，错误率降低到 1% 以下，这表明 resnet 能够解决深层网络的退化问题。



<br>