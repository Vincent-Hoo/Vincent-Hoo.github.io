---
title: Wasserstein GAN
mathjax: true
date: 2019-08-19 11:58:32
tags:
---

> 课程来源：李宏毅2018生成对抗网络课程
> 课程主页：http://speech.ee.ntu.tw/~tlkagk/courses_MLDS18.html
> 文章梗概：WGAN 作者首先详细证明了为什么原始 GAN 训练困难的问题，然后提出解决方案，用 Wasserstein 距离来衡量两个分布之间的距离。



<!-- more -->

<br>







## Original GAN review

2014 年，Ian Goodfellow 提出的生成对抗网络由生成器和判别器两个网络组成，两个网络相互对抗，从而最后实现生成器能够生成出真实照片的功能。对于生成器，它尽可能地去模仿真实分布，而判别器尽可能地区分开真实分布和生成分布。

判别器的目标函数为：

$$
V(G,D)=E_{x\sim P_{data}}[logD(x)]+E_{x\sim P_g}[log(1-D(x))]
$$

而生成器的目标函数有两种：

- 原始 GAN 论文（公式一）

$$
E_{x\sim P_g}[log(1-D(x))]
$$

- the - log D trick（公式二）

$$
E_{x\sim{P_g}}[-logD(x)]
$$


<br>
## GAN's problems

GAN 提出来之后一直面临着很多问题，如训练困难，生成器和判别器的 loss 无法指导训练过程，生成样本缺乏多样性等，WGAN 论文就上面提到的两个生成器的损失函数进行分析，来解释为什么 GAN 会存在这些问题。

### 公式一

**一句话概括：判别器训练得越好，生成器梯度消失越严重。**

在生成器固定参数的时候，最优的判别器是
$$
D^*(x)=\frac{P_{data}(x)}{P_{data}(x)+P_g(x)}
$$
判别器的评分等于一个样本 $x$ 来自真实分布和生成分布的可能性的相对比例，如果 $P_{data}(x) = 0$ 且 $P_g(x)\neq 0$，那就说明该样本来自生成器，这时判别器给出的评分就是0，反之如果 $P_{data}(x) \neq 0$ 且 $P_g(x)= 0$，那么判别器给出的评分就是1。

然而 GAN 训练有一个 trick，就是不能把判别器训练得太好，否则在实验中生成器是学不动的（loss 降不下去），为了探究背后的原因，我们可以看看在极端情况下（判别器最优），生成器的损失函数变成什么。

给公式一加上一项不依赖生成器的项，使之变成
$$
E_{x\sim P_{data}}[log(D(x))]+E_{x\sim P_g}[log(1-D(x))]
$$
当训练生成器的时候，判别器是固定的，假设判别器为最优判别器，那么生成器的目标函数即为：
$$
E_{x\sim P_{data}}log\frac{P_{data}(x)}{\frac{1}{2}[P_{data}(x)+P_g(x)]}+E_{x\sim P_g}\frac{P_g(x)}{\frac{1}{2}[P_{data}(x)+P_g(x)]}-2log2
$$
根据 JS 散度的定义，目标函数变成
$$
2JS(P_{data}|P_g)-2log2
$$
当这里，生成器的目标函数为两个分布之间的 JS 散度，训练生成器的目的是最小化该目标函数，即最小化两个分布之间的 JS 散度。判别器越接近最优判别器，生成器的目标函数就越能近似两个分布之间的 JS 散度。

问题就出在这个 JS 散度上，我们会希望如果两个分布之间越接近它们的 JS 散度越小，我们通过优化 JS 散度就能将 $P_g(x)$ 拉向 $P_{data}(x)$，最终以假乱真。这个希望在两个分布有所重叠的时候是成立的，但是如果两个分布完全没有重叠的部分，或者重叠的部分可忽略，那么 JS 散度就不能衡量他们之间的距离了。

根据 JS 散度和 KL 散度的定义
$$
JS(P_1|P_2)=\frac{1}{2}KL(P_1|\frac{P_1+P_2}{2})+\frac{1}{2}KL(P_2|\frac{P_1+P_2}{2})
$$

$$
KL(P_1|P_2)=E_{x\sim P_1}log\frac{P_1}{P_2}
$$

对于任意一个 $x$，存在四种情况

- $P_1(x)=0$，$P_2(x)= 0$：无 JS 散度。
- $P_1(x)\neq 0$，$P_2(x)\neq 0$：由于重叠部分可以忽略，因此对 JS 散度的贡献也为 0。
- $P_1(x)=0$，$P_2(x)\neq 0$：JS 散度为 $log2$。
- $P_1(x)\neq 0$，$P_2(x)= 0$：JS 散度同为 $log2$。

也就是说，无论 $P_{data}$ 和 $P_g$ 有多靠近，只要他们没有重叠，或者重叠部分可忽略，那么 JS 散度就是固定的常数 $log2$，那么对于生成器来说，目标函数的值应该不会变，这也就导致生成器接收不到任何的梯度信息。因此当判别器为最优判别器的时候，生成器接收不到梯度信息，即使判别器不是最优，生成器也极有可能面临梯度消失的问题。

