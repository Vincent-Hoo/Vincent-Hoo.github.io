---
title: 【论文阅读】—— Explaining and Harnessing Adversarial Examples
date: 2018-11-26 11:09:15
mathjax: true
tags:
---

> Title：Explaining and Harnessing Adversarial Examples
> link：https://arxiv.org/abs/1412.6572
> Abstract：该文章简单介绍对抗样本、对抗样本存在的原因，以及如何生成对抗样本进行对抗攻击——FGSM算法。

<!-- more -->


<br>
# What is adversarial example?

深度学习的发展使得机器在很多任务上都达到了接近人类的准确率，但是 Szegedy 等人在 "Intriguing properties of neural network" 中首次发现了这些深度网络的flaw并提出了对抗样本，对抗样本指的是将一些特定的扰动（perturbation）加到clean example上，使得模型对这些样本的分类结果发生错误，需要注意的是，这些扰动通常来说非常地小，即人类无法察觉，从下面的例子中就可以看出对抗样本是如何得到的。将一些扰动加到原图像上，分类器就会马上分类错误。

{% asset_img 1.png %}


<br>
# Why do adversarial examples exist?

- 过拟合导致？

  - 对抗样本刚被发现的时候，研究者都认为是模型参数过多，模型过度拟合训练集所致。
  - 如果对抗样本真的是过拟合导致，那么对抗样本的出现就不会是unique的，当我们重新训练模型，应该会随机出现其它的分类错误的“对抗样本”。但是实际上发现不同的模型会对同一个对抗样本错误分类（这种叫做对抗样本的transferability），并且如果把对抗样本和原来的样本作差，然后将差加到任何的一个clean example上，这个样本马上会变成对抗样本。
  - 所以，这些发现也表明了对抗样本不可能是过拟合所致。

- 模型的线性性导致！

  - Ian Goodfellow 在2014年发表了 "explaining and harnessing adversarial example" 这篇文章，并说明对抗样本的存在是由于模型过于线性所致。

  - 由于样本的输入特征的精度有限，一般的图像的每个像素都是8bits，所以样本中低于1/255的信息都会被丢弃，当样本 $x$ 添加一定的扰动值 $\eta$，$\eta$ 满足 $||\eta||_{\infty} < \epsilon$，即最大的扰动量小于 $\epsilon$。考虑一个线性模型，线性模型的特点在于 dot product，即 $score = w^T x$。由于 $\tilde{x} = x + \eta$，则 $score = w^Tx + w^T\eta$，可以看出当样本改变 $\eta$ 时，activation值就会改变 $w^T\eta$。

  - Ian Goodfellow 认为令 $\eta = sign(w)$，可以使得 $w^T\eta$ 最大化，假设 $w$ 有 $n$ 个维度，且平均值为 $m$，那么activation的增量最多为 $\epsilon mn$。由于 $||\eta||_{\infty}$ 不会随着维度 $n$ 的变化而变化，但是增量却会随着 $n$ 增大而线性增长，因此对于一个高维度的问题，一个样本中即使是无限小的扰动，也会叠加对最后的activation产生很大的影响

  - 神经网络似乎自诞生以来就和非线性挂钩，激活函数非线性，那么为什么深度网络还是会有对抗样本呢？Ian Goodfellow 认为现在绝大多数的激活函数都是 piecewise linear，Relu函数是分段线性的，sigmoid 和 tanh在我们care的部分也近似线性。

  - 下图表明线性模型导致对抗样本的出现，在clean example的领域内找到某一个点，它跨过了决策面被分类错误，但如果是非线性的决策面，这种情况就不会发生。

    {% asset_img 2.png %}

  - 而也有猜想说，对抗样本的存在是因为决策面距离样本点太近所致。


<br>
# Fast Gradient Sign Method(FGSM)

Ian Goodfellow 根据模型的线性性，提出一种产生对抗样本的方法，通过计算模型对输入的梯度的符号作为输入的扰动值。

$$\eta = \epsilon sign(\bigtriangledown _xJ(\theta, x, y))$$ 

$$x = x + \eta$$

我们平常用的梯度下降方法是通过求损失函数对于模型参数的梯度，进而更新参数使得模型的损失降低。而 FGSM 方法求模型损失函数对于输入的梯度，然后用**梯度上升**的方法，使得模型的损失增大。模型的损失增大意味着模型对于该输入样本的分类置信度降低，或者直接分类错误，从而达到对抗样本的目的。

Note：我们可以随意地将输入样本往任一错误类别上改变。比如输入图片是猫，我们可以求损失函数对于狗的梯度，然后将它作为 reference direction来更新猫的图片，就可以将猫往狗的方向改变。


