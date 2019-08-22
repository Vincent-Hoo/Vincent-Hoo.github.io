---
title: 【论文阅读】—— Densely Connected Convolutional Networks
date: 2018-12-04 23:23:55
mathjax: true
tags:
---

> Title：Densely Connected Convolutional Networks
> Author：Gao Huang, Zhuang Liu, et al.
> Session：CVPR, 2017
> Abstract：作者发现 ResNet 的结构有些许冗余，因此提出要将网络中的每一层都相互相连，而不是像 ResNet 那样只是 block 之间连接。作者提出的这种 DenseNet 结构参数量比 ResNet 要少，网络层数也比 ResNet 要少，但是效果却比 ResNet 要好。



<!--more-->

<br>

*preface*

作者黄高在 “Deep networks with stochastic depth” 论文中提出，ResNet 层数太多，大部分的残差块只能提高少量的信息，所以在 ResNet 基础上随机丢弃一些层，发现可以提高 ResNet 的泛化能力。 因此作者觉得网络中某一层的输入不仅仅只依赖于它上一层的输出，而依赖于更加上层的输出。由于随机丢弃某些层反而导致泛化能力变强，这意味着 ResNet 有明显的冗余，并不是每一层都需要，网络中的每一层只能提取很少量的特征。

基于上面的推断，作者提出**让网络中的每一层和前面的所有层相连**，同时把每一层设计的比较窄（即 feature map比较小），从而使每一层学到的特征变少来降低冗余。虽然网络每一层都相互连接，但是整体上网络的参数并没有比 ResNet 要多，因为每一层网络变简单了。除了参数变少外，DenseNet 的结构使得信号的前向传播和梯度的反向传递更加容易。



<br>

# DenseNet

ResNet 中每一个 residual block 可以看成以下的式子，$H_l()$ 表示第 $l$ 个 block 的 residual function，按照 ResNet-v2，应该是 BN-ReLu-Conv。

$$x_l = H_l(x_{l-1}) + x_{l-1}$$

作者认为相加可能会阻碍信息的流动，因此作者提出的 DenseNet 不仅将前面层的输出作为后面层的输入，merge 方式也采取 concatenation 而不是 summation，因此 DenseNet 中某一层可以看成下面该式子

$$x_l = H_l([x_0, x_1, ..., x_{l-1}])$$

上面的式子成立的前提是：$x_0, x_1,...,x_{l-1}$ feature map 的大小相同，然后在 channel 维度上拼接，有点像 Inception 中的 split-transform-merge 的思想。为了能够 concatenation，作者将网络分为两个 component：transition layer 和 dense block，如下图。

- transition layer：由一个 $1\times 1$ 的卷积层和一个 $2\times2$ 的 avg-pooling 层组成，pooling 层的 stride 为 2。transition layer 的作用在于降维，每次将 feature map 的大小减半。
- dense block：由一系列的 BN-ReLu-Conv 层组成，每一个 Conv 的输出都是相同大小，前面提到 transition layer 的作用在于降维，因此在 dense block 中 feature map 的大小会维持不变，Conv 采用的是 SAME 卷积。

{% asset_img 2.png %}



**Growth rate**：dense block 中每一层的卷积层输出不仅 feature map 大小一样，channel 大小也是一样，作者把 channel 大小称为 growth rate $k$，因此第 $l$ 层的输入的 channel 大小为 $k_0 + k \times(l-1)$，而输出的 channel 大小依旧为 k，下图表示的是一个 dense block，有五层，k = 4

{% asset_img 1.png %}

作者的实验表明，k 不用很大效果就很好了。因为每一层都可以访问前面所有层的 feature map，整个网络相当于有一个 global state，每个层都添加 k 个 feature map 到这个状态里面；而 k 约束了添加信息的多少；这个 global state可以从网络的任何地方访问得到，而不像传统的网络一样，每一层都需要 replicate 一下。



作者实现了几个版本的 DenseNet

- DenseNet-A：dense block 的结构为 BN-ReLu-Conv(3 $\times$ 3)
- DenseNet-B：dense block 的第 m 层的结构变为 BN-ReLu-Conv((m-1)k, 1 $\times$ 1, 4k)-BN-ReLu-Conv(4k, 3 $\times$ 3, k)，添加了一个而 bottleneck layer，bottleneck 的输出大小为 4k
- DenseNet-C：transition layer 不仅可以将 feature map 的大小减半，还可以减少 channel 数量，作者把 $\theta$ 作为 compression factor，通过 transition layer 后，channel 的数量降为原来的 $\theta$，也就是说不断地减小 growth rate k，这种版本的 DenseNet 为 DenseNet-C。实验中 $\theta$ 取 0.5
- DenseNet-BC：结合 bottleneck layer 和 compression factor



网络结构如下：

{% asset_img 3.png %}



<br>

# 实验结果

作者分别在 CIFAR-10，CIFAR-100，SVHN 和 ImageNet 上进行实验；前三个数据集分别做了有数据增强和无数据增强的对比，无数据增强的作者用 dropout 来防止过拟合。结果如下，+ 号表示有数据增强，* 号表示自己跑的实验，粗体表示超过所有其它算法，蓝色表示最好的。

{% asset_img 4.png %}

主要对比的算法均为 ResNet 和它的变体，如 WRN 和黄高的那篇 “Deep networks with stochastic depth” 论文中的方法。

- DenseNet 的结果要比其它算法普遍要好
- 达到同等好的结果，参数少一点

{% asset_img 5.png %}



下面对比的是参数里量和错误率的关系，可以看出 DenseNet-BC 最为 parameter-efficient，而对比 DenseNet 和 ResNet，DenseNet 只需要三分之一的参数量即可达到 ResNet 的效果，尽管 ResNet 对比 VGG 和 AlexNet 已经有很大的提升。

{% asset_img 6.png %}

<br>

*In Conclusion*

DenseNet 在某种程度上和 stochastic depth 相似，因为 stochastic depth 是随机的丢弃某些 residual block 使得某一层能和上几层或者后几层相连，这是随机的；而 DenseNet 将这个随机变为确定，让所有的层互相连接。

<br>