---
title: 【论文阅读】—— Image Super-Resolution Using Knowledge Distillation
mathjax: true
date: 2019-09-18 19:25:56
tags:
---

> Title: Image Super-Resolution Using Knowledge Distillation 
> Authors: Qinquan Gao, et al
> Conference: ACCV, 2018
> Abstract: 作者第一次将知识蒸馏用到超分任务上，并取得了提升。

<!--more-->



## Framework

### Teacher network

并没有采用比较通用的超分模型，而是自己设计了一个20层的网络，分为三个部分：feature extraction，Non-linear Mapping 以及 reconstruction，结构图如下

- 非线性映射部分采用 residual block 堆叠的方式，每个 block 为两个卷积层组成。
- 卷积核大小采用 $3 \times 3$，通道数固定为 64.

{% asset_img 1.png 500 500 %}





### Student Network

和 teacher 网络基本一样，层数也一样，但是为了减少学生网络的参数以及计算开销，每个 residual block 采用的是 depthwise 卷积。

{% asset_img 2.png 500 500 %}





### Knowledge Distillation

只采用了特征蒸馏，并没有采用软标签蒸馏，对于特征蒸馏，提出四种特征提取方式

- $(G_{mean})^pF=(\frac{1}{C}\sum_{i=1}^CF_i)^p$
- $(G^p)_{mean}F=\frac{1}{C}\sum_{i=1}^C(F_i^p)$
- $G_{max}F=max_{i=1,C}F_i$
- $G_{min}F=min_{i=1,C}F_i$

特征位置的选取上，由于教师和学生网络在 NonLinear Mapping 部分均为 10 层，因此作者提取 第 4、7、10 层的特征。模型整体的 Loss 如下，$Y$ 为 HR image。
$$
loss = loss_0(\tilde{Y},Y) + \lambda_1loss_1(s_1,t_1)+\lambda_2loss_2(s_2,t_2)+\lambda_3loss_3(s_3,t_3)
$$
<br>

## Experiment

### 特征选取位置的影响

实验结果如下，个人觉得这实验结果并没有说服力，因为随机性太强，作者认为最好的结果也没有特别明显，并且在过程中，改变了 $\lambda_0$，才取得最好的结果，而 $\lambda_0$ 和特征位置的选取并没有关系，理应固定；而当固定 $\lambda_0$ 为 1 的时候，可以看出结果并没有特别大的区别，(1, 0, 0) 和 (0, 1, 0) 均为一样的结果。

{% asset_img 3.png 500 500 %}





### 特征提取方式的影响

作者比较四种特征提取方式，以及不同的 $p$ 值，实验结果如下。$(G_{mean})^2$ 的结果最好，且蒸馏有效。

{% asset_img 4.png 500 500 %}>





### 蒸馏整体的有效性

{% asset_img 5.png 500 500 %}



<br>

*In Conclusion*

作者确实验证了蒸馏在超分上的有效性，但是网络选择方面没有选择通用的超分网络，而是自己搭建了一个网络，这样，自己搭建的网络是否适用于超分任务也很难说明，另外蒸馏方面只用到特征的蒸馏，而没有采用标签的蒸馏。

<br>