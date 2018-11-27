---
title: 【论文阅读】—— OverFeat, Integrated Recognition, Localization and Detection using Convolutional Networks
date: 2018-11-27 23:07:47
mathjax: true
tags:
---



> Title：OverFeat: Integrated Recognition, Localization and Detection using Convolutional Networks
> Authors：Pierre Sermanet，Yann LeCun，et al
> Abstract：本文改进了AlexNet，通过一个卷积网络来同时进行分类，定位和检测三个计算机视觉任务，并在ILSVRC 2013 中获得了很好的结果。 



<!-- more -->
<br>
*Preface*

> Along with this paper, we release a feature extractor named “OverFeat” 

作者在论文中提到 OverFeat 是一个特征提取器，因为作者充分利用了卷积神经网络特征提取的功能，通过分类任务来训练卷积神经网络，再将卷积层提取到的特征应用于定位和检测的任务上，只需要改变网络最后的几层，即可实现不同任务间的切换。

论文的motivation为：卷积神经网络对于标签数据非常依赖，而得益于大数据集，如 ImageNet，才取得了突破。但是 ImageNet 数据集上的分类图片，物体大致分布在图片中心,但是感兴趣的物体常常在尺寸和位置（以滑窗的方式）上有变化。而解决这个问题传统上有几种方法：

1. 在不同位置和不同缩放比例上应用卷积网络，即使用一个大小变化的滑窗来遍历图片，但是这种滑窗的可视窗口可能只包涵物体的一个部分，而不是整个物体；对于分类任务是可以接受的，但是对于定位和检测有些不适合。 
2. 训练一个卷积网络不仅产生类别分布，还产生一个物体位置的预测和bounding box的尺寸。
3. 积累在每个位置和尺寸对应类别的置信度。

上面提到了该论文通过一个网络解决了三个视觉任务：分类，定位和检测。

- 分类任务：给定一张图片，模型需要输出这张图片的class。
- 定位任务：给定一张图片，模型不仅需要输出这张图片的class，还要定位出物体的位置，用一个bounding box的形式。
- 检测任务：给定一张图片，图片中有多个 object，需要把它全部找出来，包括位置和类别。


<br>
# AlexNet as Prerequisite

这篇论文的网络结构跟 AlexNet 基本类似，AlexNet 的网络结构在这里不再赘述，因为这篇论文对 AlexNet 改动最大的地方在于测试阶段，这里回顾一下AlexNet 的训练和测试

- 训练阶段：每张训练图片的维度为 $256 \times 256$，裁剪出 $224 \times 224$ 大小的图片作为网络的输入。
- 测试阶段：输入图片为 $256 \times 256$，从图片的四个角和中心裁剪出 5 张 $224 \times 224$ 的图片，然后对这 5 张图片进行水平镜像操作，共得到 10 张图片。将这 10 张图片输入到网络，得到 10 个预测结果，最后取平均。

OverFeat 在训练阶段和 AlexNet 完全一样，但是在测试阶段有很大的差别。


<br>
# Fully Convolution Networks

FCN 为全卷积神经网络，顾名思义就是整个网络结构全是卷积层，没有全连接层，那么如何将一个普通的网络（卷积网络 & 全连接网络）转成一个全卷积神经网络呢？

1. 将卷积层到全连接层，看成是对整张图片的卷积操作，即 filter size = feature map size。
2. 将全连接层到全连接层，看成是采用 $1\ast1$ 大小的卷积核的卷积层。

如下图，pooling 之后的 feature map 大小为 $5 \times 5$，通过一个 $5 \times 5$ 的卷积核就等同于将图片 flatten 之后再做全连接。而后面通过 $1 \times 1$ 的卷积核也等同于在做全连接网络的前向传播。

{% asset_img fcn1.png %}


假如将上面例子的输入图片改成 $16 \times 16$，会有什么结果呢？如下图，网络最后的输出为 $2 \times 2$ 的 feature map，相当于有一个 $14 \times 14$ 的 sliding window 在 $16 \times 16$ 的图片上滑动，stride 为 2，所以输出的左上角对应原图像左上角的 $14 \times 14$ patch 通过网络的结果。

{% asset_img fcn2.png %}

从这里，我们可以看出 FCN 相对于传统网络（卷积层 & 全连接层）的优势，FCN 的输入的维度并不固定，而传统网络的输入维度必须固定，因为卷积层转全连接层的时候，全连接层的输入维度是固定的。

AlexNet在测试阶段的时候，采用了对输入图片的四个角落进行裁剪，进行预测，分别得到结果，最后的结果就是类似对应于上面 $2\ast2$ 的预测图（实际大小为 $2 \ast 2 \ast num \_class$ ）。这个 $2\ast2$ 的每个像素点，就类似于对应于一个角落裁剪下来的图片通过网络得到的预测分类结果。只不过 AlexNet 把这4个像素点，相加在一起，求取平均值，作为该类别的概率值。而 OverFeat 采用另外的方法。


