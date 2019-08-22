---
title: >-
  【论文阅读】—— Inception-v4, Inception-ResNet and Impact of Residual Connection on
  Learning
date: 2018-12-03 19:17:16
mathjax: true
tags:
---


> Title：Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning
> Authors：Christian Szegedy, Sergey Ioffe
> Session：AAAI, 2017
> Abstract：作者修改了 Inception-v3 为 Inception-v4，并且将 Inception 和 ResNet 进行结合。两个网络的性能接近。



<!-- more-->

<br>

# Inception-v4

{% asset_img 1.png %}




<br>
# Inception-ResNet

结构和上面基本相同，只是将 Inception-block 改为 Inception-ResNet-block，列举几个 block。作者在论文中提到了两个版本的 Inception-Resnet。

{% asset_img 2.png %}
{% asset_img 3.png %}



还有一个 trick 就是作者发现当卷积核的个数超过 1000 的时候，网络前向传播的值会偏向于 0，因此对 residual 进行放缩。

{% asset_img 4.png %}


<br>
# 实验结果

Inception-ResNet-v1 和 Inceptin-v3 计算代价差不多，Inception-ResNet-v2 和 Inception-v4 计算代价差不多，然而实作上 Inception-v4 慢很多可能是因为层数太多。 而 Inception-ResNet 要收敛更快。

{% asset_img 5.png %}



<br>