<br>
# Some Intuitions

**Are adversarial examples rare?**

Ian Goodfellow认为对抗样本的存在并非偶然，并非像有理数分布到实数上的这种程度，而是对抗样本更像是存在于某一个对抗子空间

**Are adversarial examples off the manifold**

manifold 简单解释即数据存在的区域，通常我们说 manifold是高维空间里面的一个低维空间，比如公路是三维空间里面的一维空间；

{% asset_img 3.png 500 500 %}

而对抗样本不属于数据的manifold，可以理解为非自然点，即不在正常数据的分布内。有人理解对抗样本的存在是因为训练数据不够大没有覆盖到对抗样本，才导致模型欠拟合出现分类错误，但实际上对抗样本不属于正常数据，即使我们的训练数据有无穷多，也不可能包含对抗样本，因为对抗样本不在正常数据的分布内。


<br>
# 对抗训练

这是基于 Ian Goodfellow 论文中提到的方法

## 线性模型的对抗训练

考虑最简单的逻辑回归模型，假设训练一个逻辑回归模型来进行分类，分类的标签 $y \in \{ -1, +1\}$，预测函数 $P(y = 1) = \sigma(w^T x + b)$，损失函数为 $J(x,y) = \zeta(-y(w^Tx + b))$，其中 $\zeta(x) = log(1 + exp(z))$ 为softplus函数。

其实这个损失函数就是我们一般的cross-entropy loss，经过一系列变形成上面的形式。

$$Likelihood(x,y) = \sigma(\hat{y})^{\frac{y+1}{2}}(1-\sigma(\hat{y}))^{1-\frac{y+1}{2}}$$

其中 $\hat{y} = w^Tx + b$  

$$Loss(x,y) = -log\ Likelihood(x,y) = \begin{cases} log(1 + exp(-\hat{y})) & y = 1 \\ log(1 + exp(\hat{y})) & y = -1 \end{cases}$$
<br>
$$\zeta(-y\hat{y}) = \begin{cases} log(1 + exp(-\hat{y})) & y = 1 \\ log(1 + exp(\hat{y})) & y = -1 \end{cases} $$ 
<br>
所以，$Loss(x,y) = \zeta(-y(w^Tx + b))$ 

对该模型使用FGSM方法，求得扰动量为

$$\eta = \epsilon sign(\bigtriangledown _x \zeta(y(w^Tx + b))) = \epsilon sign(-w) = -\epsilon sign(w)$$  

因此对抗样本的Loss为

$$\zeta(-y(w^T(x + \eta) + b)) = \zeta(y(\epsilon ||w||_1 - w^Tx -b))$$ 

其中 $w^T \eta = -\epsilon ||w||_1$

对于对抗样本的Loss，加入了模型的权重，就好比L1正则，区别在于L1正则权重是加到整体损失函数之后，而上面的式子权重是加到activation。

当模型能够做出置信度很高的预测的时候，即 $w^Tx + b$ 很大的时候，上式的正则效果会消失；而当预测的置信度很小的时候，会使得欠拟合更加严重，因为上式的主导项变成了权重项。



## 深度网络的对抗训练

Ian Goodfellow同时提出了一种基于深度网络的对抗训练方法，同样是基于FGSM算法

$$\tilde{J}(\theta, x, y) = \alpha J(\theta, x, y) + (1-\alpha)J(\theta, x + \epsilon sign(\bigtriangledown_xJ(\theta,x,y)))$$ 

这种对抗训练的方法意味着在训练过程中不断更新对抗样本，从而使得当前模型可以抵御对抗样本。 

文章表明，在不进行对抗训练的情况下，模型识别FGSM攻击方法生成样本的错误率是89.4%，但是通过对抗训练，同样的模型识别对抗样本的错误率下降到17.9%。 


<br>
# References

- [Know Your Adversary: Understanding Adversarial Examples](https://towardsdatascience.com/know-your-adversary-understanding-adversarial-examples-part-1-2-63af4c2f5830)
- [The Modeler Strikes Back: Defense Strategies Against Adversarial Attacks ](https://towardsdatascience.com/the-modeler-strikes-back-defense-strategies-against-adversarial-attacks-9aae07b93d00)
- 非常好的一片文章：[Breaking Linear Classifiers on ImageNet](http://karpathy.github.io/2015/03/30/breaking-convnets/)
- [CS231, Lecture 16 | Adversarial Examples and Adversarial Training](https://www.youtube.com/watch?v=CIfsB_EYsVI)
- [某人的论文笔记](https://zhuanlan.zhihu.com/p/32784766)
- [某博文](https://blog.csdn.net/cdpac/article/details/53170940)



<br>