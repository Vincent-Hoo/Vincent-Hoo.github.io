---
title: 【论文阅读】—— Knowledge Distillation Meets Self-Supervision
mathjax: true
date: 2020-08-06 20:02:40
tags:
---

<br>

>Title: Knowledge Distillation Meets Self-Supervision
>Conference: ECCV 2020
>Authors: Guodong Xu, Ziwei Liu, et al
>Abstract: 作者提出用自监督的方式去提取一些更加 general 的知识，并将它用来知识蒸馏，能大幅度提升不同网络架构之间的蒸馏效果。

<!--more--> 

*prefix*

很多知识蒸馏的文章都将重点放到了如何提取网络中间层的特征，比如提取注意力图、相关性矩阵、分布统计等，但是这些方法提取到的知识都是与网络所处理的任务相关的（task-specific），并且这些知识是与网络的架构相关的（architecture-relative），所以会导致不同网络结构的知识蒸馏效果不佳。

而自监督这种训练方式，能够充分地利用数据本身去学习网络，常见的自监督任务有

- exemplar：每一个样本是一个类别，训练网络进行分类任务
- rotation：把图片进行旋转，旋转的角度有4个，然后进行四分类
- jigsaw：将图片分成四块，然后随机打乱顺序，共有24种顺序，进行24分类问题
- contrastive learning：将图片进行数据增广，然后让网络判别出原图和增广后的图是positive，与其他图是negative

针对知识蒸馏任务，作者认为需要更加通用的知识，因此将自监督作为辅助任务来获得更加general的knowledge

<br>

## Self-Supervision Knowledge Distillation(SSKD)

### Contrastive Learning as Self-supervision

给定有 N 个样本 $\{x_i\}_{i=1:N}$ ，有一个 transformation $T$ 集合，任意一个 transformation $t(\cdot) \in T$ 作用到每一个样本上得到 $\{\tilde{x}_i\}_{i=1:N}$ 。

contrastive learning 的核心是要最大化 positive pair $(\tilde{x}_i, x_i)$ 的相似性，和最小化 negative pairs $(\tilde{x}_i, x_k)_{k\neq i}$ 的相似性，相似性的定义为 cosine 相似性如下

{% asset_img 1.png %}

contrastive prediction loss 如下

{% asset_img 2.png %}

### SSKD Framework

教师网络和学生网络都由三个部分组成

- backbone $f(\cdot)$
- classifier $p(\cdot)$
- 自监督模块 SS $c(\cdot,\cdot)$

{% asset_img 3.png %}

**Training the teacher network**

- 第一阶段：标准训练，用正常的数据集去训练 $f(\cdot)$ 和 $p(\cdot)$
- 第二阶段：固定住 backbone，然后训练 SS 模块，它是一个两层的 MLP，contrastive learning 用到 4 种 transformations，分别为 color dropping、rotation、clipping + resizing、color jittering。

**Training the student network**

四个 loss 组成

- cross entropy
- kd loss，教师的输出和学生的输出

{% asset_img 4.png %}

- SS loss，约束两个相似性矩阵，采取 KL 散度的形式

{% asset_img 5.png %}

- 在增广的数据上也进行约束

{% asset_img 6.png %}

由于教师网络的自监督模块只由 2 层 MLP 构成，并且训练过程 backbone 固定，所以很有可能会生成不正确的预测，即认为 $(x_k,\tilde{x}_i)_{k\neq i}$ 为 positive pair，为了避免这种情况，softmax 之后排序，若正确的在 top-k 个，则认为是 positive pair

<br>

## Experiments

**Ablation study**

感觉是数据增广的用处要比 SS 的大...

{% asset_img 7.png %}

**k值的影响**

既要保留一点的错误，也不能容忍全部错误

{% asset_img 8.png %}

**不同自监督任务对比**

{% asset_img 9.png %}

**cifar100上的结果**

可以看出不同架构的知识蒸馏提升很大

{% asset_img 10.png %}

{% asset_img 11.png %}

**further study**

在 few-shot 和 noisy label 的情况下，SSKD 也能很好地提取知识

{% asset_img 12.png %}