---
title: 【论文阅读】—— Dual Path Network
date: 2018-12-11 15:11:02
mathjax: true
tags:
---


> Title：Dual Path Network
> Authors：颜水成团队
> Session：NIPS, 2017
> Abstract：近几年，ResNet 和 DenseNet 都在视觉任务上取得了比较好的成绩，作者结合两者的优点，提出一种双通道的网络 (dual path network, DPN)，并取得了 ILSVRC-2017  目标定位的冠军。



<!--more-->

<br>

*preface*

ResNe t和 DenseNet是近几年两种比较热门的网络结构，ResNet 把输入直接加到（element-wise adding）卷积的输出上，DenseNet则把每一层的输出都拼接（concatenate）到其后每一层的输入上。在这篇论文中作者用High Order RNN 结构（HORNN）把 DenseNet 和 ResNet 联系到了一起，证明了 DenseNet 能从靠前的层级中提取到新的特征，而 ResNet 本质上是对之前层级中已提取特征的复用。通过把这两种结构的优点结合到一起，提出了 Dual Path Networks（DPN）。



<br>

# Revisiting ResNet, DenseNet and Higher Order RNN

## Higher Order RNN(HORNN)

在普通的 RNN 结构中，一个时刻的输入会与上一时刻的状态共同决定这一时刻的状态，而高阶 RNN 中，一个时刻的状态不仅与上一个时刻的状态有关，还和前面所有时刻的状态都有关，每个时刻的计算公式如下，$h^k$ 表示第 $k$ 个时刻的状态，也是当前状态，$f^k_t(h^t)$ 表示计算从 $h^t$ 传到 $h^k$ 的信息，$\sum_{t=0}^{k-1}f_t^k(h^t)$ 表示前 $k-1$ 个状态传给第 $k$ 个状态的信息和，$g^k(\cdot)$ 将汇聚起来的信息进行处理得到第 $k$ 个时刻的状态。

$$h^k = g^k[\sum_{t=0}^{k-1}f_t^k(h^t)]$$

HORNN 一个重要的特点是，对于任意的 $k$ 和 $t$，都有：

- $g^k(\cdot) = g(\cdot)$，即每个时刻的状态计算函数是相同的，就像普通的 RNN 一样，不同时刻用同一个隐层。
-  $f^k_t(\cdot) = f_t(\cdot)$，即每个状态对后面时刻状态传递的信息是一样的，不会随着不同时刻而改变。



##用 HORNN 去解释 DenseNet

DenseNet 每一层的输入是前面所有层输出 concate 的结果，concate 之后进入一个 bottleneck layer，即 $1 \times 1$ 的卷积层，然后经过一个 $3 \times 3$ 的卷积层，得到该层的输出。

假设 DenseNet 的 growth rate 为 c，每一层输出的 feature map 大小为 $h \times w$，那么第 $k$ 层的输入大小为 $h \times w \times (k-1)c$，$f^k(\cdot)$ 为 $1 \times 1 \times (k-1)c$ 的卷积层，由于是一个 $1 \times 1$ 的卷积，就相当于 feature map 的每个像素点 $x$ (一个长度为 $(k-1)c$ 的向量) 和卷积核 $w$ 进行向量內积，$w$ 和 $x$ 都是一个 $(k-1)c$ 维的向量，我们不妨将这两个向量分成 $k-1$ 份，$x = [x_1, x_2, ..., x_{k-1}]$，$w = [w_1, w_2, ..., w_{k-1}]$，每一份单独做內积再相加等同于整体內积，即 $w^Tx = \sum w_i^T x_i$。

由上面的推论，我们可以得知，concat 后作 $1 \times 1$ 的卷积，等同于前面 $k-1$ 个状态单独做一个 $1 \times 1$ 的卷积后相加，而 $f_t^k(\cdot)$ 等于 $f^k$ 对应的 $1 \times 1$ 的卷积核在 channel 上的第 $t$ 组，如 $f_1^k(\cdot)$ 就等于卷积核的 $[1,1, 0 : c]$ 这部分。

{% asset_img 1.png %}

从这可以看出，DenseNet 的形式完全符合 HORNN 的形式，$f_t^k(\cdot)$ 表示第 $k$ 层第 $t$ 组 channel，$g^k(\cdot)$ 表示 $3 \times 3$ 的卷积层。需要注意的是，DenseNet 不满足 HORNN 的一个条件，即  $f^k_t(\cdot) = f_t(\cdot)$ 和 $g^k(\cdot) = g(\cdot)$，因为 DenseNet 每一层的 $1 \times 1$ 卷积层和 $3 \times 3$ 卷积层都不同。

