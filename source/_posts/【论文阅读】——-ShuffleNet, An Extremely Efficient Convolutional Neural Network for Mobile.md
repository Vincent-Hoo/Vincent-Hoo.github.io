---
title: >-
  【论文阅读】——— ShuffleNet, An Extremely Efficient Convolutional Neural Network for
  Mobile
date: 2018-12-11 15:09:20
mathjax: true
tags:
---

> Title：ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile
> Authors：Face++团队
> Session：CVPR, 2018
> Abstract：作者提出一个效率极高的网络结构 ShuffleNet，专门应用于计算力受限的移动设备。网络利用两个操作：pointwise group convolution 和 channel shuffle，与现有先进模型相比在类似的精度下大大降低计算量。在 ImageNet 和 MS COCO 上 ShuffleNet 表现出比其他先进模型的优越性能。



<!--more-->

<br>

*preface*

现许多 CNN 模型的发展方向是更大更深，这让深度网络模型难以运行在移动设备上，针对这一问题，许多工作的重点放在对现有预训练模型的修剪、压缩或使用低精度数据表示。 

论文中提出的 ShuffleNet 是探索一个可以满足受限的条件的高效基础架构。论文的 Insight 是：现有的先进 basic 架构如 Xception 和 ResNeXt 在小型网络模型中效率较低，因为大量的 $1 \times 1$ 卷积耗费很多计算资源，论文提出了逐点群卷积 (pointwise group convolution) 帮助降低计算复杂度；但是使用逐点群卷积会有幅作用，故在此基础上，论文提出通道混洗 (channel shuffle) 帮助信息流通。基于这两种技术，作者构建一个名为 ShuffleNet 的高效架构，相比于其他先进模型，对于给定的计算复杂度预算，ShuffleNet 允许使用更多的特征映射通道，在小型网络上有助于编码更多信息。


<br>
# ShuffleNet

ResNeXt 在每个 residual block 中应用到了 group convolution 的技术，但是只用到 $3 \times 3$ 的卷积核，并没有用到 $1 \times 1$ 的卷积核上，从下图可以看出，bottleneck layer 还是一个 standard convolution，而 ResNeXt 的论文也表明，$1 \times 1$ 的 standard conv 占了 93.4% 的计算量。

{% asset_img 1.png %}

因此作者第一个想法就是在 bottleneck layer 上也应用 group convolution，把输入 channel 被分成不同的 group，在每个 group 的卷积中，输入输出 channel 数也成倍地减少，因此能达到减少计算量的效果。但是使用 group convolution 有一个缺点就是，standard convolution中，卷积核跨越的是全部的 channel，而在 group convolution，卷积核跨越的是一个 group 里面的 channel，即输入的信息变少了，这样就导致了卷积的结果只来源于一个 group 的信息，group 之间并没有 interaction，这就阻碍了信息的流通。而反观为什么 ResNeXt 可以使用 group convolution 来代替原来的操作呢？那是因为 ResNeXt 本身就分成了多条 path，而每条 path 的信息 channel 本身就等于一个 group 的 channel 大小，因此使用 group convolution 正好等于原来的多条 path 分别卷积，并且能利用 group convolution 并行运算的好处。

针对上面 group convolution 阻碍 channel 之间信息流通的问题，作者提出了第二个想法：channel shuffle，将 group convolution 输出结果进行重排，如下图，(a) 为一般的 group convolution，组内进行卷积，(b) 演示了什么叫做 channel shuffle。

{% asset_img 2.png %}

channel shuffle 在实现上如下，假设有 g 个组，每个组的 channel 数为 n，因此整个 group convolution 的输出 channel 大小为 g * n。shuffle 流程如下

- 将 $[1, g \times n]$ 的向量 reshape 成 $[g,n]$
- 将 $[g,n]$ 的矩阵进行转置得到 $[n,g]$
- 将 $[n,g]$ 的矩阵重新 reshape 成 $[1, n \times g]$ 



根据上面的两点想法：group convolution 和 channel shuffle，作者能够解决 bottleneck layer 计算量开销大的问题，仿照 ResNet，作者提出了 ShuffleNet 的一个 shuffle unit，如下，每个 unit 采用的是 两条 path 的模式，一条是 identity mapping，另一条是 residual function。

- (a)：将 $3 \times 3$ 的 standard convolution 改成 depthwise convolution（Xception 和 MobileNet 中提到），且 depthwise  convolution 后不加 ReLU 激活（Xception 中验证了其有效性）。
- (b)：将 $1 \times1$ 的 standard convolution 改成 pointwise group convolution，并在第一个 bottleneck 和 $3 \times 3$ 卷积之间加上 channel shuffle。
- (c)：该版本的 unit 用于维度变换，$3 \times 3$ 的卷积 stride 为 2，使得 feature map 大小减半，同时 identity mapping 也改为 stride 为 2 的 avg-pooling，作者认为在减小 feature map 的同时，需要增大 channel 数，所以把最后的 add 操作改为了 concat 操作，实现 feature map 减半，channel 数翻倍的效果。

