---
title: 【论文阅读】—— Data-Free Learning of Student Networks
mathjax: true
date: 2019-10-30 09:47:36
tags:
---

> Title：Data-Free Learning of Student Networks
> Authors：Hanting Chen, Yunhe Wang et.al
> Conference：ICCV 2019
> Abstract：作者提出了一种无需原始数据的蒸馏方式，通过用对抗生成网络（GAN）去生成数据集，能够达到原数据的蒸馏效果

<!--more-->

*preface*

传统的知识蒸馏框架是基于以下假设：用来训练教师网络的数据集，如 ImageNet、Cifar，同时用来做知识蒸馏时的数据集，但是在现实场景下，用来训练教师网络的数据可能会处于隐私保护的原因，不会公开出来，而公开的仅仅只有教师网络这一个模型，相当于是一个接口，这时候我们没有了训练教师网络的数据集，应该如何进行知识蒸馏来训练学生网络呢？

解决无数据下的知识蒸馏的基本思路是：构造合成数据集，使得合成的数据集能够模拟原数据集的数据分布。Data-Free 下的知识蒸馏的第一篇论文是 NIPS 2017 workshop 的一篇文章，[ Data-free knowledge distillation for deep neural networks]( https://arxiv.org/pdf/1710.07535.pdf )，这篇文章的主要思路是通过一些 meta-data 来重构数据集，meta-data 为训练教师网络时收集的一些信息，如网络输出层或中间层的激活值的均值或方差等，然后用这些 meta-data 来指导生成图片，使得合成的图片的激活值方差尽可能地接近 meta-data。

{% asset_img 1.png %}

但这种方法还是需要 meta-data 作为支撑，而 ICCV 2019 的这篇文章通过 GAN 直接模拟合成数据集，且合成的数据集蒸馏效果更好。

<br>

## GAN Loss Function

作者提出的整体架构如下，作者把教师网络作为判别器去指导生成器的训练，训练好的生成器生成图片，然后分别通过教师和学生网络，然后进行知识蒸馏。

{% asset_img 2.png %}

但是传统 GAN 中的判别器时一个二分类的判别器，用来判断生成的图片的真假，而教师网络是多分类网络，用来判断输入图像属于哪一个类别，为了能够使得生成器生成的数据能够尽可能地像原数据集，作者提出了 3 个损失来约束生成器。

### One-hot Loss

先定义一些符号：

- 生成器的输入是一些随机的向量：$\{z^1,z^2,...,z^n \}$
- 生成器的输出是一些生成的图片：$\{x^1,x^2,...,x^n \}$，$x^i=G(z^i)$
- 将生成的图片输入到判别器（教师网络）得到输出：$\{y^1_T,y^2_T,...,y^n_T \}$，$y_T^i=D(x^i)$
- 判别器（教师网络）预测的类别：$\{t^1,t^2,...,t^n \}$，$t^i=argmax_j(y^i_T)_j$

要使生成的图片尽可能像原数据集，首先要使得生成图片通过教师网络后的结果 $y_T^i$ 尽可能地像一个 one-hot 向量，即能够预测出某一个类别出来，因此我们把 $t_i$ 看作是一个伪标签来约束生成器。
$$
L_{oh}=\frac{1}{n}\sum_i H_{cross\_entropy}(y_T^i, t^i)
$$

### Information Entropy Loss

除了从单张图片去考虑生成器的生成质量，还要从整体上去考虑，要使生成器生成的图片尽可能地像训练集原始的图片，其次还要做到生成的图片每个类别是均衡的。这里作者用了信息熵来表示各种类别的均衡性，当每个类别生成的概率都一样时，信息熵达到最大值，因此通过最大化信息熵来训练网络，$H_{info}(p)=-\frac{1}{k}\sum_i p_ilog(p_i)$
$$
L_{ie}=-H_{info}(\frac{1}{n}\sum_i y^i_T)
$$
每个类别的概率为教师网络输出层在不同样本下的平均

### Activation Loss

第三个损失函数是考虑到图片的真实性，一般真实的图片对应的网络激活值都比较大，所以把网络激活值同时加到损失函数里面进行约束。这里用到的特征层为全连接层前的特征图 $f_T^i$
$$
L_a=-\frac{1}{n}\sum_i||f_T^i||_1
$$
<br>

整体的 Loss 为三个 loss 的加权
$$
L_{Total}=L_{oh}+\alpha L_a + \beta L_{ie}
$$


伪代码如下，训练分为两个阶段，先训练生成器，使得生成器合成的图片尽可能接近真实数据集分布，第二个阶段为用训练好的生成器生成图片来进行知识蒸馏。

{% asset_img 3.png %}

<br>



## Experiment

### Ablation study

先验证三个 loss 对应的作用，实验设置为：教师网络为 LeNet-5，学生网络为 LeNet-5-HALF，数据集为 MINST。下表结果为用不同损失函数训练出来的生成器进行知识蒸馏后学生网络的性能，从结果可以看出，在 MINST 这种简单的数据集，即使不训练生成器，随机的图片都有 88 的精度，而当不同 loss 被考虑的时候，可以看出信息熵损失是最有用的，当没有信息熵损失的时候，结果都很差。另外两个损失作为信息熵损失的补充，进一步提升了生成图片的质量

{% asset_img 4.png %}



### Comparison with other methods

#### Results in MINST

- Normal distribution：不训练生成器，直接用正态分布的随机数据去进行知识蒸馏训练学生网络，然后再用MINST 测试。
- Alternative data：用一个类似 MINST 的数据集 USPS，也是一个手写数字的数据集，用这个数据集去进行知识蒸馏，再用 MINST 测试，可以看出即使是相近的数据集，他们的数据分布也会有一些不一样，所以当用 MINST 测试的时候性能会下降。
- Meta-data：上面提到的 NIPS 2017 workshop 上的方法，用 meta-data 生成数据集，再用合成的数据集来进行知识蒸馏，再用 MINST 测试。
- DAFL：用 GAN 的方式生成数据集，用合成的数据进行知识蒸馏，用用 MINST 测试。

可以看出用 GAN 生成的数据集是最像原数据集的，因为用合成的图片训练的知识蒸馏比用原 MINST 数据知识蒸馏，性能并没有降太多。

{% asset_img 5.png %}



#### Results in Cifar

{% asset_img 6.png %}



### Visualization

从图片可以看出，生成图片的轮廓还是有点像原始图片的。

{% asset_img 7.png %}

用生成图片训练的学生网络的 filter 和用原数据训练的教师网络的 filter 接近。

{% asset_img 8.png %}

<br>