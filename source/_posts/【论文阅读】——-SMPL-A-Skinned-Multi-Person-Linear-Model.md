---
title: '【论文阅读】—— SMPL, A Skinned Multi-Person Linear Model'
mathjax: true
date: 2020-09-16 23:04:34
tags:
---

> Title：SMPL: A Skinned Multi-Person Linear Model
> Authors：Matthew Loper, et al
> Conferences：ToG 2015
> Abstract：作者提出了一个参数化的人体蒙皮模型，通过 23 个关节点的旋转以及 10 个 shape 参数就可以生成人体蒙皮。

<!--more-->

*prefix*

2D 数据的表示非常单一，一般就用 image 来表示；而3D 数据的表示有多种，比如 multi-view image、point cloud、mesh、voxel，这里我们用到的数据为 mesh 数据，一个 mesh 由多个 vertices 组成，每个 vertices 是一个三维坐标 (x,y,z)，除了有 vertices 外，生成 mesh 的时候还需要有一个 triangle list，一个 3D object 外观上有多个 triangle 拼接而成，而 list 里面存储的就是每个 triangle 所用到 vertices index。这里我们关注的是 3D 人体模型，同样的人体由多个 vertices 组成，而 joint 是一些特殊的 vertices，用来表征人体的一些特殊关节点，如肘、肩等，实质上就是一个 vertice，但这些 joint 有可能在人体 mesh 的 vertice 里面，也可能不在。

Linear Blending Skinning (LBS) 指的是通过操纵骨骼来使得 mesh 发生形变，LBS 定义骨骼点（joint）和 mesh 的 vertices 之间的关系是线性关系，一个 vertice 的位置会受到其他 joint 共同的加权影响，具体操作是会先计算 joint 的位置，然后根据 joint 的位置，求其他 vertices 的位置，所以可以通过操纵骨骼来改变整个 mesh。

<br>

### Model Formulation

SMPL 模型的 mesh 由 $N=6890$ 个 vertices 和 $K = 23$ 个 joints 组成，参数为 $\theta$ 和 $\beta$，分别表示 23 个关节点的旋转以及人体 shape 的参数，后者是一个 10 维的向量，前者向量大小取决于旋转用什么方式来表征，比如旋转向量、旋转矩阵等。

**Notation**

- mean template $\bar{T}\in \mathbb{R}^{3N}$，可以理解为初始的一个 mesh，rest pose
- zero pose $\theta^\ast$，初始 mesh 下各个关节点的旋转角度，这里的旋转角度实际上是相对于每个点的父节点的旋转角度，比如肘部的父节点是肩部，所以肘部的旋转值是相对于肩部来确定的
- blend weight $\mathcal{W} \in \mathbb{R}^{N\times K}$，表示的是一个关节点 k 的旋转对于所有 N 个 vertices 的影响
- blend shape function $B_S(\beta) : \mathbb{R}^{|\beta|} \rightarrow \mathbb{R}^{3N}$，输入 shape 参数，输出 vertices offset
- joint prediction function $J(\beta) : \mathbb{R}^{|\beta|} \rightarrow \mathbb{R}^{3K}$，输入 shape 参数，输出 joint 的位置
- pose-dependent blend shape function $B_P(\theta) : \mathbb{R}^{|\theta|} \rightarrow \mathbb{R}^{3N}$，输入 $\theta$，输出 vertices offset
- linear blend skinning function $W(\cdot)$ is applied to rotate the vertices around the estimated joint
- SMPL model $M(\beta,\theta;\Phi):\mathbb{R}^{|\theta| \times |\beta|} \rightarrow \mathbb{R}^{3N}$



**Blend Skinning**

