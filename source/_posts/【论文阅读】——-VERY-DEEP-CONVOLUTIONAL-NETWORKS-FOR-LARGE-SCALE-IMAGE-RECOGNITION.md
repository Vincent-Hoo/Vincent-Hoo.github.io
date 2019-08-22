---
title: 【论文阅读】—— VERY DEEP CONVOLUTIONAL NETWORKS FOR LARGE-SCALE IMAGE RECOGNITION
date: 2018-11-29 14:06:16
mathjax: true
tags:
---

> Title：VERY DEEP CONVOLUTIONAL NETWORKS FOR LARGE-SCALE IMAGE RECOGNITION
> Authors：Karen Simonyan, Andrew Zisserman
> Session：ICLR, 2015
> Abstract：VGGNet 代表了牛津大学的 Visual Geometry Group，该网络赢得了 ILSVRC 2014分类比赛的亚军、定位比赛的冠军。VGG 首次将网络深度提升到了 19 层，并且使用较小的卷积核，深刻印证网络深度对模型性能的影响。



<!-- more-->

<br>

# 网络结构

ZFNet 和 OverFeat 两个网络都是将 AlexNet 的第一层卷积层进行了修改，将卷积核缩小以求挖掘更加细致的信息，这也说明了小卷积核的优势，因此 VGG 清一色的将卷积核设为 $3 \times 3$，并且所有的卷积操作都遵循 SAME 原则，即不改变 feature map 的维度，只改变 channel 数量；而降维操作均发生在池化层，同样清一色的采取 stride 为 2 的 $2 \times 2$ 的pooling，即每次 pooling 发生，feature map 的 height 和 width 就会减半。

而小卷积核对比大卷积核的优势究竟是什么呢？作者提到主要的优势有两点。

1. 从下图可以看出，串联两个 $3 \times 3$ 的卷积层和一个 $5 \times 5$ 的卷积层拥有相同的receptive field（感受野）；而串联三个 $3 \times 3$ 的卷积层和一个 $ 7 \times 7$ 的卷积层拥有相同的receptive field。虽然感受野相同，那么串联多个小卷积核的好处在于它带来更多的non-linearity，因为经过了多层的激活函数。

{% asset_img 1.png %}

2. 小卷积核会减少模型的参数量和计算量，由于 VGG 在卷积操作的时候不改变 channel 数，因此我们假设其为 C。串联三个 $3 \times 3$ 的卷积层总共的参数为 $3(3^2C^2)$，而一层 $7 \times 7$ 的卷积层就有 $7^2C^2$ 的参数，可见小而深的网络比宽而浅的网络参数要少。

作者共涉及了 6 个 VGG网络，DE 版本就是我们常见的 VGG-16 和 VGG-19，不同网络间除了网络深度不同，其它完全一样。注意作者定义网络层数的时候，只算有参数的层，即卷积层和全连接层，池化层不算在内。里面提到 conv 均为 $3 \times 3$ 的 SAME 卷积，max-pooling 均为 stride为 2 的 $2\times 2$ 的池化。

{% asset_img 2.png %}

A-LRN 是在 A 版本下加入了 AlexNet 的 LRN，但是实验结果表明，LRN并没有带来提升，反而会使得模型的计算量增大。C 版本尝试性地使用了两个 $1 \times 1$ 的卷积核，作者受到当时 "Network in Network" 中的启发，也使用了这样的卷积核，$1 \times 1$ 的卷积核可以看成是和每个 pixel 进行內积，它只挖掘一个 pixel 在不同 channel 之间的关系，而不去管一个 feature map 中不同 pixel 之间的 connection。

<br>

# 训练和测试的细节

## 训练阶段

输入的图像是 $224 \times 224 \times 3$ 的 RGB 图像，其他参数，如 batch size、momentum、weight decay 基本和 AlexNet 一致。作者发现 VGG 收敛要比其它网络要快，分析可能的原因是 1）深度增加和卷积核变小带来的 implicit regualarization。2）某些层的预初始化。

**预初始化：**

1. BCDE 版本网络的初始化均来自于 pre-trained 的版本 A 网络，由于网络 A 较小，因此收敛较快，先训练一个网络 A，然后将其训练好的参数作为后面网络的初始值（前面四个卷积层和最后的三个全连接层）
2. 其它的参数进行随机初始化，从 $(0, 0.01)$ 的高斯分布中进行初始化。

**数据增强**

- 随机水平翻转

- 随机 RGB 颜色调整

