---
title: 【论文阅读】——Distilling Knowledge From a Deep Pose Regressor Network
mathjax: true
date: 2019-08-28 19:34:47
tags:
---


> Title：Distilling Knowledge From a Deep Pose Regressor Network
> Author：Muhamad Risqi U. Saputra, et al.
> Session：ICCV, 2019
> Abstract：作者发现知识蒸馏都基本应用在分类任务上，而回归任务的特殊性导致软标签不 work，因此作者提出自适应的蒸馏法来解决 visual odemetry 任务。



<!--more-->



*Motivation*

Knowledge Distillation(KD) 提出来后，绝大多数的研究都在分类任务上，KD 中提到的 dark knowledge 描述了分类任务中不同类别之间的相关性，因此即使教师网络误分类了某个样本，由于软标签的存在，也能给学生网络提供一定的指导信息。

而在回归任务上，神经网络输出的是连续的，假如教师网络对于某个样本的输出和真实标签发生了较大的区别，那么这个错误的样本对于学生网络就不存在任何的指导作用，不像分类任务的软标签那样。而考虑到回归任务的输出可能是一个无法预知的分布（分类任务的输出是封闭的，各个样本的概率，0-1之间的数值），因此错误所带来的代价可能会很大。

我们无法确保教师网络对于任何样本的输出都是合理的正确的，因此作者提出了一种自适应的知识蒸馏方法，针对不同的样本，学生网络需要判断是否需要接受教师网络的指导。


<br>
## Blending Teacher, Student and Imitation Loss

T 指教师网络，S 指学生网络，GT 指真实标签

- Teacher Loss: MSE(T, GT)
- Student Loss: MSE(S, GT)
- Imitation Loss: Loss(T, S)

作者提出了多种监督标签(supervised label)的方法：

- Minimum of student and imitation：对于每个样本，选择 student loss 和 imitation loss 中较小的

$$
L_{reg}=\frac{1}{n}\sum_{t=1}^nmin(||p_S-p_{gt}||^2, ||p_S-p_T||^2)
$$

- Imitation Loss as an additional loss：对于每个样本，既有 student loss，又有 imitation loss

$$
L_{reg}=\frac{1}{n}\sum_{t=1}^n \alpha||p_S-p_{gt}||^2+(1-\alpha)||p_S-p_T||^2
$$

- Teacher loss as an upper bound：NIPS2017一篇目标检测的 KD 文章提出，教师损失作为上界，即对于某个样本，教师损失比学生损失还要大的时候，就不允许教师网络指导学生网络，也比较合情合理。

$$
L_{reg}=\frac{1}{n}\sum_{t=1}^n \alpha||p_S-p_{gt}||^2+(1-\alpha)L_{imit}
$$

$$
L_{imit}=\left\{
\begin{aligned}
||p_S-p_T||^2 & & if \ ||p_S-p_{gt}||^2>||p_T-p_{gt}||^2\\
0 && otherwise\\
\end{aligned}
\right.
$$

- Probabilistic imitation loss：用教师网络和学生网络的条件概率作为 imitation loss，假设该概率服从拉普拉斯分布，也可以假设为高斯分布。

$$
P(p_S|p_T,\sigma)=\frac{1}{2\sigma}exp\frac{-||p_S-p_T||}{\sigma}
$$

$$
L_{imit}=-logP(p_S|p_T,\sigma)=\frac{||p_S-p_T||}{\sigma}+log\sigma+const
$$

- Attentive imitation loss(AIL)：作者提出的 AIL 自适应地选择是否信任教师网络，或者信任的程度是多少。该信任程度定义为 $\Phi_i$，用教师损失来定义，教师损失越大，信任程度越低。在整个数据集范围内，计算每个样本的教师损失，并且根据损失为权重确定信任程度。

{% asset_img 1.png 500 500 %}


<br>
## Learning Intermediate Representations

仿照 Hint Training(HT)，从教师网络的中间层抽取某一层的输出作为监督信息指导学生网络训练，同样采用自适应的方法，加入信任程度 $\Phi_i$，记为 attentive hint training(AHT)。 

{% asset_img 2.png 500 500 %}
<br>
## 训练细节

训练分为两个步骤：

1. 先用监督特征 $L_{hint}$ 进行训练网络的前面层
2. 固定监督特征 $L_{hint}$ 所训练的层，再用监督标签进行训练


<br>
## 实验结果

### 监督标签的有效性

对比提出的五种监督标签的损失函数，发现 AIL 的效果最好

{% asset_img 3.png 500 500 %}

### 监督特征的有效性

对比没有监督特征，加了监督特征后显著提升（提升得很多，有点夸张）。而用 AHT 的比 HT 的效果也有一定提升。

{% asset_img 4.png 500 500 %}

### 对比其它知识蒸馏方法
{% asset_img 5.png 500 500 %}


<br>