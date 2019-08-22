---
title: 【论文阅读】—— Convolutional Networks with Adaptive Computation Graphs
mathjax: true
date: 2018-12-16 17:14:12
tags:
---

> Title：Convolutional Networks with Adaptive Computation Graphs
> Authors：Andreas Veit, Serge Belongie
> Abstract：作者提出了 Adanet，是一种自适应的计算图，在 ResNet 的基础上，加入了门结构来决定是否需要执行某个 block，引入这种门结构并没有带来额外太多的计算量，反而能加快训练过程。

<!--more-->

<br>

*preface*

深度学习领域在网络深度和网络结构上都进行了很多的研究，尽管这些网络在结构细节上会有不同，但是他们有一点是相同的：他们都是一个固定的模型，网络结构与输入图像没有任何关系。

《Residual networks behave like ensembles of relatively shallow networks》论文中提到去掉  ResNet 的某几层并不会影响性能，这也表明没有任何一个独立的层是对性能起到关键性作用的，网络层存在着冗余，很多计算量是没有必要的。因此作者就提出了网络结构并不需要固定不变，对于不同的输入，可以组装不同的网络去进行预测。

作者提出的 AdaNet 是基于 ResNet 进行的改进，在每一个 residual block 中加入一个 gating function 来决定是否需要执行这个 block 的操作。

{% asset_img 1.png %}

作者的这个工作和 regularization with stochastic noise 有关，stochastic path 随机的丢弃 ResNet 的某些层，而 AdaNet 根据输入来决定跳过哪些层。

AdaNet 也可以看成是 attention 机制的一个例子，选择重要的层去执行；而 SENet 是根据卷积输出中 channel 的重要性进行 rescale。

<br>

# AdaNet

ResNet 可以定义如下，它的提出使得上千层的网络得以诞生，但是研究也表明，尽管这些层是一起训练的，去掉任何一层也不会影响最后的性能。

$$x_l = x_{l-1} + F_l(x_{l-1})$$

## Adaptive Computation Graph

作者受到《Residual networks behave like ensembles of relatively shallow networks》这篇论文结果的启发，思考卷积网络是否需要一个固定的结构，由于固定的结构会带来很多冗余的计算，因此作者想自适应地选择或组装 (assemble) 网络，自适应的网络会根据输入来决定通过哪些必要的层，而跳过不必要的层，因此提出了门结构的 residual block。$z(x_{l-1})$ 是 gating function，根据每一层的输入来决定是否执行。

$$x_l = x_{l-1} + z(x_{l-1}) \cdot F_l(x_{l-1}), z(x_{l-1}) \in \{0,1 \}$$

作者把上面的方式和 highway network 进行了对比，highway network 中每一层的输出为：$x_l = (1-z(x_{l-1})) \cdot x_{l-1} + z(x_{l-1}) \cdot F_l(x_{l-1})$，两个式子有点类似，但是 highway network 的式子更像是一种 soft 注意力机制，关注某些层和不关注其他层。而 AdaNet 是直接跳过不执行某一层。



## Gating Unit

$z_(x_{l-1})$ 分为两个阶段

1. 根据输入，评估需要执行 residual block 的概率。
2. 根据概率值，抽样出一个离散的样本。

{% asset_img 2.png %}
考虑到 AdaNet 的主要目的是为了减少不必要的计算量，因此引入的门结构必须比较轻量，不然整体的计算量无法减少。首先作者采用 global avg-pooling 进行 channel 级别的特征提取。

$$z_c = \frac{1}{H\times W}\sum \sum x_{i,j,c}$$

通过 global pooling 得到一个 $1 \times 1 \times C$ 的 channel descriptor，然后再用两层全连接网络去捕获 channel 之间的关系。

$$\beta = W_2\sigma(W_1z)$$

$W_1 \in R^{\frac{C}{d}\times C}$，$W_2 \in R^{2\times \frac{C}{d}}$，$d$ 为 reduction ratio。输出的 $\beta$ 是一个两个元素的一维向量，表征着执行和跳过该 block 的概率。

假设 $X$ 是一个随机变量，$X = 0$ 表示跳过该 block，$X = 1$ 表示执行该 block，那么 $P(X = 0) = \beta_0$，$P(X = 1) = \beta_1$，我们现在想得到一个服从该离散变量分布的一个值 $x$，即 0 或 1，一个最简单的做法是根据概率去采样，但是采样出来的只有 $x$ 的值，并没有生成 $x$ 的式子，即采样这个操作不可微，本来 $x$ 的值是和 $\beta_0$、$\beta_1$ 有关的，但是采样这个操作，我们得到的 $x$ 无法对 $\beta$ 进行求导，这就使得神经网络里面的梯度无法回传。

我们能否给出一个以 $\beta$ 为输入的函数，输出的结果为随机变量 $X$ 的采样值 $x$ 呢？答案是肯定的，作者采用了 Gumbel-Max trick 进行再参数化，如下，$z$ 是一个 onehot 向量，$g_i$ 是 Gumbel 分布 G 的采样，$G = -log(-log(U))$，$U \sim Uniform(0,1)$

$$z = arg\ max(log\beta_i + g_i)$$

即

$$P(x|Pa_x)=\begin{cases}  1, & i=argmax_j(log(\beta_j) + g_j) \\  0, & otherwise  \end{cases}$$ 

因此得到的 $z$ 是一个两个元素的一维向量，元素值为 0 或 1，按照上图的流程，若 $z_1 = 1$，则执行该 block。

上面的 Gumbel-Max trick 实现了从 $\beta$ 向量到 $z$ 向量的转变，但是上面的流程中，argmax 操作是不可导的，作者提出用可导的 softmax 函数去代替 argmax，因此得到的 $z$ 向量为

$$z_i = softmax((log(\beta_i) + g_i) / \tau)$$

$\tau$ 是温度系数，当 $\tau \rightarrow 0$ 的时候，类别之间的距离被拉得很远，softmax 的结果就会趋于一个 onehot，softmax 就能逼近 argmax；而 $\tau \rightarrow \infty$，不同类别的概率趋于一致，softmax 的结果趋于一个均匀分布。

因此作者提出，训练中前向传播，严格用 argmax 得到离散值，反向传播用 softmax 来求梯度。



**损失函数**

除了 multi-class logistics loss $L_{MC}$ 外，作者还想将计算量写入损失函数。定义一个 batch 的样本应该通过某一层的比率为 t (target rate)，$\overline{z}_l$ 代表实际中一个 batch 样本通过的样本比例，定义另外一个 loss 为，$N$ 为 batch size。

$$L_{target} = \sum_1^N (\overline{z}_l - t)^2$$

$$L_{Adanet} = L_{MC} + L_{target}$$

其中 $t = 0.6$，$\lambda = 2$



<br>

# 实验结果

作者先在 CIFAR-10 上做了实验，AdaNet 有一定的提升，且计算量减少了一点。

{% asset_img 3.png %}

下图是不同层的通过率，downsampling layer 都有极高的通过率，Wide AdaNet 在后面的层，不同类别之间的通过率有明显的不同。

{% asset_img 4.png %}

下面是 ImageNet 的结果，并没有变得很好，但是计算量少了不少。

{% asset_img 5.png %}

下图表示前 30 个 epoch，通过率的变化，通过率初始化为 0.85，可以看出后面的层和 downsampling 层都比较快的达到通过率 100%，而其它层的通过率则不断下跌。

{% asset_img 6.png %}

最后，作者还表示 AdaNet 可以防御对抗样本的攻击。




<br>