<br>
# Offset Max-Pooling

先举一个一维的例子，有 20 个神经元，选择 size = 3 的 non-overlapping 池化层，那么结果将会是下面 $\bigtriangleup = 0$ 的情况；我们除了以 1 作为初始位置外，还可以从位置 2 或者 3 开始，它们分别对应 $\bigtriangleup = 1$ 和 $\bigtriangleup = 2$ 的 情况，在一般的 CNN 网络中，我们只用 $\bigtriangleup = 0$ 的结果，而 offset pooling 可以设置 offset，从而得到多个池化的结果。

{% asset_img 1d_offset_pool.png %}

从一维升级到二维的情况，我们分别在两个维度进行 offset pooling，那么 $(\bigtriangleup_x, \bigtriangleup_y)$ 就会共有 9 种池化结果。如果我们在做图片分类的时候，在网络的某一个池化层加入了 offset pooling，然后把这 9 种池化结果，分别送入后面的网络层，最后我们的图片分类输出结果就可以得到 9 个预测结果，输出的实际维度为 $ 3  \ast  3 \ast num \_class $ （每个类别都可以得到 9 种概率值，然后我们对每个类别的 9 种概率，取其最大值，做为此类别的预测概率值）。 


<br>
# OverFeat Classification

## 网络架构

论文提出了两种网络架构，分别是 Fast 模型和 Accurate 模型，网络结构分别如下，两种架构都是在 AlexNet 上进行修改，对比 AlexNet 区别在于

- 训练时输入维度固定，测试时用多尺度输入（multi-scale），即输入的维度不固定。
- 没有用 local response normalization，因为发现没什么用。
- 没有采用 overlap 的 max-pooling。
- 第一层卷积层的参数作了稍微改动，accurate模型第一个卷积核改为 $7 \times 7$，stride为 2。3,4,5 层的卷积核个数进行了修改。

其他方便和 AlexNet 一样，训练的参数几乎一致。

{% asset_img fast.png %}
{% asset_img accurate.png %}





OverFeat 最具创新的地方在于它的测试过程，作者认为 AlexNet 测试的做法存在两个问题

- 将原来的图片从 4 个角落里裁剪出图片，会把原图片的很多区域给忽略。
- 裁剪窗口会存在重叠，因此会有冗余的计算量。

而 OverFeat 算法在测试过程，不再是固定维度的一张图片，而是采用 6 张维度不同的图片作为输入，这就是 multi-scale 多尺度，如下表格，输入的维度各不相同，然后当前向传播到第五层的时候（accurate model 的第五层是卷积层过渡到全连接层），进行 offset pooling，从 layer-5 pre-pool 到 layer-5 post-pool，这一步的实现是通过池化大小为 (3, 3) 进行池化， $\bigtriangleup_x =1,2,3$，$\bigtriangleup_y = 1,2,3$，因此对于每一个 feature map，会得到 9 个池化结果图，比如下面 scale-1，$17 \times 17 \rightarrow (5 \times 5) \times(3 \times 3)$ 。

{% asset_img multi-scale.png %}

从 layer-5 post-pool 到 classifier map(pre-reshape)：我们知道在训练的时候，从卷积层到全连接层，输入的大小是 $4096 \ast(5\ast5)$。但是我们现在输入的是各种不同大小的图片，因此就需要采用 FCN 的方法，让网络继续前向传导。我们从 layer-5 post-pool 到第六层的时候，如果把全连接看成是卷积，那么其实这个时候卷积核的大小为 $5 \ast5$，因为训练的时候，layer-5 post-pool 得到的结果是 $5 \ast5$。因此在预测分类的时候，假设layer-5 post-pool 得到的是 $7\ast9$（上面表格中的 scale-3），经过 $5\ast5$ 的卷积核进行卷积后，那么它将得到 $(7-5+1)\ast(9-5+1)=3\ast5$的输出。

注意，因为我们采用了 offset pooling，因此 classifier map (pre-reshape) 的实际维度都会乘以 9 倍，因此对于每一个 scale，由于 FCN 和 offset pooling，我们会得到很多个预测结果，比如 scale-3，我们得到 105 个（FCN导致15个，offset导致9个），然后就取最大的作为该 scale 的预测值，分析不同的 scale 得到最后的预测值。


<br>
*In Conclusion*

定位和检测任务在这里不做说明，一来不是我关注的重点且论文没有多说，二来它们只是把后面的全连接网络进行了修改而已。

这篇论文的核心在于 FCN 和 offset pooling，以及迁移学习的重要性，卷积神经网络在挖掘特征上很强势，因此才可以将特征提取层迁移到其它任务上。

 

 

 

 

 

 

 

 

 

 

<br>