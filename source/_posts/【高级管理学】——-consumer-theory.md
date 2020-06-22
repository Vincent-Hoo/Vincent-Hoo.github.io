---
title: 【高级管理学】—— consumer theory
mathjax: true
date: 2019-10-04 21:56:42
tags:
---


> 课程名称：高级管理学
> 授课人：Ying Kong
> 本章梗概：介绍了消费者的相关理论，如效用函数，需求曲线，弹性等概念。

<!--more-->


## Preference

bundle：多个不同种类商品的集合，比如 bundle x 为 5 个商品 1 和 10 个商品 2 组成。

preference：对于不同的 bundle，消费者可以选择其中的一个，对于消费者，两个 bundle 之间会有选择偏好，这就是 preference。对于两个 bundle $x$ 和 $y$，他们之间有三种关系，（由于找不到相关的经济学符号，因此用数学符号来代替）

- strict preference：$x$ is more preferred than $y$, denoted as $x>y$

- weak preference：$x$ is as at least preferred as y, denoted as $x \geq y$

- indifference：$x$ is exactly as preferred as y, denoted as $x \sim y$

preference relations

- $(x \geq y) \ and\ (y\geq x)\ \rightarrow \ (x\sim y)$

- $(x \geq y)\ and \ (not\  y \geq x) \ \rightarrow \ (x > y)$

- For any two bundles $x$ and $y$, $(x \geq y)\ or \ (y \geq x)$

- Any bundle $x$ is always at least as preferred as itself, $x \geq x$

- $(x \geq y) \  and \ (y \geq x) \ \rightarrow \ (x \geq z)$


<br>
## Indifference Curve

假设一个 bundle 为 $x$，indifference curve 指的是所有可能的 bundle $y$ 使得 $y \sim x$，即 $\{y|y\sim x\}$，因此也叫 indifference set。当我们的 bundle 只考虑两种商品 $x_1$ 和 $x_2$ 时，indifference curve 为二维平面上的一条线，如下图：

{% asset_img 1.png 500 500 %}

一条曲线代表一条 indifference curve，curve 上的任意两个点 $x = (x_1,x_2)$ 和 $y = (x_1^\prime, x_2^\prime)$ 都满足 $x\sim y$，而不同的两条线上的点必定满足 strict preference 的关系，越往右上方的点，越会被 preferred，因此 $l_1$ 上的点都 strict preferred 于 $l_2$ 上的点。

*Note：*任意两条 indifference curve 必定是无交点的

### Two Extreme Cases of Indifference Curve

#### Perfect Substitute

同样是考虑两种商品组成的 bundle，如果消费者认为商品一和商品二是等同的，那么这两个商品就是 perfect  substitute，由于两者等同，因此在比较不同 bundle 哪个更加被 preferred 的时候，只需考虑两种商品的总数量即可。indifference curve 如下，不同的 curve 的斜率均为 -1。

{% asset_img 2.png 500 500 %}

#### Perfect Complement

商品一和商品二总是以固定的比例出现，比如对于一辆小汽车，总是 4 个车轮和 1 个发动机，因此商品一和商品二的比例总是 4:1，即使一个 bundle 里面有再多的车轮或者再多的发动机，我们也只能考虑能够成对的数量，作为是否被 preferred 的依据 (only the number of pairs of units of the two commodities determines the preference rank-order of bundles)

下面的例子中，商品一和商品二总是以 1:1 的比例出现，因此 (9,5)、(5,5) 和 (5,9) 都是 equally preferred，因为他们都有 5 pairs，而 (9,9) 则 strict preferred 于前面的三个点，因为它有 9 pairs，indifference curve 的形状为直角。

{% asset_img 3.png 500 500 %}

### Marginal Rate-of-Substitution(MRS)

一条 indifference curve 的斜率就叫做 MRS，它所表示的意思是，我们愿意牺牲多少数量的商品一，去换取商品二，从而使得 equally preferred。对于 perfect substitute 来说，其 MRS 均为 -1，因为两者等同，因此少一个商品二，就会多一个商品一。

<br>

## Utility Function

在前面，我们一直用 preference 来衡量两个 bundle 之间的关系，而 utility function（效用函数） 就是用来代表 preference 的

- 若 $x > y$，则 $U(x) > U(y)$ （前者是 strict preference 符号，后者是数学符号）

- 若 $x \sim y$，则 $U(x) = U(y)$

因此一条 indifference curve 上的所有点，我们都可以说他们的效用是一样的。

*Note：*效用值是一个 ordinal 的概念，即只关注两个 bundle 间效用的大小关系，至于效用值具体数值并不是关键，比如 x 的效用值为 6，y 的效用值为 2，只能说明 x 比 y 更加 preferred，并不表示 x 比 y 三倍的 preferred。因此假设 x 比 y 更为 preferred，我们可以赋值给 $U(x)$ 和 $U(y)$ 任何值，只要满足 $U(x) > U(y)$ 即可。

