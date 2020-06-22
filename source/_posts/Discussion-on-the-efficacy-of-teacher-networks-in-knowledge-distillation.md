---
title: Discussion on the efficacy of teacher networks in knowledge distillation
mathjax: true
date: 2019-11-04 21:20:24
tags:
---

本文将介绍几篇最近看到的论文，这些论文都共同讨论了教师网络的有效性，当我们在进行知识蒸馏的时候，都会想到的一个问题是：应该选择什么样的教师网络去进行知识迁移？是否越深准确率越高的大网络就一定有利于知识蒸馏呢？答案不是肯定的。这几篇文章从实验的角度给我们展示了在不同教师网络设定下的知识蒸馏的结果，并且提出改进。

本文讨论的论文如下：

- Improved Knowledge Distillation via Teacher Assistant: Bridging the Gap Between Student and Teacher
- On the Efﬁcacy of Knowledge Distillation
-  Distilling Knowledge From a Deep Pose Regressor Network



<!--more-->

<br>

## Improved Knowledge Distillation via Teacher Assistant: Bridging the Gap Between Student and Teacher

这是 DeepMind 今年放在 arxiv 上的一篇文章，作者通过一些小实验发现，在进行知识蒸馏的时候，增加教师网络的模型大小反而会使得知识蒸馏的效果变差，结果如下，学生网络是一个 2 层的 CNN，而教师网络分别是 4-10 层的 CNN，从结果可以看出，继续增加教师网络深度，并不一定能提高知识蒸馏的性能。

{% asset_img 1.png 500 500 %}

从这个 toy experiment 可以看出，教师网络和学生网络之间的 Gap 确实是影响知识蒸馏的一个重要因素，当 Gap 变大的时候，从图中可以得出以下几个结论

- 教师网络的性能不断提高，证明不是过拟合导致的知识蒸馏性能下降。
- 教师网络的 capacity 不断增大，而学生网络的 capacity 不变，当 gap 增大到一定程度，学生网络的 capacity 不足以支撑其模仿教师网络。
- 教师网络的性能不断提高，对于训练样本置信度增高，使得教师网络提供的软标签变“硬”，这也使得知识蒸馏性能下降。

为了进一步证明 gap 确实影响知识蒸馏，作者做了一个小对比实验，固定教师网络为 10 层 CNN，改变学生网络的层数，结果如下，确实当 gap 大于某个阈值或者小于某个阈值的时候，知识蒸馏的效果都不是最优。

{% asset_img 2.png 500 500 %}

基于这个观察，作者很自然地提出在大网络和小网络之间增加一个中等大小的网络（teacher assistant，TA），先用教师网络蒸馏 TA，再用 TA 蒸馏学生网络。

{% asset_img 3.png 500 500 %}

当加入 TA 后，学生网络的蒸馏效果明显比直接蒸馏教师网络要好，NOKD = 没有 KD，BLKD = baseline KD，TAKD 是作者提出的方法。

{% asset_img 4.png 500 500 %}

一个很自然的问题是，为什么要用蒸馏的 TA，而不直接训练一个 TA 呢？结果显示蒸馏的 TA 效果更好，从右图可以看出，蒸馏的 TA (KD-TA) 比直接训练的 TA (FS-TA) 要好。

{% asset_img 5.png 500 500 %}

作者同时尝试了多阶段的 TA，即蒸馏一个 TA，再用该 TA 蒸馏下一个 TA，以此类推，结果如下，越多个阶段效果越好，但是越费时间和空间，这里的结果和上面 Table 1 的结果有一点矛盾，可以看出 10 -> 2 实际就是 BLKD，但是这里显示的结果是 42.56，Table 1 显示的结果是 44.57，而 44.57 是 10 -> 6 -> 2 的结果，不知道是笔误还是什么。

{% asset_img 6.png 700 700 %}



作者同时也对比了其他蒸馏方法，BSS 为基于对抗样本的蒸馏，MUTUAL 为 deep mutual learning，虽然结果显示作者的方法结果最好，但是这些方法差异较大，没有太多比较的必要，FITNET, AT, FSP 都是特征蒸馏方法，而 TAKD 为软标签蒸馏，但是可以看出作者提出的 TAKD 还是普遍好于其他的方法。同样的，这里也有一个和 Table 1 矛盾的地方，ResNet8 的 NOKD 在 Table 1 为 88.52，这里就变成了 86.02。