- scale jittering，原始图像首先会被调整到某一个尺度，如 $N\times S$，$S < N$，然后我们再从中 crop 出 $224 \times 224$ 的图片作为网络的输入；如果 $S = 224$，那么这个 crop 就相当于把图像的全部信息作为输入了；而如果 $S > 224$，那么输入图片为原图像的一部分。作者提出了两种方法来设定 $S$ 值。

  - 固定 $S = 256 \ or \  384$，256 是 AlexNet 和 ZFNet 使用的数值。
  - 设定 $S \in [256, 512]$ 之间的随机数值。

  因此对于一张图像，首先将其调整到 $N \times S$，再将它随机裁剪出 $224 \times 224$ 的部分。



## 测试阶段

同样，先将测试图像调整到尺度 $N \times Q$，$Q < N$，注意 $Q$ 不一定等于 $S$，两者没有任何关系。下面对比两种测试的方法。

**Dense Evaluation**

这种测试方法在 OverFeat 里面被提出来，在 OverFeat 中被称为 multi-scale classification，训练完网络后，将全连接层转成全卷积神经网络，从而令网络可以接收任意尺度的输入。输出会是一个 class score map，然后再处理这个 map，得到最后的预测结果。

**Multi-crop Evaluation**

这种测试方法在 AlexNet 中被提出来，在 AlexNet 中被称为 10 views，从测试图片中 crop 出多张 $224 \times 224$ 的图片，再进行水平镜像等操作后输入网络中，得到多个预测结果，取平均或最大作为最后的预测。

作者基于 dense evaluation 提出了两种评测方式

- single-scale evaluation：顾名思义，测试图片的尺度是固定的
- multi-scale evaluation：选择多个 Q 值，从而对于一张图片产生多个测试图片。这里需要注意的是，Q 值的选择需要参考 S 值，如果 S 值是固定的数值，如 256 或者 384，那么 Q 值就取 $ \{S - 32, S, S+ 32 \}$ 三个数值；如果 S 是 $[256, 512]$ 中的随机数，那么 Q 值就取 $\{ 256, 384,512 \}$ 三个数值。

multi-scale evaluation 会得到 3 个 Q 值 class score map，然后综合考虑，决定出最后的预测结果。

<br>

# 实验结果

## Single-scale Evaluation

在单尺度评测下，Q 值选择和 S 值 相同，从实验结果可以看出

- LRN 没有什么作用
- 深度增大，准确率提高
- $1 \times 1$ 的卷积核并没有提升网络性能
- 随机选择 S 值要比固定一个 S 值要好

{% asset_img 3.png %}

## Multi-scale Evaluation

在测试时候采取多尺度的评测（相当于 scale jittering），会提升网络的性能

{% asset_img 4.png %}


## Dense Evaluation vs Multi-crop Evaluation

- 对于 VGG-19，multi-crop 比 dense evaluation 要好一点
- 模型融合可以得到更好的结果

{% asset_img 5.png %}
{% asset_img 6.png %}

<br>

*In Conclusion*

引用 CS231n 的一句话

> The runner-up in ILSVRC 2014 was the network from Karen Simonyan and Andrew Zisserman that became known as the VGGNet. Its main contribution was in showing that the depth of the network is a critical component for good performance. Their final best network contains 16 CONV/FC layers and, appealingly, features an extremely homogeneous architecture that only performs 3x3 convolutions and 2x2 pooling from the beginning to the end. Their pretrained model is available for plug and play use in Caffe. A downside of the VGGNet is that it is more expensive to evaluate and uses a lot more memory and parameters (140M). Most of these parameters are in the first fully connected layer, and it was since found that these FC layers can be removed with no performance downgrade, significantly reducing the number of necessary parameters.

不难提炼出如下结论：

1. 深度提升性能；
2. 最佳模型：VGG16，从头到尾只有3x3卷积与2x2池化。简洁优美；
3. 开源pretrained model。与开源深度学习框架Caffe结合使用，助力更多人来学习；

4. 卷积可代替全连接。整体参数达1亿4千万，主要在于第一个全连接层，用卷积来代替后，参数量下降。


<br>
*references*

1. [某博客](https://blog.csdn.net/qq_40027052/article/details/79015827)
2. [cs231n lecture 9 ppt](http://cs231n.stanford.edu/slides/2017/cs231n_2017_lecture9.pdf)
3. [VGG 会议 ppt](http://www.robots.ox.ac.uk/~karen/pdf/ILSVRC_2014.pdf)




<br>