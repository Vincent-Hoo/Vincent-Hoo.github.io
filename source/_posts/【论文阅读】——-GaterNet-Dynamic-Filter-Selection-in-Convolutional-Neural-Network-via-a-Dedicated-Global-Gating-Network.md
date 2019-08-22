---
title: >-
  【论文阅读】—— GaterNet, Dynamic Filter Selection in Convolutional Neural Network
  via a Dedicated Global Gating Network
mathjax: true
date: 2018-12-13 23:26:09
tags:
---

> Title：GaterNet: Dynamic Filter Selection in Convolutional Neural Network via a Dedicated Global Gating Network
> Authors：Zhourong Chen, Yang Li, Samy Bengio, Si Si
> Abstract：conditional computation 是指用网络的一部分层去预测结果，而不是整个网络。这样可以使网络中不同的卷积核能够从不同的样本中学习到特征，一定程度上提高了模型的泛化性能和可解释性。作者提出了一个根据输入样本动态选择卷积核的算法，模型包含两个网络：gater network 和 backbone network，gater network 决定样本通过 backbone network 时需要经过哪些卷积核。



<!-- more -->

<br>

*preface*

conditional computation 是指：对于每一个样本，只用模型的一小部分去进行预测，那么在反向传播的时候，也是只有一小部分的参数需要被更新，这对于训练一个庞大的网络是非常有好处的。

conditional computation 相关的研究主要分为两个方面：Mixture of Experts (MoE) 和 dynamic network configuraion。MoE 通过集成多个子网络来进行预测，子网络的权重由一个 gating module 决定，最近有些研究提出只集成一小部分的子网络，这些子网络是根据不同的样本动态选择的；dynamic network configuration 只有一个主模型，但是会动态选择模型的单元、层数或者其他的 component，通常一些小的 module 会加到每一个网络的 component 中来决定该部分是否需要被选择，这种方法作者称为 local configuration，因为这些小 module 是根据这个 component 当前的信号值来决定是否被选择的，典型的代表是最近 EECV 的一篇论文 《Convolutional networks
with adaptive inference graphs》。

作者提出了一种 global configuration 的方式，通过一个 gater network 对输入进行全局探索，最后得出哪些卷积核需要被使用。

<br>

# GaterNet

GaterNet 分为两个部分：gater network 和 backbone network，给定一个输入样本，gater network 决定 backbone network 使用哪些卷积核进行预测，两个网络用端到端的方式通过反向传播进行训练。

{% asset_img 1.png %}

## Backbone network

backbone network 是模型主要的部分，用来预测最终的结果，任何的 CNN 结构都可以作为 backbone network，如 ResNet、Inception、DenseNet 等等。给定一个输入 $x$，第 $l$ 层的输出为 $O^l(x)$ ：

$$O^l_i(x) = \phi(F^l_i \ast I^l(x))$$

$O^l_i(x)$ 是输出 $O^l(x)$ 的第 $i$ 个 channel，$F^l_i(x)$ 是第 $i$ 个卷积核，$I^l(x)$ 是第 $l$ 层的输入，$\ast$ 代表卷积，$\phi(\cdot)$ 代表非线性激活函数。一般情况下，如果没有 gater network，则所有的卷积核 $F_i^l$ 都会被使用，然后得到一个 dense 的输出 $O^l(x)$，有了 gater network，模型会选择一部分的卷积核来进行卷积，而不是全部卷积核。



## Gater network

对比于 backbone network，gater network 是辅助性作用，并不会直接学习一些特征用于预测。而是通过处理输入图像，然后生成一个二进制的向量 (gating mask)，这个向量用来动态选择一部分的 backbone network 的卷积核用来预测，gater network 的方程如下：

$$G(x) = D(E(x))$$

$E(\cdot)$ 是一个图像特征提取器，$h^{\prime},w^{\prime},c^{\prime}$ 表示输入图像的宽高和通道数，$h$ 表示提取特征的维数。

$$E: x \rightarrow f, x \in R^{h^{\prime} \times w^{\prime} \times c^{\prime}}, f \in R^h$$

$D(\cdot)$ 将特征变换到二进制向量，$c$ 表示 backbone network 中卷积核的总数。

$$D: f \rightarrow g, f \in R^h, g \in \{0,1\}^c$$

综上，gater network 将一个输入图像 $x$ 映射到一个二进制向量 $g$，有了 gating network，backbone network 中第 $l$ 层的输出重新定义为：

