---
title: >-
  【论文阅读】——OpenPose, Realtime Multi-Person 2D Pose Estimation using Part Affinity
  Fields
mathjax: true
date: 2020-08-30 20:01:07
tags:
---

>Title: OpenPose: Realtime Multi-Person 2D Pose Estimation using Part Affinity Fields
>Author: Zhe Cao
>Conference: CVPR 2017（后扩展为期刊）
>Abstract: 作者提出了一种实时的 2D 关节点检测的算法，并将关节点扩展为手部、脚部、脸部共 135 个点。



<!--more-->

### Method

openpose 是一种 bottom-up 的方法，所谓的 bottom-up，就是从一张图片中先找出所有的关节点，然后再对关节点进行重组，这种方法不受图片中人数的影响，runtime 比较固定，因此可以做到实时，openpose 整体的流程如下。整张图片作为输入，经过神经网络会输出两种图，一种是 heatmap（part confidence maps，openpose里面将 keypoint 称为 part），另一种是骨骼连接图（part affinity fields，也就是连接不同关节点之间的骨骼），然后再对得到的关节点进行 match 和 parse。

{% asset_img 1.png %}



#### Confidence Maps

在 top-down 方法里面，ground-true 的关节点的热力图是以关节点为原点的一个二维高斯分布，热力图最大值的点即为关节点；而对于 bottom-up 的方法，一张图片会有多个人，因此对于某一个关节点，如脖子，热力图上会有多个 peak，是多个高斯分布的叠加，这里不同高斯分布采取的是取最大值的做法，防止关节点隔得太近，所以不能取平均，$S^{\ast}_j $ 表示的是第 j-th 个关节点的 GT 热力图

{% asset_img 2.png %}



#### Part Affinity Fields for Part Association

假设一个人体有 17 个关节点，那么 17 个点，就会有 16 根连线，这里的一根连线对应一个 part affinity field，PAF 是一个 $w\times h \times 2$ 的矩阵，每个 pixel 对应一个 2D 向量，指示连线的方向。对于 limb c 上的点，PAF 为一个指示单位向量。至于如何找到 limb c 上的点，作者连接两个关节点成一条线，距离该线在一定范围内的点即为 limb c 上的点。一张图会有 k 个人，所以会有 k 个 limb c，因此最终对于 limb c 的 GT PAF $L^\ast _c$ 是单独一个人 PAF 的平均。 

{% asset_img 3.png %}
{% asset_img 4.png %}

测试的时候，会预测出一堆的关节点，以及一堆的 limb，就比如图1的例子，两个人，预测出两个肩膀点和两个手肘点，这里可以自由组合出 4 个 limb，然后预测的 PAF 会有两条 limb，如果会用组合的 4 个 limb 去对预测的两条 limb 做积分，来算一个能量值，若组合出来的某一条 limb 的方向和预测的方向几乎一致，那么这个能量值就会很高，反之很低，然后就会用这个能量值作为 keypoint parsing 的依据

{% asset_img 5.png %}

#### network architecture

在原 openpose 论文里面（cvpr会议），网络结构是采取 simultaneous 的形式，分两条支路分别预测 part confidence 和 part affinity field，但后面改成期刊后，采取了 serial detection and association，先预测 limb 再预测 part。

{% asset_img 6.png %}

首先图像经过一个 VGG-19 作为特征提取器，得到特征图 F，特征图后有先经过 $T_P$ 个 stage 的 limb 预测，每个 stage 的预测上一个 stage 的预测结果以及特侦图 F 作为输入

{% asset_img 7.png %}

$T_P$ 个 stage 的 part affinity field 后是 $T_C$ 个 stage 的 part confidence maps，预测关节点的时候会将预测的 PAF 作为输入

{% asset_img 8.png %}

网路训练引入了 intermediate supervision，每个 stage 的输出都用来进行监督

{% asset_img 9.png %}



#### Multi-person Parsing using PAFs

我们网络预测得到 candidate parts 和 candidate limbs，组合过程采取启发式的方法，对于一条 limb c，连接它的两个关节点分别是 $j_1$ 和 $j_2$，检测到的两种关节点数量为 $D_{j_1}$ 和 $D_{j_2}$，这两个数量不一定相等，因为存在遮挡或者半身情况，然后建立一个二分图，边的权重为上面提到的能量值，然后对于一条 limb c，我们需要找出一个使得边权重最大的二分图，然后这就是最可能的 limb。

对于整个 body，分别找出每条 limb 的组合，再拼接起来即为最终结果。



#### foot detection

openpose 将 coco 的 17 个点扩展成 25 个点，其中增加了 6 个脚部的点，脚部的数据集是部分 coco 数据集中标注出来的，共 1.5k 张。叫上脚部检测后，解决了遮挡的问题，左图的 b 和 c。

{% asset_img 10.png %}

除此之外，openpose 还训练了手部和脸部关节点的检测器，整体效果如下

{% asset_img 11.png %}



<br>