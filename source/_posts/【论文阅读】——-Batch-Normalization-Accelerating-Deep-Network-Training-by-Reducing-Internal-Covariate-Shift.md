---
title: >-
  【论文阅读】—— Batch Normalization, Accelerating Deep Network Training by Reducing
  Internal Covariate Shift
date: 2018-12-01 18:26:49
mathjax: true
tags:
---

> Title：Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift
> Authors：Christian Szegedy, Sergey Ioffe
> Session：ICML, 2015
> Abstract：Inception 的作者提出 batchNorm，从而解决 internal covariate shift 的问题，使得网络训练更快，并在自己的 Inception 网络上加入 batchNorm，使得在 ImageNet 上的错误率进一步降低，该网络被称为 Inception-BN。



<!-- more -->

<br>

*preface*

为什么深度神经网络**随着网络深度加深，训练起来越困难，收敛越来越慢？**这是个在深度学习领域很接近本质的好问题。很多论文都是解决这个问题的，比如 ReLU 激活函数，再比如 Residual Network，batchNorm 本质上也是解释，并从某个不同的角度来解决这个问题的。 

<br>

# Internal Covariate Shift

深度网络难以训练的原因在于，随着前向传递，每一层输入数据的分布都在改变，一个层的输入受前面所有层的参数的影响，细微的改变会被放大，由于每个层的输入数据分布不断在改变，网络需要不断地去学习新的分布。

covariate shift 是用来描述一个学习系统的输入数据的分布发生变化。而其实 covariate shift 也可以用来描述一个子网络，即一个子网络的输入分布发生变化，它同样也会经历 covariate shift，而在神经网络里面，我们把一个层的输入数据分布发生变化叫做 internal covariate shift。

batchNorm 的思想来自于：之前的研究表明如果在图像处理中对输入图像进行白化（Whiten）操作的话——所谓**白化**，**就是对输入数据分布变换到0均值，单位方差的正态分布**——那么神经网络会较快收敛。因此作者推论：图像是深度神经网络的输入层，做白化能加快收敛，那么其实对于深度网络来说，其中某个隐层的神经元是下一层的输入，即深度神经网络的每一个隐层都是输入层，只是相对下一层来说而已，那么能不能对每个隐层都做白化呢？

而 batchNorm 的基本思想就是：把网络中每个隐层节点的**激活函数输入分布**固定下来，就可以避免 internal covariate shift。为什么需要固定激活函数输入分布呢？我们知道激活函数，如 sigmoid 函数，它的两边都是梯度饱和区域，只有 0 附近的区域是有梯度的，因此如果能将 sigmoid 的输入固定到 0 附近，那么就可以解决梯度的问题。


<br>
# Batch Normalization

上面提到，batchNorm 就好比 whitening，将激活函数的输入进行标准化，从而使它的均值为 0，方差为 1。需要注意的是，标准化的对象是某一隐层的一个神经元，假设某一隐层有 d 个神经元，则分别对每个神经元进行下面的标准化操作。均值和方差是一个 batch 数据在该隐藏层神经元输出的均值和方差。

$$x^k = Wu^k +b$$

$$\hat{x}^k = \frac{x - E(x^k)}{\sqrt{Var(x^k)}}，k \in \{1,2, ..., d \}$$  

所以经过 batchNorm，每个激活函数的输入数据分布都会是一个标准正态分布，即解决了上面提到的 internal covariate shift 的问题。但是进行这样的标准化可能会影响网络表达的能力，因为数据分布被强行拉到了 0 附近，因此作者将标准化后的数据再进行一个 scale 和 shift 的操作，$\gamma$ 和 $\beta$ 是两个学习的参数，且每个神经元都有对应的 $\gamma$ 和 $\beta$。如果 $\gamma^k = \sqrt{Var(x^k)}$，$\beta^k = E(x^k)$，那么就会将它重新映射到原来的分布。

$y^k = \gamma^k \hat{x}^k + \beta^k$

对于一个普通的前向传播网络，batchNorm 层加到激活函数前，如下图

{% asset_img bn.png %}

算法的流程如下

{% asset_img bn1.png %}


上面提到的算法是训练时候的算法，训练的时候我们是用 mini-batch 的方式去训练一个网络，但是预测的时候，我们是一个一个 example 的去预测，我们不希望一个预测的样本受到其它样本的干扰（训练的时候，batch 内样本互相影响）。作者的解决方法是：预测的时候固定均值和方差，而这个固定的均值和方差是由训练阶段各个 batch 的均值和方差的一个统计量，简单来说就是，用各个 batch 的均值和方差的平均值作为预测阶段的均值和方差。



**CNN 上的 batchNorm**

- 在 feature map 上进行标准化，一个 feature map 类比一个隐层的神经元。
- 前向传递网络中一个 batch 大小为 m，而在 CNN 中做 batchNorm，假如 feature map 的大小为 $p \times q$，那么就相当于 batch size 为 $m \times p \times q$。
- 简单来说就是，将 batch 里面同一层的 feature map 进行归一化，归一化的时候，不以 feature map 为一个样本，而是以 feature map 的一个 pixel 为一样本，再求 pixel 级别的均值方差。
- 不同的 channel 相当于隐层不同的神经元，所以每个 channel 的feature map 对应一个 $\gamma$ 和 $\beta$。



batchNorm 的好处

- 正则作用，训练时候，batch 里面的样本不再是没有关系的，而是相互影响。
- 增快训练效率，batchNorm 能够解决梯度的问题，因此学习率可以稍微调大一点，加快训练速度。


<br>
# Inception-BN

这个版本相对于 Inception-v1，除了在每一个激活函数前加入了 batchNorm 之外，还进行了下面的调整

- 跟 VGG 一样，Inception block 里面的 $5 \times 5$ 的卷积核用连续两个 $3 \times 3$ 来代替。
- 在 Inception block 里面的池化层，不仅仅使用 max-pooling，还会使用 averaged-pooling。
- 在 Inception-v1，会在 Inception block 之间添加 max-pooling 进行降维。这种结构将会被去掉，而在 concatenation 之前，会用一个 stride 为 2 的卷积层和池化层进行降维。

{% asset_img net.png %}

由于 batchNorm 能够加快网络的训练，因此作者在训练上还进行了以下的调整

- 增大学习率
- 去除 dropout
- 减小 L2 正则项参数
- 加快学习率 decay，因为网络训练加快了
- 去除 LRN

作者根据这些设置训练了好几个网络

- Inception：即 Inception-v1，初始学习率为 0.0015.
- BN-Baseline：即 Inception-BN
- BN-x5：Inception-BN，初始学习率为 5 倍，即 0.0075.
- BN-x30：跟 BN-x5 一样，初始学习率为 0.045.
- BN-x5-Sigmoid：跟 BN-x5 一样，换成 sigmoid 激活。

从下面结果可以看出，加入 batchNorm 之后能够明显加快网络训练，增大学习率使学习更快，BN-x30 比 Inception 快了接近 16 倍，并且准确率更高。注意的是：BN-x5-Sigmoid 也能达到很好的结果，是因为 batchNorm 解决了梯度的问题，如果没有 batchNorm，作者说准确率不及千分之一。

{% asset_img res1.png %}
{% asset_img res2.png %}


<br>