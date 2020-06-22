---
title: >-
  【论文阅读】——Be Your Own Teacher, Improve the Performance of Convolutional Neural
  Networks via Self Distillation
mathjax: true
date: 2019-11-08 20:03:12
tags:
---

>Title：Be Your Own Teacher, Improve the Performance of Convolutional Neural Networks via Self Distillation
>Author：Linfeng Zhang, et al.
>Session：ICCV, 2019
>Abstract：作者提出一种新的 self-distillation 的策略，用网络深层的特征和预测结果去监督浅层网络的学习。

<!--more-->

<br>

## Method

作者的方法比较直观简单，如下框图，作者共提出了三种监督信息，全都是从网络深层传至网络浅层，网络共分为四块，每一块得到一个特征图，特征图后接全连接网络和 softmax 得到预测结果，作者称其为浅层 classifier 和深层 classifier。三种监督信息分别是

- 特征监督，深层的特征图对比浅层特征图。
- 标签监督，网络最后 softmax 输出的软标签对比前面 3 个浅层分类器得到的软标签。
- 真实数据监督，one-hot 对比各个分类器的输出。

整体的 loss 如下，C 为分类器的个数，即将网络切分的个数。
$$
loss = \sum_{i=1}^C (1-\alpha)\cdot CrossEntropy(q^i,y)+\alpha \cdot KL(q^i,q^C)+\lambda \cdot ||f_i-F_C||_2^2
$$


{% asset_img 1.png %}



<br>

## Experiment

作者展示了四个分类器的结果，以及集成的效果，对比 baseline 和 classifier 4/4，可见自蒸馏的效果。

{% asset_img 2.png %}

对比其他特征蒸馏的方法，也是一致的提升。

{% asset_img 3.png %}

<br>

## Discussion

作者的另外一篇文章， SCAN: A Scalable Neural Networks Framework Towards Compact and Efficient Models，两者用的是同一个框架和训练思路，但是出发点不同，这篇文章的出发点在于自蒸馏，而 SCAN 出发点在于 adaptive computation，即自适应地去选择需要经过的网络层，如果浅层分类的结果还令人满意，就不再经过后面的网络，从而实现加速。网络结构基本一样，除了加入了注意力模块。

{% asset_img 4.png %}

accuracy 和 acceleration的 tradeoff 如下。

{% asset_img 5.png %}

<br>