{% asset_img 2.png %}



下图从通道的角度去理解 DenseNet，左边表示 channel，即不同层 concat 的结果，或者不同时刻状态 concat 的结果，一个灰色的圆圈代表一个时刻的状态或一层的输出（虽然第一个圈和第二个圈对应的 channel 的宽度不成比例，但是我们将其理解为一个状态），我们把目光放到第二个时刻，这时候 channel 中有两层的输出，我们将其单独提取出来，每一个状态（每一层的输出）经过一个 $1 \times 1$ 的卷积层后进行相加，这里的 $1 \times 1$ 的卷积层就是 $f^k_t(\cdot)$，而一个灰圈代表 $h^t$。按照上面的图，相加后的结果应该直接经过 $g^k(\cdot)$ 函数，即 $3 \times 3$ 的卷积层，但是这里作者另外加了一个 $1 \times 1$ 的卷积层（带下划线），目的是为了和 ResNet 匹配过来，没有其他用途，$3 \times 3$ 卷积核的输出作为这一时刻的状态，concat 到左边的 channel 中，下一层从左边取出来的就是 3 个时刻的状态，即 3 个灰圈。

{% asset_img 4.png %}



##用 DenseNet 去解释 ResNet

我们知道，DenseNet 可以写成 HORNN 的形式。

$$h^k = g^k[\sum_{t=0}^{k-1}f_t^k(h^t)]$$

将上式子进行一些变量替换，并且假设 $f^k_t(\cdot) = f_t(\cdot)$，令

$$r^k = \sum_{t=0}^{k-1}f_t^k(h^t) = \sum_{t=0}^{k-1}f^k(h^t) = r^{k-1} + f_{k-1}(h^{k-1})$$

而 $h^k = g^k(r^k)$，所以

$$r^k = r^{k-1} + f_{k-1}(g^{k-1}(h^{k-1})) = r^{k-1} + \phi^{k-1}(r^{k-1})$$

上式就是 ResNet 的结构，$\phi^{k-1}(\cdot) = f_{k-1}(g^{k-1}(\cdot))$ 为 residual function。由上面的推导可以得出，ResNet 是 DenseNet 在 $f^k_t(\cdot) = f_t(\cdot)$ 情况下的特殊表达式，下图上半部分是我们熟知的 ResNet 结构，一个 block 由一个 $3 \times 3$ 的卷积层和 $ 1\times 1$ 的卷积层组成，每个 block 的状态为 $r^k$，状态转移是 ResNet 的方式；下半部分是其等效图，一个 block 的组成成分变为 $1 \times 1$ 和 $3 \times 3$，每个 block 的状态为 $h^k$，状态的转移需要先将前面 $k-1$ 个状态经过一个 $f_t(\cdot)$ 函数然后相加后，经过 $g^k(\cdot)$ 得到该时刻的状态 $h^k$。

{% asset_img 3.png %}



上下两幅图对于 $g^k(\cdot)$ 有一点出入，下图是按照论文来的

{% asset_img 7.png %}

下图是 ResNet 的图，左边的矩形代表 channel，灰圈代表 $r^k$，$r^k$ 经过一系列卷积后和 $r^k$ 相加得到 $r^{k+1}$，这幅图并没有从 HORNN 的角度去解释 ResNet（即上图的下半部分）。

{% asset_img 5.png %}

上面我们证明了 ResNet 是 DenseNet 在 $f^k_t(\cdot) = f_t(\cdot)$ 下的特殊表达式，下图证明了这一点，(b) 中每一时刻的 $1 \times 1$ 卷积都不同，即绿色和橙色的箭头在每个时刻都不同。而 (c) 将 (b) 的形式 reformulate 了一下，将加法写成了一个独立的主分支，由于不同时刻的 $f_t(\cdot)$ 都一样。那么对于 k 时刻，$f_1(h^1) + ... + f_{k-1}(h^{k-1})$，而对于 k+1 时刻，$f_1(h^1) + ... + f_{k-1}(h^{k-1}) + f_k(h^k)$，因此只需要每次往右边的主干上加入 $f_k(h^k)$ 即可，这也就是下面的 (c) 图，$\underline{1 \times 1}$ 和 $3 \times 3$ 表示 $g^k(\cdot)$，经过 $g^k(\cdot)$ 得到 $h^k$，然后经过一个 $1\times1$ 的卷积得到 $f_k(h^k)$（橙色箭头）加入到右边。绿色和橙色的箭头只会出现在residual block中一次，也就意味着被加到了主分支之后，再也不会改变了。这也就是 with shared connections 的含义。  

