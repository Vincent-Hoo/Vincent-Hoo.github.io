---
title: '【论文阅读】—— MobileNetV2, Inverted Residuals and Linear Bottlenecks'
date: 2018-12-11 15:11:34
mathjax: true
tags:
---

> Title：MobileNetV2: Inverted Residuals and Linear Bottlenecks
> Authors：Google 团队
> Session：CVPR, 2018
> Abstract：作者在 MobileNet-v1 的基础上，提出了两个 tricks：Linear Bottleneck 和 Inverted Residual，并提升了网络的性能。

<!--more-->

<br>

# depthwise separable convolution

深度可分离卷积分为两个部分：depthwise convolution 和 pointwise convolution，前者在 Xception 中提出，深度可分离卷积实现了 spatial 和 channel 之间的解耦，下图对比了传统的卷积和深度可分离卷积，regular 的卷积核是从 spatial 和 channel 两个不同角度去提取特征，而深度可分离卷积，先从 spatial 层面进行过滤，再用点卷积进行特征提取。

{% asset_img 1.png %}


<br>

# 网络结构

先从整体上看 MobileNet-v2 的网络结构，再去仔细分析每个 trick 背后的 intuition。下面先对比一下 MobileNet-v1 和 MobileNet-v2 每个 block 。

  

{% asset_img 2.png %}
> 相同点

- **都采用 Depthwise (DW) 卷积搭配 Pointwise (PW) 卷积的方式来提特征。**这两个操作合起来也被称为 Depthwise Separable Convolution，之前在 Xception 中被广泛使用。这么做的好处是理论上可以成倍的减少卷积层的时间复杂度和空间复杂度。 
- 在 Depthwise 后都是接 ReLU6 作为激活函数，ReLU6 表示将 ReLU 的最大值设为 6。虽然在 Xception 中已经提到过，Depthwise 卷积后面接线性激活的效果要比非线性激活好，但是 MobileNet 的作者依旧在 Depthwise 卷积后加了 ReLU 激活，无论是 v1 还是 v2。

> 不同点

- MobileNet-v2 在 DW 前加了一个 PW，这么做的原因，是因为 DW 卷积由于本身的计算特性决定它自己没有改变通道数的能力，上一层给它多少通道，它就只能输出多少通道。所以如果上一层给的通道数本身很少的话，DW 也只能在低维空间提特征，因此效果不够好。 MobileNet-v2 为了改善这个问题，给每个 DW 之前都配备了一个 PW，专门用来升维，定义升维系数（expansion ratio）$t = 6$ ，这样不管输入通道数 $C_{in}$ 是多是少，经过第一个 PW 升维之后，DW 都是在相对的更高维 ( $t \cdot C_{in}$ ) 进行着特征提取。
- **V2 去掉了第二个 PW 的激活函数**。论文作者称其为 Linear Bottleneck。这么做的原因，是因为作者认为激活函数在高维空间能够有效的增加非线性，而在低维空间时则会破坏特征，不如线性的效果好。由于第二个 PW 的主要功能就是降维，因此按照上面的理论，降维之后就不宜再使用 ReLU6 了。
{% asset_img 3.png %}


<br>
下面再对比一下 MobileNet-v2 和 ResNet

{% asset_img 4.png %}

> 相同点

- MobileNet-v2 借鉴了 ResNet block 的结构，采用了 $1 \times 1 \rightarrow 3\times 3 \rightarrow 1\times1$
- MobileNet-v2 借鉴了 ResNet，同样适用 shortcut 将 block 的输入输出相连（上图未显示）

> 不同点

- ResNet 使用 **标准卷积** 提特征，MobileNet 始终使用 **DW** 提特征。 
- ResNet **先降维** (0.25倍)、卷积、再升维，而 MobileNet V2 则是 **先升维** (6倍)、卷积、再降维。作者将 MobileNet-v2 的结构称为 Inverted Residual Block。这么做也是因为使用 DW 卷积而作的适配，希望特征提取能够在高维进行。 
- 最后的 PW 卷积用线性激活，即上面提到的 Linear Bottleneck。

下图中，浅色的代表一个 block 的输出，有斜线的代表用线性激活，下图可以看出来，ResNet 中 shortcut 连接的是两个 channel 数很大的层，而 Inverted residual block 连接的却是两个 bottleneck。

