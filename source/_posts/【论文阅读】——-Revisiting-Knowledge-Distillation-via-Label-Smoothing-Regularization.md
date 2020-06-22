---
title: 【论文阅读】—— Revisiting Knowledge Distillation via Label Smoothing Regularization
mathjax: true
date: 2020-06-22 10:01:02
tags:
---

> Title : Revisiting Knowledge Distillation via Label Smoothing Regularization
> Authors : Li Yuan, et al
> Conference : CVPR 2020
> Abstract : 作者做了两组 demo 实验，一是用学生网络蒸馏教师网络，二是用未完全训练的教师网络（性能比学生网络还差）去蒸馏学生网络，两组实验的结果均有效。从这两组反常的实验中，作者提出了知识蒸馏的另一个解释：label smoothing regularization，并提出了两个 teacher-free 的知识蒸馏方法，实验表明均有效。



<!--more-->

<br>

## Exploratory Experiments and Counterintuitive Observations

我们一般对知识蒸馏的印象是：通过一个复杂的教师网络，把教师网络的输出对学生网络进行指导，从而提升学生网络的性能。而对分类任务中，知识蒸馏的有效性解释一般为软标签中蕴含了类别之间的相关性信息，这是软标签对比于硬标签的优势。

这里作者探索了两种特殊的知识蒸馏设定

- 学生网络指导教师网络，叫做 reversed knowledge distillation (Re-KD)
- poorly-trained 的教师网络指导学生网络，这里的 poorly-trained 指的是教师网络只训练了几个迭代，结果远比学生网络差，叫做 defective knowledge distillation (De-KD)

{% asset_img 1.png %}

按照我们对知识蒸馏的理解，上面这两种知识蒸馏都不会有效，可是作者进行了大量的实验证明，这两种蒸馏均是有效果的，这驱使作者去思考知识蒸馏背后的解释。

{% asset_img 2.png %}

{% asset_img 3.png %}

<br>

## Knowledge Distillation and Label Smoothing Regularization

上面的实验结果也反映了，知识蒸馏有效性的解释不止在于软标签中蕴含的类别相似性信息，因为学生网络或者 poorly-trained 的教师网络他们的软标签里面相似性信息肯定不足，并且甚至很多软标签本身分类就是错误的，这样的软标签竟然有用，作者给出的解释是，这里的软标签就类似于一种标签平滑的正则，下面先回顾一下 label smoothing regularization (LSR)。

在 LSR 里面，标签并不是一个 one-hot 的硬标签，而是经过一定的平滑，没有 LSR 的时候，网络的损失函数如下，其中 $q(k)$ 就是 one-hot 的硬标签。
$$
H(q,p)=-\sum_{k=1}^Kq(k)log(p(k))
$$
而 LSR 是把 $q(k)$ 进行平滑，加入了一个均匀分布的 $u(k)=1/K$， 使得
$$
q^{\prime}(k)=(1-\alpha)q(k)+\alpha u(k)
$$
此时的损失函数变为

{% asset_img 4.png %}

由于 $H(u)$ 是一个常数
$$
L_{LS}=(1-\alpha)H(q,p)+\alpha D_{KL}(u,p)
$$
而知识蒸馏的设定里面，学生网络的损失函数同样包含两项，$\tau$ 是温度 temperature
$$
L_{KD}=(1-\alpha)H(q,p)+\alpha D_{KL}(p_{\tau}^t, p_\tau)
$$
对比 $L_{LS}$ 和 $L_{KD}$，唯一不同在于 $u$ 和 $p_\tau ^t$，我们可以将 KD 看成是一个特殊的 LSR，特殊点在于用于平滑的分布是一个学习出来的教师网络，而不是一个预先设定好的分布；或者我们可以将 LSR 看成一个特殊的知识蒸馏，$u$ 是一个虚拟的教师网络，对于每个类别都给出相同的概率。

从下面的可视化图看出，当增大温度，网络的输出就更加趋向于一个均匀分布，KD 也就越加取向 LSR

{% asset_img 5.png %}

因此 Re-KD 和 De-KD 可以解释为这些 poorly-trained 的网络输出的软标签在高温度下能给学生网络提供一个正则的作用，因此上面的实验结果有效更多是在高温度的设定下进行的，作者在附录中也给出了参数，温度设为 20.

<br>

## Teacher-free Knowledge Distillation (Tf-KD)

基于上面的结论，软标签更多的作用在于正则项，而不是类别相似性，因此我们可以压根不需要教师网络来提供这样的一个正则项，作者提出了两种 Tf-KD，第一种为 $Tf-KD_{self}$，这个更多像 born again network，先训练一个网络，再用该网络作为教师网络，重新训练一个学术网络。

第二种为 $Tf-KD_{reg}$，通过自己设定了一个教师网络的输出，使得网络在正确类别上有着相当高的概率，这里的 $a$ 通常设置为 0.9 以上

{% asset_img 6.png %}

同样的，两种设定下的温度系数都是 $\tau \ge 20$，使得软标签更像 LSR

<br>

## Experiments

{% asset_img 7.png %}

{% asset_img 8.png %}

<br>

