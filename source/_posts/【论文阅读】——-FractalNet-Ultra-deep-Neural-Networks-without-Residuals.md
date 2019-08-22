---
title: '【论文阅读】—— FractalNet, Ultra-deep Neural Networks without Residuals'
date: 2018-12-11 15:10:19
mathjax: true
tags:
---

> Title：FRACTALNET: ULTRA-DEEP NEURAL NETWORKS WITHOUT RESIDUALS
> Authors：Gustav Larsson, Michael Maire, Gregory Shakhnarovich
> Abstract：作者提出了一个分形网络 (fractalNet)，并且认为 ResNet 中的残差计算并不是深层网络必须的，可以通过一种分形的结构，也可以达到很好的效果。



<!--more-->

<br>

*preface*

自从 ResNet 提出以来，倍受人们的追捧，在 ResNet 基础上延伸出来的模型结构越来越多，如：Wide ResNet、stochastic Depth ResNet 等等，都取得了非常好的效果，而 Residual 的结构也被认为对于构建极深网络非常重要的结构。而本文，提出了一种新的分形网络结构，它的提出就是为了证明 Residual 对于极深网络并不是必须的，它通过一种分形的结构，达到了类似于教师-学生机制、深度监督的效果，同样取得了非常好的效果。虽然这种方法在目前来说用的比较少，但是我觉得它的这种思路是非常值得学习的。



# FractalNet

整个分形网络的架构图如下，先从最右边看起，网络中包含 5 个 block，每个 block 之间用池化层相连，最后一个全连接层进行预测，然后我们细看每个 block 的结构，初看有一种递归或迭代生成的结构，左上角给出了 fractal 展开的法则，$f_1(\cdot)$ 表示一个最基本的层，可以是卷积层，也可以是自己定义的一些结构，从 $f_1(\cdot)$ 拓展到 $f_2(\cdot)$ 需要将输入分成两条路径，一条路径上为卷积层，另一条路径上为两个依次连接的 $f_1(\cdot)$，然后将两条路径上的信号 join 起来。依此类推 $f_3(\cdot)$ 和 $f_4(\cdot)$，下图中的 $f_4(\cdot)$ 做了一些小小的改动，从 fractal expansion rule 看，join 操作的输入只能是两条路径，比如 A 和 B join 的结果为 C，C 和 D 再做 join，作者修改其为 A、B 和 C 一起 join，因此 join 操作的输入不限定为 2 个。

从 fractal expansion rule 看，每拓展一个 level，网络的层数就 double，所以对于 $f_C(\cdot)$，其网络结构有 $2^{C-1}$ 层。在 FractalNet 中 join 操作，不像之前的工作那样采取求和或拼接，因为 join 的输入路径数量不固定，因此作者将输入信号进行取均值处理，从这可以看出，一个 block 中每一个卷积操作应该都是有相同的卷积核大小和卷积核个数的，并且在 block 中不改变 feature map 的大小，不然无法进行均值化。

分形网络不像 ResNet 那样连一条捷径，而是通过不同长度的子路径组合，其结构也有点像 Inception，将输入分成不同路径，但是 FractalNet 每条路径长度又不完全相同。在思路上，有点像 DenseNet 和 Stochastic Depth ResNet，跨越不同层将信号相连，如果把下面的 $f_4(\cdot)$ 看成是 8 层的网络，那么最左边的路径就是跨越 8 层的连接。

{% asset_img 1.png %}



同时，作者受到 Dropout 的启发，提出了 drop path，即训练过程中随机丢弃一些路径，drop path 是 dropout（防止co-adaption）的天然扩展，是一种正则方法，可以防止过拟合，提升模型表现。drop path 分为两种类型：local 和 global。local 指 join 操作会随机丢弃一些输入，且保证至少会有一个输入；global 指从整个网络中随机选择一条路径来进行训练，这样做会使得网络中任意的一条路径都能作为一个分类器使用。drop path 的好处有几点

1. 减少过拟合
2. 强化每条路径的输出
3. 是作者重点提出，也是我认为非常值得学习的一点，那就是不同路径的联合，在 drop path 的机制下，起到了一种简单的教师—学生的学习方式。也就是说，如果某一条路径学到了对最终分类起到非常重要的特征时，假如在某一次迭代中，该路径被关闭掉了，那么通过 loss 的反向传播中，可能就会指导和该路径进行联合的另一条路径也学到这种重要的特征。那么通过模型内部的这种不断的教师—学生的学习方式，不仅可以提高整个模型的效果，并且当提取出其中的任意一条路径出来单独使用时，也能够达到非常好的效果。作者在后面也证明了，对于 plain network，在深度达到 40 层以上时会出现退化的问题，但是在 FractalNet 中完全不会，从整个模型中提出其中最深的路径来单独使用时可以达到和整个 FractalNet 接近的效果。


{% asset_img 2.png %}
<br>
# 实验结果

实验使用的数据集包括：CIFAR-10，CIFAR-100，SVHN，ImageNet。对于前三个数据集，输入图片大小为 $32 \times 32$，FractalNet 有 5 个 block，每个 block 内每个卷积核的输出 channel 数分别为 (64, 128, 256, 512, 512)，block 之间采取 non-overlapping 的池化层，遵循 feature map 减半的同时，channel 数量翻倍。对于 ImageNet 数据集，对比 ResNet-34，作者保留 ResNet-34 的头尾两层，将中间层改为 4 个 fractalnet block，每个 block 内卷积核的输出 channel 数分别为 (128, 256, 512, 1024)。

实验主要对比 ResNet，结果如下

- +表示使用了水平镜像翻转和平移，++表示使用了更多的数据增强

- 20 层的 FractalNet 超过了最原始的 ResNet
- 有数据增强的话，CIFAR-100 上 FractalNet 跟最好的 ResNet 变种接近
- 没有数据增强的话，FractalNet 比 ResNet 和 Stochastic Depth 要好，证明 FractalNet 的训练模式更不容易过拟合。
- drop path 是有用的，能够防止过拟合，在 CIFAR-100 上，加了 drop path，错误率从 35% 降到 28%
- 当取出最深的一条路径时，和整个 FractalNet 的表现接近

{% asset_img 4.png %}

比较 FractalNet 和 Plain Network

- fractalNet 到了160层开始出现退化
- plain network 到了40层就出现了退化，到了160层不能收敛 
- 使用了drop-path的 fractalNet 提取的 path 比平常的网络取得了更优的表现，而且克服了退化问题。
{% asset_img 4.png %}
{% asset_img 5.png %}

<br>

*In Conclusion*

1. 论文的实验说明了路径长度才是训练深度网络的需要的基本组件，而不单单是残差块
2. 分形网络和残差网络都有很大的网络深度，但是在训练的时候都具有更短的有效的梯度传播路径
3. 分形网络简化了对这种需求（更短的有效的梯度传播路径）的满足，可以防止网络过深
4. 多余的深度可能会减慢训练速度，但不会损害准确性

 

<br>