---
title: GAN theory
date: 2018-11-21 18:56:23
mathjax: true
tags:
---

> 课程来源：李宏毅2018生成对抗网络课程
> 课程主页：http://speech.ee.ntu.edu.tw/~tlkagk/courses_MLDS18.html
> 论文：{% asset_link "Generative Adversarial Nets.pdf" %}
> 文章梗概：该文章讲述 Ian Goodfellow 在 2014 年提出 GAN 时候的理论基础。


<!-- more -->

<br>

*preface*

GAN所做的事情就是生成，学习真实数据的分布，所谓数据的分布，可以看下面这张图，假如数据是二次元图像，图像的维度是 $10 \times 10$，因此整个Image Space就是 $10 \times 10$ 的空间，$x$ 就是这个空间中的一个样本，即一张图片，显然这个空间囊括了所有 $10 \times 10$ 的图片，而假如真正的二次元图像在这个空间里面服从某一个分布 $P_{data}(x)$，我们可以暂且把Image Space看成是离散的点所构成，因此 $P_{data}(x)$ 就是空间中某个点 $x$ 属于二次元图像的概率，下图中区域内的点属于二次元图像的概率较高，而区域外的点概率较低。

{% asset_img 1.png 500 500 %}


<br>
## maximum likelihood estimation

在GAN之前，是通过最大似然的方法是估计真实数据的分布，从而去做一个generator的，假设我们用一个类型已知的分布 $P_G(x; \theta)$ 去逼近真实的分布 $P_{data}(x)$，这个已知的分布可能是高斯分布，则 $\theta$ 就代表均值和方差，根据极大似然估计的方法，流程如下

1. 从真实的数据分布 $P_{data}(x)$ 中，抽样 $m$ 个样本 {$x^1, x^2, ..., x^m$} 
2. 计算每个样本点在 $P_G(x;\theta)$ 分布下的概率 $P_G(x^i; \theta)$
3. 似然函数 $L = \prod_{i=1}^m P_G(x^i; \theta)$，代表的就是 $P_G(x;\theta)$ 生成该 $m$ 个样本点的概率
4. 找出 $\theta^*$ 使得该概率最大化

两个概率分布的 KL-divergence 的公式如下，当两个概率分布越接近，KL-divergence 越小，当两个分布完全相同时，divergence 为 0.

$$D_{KL}(P || Q) = \int p(x)log(\frac{p(x)}{q(x)})$$ 

而实际上最大化概率可以转换为最小化两个概率分布的 KL-divergence，推导如下

$$\theta^* = {arg\max} \prod_{i=1}^m P_G(x^i;\theta) = arg\max \sum logP_G(x^i;\theta)$$

上式近似等于

$$\approx arg \max E_{x \sim P_{data}} [logP_G(x; \theta)] = arg\max \int_x P_{data}(x)logP_G(x;\theta)dx $$   

由于待求的参数是 $\theta$，加上一项与 $\theta$ 无关的项，对结果没有影响

$$= arg\max (\int_x P_{data}(x)logP_G(x;\theta)dx - \int_x P_{data}(x)logP_{data}(x)dx)$$ 

$$=argmax(\int_xP_{data(x)}log(\frac{P_G(x;\theta)}{P_{data}(x)})) = arg \min D_{KL}(P_{data}||P_G)$$      

因此，最大似然的方法去求解生成数据的概率分布，实际上就是在最小化两个分布的 KL-divergence。

<br>
## generator objective function


上面的推导是基于 $P_G(x;\theta)$ 分布类型已知的情况，如高斯分布，用高斯分布去拟合高维的数据，有时候是很难实现的，无论怎样调节均值和方差。因此我们需要一个更加普通的概率分布，但是这样我们就无法计算概率值，因此上面的方法就变得不适用了。


GAN就是用一个generator就实现一个普遍的概率分布 $P_G(x)$，generator的作用就是将一个普通的概率分布，如正态分布，转变成一个高维空间的概率分布。而generator的目标是使得生成图片的概率分布 $P_G(x)$ 和 $P_{data}(x)$ 尽可能地接近，即两个概率分布的divergence尽可能小。

{% asset_img 2.JPG 500 500 %}

我们不知道 $P_G(x)$ 和 $P_{data}(x)$ 的概率分布，因而也无法计算它们之间的divergence，无法求divergence的最小值。虽然概率分布和分布类型是未知的，但是我们可以进行抽样，样本的概率分布与总体的概率分布是一致的，因此我们需要从两个概率分布的样本值去计算这两个概率分布的divergence。**GAN神奇的地方在于可以透过discriminator计算两个概率分布的divergence**，

<br>
## discriminator 与 divergence的联系

discriminator的objective function如下，G是固定的，discriminator的目标是最大化objective function，即求解 $D^* = arg \ max \ V(G,D)$ 