{% asset_img 3.png %}



假设输入维度大小为 $h \times w \times c$，bottleneck layer 的 channel 为 m，下面对比 ResNet，ResNeXt 和 ShuffleNet  一个单元的计算量（忽略 add，BN，ReLu）。

- ResNet：三个卷积均为 standard convolution，三层卷积层的卷积核大小分别为 $1 \times 1 \times c \times m$，$3 \times 3 \times m \times m$，$1 \times 1 \times m \times c$，图片大小变化为 $h \times w \times c$，$h \times w \times m$，$h \times w \times m$，$h \times w \times c$。因此三次卷积的计算量总和为：$hwmc + hw9m^2 + hwcm = hw(2cm + 9m^2)$
- ResNeXt：bottleneck layer 的卷积为 standard convolution，$3 \times 3 $ 的卷积为 group convolution，因此首尾两层的计算量没变，group convolution 执行卷积操作的时候，实际上接收的图片大小为 $h \times w \times \frac{m}{g}$，卷积核的大小为 $3 \times 3 \times \frac{m}{g} \times \frac{m}{g}$，因此一个 group 的卷积计算量为 $9hw\frac{m}{g}\frac{m}{g}$，共有 g 组卷积，因此计算量为 $hw9m^2/g$，因此三次卷积的计算量总和为：$hw(2cm + 9m^2/g)$
- ShuffleNet：bottleneck layer 改为 group convolution，$3 \times 3$ 卷积改为 depthwise convolution，按照上面的计算方法，group convolution 能把原计算量减低为 1/g，因此首尾两层的计算量降为 $2hwcm/g$，depthwise convolution 每个卷积核只负责一个 channel，共有 m 个卷积核，每个卷积核为 $3\times 3$，因此计算量为 $3 \times 3 \times h \times w \times m = hw9m$，因此三次卷积的计算量总和为：$hw(2cm/g+9m)$

因此 ShuffleNet 的计算量最少，给定一个计算限制，ShuffleNet 可以使用更宽的特征映射。

基于上面的 shuffle unit，作者提出 ShuffleNet 的架构如下

- shuffleNet 分成了三个 stage，每个 stage 的 channel 数会翻倍，用的就是上面 shuffle unit (c)
- bottleneck layer 的 channel 大小设为输出的 1/4
- 作者按照 group 的大小分成了五个网络，每个网络的 channel 大小都不同，但最后整体的 complexity 接近。
- 下面的网络称为 ShuffleNet 1 $\times$，作者称 ShuffleNet $s\times$ 为整体网络每个 channel 大小都乘上 s 的网络。

{% asset_img 4.png %}



<br>

# 实验结果

**On the importance of Pointwise Group Convolution**

实验对比了 g 取不同值下的模型在 ImageNet 的错误率，g = 1，即没有 group convolution，网络变成像 Xception 的样子，结果如下；从结果来看，有 group convolution 的一致比没有 group convolution (g=1) 的效果要好。注意 group convolution 可允许获得更多通道的信息，我们假设性能的收益源于更宽的特征映射，这帮助我们编码更多信息。并且，较小的模型的特征映射通道更少，这意味着能多的从特征映射上获取收益。

对于一些模型，随着 g 增大，性能上有所下降。意味 group 增加，每个卷积滤波器的输入通道越来越少，损害了模型表示能力。

值得注意的是，对于小型的 ShuffleNet 0.25×，组数越大性能越好，这表明对于小模型更宽的特征映射更有效。

{% asset_img 5.png %}



**Channel Shuffle vs. No Shuffle**

在三个不同复杂度下带 shuffle 的都表现出更优异的性能，尤其是当 group 更大 (g = 8)，具有 shuffle 操作性能提升较多，这表现出 shuffle 操作的重要性。 

{% asset_img 6.png %}



**Comparision with other models**

作者对比不同模型 unit 之间的性能差异，如 residual block，Inception block，用各个 unit 取替换 stage 2 - 4 之间的 shuffle unit，调整通道数保证不同模型的复杂度接近。可以看出 Shufflenet 在相同 complexity 下，表现最好。

{% asset_img 7.png %}
{% asset_img 8.png %}



**Comparision with MobileNet**

- 作者发现在不同 complexity 下，ShuffleNet 都比 MobileNet 要好
- 作者还尝试了更浅的网络，因为 MobileNet 是 26 层，而 ShffuleNet 为 50 层，实验表明更浅的 ShuffleNet 依旧比 MobileNet 要好。

{% asset_img 9.png %}



**Actual Speedup Evaluation**

三种分辨率输入做测试，由于内存访问和其他开销，原本理论上 4 倍的加速降低到了 2.6 倍左右。但是比 AlexNet 快了十几倍。

{% asset_img 10.png %}
<br>