---
title: 【论文阅读】—— Visualizing and Understanding Convolution Networks
date: 2018-11-26 21:54:41
mathjax: true
tags:
---


> Title：Visualizing and Understanding Convolutional Networks
> Authors：Matthew D. Zeiler，Rob Fergus
> Session：ECCV, 2014
> Abstract：作者提出了一种可视化卷积神经网络的方法，并且根据可视化结果，将 AlexNet 网络参数进行调整，在 ImageNet 上的结果有明显提升，并且得到了 ILSVRC 2013 的冠军，并将网络取名 ZFNet（两位作者的名字首字母）。



<!-- more -->

<br>

# AlexNet as prerequisite

AlexNet 是由 Alex Krizhevsky 提出并获得 ILSVRC 2012的冠军，网络结构如下，网络采用了5个卷积层和2个全连接层，并且用ReLu作为激活函数，采用了Dropout等正则手段。

{% asset_img AlexNet.png %}

<br>

# ZFNet

ZFNet 在 AlexNet基础上进行了微妙的调整，将第一个卷积层的 filter 从 $11 \times 11$ 调整为 $7 \times 7$，并且将 stride 从 4 减小到 2，除此之外，ZFNet 和 AlexNet 完全一样，有人会觉得 ZFNet 就是将 AlexNet 做了一个调参工作，然后就拿了一个比赛冠军且发了一篇论文，事实确实是这样的。。

{% asset_img ZFNet.png %}

但是这篇文章的主要贡献并不在于 ZFNet 的提出，而是在于作者是如何可视化卷积神经网络，从而找到调整 AlexNet 的方向，这才是这篇论文的精髓所在。

<br>

# Visualization Approach

作者想探索输入图片的每个 pixel 和网络中间的卷积层输出的联系，卷积层输出又称为 feature map，因此作者提出了一种能将中间卷积层的输出逆映射到输入空间的方法，采取的方法是deconvolution network。

正常的 convolution network 由卷积层、激活函数、池化层组成，deconvolution network要进行逆映射，将前向传递的过程逆转。deconvnet 包含三个部分

- Unpool：作者在 convnet 中采用的是 max-pooling，在进行反转的时候，我们需要知道最大值所在的位置，因此作者在 convnet 中保存了一个 switch，用来指示最大值所在的 pixel 位置。

  {% asset_img unpool.png %}

- Rectification：convnet 中用的是 Relu 作为非线性激活函数，在 deconvnet 中同样采用 Relu。

- Filtering：convnet 中用 filter 来对上一层的输出进行卷积操作，而 deconvnet 用转置的 filter 对 rectified unpooled map 进行卷积。

{% asset_img deconvnet.png %}

<br>

# Visualization Discoveries

## Feature Visualization

- 对于每一层的 feature map，在数据集上选取激活值最强的 9 张图，画成一个九宫格。把它们映射回输入空间后可以看到不同结构的重建特征图（灰色图），以及这些特征图对应图像块（彩色图）。
- 灰色图并不是模型的样本，灰色图展示了使得该层激活值较大的图的pattern，也变相说明了每一个卷积层究竟在挖掘什么样的特征。
- 从结果可以看出，前面的层学习一些基本的形状，后面的层学习的内容更加高级，越多样性。

{% asset_img 1.png %}

## Feature Evolution during Training

- 下图是随着训练的迭代，feature map 的变化，每一层的八列分别代表不同 epoch 时的特征图
- 可以看出，低层的 feature map 收敛较快，高层的收敛较慢，要到 40-50 个 epoch 才开始收敛。

{% asset_img 2.png %}


## Architecture Selection

下面展示的是作者为什么调整 AlexNet 第一层的原因，可以看出作者调整过后的 feature map 要更加的清晰，意味着 Alexnet 的卷积核过大，忽略了一些细节信息。

{% asset_img 3.png %}


## Occlusion Sensitivity

- 作者通过遮挡图片的不同位置，观察 feature map 以及最后分类结果的变化，从而推断模型是否真的学习到 object 的位置。
- 结果表明，模型知道物体的位置，因为当遮挡住物体的时候，分类结果迅速下降。
- b 图展示的是第五层的激活数值，可以看出当遮住小狗的脸的时候，激活值迅速下降。
- d、e 图和 b 图差不多，当遮住小狗的脸的时候，分类的正确率迅速下降。

{% asset_img 4.png %}




<br>