$$O^l_i(x)=\left\{\begin{array}{cc}  zero \ matrix, & g^l_i=0\\  \phi(F^l_i \ast I^l(x)) & g^l_i = 1  \end{array}\right.$$ 

$g^l_i$ 是 $g$ 向量中对应第 $l$ 层第 $i$ 个卷积核的元素，可以看出如果 $g^l_i$ 为 0，第 $l$ 层第 $i$ 个卷积核被跳过，用 0 去代替卷积的输出。当第 $l$ 层有很多的卷积核被跳过时，输出就是一个稀疏的图像。具体实现上，将上式合并成以下的形式：

$$O^l_i(x) = \phi(F^l_i \ast I^l(x)) \cdot g^l_i$$

接下来将讨论何如选择函数 $E(\cdot)$ 和 $D(\cdot)$

### Feature Extractor

$E(\cdot)$ 是一个特征提取器，任意的 CNN 都可以用来作为 $E(\cdot)$，对比 backbone network，$E(\cdot)$ 有两点不同：

1. 输出层被去除，使得 $E(\cdot)$ 输出的是特征，然后被 $D(\cdot)$ 使用
2. $E(\cdot)$ 不需要像 backbone network 那样复杂的 CNN，$E(\cdot)$ 只需要输出输入图像的一个简单的概括即可，太复杂的 $E(\cdot)$ 会带来过多的参数和计算量，且作者想避免 $E(\cdot)$ 喧宾夺主，毕竟 backbone network 才是真正用来预测的网络。

### Features to Binary Gates

$D(\cdot)$ 函数将一个 $h$ 维的向量 $f$ 映射到一个 $c$ 维的二进制向量 $g$，首先考虑用一层全连接网络进行映射，那么这一层网络的参数就有 $h \times c$（忽略 bias），由于 $c$ 代表 backbone network 中卷积核的总数，往往是几千个，而 $h$ 表示特征的个数，通常也不少，因此这一层的参数量和计算量都不少。为了减少计算量，作者用两层全连接层，第一层使用一个 bottleneck 进行降维到 $b$，因此参数量改为 $(h+c)\times b$。

$$f^{\prime} = FC_1(f)$$

$$g^{\prime} = FC_2(ReLU(BatchNorm(f^{\prime})))$$

$g^{\prime}$ 向量是一个实向量，作者通过一个离散化的方法将其转成一个二进制向量，该方法为 Improved SemHash (改进的语意哈希)。

在训练过程中，将 0 均值和单位标准差的 $c$ 维高斯噪声 $\epsilon$ 加到 $g^{\prime}$，即 $g^{\prime}_{\epsilon} = g^{\prime} + \epsilon$，计算两个向量

$$g_{\alpha} = \sigma^{\prime}(g^{\prime}_{\epsilon}) $$

$$g_{\beta} = 1(g^{\prime}_{\epsilon} > 0)$$

$\sigma^{\prime}(\cdot)$ 是饱和 sigmoid 函数

$$\sigma^{\prime}(x) = max(0, min(1, 1.2\sigma(x) - 0.1))$$

$\sigma(x)$ 是 sigmoid 函数，$g_\alpha$ 是一个实向量，范围在 0 - 1，可微分；而 $g_\beta$ 是一个二进制向量，不可微分。因此作者在前向传播的过程中，随机抽取一半的数据用 $g = g_\alpha$，另一半的数据用 $g = g_\beta$，当用 $g_\beta$ 的时候，用 $g_\alpha$ 的梯度去代替 $g_\beta$ 的梯度。

{% asset_img 2.png %}
在预测阶段，不需要引入高斯噪声，且全部适用 $g = g_\beta$。 

最后，作者希望 gater network 得到的 gates (二进制向量) 越稀疏越好，因此将 gater network 作为正则项

$$L = -logP(y | x, \theta) + \lambda \frac{||G(x)||_1}{c}$$



<br>

# 实验结果

作者先在 ResNet 上验证 GaterNet 的有效性，gater network 选用 ResNet-20，backbone network 选用 ResNet-20, ResNet-56, ResNet-164。结果如下，ResNet-Wider 指添加一些卷积核使得参数量和 GaterNet 接近的 ResNet，Gated Filter 指 backbone network 中卷积核的个数。

- GaterNet 比所有的原始 ResNet 要好，在  CIFAR-100 上，GaterNet 比 ResNet-164 的错误率要低了 1.83%。
- GaterNet 同样要比 ResNet-SE 要好，GaterNet 产生二进制的 gate 去选择卷积核，而 ResNet-SE 是 re-scale 通道值，尽管 GaterNet 在前向传递时会有信息损失，因为只用了部分的卷积核，但是模型却有更好的泛化性能，也证明了处理一个样本只需要部分的卷积核，不需要全部。
- ResNet-Wider 要比 ResNet 要好，这是显然的，ResNet-20-Wider 甚至要比 GaterNet 要好，这也比较容易理解，因为 gater network 是用 ResNet-20，GaterNet 有一半的参数都在做 gating strategy，而不是直接参与到预测中。但是 GaterNet 依旧和 ResNet-20-Wider 接近。
- 而对于 ResNet-50 和 ResNet-164，增加一些卷积核变成 ResNet-Wider 并不会增加多少准确率，而 GaterNet 却能明显提高准确率，这也说明了 GaterNet 的有效性，不止在参数量的增多，更在于 gating strategy。
- gater network 不需要很复杂，ResNet-20 就已经足够了，当 backbone network 越复杂的时候，gater network 所带来的参数量就越微不足道。

{% asset_img 3.png %}



<br>

**Gate Distribution**

这里的 gate 指的是一个卷积核是否被卷积，作者想探究一下 gater network 是否有用，想知道对于每个样本，学习到的二进制向量会是怎样的。作者用  ResNet-164-Gated 这个网络，把 CIFAR-10 的测试集通过 gater network 得到每个测试样本的二进制向量。一个 gate 代表一个卷积核，自然而然 gate 会有三种类型：1）Always On：对于所有样本都是 ON，2）Always Off：对于所有样本都是 OFF，3）Input-dependent gate：或 ON 或 OFF。ResNet-164 共有 54 个 residual unit，每个 unit 中会有一定数量的门，下图表示每个 unit 中三种类型的门的分布情况。

- 在浅层的网络，很大一部分的卷积核都是 OFF 的；
- 而随着网络加深，Always On 和 Input-dependent 增多。
- 在最后的两个 unit，Input-dependent 占最大的比例。

这和我们的一般认识相符合，浅层网络提取 low-level 的特征，深层网络提取 high-level 特征。

{% asset_img 4.png %}

作者认为 Input-dependent 这个定义不够精细，对于 CIFAR-10 中 10000 个测试样例，如果对于某一个卷积核，它只卷积了 1 个样本，skip 了 9999 个样本，它也是属于 Input-dependent 类型的。作者想知道每个 Input-dependent 类型的门，有多少个样本通过了这个门。ResNet-164 共有 7200 个卷积核，即 7200 个门，其中有 1567 个 Input-dependent 的门，在这 1567 个门中，有 443 个门几乎是全通过或全拒绝，即上面说到的情况。剩下 1124 个门的通过样本数量在 100 - 9900 之间。

下图的 x 轴表示一个 input-dependent 的门通过样本的个数，y 轴表示 input-dependent 门的个数，整体表示的意识是：通过样本数为 x 的门有 y 个。下图的两边表示的就是那 443 个门，几乎全部通过或几乎全部拒绝。如果将下图进行求和 (积分)，结果必定为 1567。

{% asset_img 5.png %}

上面的实验都是基于一个门来进行的，探索了每个门通过的样本数，下面坐着探索每个样本通过的门数，经过实验得到，通过的门数最小值和最大值分别是 5380 和 5506，平均通过门数为 5453，通过门数似乎服从正态分布。

{% asset_img 6.png %}

最后，作者想探究 gater network 究竟学到了什么 gating strategy，因此它用每个样本通过 gater network 得到的二进制向量 (7200维) 去代表每个样本，然后用 PCA 将其降维到 400，再用 t-SNE 映射到二维空间，如下图，可以看出，相同 label 的二进制向量都比较靠近，因此 gater network 学习到：对于相同 label 的样本，它会试着让它通过相似的门，也就是说用 backbone network 的相同部分去处理这些 label 相同的样本。另外，下图的簇并不是区分地很好，这也说明了 gater network 并没有占据 backbone network 的地位。

{% asset_img 7.png %}



**CIFAR-10 上的结果**

{% asset_img 8.png %}



**ImageNet 上的结果**

{% asset_img 9.png %}


<br>