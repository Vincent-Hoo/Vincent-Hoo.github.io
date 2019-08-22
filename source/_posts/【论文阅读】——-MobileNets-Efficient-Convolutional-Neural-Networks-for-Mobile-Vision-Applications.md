---
title: >-
  【论文阅读】—— MobileNets, Efficient Convolutional Neural Networks for Mobile Vision
  Applications
date: 2018-12-04 23:24:47
mathjax: true
tags:
---

> Title：MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications
> Authors：Google 团队
> Abstract：MobileNet 是为移动和嵌入式设备提出的高效模型。 使用了**深度可分离卷积**来构造的轻量级深度神经网络，计算量大幅度减小，且性能能媲美其它网络。

<!--more-->

<br>

# Depthwise Separable Convolution

假设卷积输入维度为 $D_F \times D_F \times M$，卷积核为 $D_K \times D_K \times M \times N$，采取的是 SAME 卷积，因此卷积输出为 $D_F \times D_F \times N$，这种标准的卷积层的计算量为

$$D_K \cdot D_K \cdot M \cdot N \cdot D_F \cdot D_F$$

而 Depthwise Separable Convolution 分为两部分：depthwise convolution 和 pointwise convolution。

depthwise 卷积使用 M 个大小为 $D_K \times D_K \times 1$ 的卷积核分别对输入的 M 个 channel 进行卷积，即每个卷积核负责一个 channel，如果采取的是 stride 为 1 的 SAME 卷积，那么卷积出来的结果维度和输入一样 $D_F \times D_F \times M$，depthwise 卷积的计算量为

$$D_K \cdot D_K \cdot M  \cdot D_F \cdot D_F$$

pointwise 卷积是对 depthwise 卷积的结果再进行 channel 级别上的 interaction，用 N 个 $1\times 1 \times M$ 的卷积核去处理，得到结果大小为 $D_F \times D_F \times N$，pointwise 卷积的计算量为

$$1 \cdot 1 \cdot M \cdot N \cdot D_F \cdot D_F$$

总的 depthwise separable convolution 的计算量为

$$D_K \cdot D_K \cdot M  \cdot D_F \cdot D_F + M \cdot N \cdot D_F \cdot D_F$$

对比标准的卷积，如果用 $3 \times 3$ 的卷积核，大概能减少个 8 - 9 倍的计算量。

$$\frac{D_K \cdot D_K \cdot M  \cdot D_F \cdot D_F + M \cdot N \cdot D_F \cdot D_F}{D_K \cdot D_K \cdot M \cdot N \cdot D_F \cdot D_F} = \frac{1}{N} + \frac{1}{D_K^2}$$

下图是标准卷积核和深度可分离卷积核的对比

- standard conv 是既有 feature map 的 local 信息的挖掘，也有 channel 之间信息的挖掘，因为每一个卷积核的深度就等于输入 channel 的大小，但是这也是它计算量大的原因，尤其当输入 channel 比较大的时候，卷积核的 channel 也必须与输入的 channel 大小 match。
- depthwise conv 将输入图像的每一个 channel 单独用一个卷积核去过滤，相当于每一个 channel 先进行一个过滤操作，但是它并没有 channel 之间的 interaction。
- point-wise conv 就等于 standard conv，组合不同 channel 的信息，来构造新的特征。

因此 depthwise conv & pointwise conv 就能比较好地去代替 standard conv。

{% asset_img 1.png %}

<br>

# MobileNet

网络结构如下，MobileNet 去掉了用池化层去降维，而是用 stride 为 2 的卷积层，Conv dw / s1 表示 stride 为 1 的 depthwise conv，$3 \times 3 \times 32$ dw 表示有 32 个 $3 \times 3$ 的卷积核。

{% asset_img 2.png %}

下图展示的是 MobileNet 不同 component 的计算量占比，可以看出 Conv $1 \times 1$ 计算量和参数占比都是最大，那是因为 pointwise conv 实际上就是一个 standard conv，连接全部 channel 来输出一个特征，所以计算量也大。而 depthwise conv 计算量就少很多。

{% asset_img 3.png %}

<br>

# Two Multipliers

我们把上面的 MobileNet 称为 baseline model。

**Width Multiplier：Thinner Models**

宽度乘子 $\alpha$ 的作用在于把每一层的 channel 数量进行统一的缩减，因此输入 channel 大小为 M 则变为 $\alpha M$，输出 channel 大小为 N 则变为 $\alpha N$。实验中 $\alpha$ 取值 1，0.75，0.5，0.25，$\alpha = 1$ 则为 baseline model，加上宽度乘子后的计算量为

$$D_K \cdot D_K \cdot \alpha M  \cdot D_F \cdot D_F + \alpha M \cdot \alpha N \cdot D_F \cdot D_F$$



**Resolution Multiplier：Reduced Representation**

分辨率说的就是 feature map 的大小，分辨率乘子 $\rho$ 作用于 feature map 的宽和高，使得 feature map 的大小得到缩减，作者实验中取 feature map 大小为 224，192，160 和 128 来模拟不同的 $\rho$ 值，224 则为 $\rho = 1$ 的情况，加上分辨率乘子的计算量为

$$D_K \cdot D_K \cdot \alpha M  \cdot \rho D_F \cdot \rho D_F + \alpha M \cdot \alpha N \cdot \rho D_F \cdot \rho D_F$$

假设输入为 $14 \times 14 \times 512$，standard conv 的卷积核为 $3 \times 3 \times 512 \times 512$，计算量为 $14 \cdot 14 \cdot 512 \cdot 3 \cdot 3 \cdot 512 \approx 462M$，而 depthwise separable conv 的计算量降了接近十倍，加上乘子后进一步减少计算量。

{% asset_img 4.png %}

<br>

# 实验结果

**depthwise separable conv vs full conv**

depthwise 虽然准确率不如 full，但是参数量和计算量大幅度的减少

{% asset_img 5.png %}

**thinner vs shallower**

thinner 指用宽度乘子为 0.75，shallower 指将原网络结构的 5  层 feature map 大小为 $14 \times 14 \times 512$ 的卷积层去掉。两者的参数量和计算量都差不多，但是明显 thinner 要比 shallower 要好。

{% asset_img 6.png %}

**乘子不同取值比较**

当宽度乘子不断减小，channel 数量不断减小，准确率不断下降，计算量也大幅度下降，同理分辨率乘子。

{% asset_img 7.png %}

宽度乘子和分辨率乘子各取 4 个值，共得到 16 个模型，作者比较这 16 个模型的计算量、参数量和准确率之间的关系，结果如下，毫无疑问，计算量越大或者参数越多，准确率越高，但是作者需要寻找的是他们两者之间的一个 tradeoff，找到一个既能保证准确率，计算量又不算太大的模型。

{% asset_img 8.png %}

同时作者对比了 baseline mobileNet 和 GoogLeNet，VGG 在 ImageNet 上的性能，mobileNet 比 GoogLeNet 要好，比 VGG 稍微差一点，但是计算量和参数却是另外两个模型无法比拟的。

{% asset_img 9.png %}

作者对比缩减版的 mobileNet 和其它模型，也是相同的结论。

{% asset_img 10.png %}

作者还在其它的分类数据集上进行了实验，还在其它的视觉应用上进行了实验，如人脸识别，face attribute，用到了蒸馏技术。



<br>

*In Conclusion*

mobileNet 提供给我们的是一种轻量级别的深层神经网络，更加值得我们思考的是，我们是否需要一味地追求准确率，还是需要在准确率和计算性能上进行平衡。

<br>