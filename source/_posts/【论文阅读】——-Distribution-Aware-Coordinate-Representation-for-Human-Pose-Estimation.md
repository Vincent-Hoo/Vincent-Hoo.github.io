---
title: >-
  【论文阅读】—— Distribution-Aware Coordinate Representation for Human Pose
  Estimation
mathjax: true
date: 2020-09-15 16:10:05
tags:
---

> Title: Distribution-Aware Coordinate Representation for Human Pose Estimation
> Authors: Feng Zhang, Xiatian Zhu, et al
> Conferences: CVPR 2020
> Abstract: 作者发现姿态检测任务中，除了网络结构的设计之外，label representation 也很重要，即如何在 keypoint coordinate 和 heatmap 之间相互转换，作者提出了一种新的转换方式，能无缝地与各种姿态检测网络相结合，并恒定地提升网络的性能。

<!--more-->

*prefix*

姿态检测任务不是一个简单的任务，它的难点在于关节点的遮挡、复杂的背景等，过去的很多工作都在设计更深更强的网络结构，而忽略了最本质的 label representation 的部分，姿态检测的 label 分为两类，一种是直接回归坐标，第二种是热力图回归，即先将坐标点转化成热力图上响应最大的点，然后网络输出一个热力图。第二种方法成为了现今姿态检测的主流方法，但回归热力图带来的问题就是计算开销大，所以常规的做法会先有 resolution reduction 后 resolution recovery，如下图，首先会将 bbox 下采样到一个 predefined 的分辨率，热力图也设置成一个预先定义好的分辨率，网络输出的热力图最后会上采样会原来的 image space。

作者把 coordinate 到 heatmap 的过程定义为 coordinate encoding，heatmap 到coordinate 到过程定义为 coordinate decoding，值得注意的是，encoding 的过程有 resolution reduction，所以会引入量化误差，为了解决这一问题，现在一般的做法是在 decoding 的时候，会有一个 0.25 像素的偏移来弥补这个误差，这个做法能够明显提升最后的结果。

{% asset_img 1.png %}

<br>

## DARK

作者提出的 DARK 分为 encoding 和 decoding 部分

### Coordinate Decoding

标准的 coordinate decoding 做法是从 heatmap 中挑出响应值最大和次大的两个点 $m$ 和 $s$，最后的点会是最大点向次大点偏移 0.25 个像素，最后再恢复成原图像 space。这种 sub-pixel shifting 是用来补偿 resolution reduction 带来的量化误差，热力图上响应值最大的点未必对应原空间上最大的点，可能只是一个粗略的坐标。
$$
p = m + 0.25 \frac{s-m}{||s-m||_2}
$$

$$
\hat{p} = \lambda p
$$

这种做法是 hand-crafted 的，并且没有太多的理论依据，作者提出一种基于分布的 decoding 方法，来获得更加准确的 sub-pixel，首先假设输出的 heatmap 同样是服从二维高斯分布
$$
G(x;\mu,\Sigma)=\frac{1}{2\pi|\Sigma|^{1/2}}exp(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu))
$$
取 log 后得
$$
P(x;\mu,\Sigma)=ln(G)=-ln(2\pi)-\frac{1}{2}ln(|\Sigma|)-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)
$$
二元高斯分布的 $\mu$ 是中心，同样也是响应值最高的地方，因此对其求一阶导数会等于 0。
$$
D^\prime(x)|_{x=\mu}=-\Sigma^{-1}(x-\mu)|_{x=\mu}=0
$$
但由于实际上得到的 heatmap 是离散值，假设最大响应点为 $m$，我们用泰勒展开，用 $m$ 点去估计真正最大的点 $\mu$
$$
P(\mu)=P(m)+D^\prime(m)(\mu-m)+\frac{1}{2}(\mu-m)^TD^{\prime \prime}(m)(\mu-m)
$$
其中二阶导数（Hessian）为
$$
D^{\prime \prime}(m)=D^{\prime \prime}(x)|_{x=m}=-\Sigma^{-1}
$$
联合几条式子最后得到最大响应值 $\mu$ 为
$$
\mu = m - (D^{\prime \prime}(m))^{-1}D^{\prime}(m)
$$
对比现在的标准做法人为像第二大的点偏移 0.25 个像素，作者提出的做法充分利用了热力图的分布信息，但该方法是基于网络输出的分布是高斯分布的前提，但实际上输出的热力图不是完全的高斯分布，如下图，输出的热力图会出现在关节点附近好几个峰值。

{% asset_img 2.png %}

因此作者在这里对输出的热力图做了一个平滑（modulation）操作来 smooth out 其他的峰值，具体的操作是用一个高斯核 K 来进行卷积，然后再调整热力图的峰值为原热力图的峰值
$$
h^\prime = K \otimes h
$$

$$
h^\prime = \frac{h^\prime - min(h^\prime)}{max(h^\prime) - min(h^\prime)} \ast max(h)
$$

总结一下，作者提出的 decoding 分为三步

1. heatmap distribution modulation
2. Distribution-aware joint localisation by Taylor expansion
3. resolution recovery to the original coordinate space

{% asset_img 3.png %}



### Coordinate Encoding

假设真实的坐标点为 $g=(u,v)$，resolution reduction 后的坐标点为
$$
g^\prime=(u^\prime, v^\prime) = \frac{g}{\lambda}=(\frac{u}{\lambda}, \frac{v}{\lambda})
$$
然后会将 $g^\prime$ 进行量化，比如 floor、ceil、round 等操作得到 $g^{\prime\prime}$，现在 encoding 的方法就是在 $g^{\prime\prime}$ 上进行 GT heatmap 的生成，但这肯定会引入一个量化误差在里面，作者提出用 $g^\prime$ 去生成 heatmap 能够更加准确一点。



<br>

## Experiments

### Evaluating Coordinate Representation

#### Coordinate decoding

作者对比了 0.25 hand-crafted shift 和 DARK，用的网络是 HRNet-w32，输入维度是 128x96，结果表明以前的那种 0.25 shift 的方法能够极大提升性能，但竟然没有被研究

{% asset_img 4.png %}
{% asset_img 5.png %}

#### Coordinate encoding

Unbiased encoding 能恒定提升性能，无论 decoding 方式如何

{% asset_img 6.png %}

#### input resolution

降低分辨率性能掉点严重，但 DARK 在分辨率低的情况下提点比较多

{% asset_img 7.png %}

#### generality

测试了不同的网络，都能有恒定的提点

{% asset_img 8.png %}
{% asset_img 9.png %}
{% asset_img 10.png %}



<br>