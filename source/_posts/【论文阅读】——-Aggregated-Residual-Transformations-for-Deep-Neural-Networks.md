---
title: 【论文阅读】—— Aggregated Residual Transformations for Deep Neural Networks
date: 2018-12-03 19:18:16
mathjax: true
tags:
---


> Title：Aggregated Residual Transformations for Deep Neural Networks
> Authors：Saining Xie，Kaiming He
> Session：CVPR, 2017
> Abstract：提高网络性能，除了深度和宽度外，作者提出了 cardinality 的概念，并修改 ResNet 成一个高度模块化的网络 ResNeXt，对比 ResNet 和 Inception，ResNeXt 的超参数很少，不需要精细的网络设计，且网络参数和计算量都比 ResNet 少。

<!--more-->

<br>

*preface*

- VGG, ResNet：通过堆叠相同形状的的网络 block，即 depth。
- Inception：split-transform-merge 的模式，首先用 $1 \times 1$ 的卷积核进行 split，然后用 $3 \times 3$ 或 $5 \times 5$ 的卷积核进行 transform，最后进行 channel 级别上的 merge。
- ResNeXt：采用 VGG/ResNet 的网络的 depth 加深方式，同时利用 split-transform-merge 策略。


<br>
# ResNeXt

作者想把 Inception 的思想融入 ResNet，但是 Inception block 都是需要人工设计的，就好像 Inception-ResNet 中的各个 residual block那样。作者想希望设计一个高度模块化的网络，因此提出了下面的 residual block，左边的图是 ResNet 的 residual block，右边是 ResNeXt 的，可以看出右边的图 residual block 的形式就等同于 Inception block，只是右边的图每一条 path 都是相同的卷积层，或者说相同的拓扑结构，作者把 path 的数量称为 cardinality，而与 Inception block 不同的是，每一条 path 最后的并不是 concatenate，而是 summation，因为每一条 path 的拓扑相同，因此每一条 path 的输出大小相同，因此作者采取相加的方式进行 merge。

{% asset_img 1.png %}

作者提出的这种 residual block 可以看成是 Inception-ResNet 论文里面的 block，或者看成 grouped convolution（在 AlexNet 论文中提到）。

下图的 c 是 grouped convolution，AlexNet 中用到 grouped convolution 主要是为了并行训练，放到两个GPU中训练网络；而 grouped convolution 首先会将输入按照 channel 分成多个 group，然后每个 group 再进行卷积，卷积后再进行拼接。ResNeXt 可以看成是 grouped convolution 是因为本身在一个 block 里面就有 32 条 path，现在用 grouped convolution 的角度只是先把这 32 条 path 整合成 channel 为 128 的数据，再分 32 组 group 去卷积，每组卷积的输入输出依旧为 4。作者在实现 ResNeXt 的时候就是采取 grouped convolution 的方式，这一思想再后面的论文也有体现，比如 shuffleNet。  

{% asset_img 2.png %}



作者在做对比实验的时候，是保持 ResNet 和 ResNeXt 的 complexity 一致，complexity 一般是指网络的参数和计算量，具体到每个 block，为了维持一致的 complexity，需要计算 ResNet 的参数量和 ResNeXt 的参数量。假设 cardinality 为 C，bottleneck width 为 d，则

$$256 \times 64 +3 \times 3 \times 64 \times 64+64 \times 256 = C \times(256 \times d + 3 \times3 \times d \times d + 64 \times 256) \approx 70k$$ 

$C = 32,d=4$ 为上图的情况。**而 ResNet 相当于是 $C=1$ 的情况，只有一条 path。**

{% asset_img 3.png %}


<br>
# 实验结果

我们把 50 层 ResNeXt，C = 32，d = 4 的网络标记为 $ResNeXt-50（32 \times 4d）$

**Cardinality vs bottleneck width**

首先作者需要找出一对最好的 cardinality 和 width，且和 ResNet 保持一致的 complexity。

- ResNext-50 最好的 error 是 22.2%，比 ResNet-50 低了 1.7%。
- 当 cardinality 增大，error 持续减小

{% asset_img 4.png %}



**Increasing Cardinality vs Going Deeper/Wider**

deeper 指网络层数增多，ResNet-101 到 ResNet-200；wider 指 bottleneck width 增宽，即 channel 数增多；

- 将 ResNet 加深和加宽，error 下降得不多，分别下降了 0.3% 和 0.7%。
- 增大 cardinality，error 下降得比较明显，作者分别对 ResNet 和 ResNeXt 进行 cardinality 增大，ResNet 原来 cardinality 为 1，ResNeXt 原来为 32，均增大一倍。

{% asset_img 5.png %}



**对比其它模型**



{% asset_img 6.png %}

<br>
*In Conclusion*
个人觉得，ResNeXt 最大的贡献在于提供了一种高度模块化的网络，不用刻意地去设计，只需要改变 cardinality 和每个 block 的卷积层就可以设计一个模型了；而 cardinality 这个点也是很有创新性。
cardinality 是衡量神经网络在深度和宽度之外的另一个重要因素。作者还指出，与 Inception 相比，这种新的架构更容易适应新的数据集/任务，因为它有一个简单的范式，而且需要微调的超参数只有一个，而 Inception 有许多超参数（如每个路径的卷积层核的大小）需要微调。

<br>