而在绝大多数情况下，$P_{data}$ 和 $P_g$ 是不会有重合的，原因为

- 真实分布和生成分布都只是高维空间下的一个低维流体，就好比二维空间中的两条线，即使有重合，也只是某几个点，可以忽略。
- 现实中我们只能得到真实分布的采样分布，如果没有足够多的样本，即使两个分布有重合，我们也很难得到真实重合部分。

{% asset_img 1.png 500 500 %}

因此，原始 GAN 训练不稳定的原因就是：如果判别器训练得太好，生成器梯度消失，loss 降不下去；如果判别器训练得不好，那么生成器的梯度不准。只有判别器训练得不好不坏才行，这就无形中增加了 GAN 的训练难度。

### 公式二

**一句话概括：最小化第二种生成器 loss 函数，会等价于最小化一个不合理的距离衡量，导致两个问题，一是梯度不稳定，二是 collapse mode 即多样性不足。**

我们已知
$$
E_{x\sim P_{data}}[log(D^*(x))]+E_{x\sim P_g}[log(1-D^*(x))]=2JS(P_{data}|P_g)-2log2
$$
把 KL 散度变成含有 $D^*(x)$ 的形式
$$
KL(P_g|P_{data})=E_{x\sim P_g}[log\frac{P_g(x)}{P_{data}(x)}]=E_{x\sim P_g}[log\frac{P_g(x)/(P_{data}(x)+P_g(x))}{P_{data}(x)/(P_{data}(x)+P_g(x))}]
$$

$$
=E_{x\sim P_g}[log\frac{1-D^*(x)}{D^*(x)}]=E_{x\sim P_g}log[1-D^*(x)]-E_{x\sim P_g}logD^*(x)
$$

因此公式二的目标函数等价于
$$
E_{x\sim P_g}[-logD^*(x)]=KL(P_g|P_{data})-2JS(P_{data}|P_g)+2log2+E_{x\sim P_{data}}logD^*(x)
$$
注意上式最后两项不依赖于生成器 G，因此等价于最小化
$$
KL(P_g|P_{data})-2JS(P_{data}|P_g)
$$
这个等价最小化目标存在两个严重的问题。第一是它同时要最小化生成分布与真实分布的 KL 散度，却又要最大化两者的 JS 散度，一个要拉近，一个却要推远！这在直观上非常荒谬，在数值上则会导致梯度不稳定，这是后面那个 JS 散度项的毛病。

第二，即便是前面那个正常的 KL 散度项也有毛病。因为 KL 散度不是一个对称的衡量，$KL(P_g|P_{data})$ 与 $KL(P_{data}|P_g)$ 是有差别的，以前者为例

- 当 $P_g(x) \rightarrow 0$ 而 $P_{data}(x) \rightarrow 1$ 时，

$$
KL(P_g|P_{data})=E_{x\sim P_g}log\frac{P_g(x)}{P_{data}(x)}=\int P_g(x)log\frac{P_g(x)}{P_{data}(x)}dx \rightarrow 0
$$

- 当 $P_g(x) \rightarrow 1$ 而 $P_{data}(x) \rightarrow 0$ 时，

$$
KL(P_g|P_{data})\rightarrow+\infty
$$

换言之，$KL(P_g|P_{data})$ 对于上面两种错误的惩罚是不一样的，第一种错误对应的是生成分布还未能覆盖真实分布，惩罚微小；第二种错误对应的是“生成器生成了不真实的样本” ，惩罚巨大。第一种错误对应的是缺乏多样性，第二种错误对应的是缺乏准确性。**这一放一打之下，生成器宁可多生成一些重复但是很“安全”的样本，也不愿意去生成多样性的样本，因为那样一不小心就会产生第二种错误，得不偿失。这种现象就是大家常说的 collapse mode。**

*In Conclusion*

**在原始 GAN 的（近似）最优判别器下，第一种生成器 loss 面临梯度消失问题，第二种生成器 loss 面临优化目标荒谬、梯度不稳定、对多样性与准确性惩罚不平衡导致 mode collapse 这几个问题。**


<br>
## Wasserstein 距离的优越性

Wasserstein 距离又叫 Earth-Mover 距离，定义如下
$$
W(P_{data}, P_g)=inf_{\gamma\sim \Pi(P_{data},P_g)}E_{(x,y)\sim\gamma}||x-y||
$$

$\Pi(P_{data},P_g)$ 是 $P_{data}$ 和 $P_g$ 组合起来的所有可能的联合分布的集合，反过来说，$\Pi(P_{data},P_g)$ 中每一个分布的边缘分布都是 $P_{data}$ 和 $P_g$ 。对于每一个可能的联合分布 $\gamma$ 而言，可以从中采样 $(x,y)\sim\gamma$ 得到一个真实样本 $x$ 和生成样本 $y$，并算出这对样本之间的距离，所以可以计算该联合分布 $\gamma$ 下对距离的期望值。在所有可能的联合分布中能够对这个期望值去到的下界，就定义为 Wasserstein 距离。