骨骼蒙皮通过骨骼来控制整个 mesh 的变化，$\theta$ 由 K 个关节点的旋转向量组成 $\theta = [w_0, ..., w_K]^T$，$w_k \in \mathbb{R}^3$ denotes the axis-angle representation of the relative rotation of part k with respect to its parent in the kinematic tree. $|\theta| = 3 \times 23 + 3 = 72$ 个参数，每个关节点 3 个参数外加整体的 root orientation，首先旋转向量会通过 Rodrigues formula 转换成 $3\times3$ 的旋转矩阵 $exp(w_k)$，标准的 LBS 为以下的函数
$$
W(\bar{T}, J,\theta, \mathcal{W}):\mathbb{R}^{3N\times 3K \times |\theta| \times |\mathcal{W}|} \rightarrow \mathbb{R}^{3N}
$$
函数会输入 rest pose $\bar{T}$，rest pose 下的 joint location $J$，pose $\theta$，blend weight $\mathcal{W}$，最后输出 posed vertices，对于每一个 vertices 的 transformation 如下，实际做的就是对 rest pose 下的每一个 joint 计算变换，然后再将计算出来的变换应用到每一个 vertice 上。

{% asset_img 1.png %}

$G_k(\theta,J)$ 表示第 k 个关节点的 world transformation，这里的 world 是指世界坐标，所以用到的是齐次坐标系，$\bar{t}_i$ 和 $\bar{t}_i^\prime$ 分别表示 T-pose 下和变换后的第 i 个vertice，也都是齐次坐标系，4 维向量。$A(k)$ 表示的是第 k 个关节点的所有父节点，比如手腕的父节点分别有手肘、肩部等，因此在计算第 k 个关节点的变换的时候，依次乘以父节点的旋转矩阵，最后得到 $G_k(\theta, J)$，$G_k(\theta^\ast,J)$ 表示的是得到 rest pose 所需要的关节点的旋转变换，因此 $G^\prime_k(\theta,J)$ 是减去得到 rest pose 的变换，相当于得到一个相对的变换矩阵，然后变换矩阵乘以 rest pose 下的第 i 个 vertices 就可以得到变换后的第 i 个 vertice，然后乘上 $w_{k,i}$ 分量，表征第 k 个关节点对第 i 个 vertice 的影响。

SMPL 模型对 LBS 进行了修改，具体的流程可以看下图，首先会根据 $\beta$ 调整 T-pose，然后再根据 $\theta$ 调整（看臀部的位置，这是为后面的抬腿动作做准备），最后一步就是 LBS。
$$
M(\beta,\theta) = W(T_P(\beta,\theta), J(\beta), \theta, \mathcal{W})
$$

$$
T_P(\beta, \theta) = \bar{T} + B_S(\beta) + B_P(\theta)
$$

{% asset_img 2.png %}

**Shape blend shapes**

$S = [S_1, ..., S_{|\beta|}] \in \mathbb{R}^{3N\times |\beta|}$，表示 shape 每个参数对 vertices 的影响，S 是通过数据学习出来的
$$
B_S(\beta; S) = \sum_{n=1}^{|\beta|} \beta_n S_n
$$
**Pose blend shapes**

每个 pose 参数都用旋转矩阵表示，所以是 9K，同样的 $P\in \mathbb{R}^{3N \times 9K}$ 矩阵通过数据学习出来
$$
B_P(\theta; P) = \sum_{n=1}^{9K}(R_n(\theta) - R_n(\theta^\ast))P_n
$$
**Joint locations**

这里谈到的 joint 和上面用到的 $J$ 也都是在 rest pose 下的 joint 的 3D location，SMPL 在计算 joint 的时候引入了 $\beta$，更加精确，$\mathcal{J}$ is a matrix that transforms rest vertices into rest joints，同样通过数据学习出来。
$$
J(\beta; \mathcal{J}, \bar{T}, S) = \mathcal{J}(\bar{T} + B_S(\beta;S))
$$
上面提到需要学习的参数有 $\Phi = \{\bar{T}, \mathcal{W}, S, \mathcal{J}, P \}$，也都是一些 regressor，或者说是矩阵，这里不介绍模型的训练过程，有兴趣的可以仔细看原论文。

<br>

### 代码分析

作者共提供了三个预训练模型，分别是 male、female 和 neutral，下面以 `SMPL_NEUTRAL.pkl` 为例子，讲一下模型里面的参数，pkl 文件里面是一个字典，关键的 key 如下