{% asset_img 6.png %}



## Motivation

从上面的分析可以看出，由于 ResNet 中 $f^k_t = f_t$，所以后面的层在复用前面层已经提取过的特征，除去这些直连的复用特征外，真正由卷积提取出来的特征“纯度”就比较高了，基本都是之前没有提取到过的全新特征，所以作者说 ResNet 提取的特征中冗余度比较低，且鼓励 feature reuse。而 DenseNet 每一层的 $f^k_t$ 都是不相同的，前面层提取出的特征不再是被后面层简单的复用，而是创造了全新的特征，这种结构后面层用卷积提取到的特征很有可能是前面层已提取过的，所以作者说 DenseNet 提取的特征冗余度高，但是鼓励探索新特征。

ResNet 有高的特征复用率，但冗余度低，DenseNet 能创造新特征，但是冗余度高，作者提出把两种结构结合起来，提出 dual path network。

<br>

# Dual Path Network

DPN 顾名思义就是有两条通道的网络，而这两条通道分别是 DenSeNet 和 ResNet

$$x^k = \sum_{t=1}^{k-1}f_t^k(h^t)$$

$$y^k = \sum_{t=1}^{k-1}v_t(h^t) = y^{k-1} + \phi^{k-1}(y^{k-1})$$

$$r^k = x^k + y^k$$

$$h^k = g^k(r^k)$$

下图是 DPN 的结构，(e) 图是将 (d) 图的两个主干合并的结果，卷积后的结果被切分成两部分，一部分加到 ResNet 的通道里面，一部分 concat 到 DenseNet 中。(d) 图和上面的公式有点不符，ResNet 通道出来的 $1 \times 1$ 卷积不知道怎样解释。

{% asset_img 8.png %}

整体的网络结构如下，DPN 和 DenseNet 和 ResNeXt 结构不同之处在于每个 block 的设计

- $3 \times 3$ 的卷积层采用的是 group convolution
- $1 \times 1 \times 256(+16)$ 中的 256 代表的是 ResNet 的通道数，16 代表的是 DenseNet 一层的输出通道数，将结果分成 256 和 16 两部分，256 的 element-wise 的加到 ResNet 通道，16 的 concat 到 DenseNet 通道，然后继续下一个 block，同样输出 256 + 16 个通道，重复操作。
- 在 conv 之间过渡的时候，如 conv2 输出到 conv3 输入，需要经过一个下采样，即 feature map 大小减半，通道数翻倍，具体实现上是将上一个 conv 的输出经过一个 stride 为 2 的 $1 \times 1 $ 的卷积层，将通道数翻倍。而在 conv2 中 max-pooling 层过渡到 DPN block 的时候，也是用这种方式，经过一个 $1 \times 1$ 的卷积层，将通道数调整为 256 + 16$\times$2，然后将通道分成两份 256 和 16$\times$2，至于为什么 DenseNet 的通道数需要初始化为 2$\times$+k，作者没有说明。
- 一点细节：每个 conv 中的 $3 \times 3$ 卷积层的输出 channel 数为 ResNet 在这一层的 channel 数，+k 则为该层 DenseNet 的 channel 数。

具体实现的细节，可以参考：<https://github.com/cypw/DPNs/tree/master/settings>。 

{% asset_img 9.png %}

<br>

# 实验结果

作者在 classification、object detection 和 semantic segmentation 三个任务中验证了 DPN 的有效性，这里只讲基于 ImageNet 的分类任务

- 先看第一个 block，DPN-92 在 top-1 error 中最好，比 ResNeXt 要低 0.5%，且计算量要低。
- 再看第二个 block，深一点的 DPN-98 还是要比 ResNeXt 要好，计算量也少了 25%。
- 第三个 block，继续加深 DPN 到 131 层，效果依旧明显。

{% asset_img 10.png %}



<br>

*In Conclusion*

作者从 HORNN 的角度去理解 ResNet 和 DenseNet，并发现 ResNet 是 feature reuse，而 DenseNet 是探索新特征，基于这两个 motivation，提出 DPN，实验结果也证明了其在多项任务上的有效性。



<br>

*References*

1. https://zhuanlan.zhihu.com/p/42906045
2. https://zhuanlan.zhihu.com/p/34993147
3. https://github.com/cypw/DPNs/tree/master/settings
4. https://www.jianshu.com/p/c13a9900d643

<br>