$$V(G,D) = E_{x\sim P_{data}}[logD(x)] + E_{x \sim P_G}[log(1-D(x))]$$

discriminator实际上就是一个二分类器，最大化上面的objective function实际上就是最小化cross entropy，而当我们把discriminator训练好的时候，**目标函数 $V(G,D)$ 的最大值就是我们想要的两个概率分布的divergence**，一个直观的感受如下，当两个概率分布的divergence较小的时候，discriminator比较难区分它们，因此objective function value也比较难拉高；而当两个概率分布差别很大时，objective function value就会容易取到比较大的值。即：$max(V(D,G))$ 越小，divergence越小。

{% asset_img 3.png 500 500 %}

<br>

**数学证明 $max(V(G,D))$ 和 divergence的关系如下**

$V(G,D) = E_{x\sim P_{data}}[logD(x)] + E_{x \sim P_G}[log(1-D(x))]$

​                 $$ = \int_x P_{data}(x)logD(x)dx + \int_x P_G(x)log(1-D(x))dx$$

​                 $$=\int_x [P_{data}(x)logD(x) + P_G(x)log(1-D(x))]dx$$ 

这里我们假设 $D(x)$ 可以是任意的函数，即我们假设discriminator的网络可以有无穷多的神经元，给定任意的x，可以输出任意的值。如果这样的话，我们可以把积分看成是求和，然后把被积部分的每个x单独拿出来，求出取得最大值时候的 $D(x)$ 的值，所以 $D(x)$ 就是我们构造出来的一个函数，如下

$$D^*(x) = arg\ max\ P_{data}(x)logD(x) + P_G(x)log(1-D(x))$$

接下来就是求出这个 $D^*(x)$

{% asset_img 4.png 500 500 %}

求到 $D^*(x)$ 之后，我们可以算出 $max(V(G,D))$ 

$max(V(G,D)) = V(G, D^*) = E_{x\sim P_{data}}[log\ \frac{P_{data}(x)}{P_{data}(x) + P_G(x)}] + E_{x\sim P_G}[log\ \frac{P_{G}(x)}{P_{data}(x) + P_G(x)}]$

$=\int_x P_{data}(x)log[ \frac{P_{data}(x)}{P_{data}(x) + P_G(x)}]dx + \int_x P_G(x) log[\frac{P_{G}(x)}{P_{data}(x) + P_G(x)}]dx$   

进行一些简单的变换，分子分母同除2

$=\int_x P_{data}(x)log[ \frac{0.5P_{data}(x)}{0.5（P_{data}(x) + P_G(x)）}]dx + \int_x P_G(x) log[\frac{0.5P_{G}(x)}{0.5(P_{data}(x) + P_G(x))}]dx$   

$=\int_xP_{data}(x)log(0.5)dx + \int_xP_G(x)log(0.5)dx + \int_x P_{data}(x)log[ \frac{P_{data}(x)}{0.5（P_{data}(x) + P_G(x)）}]dx + \int_x P_G(x) log[\frac{P_{G}(x)}{0.5(P_{data}(x) + P_G(x))}]dx$

$=2log(0.5) + D_{KL}(P_{data} || \frac{P_{data} + P_G}{2}) + D_{KL}(P_{G} || \frac{P_{data} + P_G}{2})$

$=2log(0.5) + 2D_{JSD}(P_{data} || P_G)$ 

可以看出，$max(V(G,D))$ 实际上就是两个概率分布的JS-divergence和一个常数的和，两个分布divergence越小，$max(V(G,D))$ 越小，因为两种数据分布接近，难以区分，所以 $V(G,D)$ 能取到的最大值也越小。


<br>
## 回到最原始的objective function

最原始的objective function是想找到一个generator，使得generator生成图片的概率分布 $P_G$ 和 真实数据 $P_{data}$ 的divergence最小，即 $G^* = arg \ min \ Div(P_G, P_{data})$ 

然后我们通过上面的证明可以知道，$max(V(G,D))$ 就是两个概率分布的divergence，所以原始的目标函数可以变为 $G^* = arg \ min_G \ max_D V(G,D)$ 

下面这张图描述了如何解决 $minmax$ 这种优化问题，我们假设就只有三个 $G$，固定一个 $G$，可以得到选择不同 $D$ 下的 $V(G,D)$，然后比较不同 $G$ 下的 $max \  V(G,D)$，选择最小的，即 $G_3$。可以看出 $V(G,D)$ 的最大值就是 $P_G$ 和 $P_{data}$ 之间的divergence

{% asset_img 5.png 500 500 %}


<br>
## Algorithm for minmax problem

objective function：$G^* = arg \ min_G \ max_D V(G,D)$

令 $L(G) = max_D \ V(G,D)$ 