{% asset_img 7.png  %}



这篇文章提出了一个影响知识蒸馏的因素：gap，学生网络的表达能力或者 capacity 有限，而如果教师网络太过强，就会提供一些超过学生网络能够学习范围的知识，就会导致反作用，因此如何找到恰当大小的教师网络是一个比较难的问题，作者采用的 TA 的方式，层层蒸馏，下一层吸收上一层的知识，再传递给下一层，确实是一个很好的办法，这解决了我们不知道选什么大小的教师网络的问题。



<br>



## On the Efﬁcacy of Knowledge Distillation

>Authors: Jang Hyun Cho,  Bharath Hariharan 
>Conference: ICCV 2019

跟上面的文章一样，作者也认为 bigger models are not better teachers，但是他并不是做的 toy experiment，而是用 wide-resnet 在 cifar 和 ImageNet 上做了实验。教师网络无论是 wide-resnet 还是 DenseNet，无论是增加深度还是宽度，都是会有 degradation。

{% asset_img 8.png  %}

{% asset_img 9.png   %}

而在 ImageNet 上的结果，有点出人意料，可能是 wide-resnet 的缘故，也可能是 ImageNet 的缘故，增加深度，性能不断下降，且 KD 完全无效。学生网络为 ResNet18，教师网络无论选什么，都比学生网络没有蒸馏的差。

{% asset_img 10.png 500 500 %}

作者将 degradation 的原因归于教师网络和学生网络 capacity 的 mismatch，教师网络太过强大，因此作者提出两种小策略，来选出 capacity 相对较小的教师网络。

### Early stopping KD(ESKD)

作者比较 ResNet18 在蒸馏下和无蒸馏下的 ImageNet 训练曲线发现，KD 一开始是有用的，但到后来就比 scratch 的要差，无论教师网络选择 ResNet34 还是 ResNet50。这两张图直接否定了 KD 的有效性，因此 ImageNet 是否是一个比较难进行知识蒸馏的数据集呢？因为很多 KD 的文章都没有在 ImageNet 上进行验证。

{% asset_img 11.png  %}

基于上面的观察，作者决定提前结束 KD 过程，即 ESKD，结果如下，这时候 KD 确实有效果了，但是网络深度增加，degradation 现象依旧存在。

{% asset_img 12.png 500 500 %}

ESKD 的训练曲线

{% asset_img 13.png  %}



### Early-stopped teacher

解决 degradation 问题，就要把 teacher 的 capacity 降到 student 能够触碰的范围内，一个简单的方法就是早停策略，即不完全训练的 teacher。不完全训练的 teacher 表现上就和小网络一样。完全训练的 teacher 的实验设定为 200 epoch，60 epoch 将学习率（60/200），不完全训练的 teacher 遵循 (15/50，10/35) 等策略。

下图为训练不完全的 teacher 用来蒸馏的效果，可以看出都比训练完全的 teacher 要好。

{% asset_img 14.png  %}

用不同的学生和教师网络同样验证 ES teacher 的有效性

{% asset_img 15.png  %}

这篇文章主要是实验型文章（实验结果很多），研究教师网络对于 KD 有效性的影响，并且提出两种策略来使得教师网络的 capacity 和学生网络相匹配，第一种方法为提前结束 KD 过程，至于为什么 KD 到了后面会不如 scratch，原因或许出于软标签上；而第二种方法为教师网络主动降低 capacity 来 match 学生网络。

目前为止的这两篇文章都是围绕如何找到 match 学生网络 capacity 的教师网络。这里改进的点有很多。

<br>

## Distilling Knowledge From a Deep Pose Regressor Network

这篇文章在前面有专门的博客介绍，其关注的是回归问题，而不是分类问题，作者认为回归问题的解空间过大，因此对于学生网络不容易学习，并且回归问题中，网络输出是一张图，而不是一个概率分布，一张图是没有任何限制的，因此教师网络很有可能会给出非常错误的知识，因为回归网络的输出并没有任何约束。所以作者提出要对教师网络的输出进行约束，约束为：若教师网络生成的结果很差，就不用 KD，在 KD Loss 前面为每个样本添加一个系数，来衡量该样本是否值得知识迁移。



<br>