{% asset_img 5.png %}

<br>

再来看整体的网络结构，t 代表 expansion ratio，c 代表输出的 channel 数，n 代表重复的次数，s 代表 stride。

{% asset_img 6.png %}

需要注意的是：

- 当 n > 1 的时候，即该 bottleneck 重复次数大于 1 次，只有在第一个 bottleneck 的 stride 为 s，其它的 stride 为 1.

- 只有在 stride 为 1 的情况下，才会有 shortcut 连接。见下图，stride 为 2 的时候没有 shortcut。

{% asset_img 7.png %}

-  当 n > 1 时，只在第一个 bottleneck 的输出 channel 为 c，其他 bottleneck 输出的 channel 不变。
<br>
例如，对于上图中 $56^2 \times 24$ 的那层。

1. n = 3，共有 3 个 bottleneck，只在第一个 bottleneck 使用 stride = 2，后两个bottleneck 的 stride = 1；
2. 第一个瓶颈层由于输入和输出尺寸不一致，因而无 shortcut 连接，后两个由于stride = 1，输入输出特征尺寸一致，会使用 shortcut 将输入和输出特征进行相加；
3. 只有第一个 bottleneck 最后的 $1\times1$ conv 的输出 channel 为 c，后两个 bottleneck 的输出 channel 恒定不变，保持为 c。

该层输入特征为$56\times56\times24$，第一个 bottleneck 输出为 $28\times28\times32$（ feature map 尺寸降低，channel 数增加，无 shortcut），第二、三个 bottleneck 的输入和输出均为$28\times28\times32$（此时 c = 32，s=1，有shortcut）。

<br>

# Intuition behind MobileNet-v2

MobileNet-v2 正如论文标题所说，两个最大的特点就是 Linear Bottleneck 和 Inverted  Residuals，虽然线性激活函数在 Xception 中已经提到，被放在 DW 卷积后，但是作者并没有那么做，而是把线性激活函数放在 bottleneck 上，下面看一下作者这样做的原因。

对于一组真实图片，作者称激活层后的张量为 manifold of interest，根据之前的研究，manifold of interest 可能仅分布在激活空间的一个低维子空间里，那么我们可以通过减少每一层 channel 的数量来减少空间的维数，MobileNet-v1 就是通过一个宽度乘子来实现的，但是由于非线性变换的存在，如 ReLU，使得降维操作会损失较多的信息，作者做了一个小实验来验证这一点。

下图中输入在二维空间，作者通过一个 $n \times 2$ 的矩阵 T 将输入映射到 n 维空间中，再经过 ReLU 函数，然后再用 $T^{-1}$ 将其映射回二维空间，下图表示当 n 取不同值时候的结果，可以看出当 n 取 2 或 3 的时候，恢复的数据和原来的数据有较大的差异，信息丢失得比较严重；而当 n 比较大的时候，信息能够得到较大的保留，作者给出的解释是如果维度较大，即有较多的 channel，一个 channel 信息的丢失可能会在另外的 channel 中得到保留。

{% asset_img 8.png %}

由于网络中每一层激活后的空间都可以理解为 manifold of interest 的一个表征，如果这个激活空间的维数过低，再经过 ReLU 非线性激活，根据上面的结论，信息会丢失较多，那么这个激活空间就不能很好地表征这个 manifold of interest，因此作者才提出用 Linear  Bottleneck 去代替 ReLU。

作者在附录中证明了：如果 input manifold 可以是一个低维激活空间的表征，那么 ReLU 激活函数就可以保留所有的信息，且实验证明了线性层能够防止 ReLU 函数破坏太多的信息。

而上面提到的，高维空间 ReLU 能够保留较多的信息，这也是作者提出 Inverted  Residuals 的原因，升维后使用 ReLU，降维后使用线性激活。


<br>
# References

1. [谷歌官方博客](https://ai.googleblog.com/2018/04/mobilenetv2-next-generation-of-on.html)
2. [论文笔记1](https://zhuanlan.zhihu.com/p/33075914)
3. [论文笔记2](https://blog.ddlee.cn/posts/c9816b0a/)
4. [论文笔记3](https://blog.csdn.net/u014380165/article/details/79200958)
5. [论文笔记4](https://www.cnblogs.com/darkknightzh/p/9410574.html)




<br>