现在我们可以将 indifference curve 和 utility function 放在同一个坐标系里面，xOy 平面依旧是 indifference curve，z 维度表示效用值。

{% asset_img 4.png 300 300 %}

indifference map：所有的 indifference curve 的集合，其等同于 utility function。

几种不同的 utility function：

- perfect substitute：$U(x_1,x_2)=x_1+x_2$

- perfect complement：$U(x_1,x_2)=min(k_1x_1,k_2x_2)$

- quasi-linear：$U(x_1,x_2)=f(x_1)+x_2$
- Cobb-Douglas：$U(x_1,x_2)=x_1^ax_2^b$，indifference curve 都是双曲线状



### Marginal Utility

The marginal utility of commodity $i$ is the rate-of-change of total utility as the quantity of commodity $i$ consumed changes.
$$
MU_i=\frac{\partial U}{\partial x_i}
$$
**Marginal Utility & Marginal Rate-of-Substitution**

考虑具体的一条 indifference curve：$U(x_1, x_2)=k$

全微分后得：
$$
\frac{\partial U}{\partial x_1}dx_1+\frac{\partial U}{\partial x_2}dx_2=0
$$
化简后得到 MRS 和 MU 的关系
$$
MRS=\frac{dx_2}{dx_1}=-\frac{\partial U/\partial x_1}{\partial U/\partial x_2}=-\frac{MU_1}{MU_2}
$$
**Monotonic transformation & MRS** 

假设 $V=f(U)$，对于新的效用函数 $V$，其对应的 MRS 为
$$
MRS=-\frac{\partial V/\partial x_1}{\partial V/\partial x_2}=-\frac{f^\prime(U) \times\partial U/\partial x_1}{f^\prime(U) \times\partial U/\partial x_2}=-\frac{\partial U/\partial x_1}{\partial U/\partial x_2}
$$
因此 MRS 在经过一个正单向变换后，并不会改变

<br>

## Optimal Choice and Demand Function

### Rational Constrained Choice

作为消费者，我们当然想选择 utility 尽可能高的 bundle，但同时我们受商品价格以及预算的约束，假设商品一的价格为 $p_1$，商品二的价格为 $p_2$，预算为 $m$，那么我们选择的 bundle 必须满足 $p_1x_1+p_2x_2\leq m$，在这个约束下，我们再去考虑 utility 最大的 bundle。

假设 $p_1=p_2=1$，$m=4$，在约束下的为 affordable bundles，然后找到一条刚好与约束线相切的 indifference curve，相交点即为 most preferred affordable bundle，记为 $(x_1^{\ast},x_2^{\ast})$，又称为 ordinary demand。

{% asset_img 5.png 300 300 %}

对于 ordinary demand，其满足两个条件：

- 预算用完：$p_1x_1^\ast+p_2x_2^\ast=m$

- $(x_1^\ast,x_2^\ast)$ 处的斜率等于 constraint 的斜率 $-p_1/p_2$

{% asset_img 6.png 300 300 %}

### 几种不同效用函数下的 ordinary demand

#### Cobb-Douglas

$U(x_1,x_2)=x_1^ax_2^b$，该效用函数的 indifference curve 为双曲线，就像上面的图，因此我们需要找到切点，用 MU 计算 MRS，让其等于约束方程的斜率，即可算到 ordinary demand 为
$$
(x_1^*,x_2^*)=(\frac{am}{(a+b)p_1},\frac{bm}{(a+b)p_2})
$$

#### Perfect Substitute

简单的通过图示法即可得到 ordinary demand，因为 perfect substitute 的 indifference curve 均为斜率 -1 的线，找到效用最大的即可。根据 $p_1$ 和 $p_2$ 的大小关系分为三种情况：

- $p_1 > p_2$，$(x_1^\ast,x_2^\ast)=(0,\frac{m}{p_2})$
- $p_1 < p_2$，$(x_1^\ast,x_2^\ast)=(\frac{m}{p_1},0)$
- $p_1 = p_2$，$(x_1^\ast,x_2^\ast)$ 为 budget constraint 上的点

{% asset_img 7.png 700 700 %}

#### Perfect Complement

同样用图示法去解，ordinary demand 必定在直角拐点处，因此可以得到 ordinary demand 为
$$
(x_1^*,x_2^*)=(\frac{m}{p_1+ap_2},\frac{am}{p_1+ap_2})
$$


{% asset_img 8.png 500 500 %}

### Demand Function

又叫 ordinary demand funtion，研究 $(x_1^\ast,x_2^\ast)$ 会随着 $p_1$、$p_2$ 和 $m$ 的变化而怎么变化，由于 $p_1$、$p_2$ 和 $m$ 会影响到 budget constraint，因此研究 demand function 可以让我们更快更直接的得到 ordinary demand。

在这里我们重点关注 own-price changes，比如在 $p_2$ 和 $m$ 不变的情况下，只改变 $p_1$，如下图，右边的即为 demand function，描述 $x_1^\ast$ 和 $p_1$ 之间的关系；