- `J_regressor`: (24, 6890)，从 rest pose 中回归出 joint，上面的 $\mathcal{J}$ 矩阵
- `f`: (13776, 3)，faces，上面我们说到 mesh 除了有 vertices 组成，还有一个 triangle list，这里就是这个 list，可以看出人体共有 13776 个 triangle，每个 triangle 由三个 vertices index 组成，所以 faces 最大的数字就是 6889，因为共 6890 个点。
- `kintree_table`: (2, 24)，一般取第一行，这就是上面提到的每个点的父节点
- `weights`: (6890, 24)，blend weight，上面的 $\mathcal{W}$ 矩阵
- `posedirs`: (6890, 3, 207)，上面的 $P$ 矩阵
- `shapedirs`: (6890, 3, 10)，上面的 $S$ 矩阵
- `v_template`: (6890, 3)，上面的 $\bar{T}$

smplx 的库文件里面包含三个模型，分别是 SMPL、SMPLH 和 SMPLX，这里只分析最基础的 SMPL 的代码里面的 lbs 函数

{% asset_img 3.png %}

1. 计算 shape blend shape offset

```python
def blend_shapes(betas, shape_disps):
    blend_shape = torch.einsum('bl,mkl->bmk', [betas, shape_disps])
    return blend_shape

v_shaped = v_template + blend_shapes(betas, shapedirs)
```

2. 计算 rest pose 下的 joint

```python
def vertices2joints(J_regressor, vertices):
    return torch.einsum('bik,ji->bjk', [vertices, J_regressor])

J = vertices2joints(J_regressor, v_shaped)
```

3. 计算 pose blend shape offset，如果输入的 pose 是旋转向量，那先需要转成旋转矩阵，去掉第一个点，因为是 root，至于为什么旋转矩阵要再减去一个 identity matrix，暂时不清楚

{% asset_img 4.png %}

4. 计算旋转后的关节点，输入有旋转矩阵，rest pose 下的 joint，每个 joint 的parent
   - rel_joints: relative joints，指的是每个关节点其父节点指向自己的向量，维度是 (b, 24, 3)
   - transforms_mat：变换矩阵，上面提到的未连乘起来的 $G$，旋转矩阵的维度是 (b, 24, 3, 3)，和 relative joint 一起拼成 (b, 24, 4, 4) 的齐次变换矩阵
   - transform_chain：这里实现的应该是每个节点只有一个 parent 节点，所以 transform_chain 实际就是将 transform_mat 中的每个关节点的变换矩阵乘上其父节点的变换矩阵
   - posed_joints：不是很懂为什么 transform_mat 相乘后会将 rel_joints 直接也进行变换，但可能就是会得到变换后的点吧
   - rel_transform：不是很懂

{% asset_img 5.png %}
{% asset_img 6.png %}

5. 最后一步就是 skinning，
   - 首先将 blend weight 扩展成维度 (b, 6890, 24)
   - A 矩阵就是上面函数得到的 rel_transform 变换矩阵，blend weight 乘以变换矩阵得到 T
   - vposed_homo 就是将 vertices 转成齐次坐标，然后和 T 矩阵相乘得到最后变换后的 vertices，取前三个回到正常坐标。

```python
# W is N x V x (J + 1)
W = lbs_weights.unsqueeze(dim=0).expand([batch_size, -1, -1])
# (N x V x (J + 1)) x (N x (J + 1) x 16)
num_joints = J_regressor.shape[0]
T = torch.matmul(W, A.view(batch_size, num_joints, 16)) .view(batch_size, -1, 4, 4)

homogen_coord = torch.ones([batch_size, v_posed.shape[1], 1], dtype=dtype, device=device)
v_posed_homo = torch.cat([v_posed, homogen_coord], dim=2)
v_homo = torch.matmul(T, torch.unsqueeze(v_posed_homo, dim=-1))

verts = v_homo[:, :, :3, 0]
```



<br>