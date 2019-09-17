---
title: Similarity-Based Feature Distillation Overview
mathjax: true
date: 2019-09-10 11:08:14
tags:
---


该文章总结了 CVPR 和 ICCV 中提到的基于特征相似度蒸馏策略，共五篇文章。

- Structured Knowledge Distillation for Semantic Segmentation
- Similarity-Preserving Knowledge Distillation
- Correlation Congruence for Knowledge Distillation
- Knowledge Distillation via Instance Relationship Graph
- Relational Knowledge Distillation

<!--more-->

<br>

## Structured Knowledge Distillation for Semantic Segmentation



> Authors: Yifan Liu, Ke Chen, Chris Liu, et al.
> Conference: CVPR 2019

该论文主要针对的是图像分割领域的知识蒸馏问题，作者共提出了三个 loss

- Pixel-wise distillation：由于分割任务可以看成是一个在每个像素点上的分类任务，因此可以将原始 KD 的那套方法拿过来，假设模型输出的分割图大小为 $W^\prime \times H^\prime \times N$，$N$ 为类别数，因此 Loss 定义为

$$
{\cal L}_{pi}(S)=\frac{1}{W^\prime \times H^\prime}\sum_{i \in R}KL(q_i^s|q_i^t)
$$

- Pair-wise distillation：上面的 pixel-wise KD 可以看成是软标签上的蒸馏，而这部分的蒸馏是特征图上的蒸馏，但是作者并不是像 Hint Distillation 那样直接让学生网络 mimick 教师网络的特征图，而是将特征图在空间维度上提取了一个相似度矩阵。假设一个特征图的大小为 $W\times H \times C$，每个 pixel 可以看成是一个 $C$ 维的向量，$a_{ij}$ 表示为 第 $i$ 个像素点 $f_i$ 和第 $j$ 个像素点 $f_j$ 之间的 similarity，定义为

$$
a_{ij}=\frac{f_i^T \cdot f_j}{||f_i||_2 \cdot ||f_j||_2}
$$

​		因此相似度矩阵会是一个 $WH \times WH$ 的对称矩阵，pair-wise distillation 定义为
$$
{\cal L}_{pa}(S)=\frac{1}{(W^\prime \times H^\prime)^2}\sum_{i \in R}\sum_{j\in R}(a_{ij}^s-a_{ij}^t)^2
$$

- Holistic Distillation：作者将整个学生网络看成是一个 generator，模型生成的分割图看成是一个 fake sample，然后教师网络的输出看成是 true sample，一起输入到一个 discriminator 里面，希望 discriminator 能够判别出哪些是学生网络生成的，哪些是教师网络生成的，进而再固定判别器，调整生成器使得学生网络生成的分割图尽可能地像 GT，其中 GAN 运用的是 Wasserstein GAN，loss 函数如下

$$
{\cal L}_{ho}(S,D)=E_{Q^s\sim p_s(Q^s)}[D(Q^s|I)]-E_{Q^t\sim p_t(Q^t)}[D(Q^t|I)]
$$

整体的框架如下


{% asset_img 1.png 500 500 %}

实验结果如下，三个 Loss 一起用的话，实验结果最好。

{% asset_img 2.png %}

<br>

## Similarity-Preserving Knowledge Distillation



> Authors: Frederick Tung,  Greg Mori, et al.
> Conference: ICCV 2019

该论文做的是分类任务，上面的论文对于特征的提取方式，采取的是空间相似度的方式，而这篇论文关注的是样本之间的相似性，如下图，得到一个 $b\times b$ 的相似度矩阵。

{% asset_img 3.png %}

假设特征图经过 reshape 后为 $Q\in R^{b\times chw}$，样本相似度矩阵定义为
$$
\tilde{G}^{(l)}=Q^{(l)} \cdot Q^{(l)T};G^{(l)}_{[i,:]}=\frac{\tilde{G}_{[i,:]}^{(l)}}{||\tilde{G}_{[i,:]}^{(l)}||_2}
$$
特征蒸馏的 Loss 为
$$
L_{SP}(G_T,G_S)=\frac{1}{b^2}\sum||G_T-G_S||^2
$$

<br>

## Correlation Congruence for Knowledge Distillation



> Authors: Baoyun Peng,  Xiao Jin, et al.
> Conference: ICCV 2019

这篇论文和上面那篇几乎一样，同样是 ICCV 的文章，同样是做分类任务，同样是提取样本之间的相似性，唯一不同的是改了个名字叫做 correlation congruence。

另外不同的是，这篇文章还给出了几个相似度的度量方式

{% asset_img 4.png %}

<br>

## Knowledge Distillation via Instance Relationship Graph



> Authors: Yufan Liu,  Jiajiong Cao, et al.
> Conference: CVPR 2019

这篇论文其实跟上面的两篇的核心思想是一样的，都是样本之间的相似性，但是作者用 Graph 进行包装，提出的是 Instance Relationship Grpah(IRG)，图中节点为一个 sample，节点之间的边为 sample 之间的 relationship，如下图，但是这个 graph 可以看出来是一个全连接的 graph，其邻接矩阵不存在 0 项，因此和上面提到的样本相似度矩阵是一个东西。

{% asset_img 5.png %}

但与前面论文稍微不同的是，作者除了直接蒸馏这个邻接矩阵之外，还提出蒸馏其他信息。首先一个 IRG 可以看成是节点集合和边集合，节点为每个样本的特征图，边为两个样本特征图之间的距离，因此第 $l$ 层的 IRG 定义如下

{% asset_img 6.png %}

除了定义 IRG 之外，作者参考了 FSP 的思路，考虑了网络中不同层之间 IRG 之间的关系，想看一个 IRG 如何过渡到下一个 IRG，定义为 $IRG-t$，其包含节点的过渡以及边的过渡，“过渡”的定义为 Euclidean 距离。

{% asset_img 7.png %}

对于特征的蒸馏方式，作者提出了两种 mode，一种是一对一，学生网络和教师网络的 IRG 一对一，另一种是一对多，教师网络只拿最后一层去指导学生网络的后面几层。

{% asset_img 8.png %}

上面提到了两个图，一个是 IRG，另一个是 IRG-t，两个都是以 graph 的形式出现，有节点有边，对于 IRG，其 Loss 如下，其包含节点的 Loss 和边的 Loss。

{% asset_img 9.png %}

而作者最后采取的 Loss 如下，第一项为最后一层教师和学生网络之间 IRG 节点的 loss，最后一层 IRG 节点即网络的输出，作者把前面特征图中 IRG 的节点信息没有加入到 Loss 项里；而第二项采用的是一对多的 mode，教师网络只考虑最后一层的边信息。

{% asset_img 10.png %}

对于 IRG-t，同样存在节点信息和边信息，general 的 Loss 形式如下

{% asset_img 11.png %}

但是由于边信息太多，作者把第二项去掉了，只保留第一项

{% asset_img 12.png %}

总的 Loss 如下

{% asset_img 13.png %}

<br>

## Relational Knowledge Distillation



> Authors: Wonpyo Park,  Dongju Kim, et al.
> Conference: CVPR 2019

同样是分类任务中，挖掘样本之间的相关性

{% asset_img 14.png %}

对于相关性度量，作者提出两种方式

- Distance-wise distillation：就是简单的距离公式
{% asset_img 15.png 400 400 %}

- Angle-wise distillation：采用的三元组的方式，两个样本之间先求距离并作归一化，然后再求角度。

{% asset_img 16.png 400 400 %}