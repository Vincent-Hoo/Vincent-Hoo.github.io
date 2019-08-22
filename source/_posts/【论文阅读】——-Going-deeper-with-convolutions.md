---
title: 【论文阅读】—— Going deeper with convolutions
date: 2018-11-29 14:07:13
mathjax: true
tags:
---


> Title：Going deeper with convolutions
> Authors：Christian Szegedy et al
> Session：CVPR, 2015
> Abstract：这就是传说中的 GoogLeNet（Inception-v1），获得了 ILSVRC 2014 分类任务的冠军。该网络设计了 Inception module 代替人工选择卷积核，通过堆叠 Inception module 得到 GoogLeNet。



<!--more -->

<br>

*preface*

作者设计 GoogLeNet 的一大动机是想减少网络的参数和计算量，从而应用到实际应用中，但是为了提高模型的性能，网络深度和宽度必须要得以保证；而模型的增大会导致参数的增多，既增大了计算量，又容易使得模型陷入过拟合。而作者设计的 Inception module 参数少且宽，因此通过堆叠 Inception module 既能提高模型的性能，又能减少模型的参数。Inception-v1 虽然有 22 层，但是参数却只有 5 million，相对于 VGG-16 的 138 million 和 AlexNet 的 60 million，有了很大的提升。

一点需要注意的是：GoogLeNet 并非 GoogleNet，虽然作者来自于 Google，GoogLeNet 取名是为了致敬 LeNet；而 Inception 的取名来自于电影 Inception（盗梦空间）里面的一句经典台词 "we need to go deeper"，意味着我们需要搭建更深的网络。

<br>

# 网络结构

GoogLeNet 有以下几个特点

- 将全连接层去掉，用 global pooling 的方式来做最后的 flatten，之前的论文参数过多的一大重要原因就是全连接层的使用，因此作者将其去除，既能减少参数，性能又没有被太多的破坏。

- 所有的卷积的操作都是 SAME 卷积，即不改变 feature map 的大小，只改变 channel 的数量，而降维留给了 max-pooling。

- 堆叠 Inception module（如下图），Inception module 省去了选择卷积核的烦恼，作者统一使用三种卷积核 $1 \times 1$、$3 \times 3$ 和 $5\times 5$，并且在每个 Inception module 中加入了 pooling。为了减少卷积核的参数和计算量，作者在 $3 \times 3$ 和 $5\times 5$ 卷积核前面加入 $1 \times 1$ 的卷积核，这样做有两个优点

  - 减少计算量和参数量，假如图像维度为 $28 \times 28 \times 192$，用 128 个 $3 \times 3$ 的卷积核去卷积，参数量为 $3 \times 3 \times 192 \times 128 \approx 200k$，而如果先用 96 个 $1 \times 1$ 的卷积核再用 128 个 $3 \times 3$ 的卷积核会得到相同维度的结果，但是参数量则为 $1 \times 1 \times 192 \times 96 + 3 \times 3 \times 96 \times 128 \approx 130k$，所以参数量减少了很多。
  - 两层卷积层能带来更多的 non-linearity。

  Inception module 的 max-pooling 后同样接了 $1\times1$ 的卷积核，进而减少 channel 数量，因为 pooling 操作是不会改变 channel 数量的。

 {% asset_img 1.png %}

  

GoogLeNet 总共有 22 层（只算带参数的层），网络结构如下，输入为 $224 \times 224 \times 3$ 的 RGB 图像，SAME 卷积，#$ 3 \times 3 \ reduce$ 指的是 $3 \times 3$ 前面的 $1 \times 1$ 卷积核的个数。

{% asset_img 2.png %}



**注意：** GoogLeNet 中作者认为网络太深可能会影响梯度的反向传递，因此设置了两个辅助输出，分别在 Inception(4b) 和 Inception(4e) 后 接入一个小网络输出结果，因此整个网络会有 3 个 loss，这样可以增加梯度的回传，还有附加的正则化作用。


# 测试细节

用的是 multi-crop 的方法，和 AlexNet 一样，但是 GoogLeNet 将这种方法做到了极致。

- 首先将原图像 rescale 到 4 种尺度，最小边分别为 256、288、320 和 352。此时得到 4 种 crop。
- 然后沿着最小边从左中右三个方位裁剪出 3 张正方形图像。即假设上一步的图像为 $N \times 288$，然后就左中右的裁剪出 3 个 $288 \times 288$ 的图像。此时得到 $4 \times 3$ 种 crop。
- 对于上面的每一张图，再从 4 个 corner 和中间裁剪出 $224 \times 224$ 的图，外加一张 rescale 到 $224 \times 224$ 的图，比如上一步得到的是 $288 \times 288$，将整张图片缩小到 $224 \times 224$；再将这 6 张图片进行水平镜像。此时得到 $4 \times 3 \times 6 \times 2 = 144$ 种 crop。

这种方法相当于 scale jittering，不过做得非常极致


<br>
*In Conclusion*

GoogLeNet 首次将网络从深度和宽度上进行了探索，并且尝试不同的卷积 block，为后面的研究奠定了一些基调。

<br>