左边绿色的线是所有 ordinary demand 的连线，p1 price offer curve 指的是固定 $p_2$ 和 $m$， 只改变 $p_1$ 下的 ordinary demand 的变化线。因此可以从每个点中，算出对应的 $p_1$。
{% asset_img 9.png 500 500 %}


<br>
## Market Demand & Elasticity

市场的 demand function 等于市场中每个消费者各自的 demand function 的和，即整个市场的需求量等于各个个体需求量之和
$$
X_j(p_1,p_2,m^1,...,m^n)=\sum_{i=1}^nx_j^{*i}(p_1,p_2,m^i)
$$
弹性的一般定义如下，研究的是一个变量 x 会随着另一个变量 y 如何变化
$$
\epsilon_{x,y}=\frac{\%dx}{\%dy}=\frac{dx/x}{dy/y}
$$

**Own-Price Elasticity：**Own-Price Elasticity 指的是价格 $p$ 和需求 $x^\ast$ 之间的弹性，它是一个点概念，一般我们不说某一条 demand function 的弹性，而说 demand function 的某一点的弹性，因为市场中我们一般处在 demand function 中的某一点，我们需要关注的是在这一点的基础上，增加或减少价格所带来需求的变化。其定义如下
$$
\epsilon_{X_i^*,p_i}=\frac{p_i}{X_i^*}\times\frac{dX_i^*}{dp_i}
$$
demand function 中每个点的弹性可能都不一样，下面的例子中，根据弹性的大小分为三类：

- $-\infty < \epsilon < -1$，own-price elastic
- $\epsilon=-1$，own-price unit elastic
- $-1 < \epsilon < 0$，own-price inelastic

{% asset_img 10.png 500 500 %}



**revenue & own-price elasticity**

销售者的 revenue 为：$R(p)=p \times X^*(p)$

revenue 随着价格 $p$ 的变化率为：
$$
\frac{dR}{dp}=X^*(p)+p\frac{dX^*}{xp}=X^*(p)(1+\frac{p}{X^*(p)}\frac{dX^*}{dp})=X^*(p)(1+\epsilon)
$$
因此：

- $-\infty < \epsilon < -1$，$\frac{dR}{dq}<0$，升价会导致销售者收入减少
- $\epsilon=-1$，$\frac{dR}{dq}=0$，升价不会影响销售者收入
- $-1 < \epsilon < 0$，$\frac{dR}{dq}>0$，升价会导致销售者收入增加

我们考虑需求函数的反函数 $p(q)$，其代表当销售者卖出 $q$ 件商品时的价格 $p$，因此 revenue 可以表示为 $R(q) = p(q)\times q$

同理 mariginal revenue 为：
$$
MR(q)=\frac{dp(q)}{dq}q+p(q)=p(q)(q+\frac{q}{p(q)}\frac{dp(q)}{dq})=p(q)(1+\frac{1}{\epsilon})
$$
因此：

- $-\infty < \epsilon < -1$，$MR(q)>0$，需求增多会提高收入
- $\epsilon=-1$，$MR(q)=0$，需求增多不会影响收入
- $-1 < \epsilon < 0$，$MR(q)<0$，需求增多会减少收入

**Note：**$\frac{dR}{dp}$ 不能称为 marginal revenue，所有边界的概念都是指增加一单位的物品所带来的另外变量的变化，因此 $dp(q)/dq$ 才是 marginal revenue，而 $dR/dp$ 只能叫做 revenue 随着价格的变化率。

两种表达方式所表述的东西是一样的，假设 $\epsilon < -1$，那么增加一单位的价格带来的需求的减少量必定超过一单位，因此收入减少；而增加一单位的商品，价格的减少量是小于一单位的，因此收入增多。


<br>
## Consumer Surplus

对于 demand curve $p_1=v^\prime(x_1)$，当消费者计划消费 $x_1^\prime$ 个商品一的时候，其消费者剩余为
$$
CS=\int_0^{x_1^\prime}v^\prime(x_1)dx_1-p_1^\prime x_1^\prime
$$
{% asset_img 11.png 300 300 %}

消费者剩余指的是：消费者本身对商品有一个预期的价格，而市场价格比预期价格要低，因此出现了剩余，从公式中可以看出，第二项为实际支付的钱，而第一项则为需求在 $x_1^\prime$ 下的该消费者的平均预期所需要支付的钱，我们可以将积分项拆开
$$
\int_0^{x_1^\prime}v^\prime(x_1)dx_1=lim_{n\rightarrow \infty}\sum_{i=1}^nv^\prime(x_1)\frac{x_1^\prime}{n}=lim_{n\rightarrow \infty}\frac{1}{n}\sum_{i=1}^nv^\prime(x_1)x_1^\prime
$$
$v^\prime(x_1)x_1^\prime$ 为价格为 $v^\prime(x_1)$ 下购买 $x_1^\prime$ 个商品所需要支付的钱，将不同价格下所需要支付的钱进行一个平均，即为该消费者平均预期所需要支付的钱。

<br>