直观上可以把这个期望值理解为在 $\gamma$ 这个“路径规划”下把 $P_g$ 这堆沙土挪到 $P_{data}$ 所需的平均消耗，而 Wasserstein 距离则是“最优路径规划”下的消耗，所有才叫 Earth-Mover（推土机）距离。

Wasserstein 距离相比 KL 散度和 JS 散度的优越性在于，即便两个分布没有重叠，也能反映他们的远近。

<br>
## 从 Wasserstein 距离到 WGAN

既然 Wasserstein 距离能够度量任意两个分布的距离，如果能够把它定义为生成器的 loss，不就可以产生有意义的梯度来更新生成器，使得生成分布被拉向真是分布了吗？

Wasserstein 距离定义中的下界无法直接求解，WGAN 作者将其变换成以下形式
$$
W(P_{data},P_g)=\frac{1}{K}sup_{||f||_L\le K}(E_{x\sim P_{data}}[f(x)]-E_{x\sim P_g}[f(x)])
$$
首先需要介绍 Lipschitz 连续，它是加在连续函数 $f$ 下的一个约束，要求存在一个常数 $K\ge0$ 使得定义域内的任意两个数 $x_1$ 和 $x_2$ 都满足
$$
|f(x_1)-f(x_2)|\le K|x_1-x_2|
$$
此时称函数 $f$ 的 Lipschitz 常数为 K。Lipschitz 约束了连续函数的变化幅度，要求函数不能太急，要平缓。

作者希望借助一个神经网络去拟合这样的一个函数 $f$，并且借助梯度上升的方法，取到该目标函数的最大值，至于 Lipschitz 约束，作者采取了一个简单的方法—— gradient clipping，限制神经网络的参数在某个范围。

到此为止，我们可以构造一个参数为 $w$ 、最后一层不是非线性激活层的判别器 $f_w$，在限制 $w$ 不超过某个范围的条件下，使得 $L=E_{x\sim P_{data}}[f_w(x)]-E_{x\sim P_g}[f_w(x)]$ 尽可能大，此时 $L$ 就会近似真实分布和生成分布的 Wasserstein 距离。

注意原始 GAN 的判别器是在做二分类问题，所以最后一层为 sigmoid，而 WGAN 的判别器做的是近似 Wasserstein 距离，所以最后一层不需要 sigmoid。

WGAN 的完整算法如下

{% asset_img 2.png 500 500 %}

对于判别器，目标函数如下，上面已经证明过，该目标函数的最大值即为两个分布之间的 Wasserstein 距离，通过梯度上升法更新判别器，使其能够近似 Wasserstein 距离。
$$
E_{x\sim P_{data}}D(x)-E_{x\sim P_g}D(x)
$$
对于生成器，目标函数如下，找到最优判别器后，下面的目标函数可以等价于 Wasserstein 距离，然后通过梯度下降法更新生成器，从而使得 Wasserstein 距离减少。
$$
-E_{x\sim P_g}D(x)
$$

对比原始 GAN，WGAN 只改了四点：

- 判别器最后一层去掉 sigmoid
- 生成器和判别器的 loss 不取 log
- 判别器更新采用 gradient clipping
- 不采用基于动量的优化算法，推荐 RMSProp，SGD


<br>
## Improved WGAN (WGAN-GP)

作者们发现 WGAN 有时候也会伴随样本质量低、难以收敛等问题。WGAN 为了保证 Lipschitz 限制，采用了 weight clipping 的方法，然而这样的方式可能过于简单粗暴了，因此他们认为这是上述问题的罪魁祸首。

他们提出的替代方案是给 Critic loss 加入 **gradient penalty (GP)**，这样，新的网络模型就叫 **WGAN-GP**。

作者认为，当且仅当一个可微函数的梯度范数（gradient norm）在任意处都不超过 1 时，该函数满足 1-Lipschitz 条件。

{% asset_img 3.png 500 500 %}

但是我们不可能让所有的 $x$ 的 gradient norm 都小于 1，因此作者将条件放缩为对于所有 $x\sim P_{penalty}$，而 $P_{penalty}$ 为生成样本和真实样本之间的线性插值。

{% asset_img 4.png 500 500 %}

而在 WGAN-GP 论文中，判别器的目标函数为

{% asset_img 5.png 500 500 %}

至于为什么限制 gradient norm 趋向 1（two-sided penalty）而不是小于 1（one-sided penalty），作者给出的解释是，从理论上最优的 gradient norm 应当处处接近 1，对 Lipschitz 条件的影响不大，同时从实验中发现 two-sided penalty 效果比 one-sided penalty 略好。

{% asset_img 6.png 500 500 %}
<br>
## References

- [Towards principled methods for training generative adversarial networks](https://arxiv.org/pdf/1701.04862.pdf)
- [Wasserstein GAN](https://arxiv.org/pdf/1701.07875.pdf)
- [Improved training of wasserstein gans](https://arxiv.org/pdf/1704.00028.pdf)



<br>