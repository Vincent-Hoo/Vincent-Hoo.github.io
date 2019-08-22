---
title: 【论文阅读】—— Rethinking the Inception Architecture for Computer Vision
date: 2018-12-03 19:16:01
mathjax: true
tags:
---

> Title：Rethinking the Inception Architecture for Computer Vision
> Authors：Christian Szegedy et al.
> Session：CVPR, 2016
> Abstract：Inception 的作者进一步地优化网络结构，提出了非对称卷积的方法，使得网络的训练速度和性能得到进一步的增强，该版本被称为 Inception-v3。



<!-- more -->

<br>

# Inception-v2

作者先提出了一些修改，并将其命名为 Inception-v2，然后再 Inception-v2 的基础上再提出 Inception-v3，下面先列举 Inception-v2 相对于 Inception-v1 的改变。

- 将大卷积核分解

  - 将一个 $5 \times 5$ 的卷积核分解为两个 $3 \times 3$ 的卷积核，和 VGG 一样。

    
{% asset_img 1.png %}

  - 采取非对称卷积。将 $3\times1$ 的卷积和 $1\times3$ 的卷积串联起来，与直接进行 $3\times3$ 卷积的结果是等价的。这种卷积方式大大降低了参数量，从 $n\times n$ 降到了 $2\times n$，所以当 n 越大，降低得越多。但是也并不是适用于所有的卷积方式，论文说明，在实践中在 feature map 为 $12\times 12$ ~ $20\times20$ 时效果较好，也就是在较深层使用时效果好，浅层不太行，并且使用 $7\times1$ 和 $1\times7$ 卷积的串联可以得到很好的效果。

    下图的最后边的 Inception block 中，$1 \times 1$ 的卷积核后面，并联了两个卷积核，分别是 $1 \times 3$ 和 $3 \times 1$，注意他们的卷积方式是 SAME，因此不会出现 dimension dismatch 的情况。 

{% asset_img 2.jpg %}

- auxiliary classifier 重新解释

  - 作者认为辅助的分类器并不能帮助网络收敛，实验表明有无辅助的分类器，网络在收敛前几乎一样，也就是意味着辅助分类器并不是像 Inceptioin-v1 论文里面提到的解决梯度问题，且没有辅助分类器的网络准确率更高。
  - 作者重新解释，认为辅助分类器可能起到正则的作用。

- feature map size reduction

  - 在 Inception-v1 中作者通过 Inception block 之间插入 pooling layer 来降维。

  - 而 Inception-v2 里，作者像 Inception-bn 那样，通过在某一个 Inception block 里面调整卷积核和池化层的 stride 来进行降维

    {% asset_img 3.png %}



整个网络的结构如下图，第一个 Inception 采用的就是大卷积核转两个小卷积核，第二第三的 Inception 采用的是非对称卷积。

{% asset_img 4.png %}

<br>

# Inception-v3

作者在 Inception-v2 的基础上，对辅助分类器加上 batchNorm，得到最好的分类结果，并称其为 Inception-v3。

简单理解就是：**Inception-v3 = Inception-v2 + Factorization + Batch Normalization** 

{% asset_img 5.png %}








<br>