因此问题变成找 $L(G)$ 的最小值，自然可以用梯度下降去解决 $\theta_G  = \theta_G - \alpha \frac{\partial L(G)}{\partial \theta_G}$ 

$L(G) = max_D V(G,D) = max(L_1(G), L_2(G), L_3(G), ...)$ 

$L(G)$ 函数的意思是，给定一个 $G$，我就搜索一个 $D$ 使得 $V(G,D)$ 最大，我们可以将它看成是无数个函数 $L_i(G)$ 求最大值，一个 $L_i(G)$ 对应一个 $D$，好比下面的图中，有三条曲线，相当于只有 3 个 $D$，而实际上应该是有无穷多个 $D$，当给定一个 $G_0$ 的时候，就会从众多条曲线 $L_i(G)$ 中，选择一条使得 $L(G_0)$ 取最大值的曲线，然后把这条曲线记作是 $L^*(G)$，就可以求梯度了 $\frac{\partial L^{\ast}(G)}{\partial \theta_G}$

{% asset_img 6.png 500 500 %}


上面提到从众多条曲线 $L_i(G)$ 中，选择一条使得 $L(G_0)$ 取最大值的曲线 $L^*(G)$，实际上 $L^{\ast}(G) = V(G, D^{\ast})$，$D^{\ast} = arg \ max \ V(G_0, D)$，所以解决这个minmax problem的算法如下

1.  给定一个初始的 $G_0$
2. 找到 $D^*$ 使得 $V(G_0, D)$ 取最大值，即找出那条曲线 $L^{\ast}(G)$
3. 得到 $G_1 = G_0 - \frac{\partial V(G, D^{\ast})}{\partial G}$
4. 重复2、3步

下面是实际上的algorithm，我们用均值来代替期望，一点需要注意的是discriminator需要重复多次，尽可能找到更好的D，能够使得 $V(G,D)$ 较大，实际上我们只能找到 $V(G,D)$ 的下界，因为理论上我们假设的D可以是任意的函数，但是实际上D的神经元并不可能无穷多。所以我们只能尽可能地找到一个使得 $V(G,D)$ 大一点的D，来衡量 $P_{data}$ 和 $P_G$ 的divergence。

固定D，我们需要求解 $V(G,D)$ 的最小值，来降低divergence，由于 $V(G,D)$ 的第一项和G没有关系，所以可以去掉。注意的是G一次只需要更新一次。

{% asset_img 7.png 500 500 %}

这里解释一下为什么只能更新一次G，我们在更新G的参数的时候，是固定D的，这个D是我们找到的 $D^{\ast}$，它能够使得 $V(G_0, D^{\ast})$ 取到最大值，也意味着这个 $D^{\ast}$ 是用来衡量 $P_{G_0}$ 和 $P_{data}$ 之间的divergence，然后我们用梯度下降，想减小这个divergence，更新 $G_0$ 成 $G_1$，这时候我们的D还是固定不变的，那么 $V(G_1, D^{\ast})$ 是否同样取到最大值，是否能够表示 $P_{G_1}$ 和 $P_{data}$ 的divergence呢？答案是不肯定的，假如下面的情况出现，更新后的 $G_1$ 所对应的 $V(G_1,D)$ 曲线发生了较大的变化，导致取到最大值的点并不是原来的 $D_0^{\ast}$，而是 $D_1^{\ast}$，这时候如果再用原来的 $D_0^{\ast}$ 去计算 $V(G,D)$ 就不能表达两个数据之间的divergence了。所以G不能更新太多，或者一次不能更新太多，使得下面的情况出现，我们只能更新一点点，并且假设 $D_1^{\ast} \approx D_0^{\ast}$，两条曲线没有太大变化，这样才能保证算法的正确性。

{% asset_img 9.png 500 500 %}


<br>
## minmax GAN vs non-saturating GAN

在更新generator的时候，minmax GAN是更新一个和discriminator相同的objective function，这会导致gradient vanishing的问题，Ian Goodfellow在2016年的GAN tutorial里面提到

> In the minimax game, the discriminator minimizes a cross-entropy, but the generator maximizes the same cross-entropy. This is unfortunate for the generator, because when the discriminator successfully rejects generator samples with high confidence, the generator’s gradient vanishes. 



而non-saturating GAN则是把generator的objective function改为

$$J^G = -\frac{1}{2}E[logD(G(z))]$$ 

non-saturating GAN另外一个好处是，在训练的一开始，$D(G(z))$ 一般都很小，minmax GAN的话，梯度很小，更新很慢；而non-saturating GAN一开始的梯度都比较大，因此收敛快一点。

{% asset_img 10.png 200 500 %}



<br>

*In Conclusion*

这是Ian Goodfellow在2014年提出GAN的时候的理论，但是理论总是在发展，discriminator是否真的在描述两个概率分布的divergence也不好说，最后discriminator是否真的会坏掉，还得继续考察。

<br>