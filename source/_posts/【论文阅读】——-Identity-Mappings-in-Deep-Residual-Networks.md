---
title: 【论文阅读】—— Identity Mappings in Deep Residual Networks
date: 2018-12-01 00:43:15
mathjax: true
tags:
---


> Title：Identity Mappings in Deep Residual Networks
> Authors：Kaiming He et al
> Session：ECCV, 2016
> Abstract：作者在 ResNet-v1 基础上进行了修改，使得表现得到了提升，该网络称为 ResNet-v2.



<!-- more -->

<br>

# ResNet Analysis 

ResNet 可以看成是由一个个的 residual unit 堆叠而成，每一个 unit 可以看成下面的两个计算。$W_l$ 看成是多个卷积核，在 ResNet-v1 里面，$h(x_l) = x_l$ 是 identity mapping，$f$ 函数是 ReLu。

$$y_l = h(x_l) + F(x_l, W_l)$$

$$x_{l+1} = f(y_l)$$

假设 $h$ 和 $f$ 都是 identity mapping，那么上面两个公式可以写成下面的形式。

$$x_{l+1} = x_l + F(x_l, W_l)$$

所以某一个 unit 的输出可以由前面某一 unit 的值，加上各个unit 的函数值。

$$x_L = x_l + \sum_l^{L-1} F(x_i, W_i)$$

这里可以看出 residual network 和 plain network 一个很大的不同，resnet 的输出是每一个 unit 的 $F(x, W)$ 的相加；而 plain network 的输出是一系列矩阵的相乘（忽略 ReLu 和 Batch Normalization）。

因此损失函数 $\epsilon$ 对于某一个 unit 的梯度就能写成下面的形式。梯度可以分解为两项，一项是直接从 loss 不经过任何 weight 传到 $x_l$，而另外一项是经过 weight 来传播；另外一点是，括号里不可能为 0，因为后一项不可能总是 -1。

$$\frac{\partial \epsilon}{\partial x_l} = \frac{\partial \epsilon}{\partial x_L}\frac{\partial x_l}{\partial x_l} = \frac{\partial \epsilon}{\partial x_L}(1 + \frac{\partial}{\partial x_l}\sum_l^{L-1}F(x_i, W_i))$$ 

**注意：** 在resnet-v1，h 函数不一定等于恒等函数，当出现维度不匹配的时候，就不是恒等函数，但是作者认为 projection 在整个 100 多层的 resnet 中也只出现了几个，其他的都是 identity mapping，因此 projection 不会产生太多的影响，所以默认 h 等于恒等函数，来进行后面的推导。

从上面的推导可以看出，信号可以直接从一个 unit 传播到另外一个 unit，不管是前向传递还是反向传播。而这基于两个条件：(1) h 函数是恒等函数，(2) f 函数也是恒等函数。

下面将会尝试不同的 h 和 f 函数，进而观察网络性能的变化。

<br>

# Choice for $h$ function

作者共提出了 6 种 h 函数（包括原始的 identity mapping），如下图。为了简单起见，省去了 BN 层和激活函数。下面在做测试的时候，固定了 f 函数是 ReLu 激活。

{% asset_img h.png %}

结果如下：

- constant scaling：这相比于 identity mapping 多了一个乘法算子 $\lambda$，因此在前向和反向传递的时候，这个 $\lambda$ 都会累乘，呈指数型增长或下降。因此结果很差。
- exclusive gating 和 shortcut-only gating：类似于门电路，效果也一般。
- conv shortcut：类似于 resnet-v1 的 projection shortcut 的 C 策略，效果之所以没有 resnet-v1 的好，是因为现在网络深度更深了。
- dropout：效果也不好。

{% asset_img result_h.png %}

综上所述：original 的效果最好，接下来也会直接用 original，相当于直接的 identity mapping。

<br>

# Choice for $f$ function

在第一小节中提到，当 $f$ 函数是 identity mapping 的时候，信号可以在前向传播和反向传递中直接体现；现在我们需要将 $f$ 函数变成 identity mapping，就需要将 ReLu 和 BN 进行重新排布。如下图

{% asset_img f.png %}

- original：$f$ 函数就是 ReLu 激活函数。
- BN after addition：$f$ 函数相当于 BN 加 ReLu。
- ReLu before addition：将 original 版本的 ReLu 函数放到了 $F(x,W)$ 里面，这时 $f$ 函数就完完全全变成一个恒等函数了。但是这样存在的问题是：$F(x,W)$ 的输出是非负的，而理论上 $F(x,W)$ 的输出应该是全实数。因此前向传递的信号是单调上升的，因此也影响了整个网络的性能。
- 后面的两个都属于 pre-activation

**Post-activation or pre-activation?** 

在 original 版本里面，ReLu 函数既作用在 x 这条通道上，也作用在 $F(x,W)$ 上，这种我们称为对称 activation。

$$y_{l+1} = x_l + F(x_l, W_{l+1}) = f(y_l) + F(f(y_l), W_{l+1})$$ 

而作者提出了一种非对称的 activation，即 ReLu 只作用到 $F(x,W)$ 上，不作用到 x 上，我们令这个非对称 activation 的函数为 $\hat{f}$，因此

$$y_{l+1} = x_l + F(x_l, W_{l+1}) = y_l + F(\hat{f}(y_l), W_{l+1})$$

变换一下变量

$$x_{l+1} = x_l + F(\hat{f}(x_l),W_l)$$

然后我们可以将 post-activation 转成 pre-activation，如下图

{% asset_img pre_ac.png %}

而根据 pre-activation 的内容又分了两种类型 ReLu-only pre-activation 和 full pre-activation。

实验的结果如下图，可以看出 full pre-activation 的效果最好。

{% asset_img result_f.png %}

<br>

# Analysis

作者将最优的 $h$ 函数和 $f$ 函数构造的 resnet 和其他模型进行对比，发现性能还能提升。

{% asset_img 1.png %}

pre-activation 有两点好处

1. 网络更容易优化，因为 $f$ 函数是 identity mapping。
2. pre-activation 的 BN 给模型带来不少正则。

ResNet-v1 虽然也有 BN 层，但是 BN 完之后马上和信号相加，因此相当于没有标准化。

<br>

*In Conclusion*

作者将 ResNet-v1 进行了修改，探索了不同的 shortcut，以及 mapping function，最后得出更加好的结果。


<br>