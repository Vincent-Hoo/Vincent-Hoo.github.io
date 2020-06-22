---
title: 【高级管理学】—— producer theory
mathjax: true
date: 2019-10-16 12:23:48
tags:
---



> 课程名称：高级管理学
> 授课人：Ying Kong
> 本章梗概：本章讲述生产者理论，从利润最大化，成本最小化角度进入，并介绍纯粹竞争以及垄断者模型，最后介绍博弈论模型。

<!--more-->

## Production Function

生产者理论和消费者理论很接近，下表总结两者的异同

|                          生产者理论                          |                          消费者理论                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| production function: $y = f(x_1, ...,x_n)$，，$x_i$ 为 input level，即每个输入单位数，$y$ 为输出的单位数 |             utility function：$U=f(x_1,...,x_n)$             |
|   y output unit isoquant：生产单位为 $y$ 的各种 input 组合   |          indifference curve：效用为 U 的各种 bundle          |
|          perfect substitute：$y=a_1x_1+...+a_nx_n$           |               perfect substitute：$U=x_1+x_2$                |
|        perfect complement：$y=min(a_1x_1,...,a_nx_n)$        |          perfect complement：$U=min(k_1x_1,k_2x_2)$          |
| marginal product：增加一个单位的 $x_i$ 所带来的输出单位数的增加，$MP_i=\frac{\partial P}{\partial x_i}$ | marginal utility：增加一个单位的 $x_i$ 所带来的效用的增加，$MU_i = \frac{\partial U}{\partial x_i}$ |
| technical rate-of-substitution：isoquant 某个点的斜率，$TRS=-\frac{MP_1}{MP_2}$ | marginal rate-of-substitution(MRS)：indifference curve 某个点的斜率，$MRS=-\frac{MU_1}{MU_2}$ |
| long run：生产函数中每一个输入 $x_i$ 都可变，即生产过程不受任何输入约束；short run：生产过程至少一个输入变量为常数。 |                                                              |

<br>

## Profit and Cost theory

### Profit Maximization

生产者在生产过程需要注意的几个变量：

- 输入原料：$x_1,...,x_m$
- 输出产品：$y_1,...,y_n$
- 输入原料价格：$w_1,...,w_m$
- 输出产品价格：$p_1,...,p_n$

生产者利润 profit 的定义：
$$
\Pi = p_1y_1 + ...+ p_ny_n - w_1x_1 - ... - w_mx_m
$$
假设现在输入只有 $x_1$ 和 $x_2$，输出只有一个 $y$，并且输入为 long run，需要求利润 $\Pi$ 的最大值。问题中，变量为 $x_1, x_2,y$，常量为 $p,w_1,w_2$，利润为 $\Pi=py-w_1x_1-w_2x_2$，而 $y = f(x_1,x_2)$，因此变量减少为两个，分别对 $x_1,x_2$ 求偏导，并令其为 0，则可求得最大值。 
$$
\frac{\partial \Pi}{\partial x_1}=p \times \frac{\partial y}{\partial x_1}-w_1=p \times MP_1 - w_1 = 0
$$

$$
\frac{\partial \Pi}{\partial x_2}=p \times MP_2 - w_2 = 0
$$

可得
$$
p\times MP_1=w_1, \ p\times MP_2=w_2
$$
$p\times MP$ 为 marginal revenue，因此 profit 最大的时候，marginal revenue with respect to $x_i$ 等于 marginal cost with respect to $x_i$，这里的 marginal cost with respect to $x_i$ 代表的是增加一单位的输入$x_i$ 所增加的 cost，那明显是等于输入单价 $w_i$



### Cost Minimization

当生产者需要生产 $y$ （已知常数）个产品时，应该以怎样的输入组合才会实现花费最少？这个问题的解法思路和消费者理论中寻求在 budget 约束下的最大效用一样。

画出 y output unit isoquant，$cost=w_1x_1+w_2x_2$，这个问题就是寻找一个跟曲线相切的点，这个点即为成本最低的能生产 y 个产品的输入 bundle。

{% asset_img 1.png 500 500 %}



### Cost Function

我们研究一个企业的开销成本，只考察 cost 和 output level $y$ 的关系，不考察其和 input level $x_i$ 的关系，因此 cost function 定义为 $c(y)$，其可以表示为以下形式
$$
c(y)=F+c_v(y)
$$

- $F$ 为生产过程中固定的费用，无论生产多少个产品都固定的费用

- $c_v(y)$ 为 variable cost function，它随着 input level 而改变

{% asset_img 2.png 500 500 %}

将 output level 加入 cost function 可得 averaged cost function(AC 或 ATC)
$$
AC(y)=\frac{c(y)}{y}=\frac{F}{y}+\frac{c_v(y)}{y}=AFC(y)+AVC(y)
$$
随着 y 不断增大，AFC 会趋近于 0，AVC 趋近 AC

{% asset_img 3.png 500 500 %}



### marginal cost function

这里的边界成本指的时 output level 变化时所增加的 cost，而上面提到的 marginal cost with respect to $x_i$，是某个输入发生变化时的带来 cost 的变化，其为一个恒定的值，等于 $w_i$，与这里提到的 marginal cost 不同，marginal cost 的定义如下
$$
MC(y)=\frac{\partial c(y)}{\partial y}=\frac{\partial c_v(y)}{\partial y}
$$
由于 $MC(y)$ 为 $c_v(y)$ 的导数，那么 $c_v(y)$ 则为 $MC(y)$ 的不定积分
$$
c_v(y^\prime)=\int_0^{y^\prime}MC(z)dz
$$
**MC 和 AVC 的关系**
$$
\frac{\partial AVC(y)}{\partial y}=\frac{y\times MC(y)-c_v(y)}{y^2}
$$

- $\frac{\partial AVC(y)}{\partial y}>0$，$MC(y)>\frac{c_v(y)}{y}=AVC(y)$

- $\frac{\partial AVC(y)}{\partial y}<0$，$MC(y)<\frac{c_v(y)}{y}=AVC(y)$
- $\frac{\partial AVC(y)}{\partial y}=0$，$MC(y)=\frac{c_v(y)}{y}=AVC(y)$

即 AVC 的最低点是其和 MC 的交点，当 AVC 上升的时候，MC 比它大，当 AVC 下降的时候，MC 比它小



**MC 和 ATC 的关系**
$$
\frac{\partial ATC(y)}{\partial y}=\frac{y\times MC(y)-c(y)}{y^2}
$$

- $\frac{\partial ATC(y)}{\partial y}>0$，$MC(y)>\frac{c(y)}{y}=ATC(y)$

- $\frac{\partial ATC(y)}{\partial y}<0$，$MC(y)<\frac{c(y)}{y}=ATC(y)$
- $\frac{\partial ATC(y)}{\partial y}=0$，$MC(y)=\frac{c(y)}{y}=ATC(y)$

同样的结论，即 ATC 的最低点是其和 MC 的交点，当 ATC 上升的时候，MC 比它大，当 ATC 下降的时候，MC 比它小

{% asset_img 4.png 500 500 %}

<br>
## Competition theory
### Pure Competition

> Also regarded as perfect competition, pure competition is a situation in which the market for a product is populated with so many consumers and producers that no one entity has the ability to influence the price of the product sufficiently to cause a fluctuation. Within this type of market setting, sellers are considered to be price takers, indicating that they are not in a position to set the price for their products outside a certain range, given the fact that so many other producers are active within the market. At the same time, consumers have little influence over the prices offered by the producers, since there is no singular group of consumers that dominates the demand. 

一个企业对某样产品的市场价格没有影响，而这个产品的市场价格由市场上所有的公司共同决定，其为市场需求和市场供给的交点，如果一个企业将产品价格设高于市场价格，那么它的需求量为 0，因为没有人会买；而如果将产品价格设低于市场价格，那么该企业的需求量为市场的总需求量。

{% asset_img 5.png 500 500 %}

因此我们可以得到一个企业对于某个产品的需求曲线如下，它是一条平行于 x 轴的直线，高度等于市场价格，意味着企业如果定价等于市场价格，它的需求是没有限制的，而如果价格高于市场价，需求为0。

{% asset_img 6.png 500 500 %}



对于给定的市场价格 $p$，一个企业应该如何决定生产量 $y$ 使得利润最大化呢，问题定义为
$$
max_{y\ge0}\Pi_s(y)=py-c_S(y)
$$
对于利润 $\Pi_S(y)$，唯一自变量为 $y$，求利润最大值需要解下面两个条件

- 一阶导数为 0，得出极值点，$\frac{d\Pi_S(y)}{dy}=p-MC_S(y)=0$

- 二阶导数小于 0，得出极大值点，$\frac{d^2\Pi_S(y)}{dy^2} = -\frac{dMC_S(y)}{dy}<0$

从上面条件，我们可以得出利润最大化的 $y_S^\ast$ 应该满足

- $MC_S(y)$ 与 p 的交点
- 交点处于 $MC_S(y)$ 的上升阶段



上面的步骤只是求得利润的最大值，但并非最大值一定是正的，也可能为负值，当为负值的时候，证明企业在亏损，我们可以比较在交点处的 AC 值和 p 的大小关系，来判断取到最大值时，利润是否为正
$$
\Pi_S(y)=p(y-\frac{C_S(y)}{y})=p(y-AC(y))
$$
若 $AC(y_S^\ast) \le p$，则不亏， 若 $AC(y_S^\ast)=p$，我们称其为 breakeven price，保本价格，因为无亏损也无盈利，若 $AC(y_S^\ast)>p$，则利润为负，再把成本拆开
$$
\Pi_S(y)=p(y-\frac{C_v(y)}{y})-F=p(y-AVC(y))-F
$$
若 $AVC(y_S^\ast)<p<AC(y_S^\ast)$，则企业勉强可以生存，虽然依旧亏损，但是起码能支付得起 $c_v(y)$

若 $AVC(y_S^\ast)>p$，则企业需要关闭，称 $AVC(y_S^\ast)=p$ 的点为 breakdown price

<br>

### Producer surplus & Profit

$PS=p\times y^\ast(p)-\int_0^{y^\ast(p)}MC_S(y)=revenue-c_v(y^\ast(p))$

$\Pi=revenue-c(y^\ast(p))=revenue-c_v(y^\ast(p))-F$

$PS=Profit+F$

{% asset_img 7.png 500 500 %}

<br>

### Long-Run Market Equilibrium Price

假设 market demand 曲线固定不变，而 market supply 由市场中的企业所决定，假设一开始只有两个企业，那么市场价格为两条曲线的交点 $p_2$，在 $p_2$ 下企业会盈利，就会导致有第三个企业进入市场，这时候 market supply 曲线发生改变，市场价格 $p_3$ 也因此降低，但此时企业依旧盈利，第四个企业进入，市场价格为 $p_4$，此时企业亏损，有企业会离开市场。

{% asset_img 8.png %}

以此类推，当市场中的企业足够的多和小的话，那么 market supply 曲线将会是一条水平线，水平线的高度为 $min\ AC(y)$，所以 long run market price $p^e=min\ AC(y)$

{% asset_img 9.png 300 300 %}

<br>


## Monopoly theory
### Pure monopoly

> A market in which one company has control over the entire market for a product, usually because of a barrier to entry such as a technology only available to that company.

pure competition 中的企业都是 market price taker，而pure monopoly 的市场价格完全由垄断者所决定，因此垄断者的利润定义为
$$
\Pi(y)=p(y)y-c(y)
$$
价格由自己的 demand function 所决定

{% asset_img 10.png 500 500 %}

要使得利润 $\Pi(y)$ 最大，求其对 output level $y$ 的偏导，利润最大值下的 output level $y^\ast$ 满足 marginal revenue 等于 marginal cost。
$$
\frac{d \Pi(y)}{dy}=\frac{d}{dy}(p(y)y)-\frac{dc(y)}{dy}=MR(y)-MC(y)=0
$$
例子：

- $p(y)=a-by$，$R(y)=p(y)y=ay-by^2$，$MR(y)=a-2by$

- $c(y)=F+\alpha y+\beta y^2$，$MC(y)=\alpha+2\beta y$

{% asset_img 11.png 600 600 %}

<br>

### Monopolistic Pricing & Own-Price Elasticity


$$
MR(y)=\frac{d}{dy}p(y)y=p(y)+y\frac{dp(y)}{dy}=p(y)(1+\frac{1}{\epsilon})
$$

假设 $MC(y) = k$，$k$ 为一个常数，那么
$$
p(y^\ast)=\frac{k}{1+\frac{1}{\epsilon}}
$$
当 $\epsilon$ 往 -1 增大时，垄断者会调整 output level 使得市场价格上升，并且垄断者为了使得自己的 marginal revenue 为正值，会选择弹性商品，即 $\epsilon < -1$

**markup pricing：**市场价格和成本之差
$$
markup = p(y^\ast)-MC(y^\ast)=\frac{k}{1+\frac{1}{\epsilon}}-k=-\frac{k}{1+\epsilon}
$$
所以当 $\epsilon$ 往 -1 增大时，markup price 也随之增大

<br>

## Two-player Game Theory

A game consists of

- a set of players
- a set of strategies for each player
- the payoffs to each player for every possible choice of strategies by the players. Payoff matrix is always used to represent the relationship.

我们只研究两个 player 且每个 player 两种决策的博弈论

### Pure Strategy

每个 player 只能从自己的决策选项中选择一个，考虑下面的例子

{% asset_img 12.png 500 500 %}

**Nash Equilibrium(NE)：**A play of the game where each strategy is a best reply to the other.

上面例子中 (U,L) 和 (D,R) 都是 NE，比如当 player A 选择 U 的时候，player B 肯定会选择 L，反过来一样。

一个 game 可能会有多个 NE，上面的例子，player A 和 B 是同时做选择的，称为 simultaneous play games，我们很难判断这两个 NE 哪个较好；而 sequential play games 通常是先后做选择，先做选择的为 leader，后作选择的为 follower，当选择有先后时，就比较容易判断哪个 NE 较好。下面的图表示如果 A 先做选择，(U,L) 会较好。

{% asset_img 13.png 500 500 %}



### Mixed Strategy

有些 game 如果只用 pure strategy 的话，可能得不到 NE，即无论 player A 怎么选择，player B 总能选出更好的，使得 player A 接着改变选择。而 Mixed strategy 下，player 可以以某个概率去选择某个 strategy，使得整体的 expected payoff 达到 NE。

下面的例子没有 pure strategy，但是有 Mixed strategy

{% asset_img 14.png 500 500 %}

对于有限个玩家且玩家的决策有限的情况，至少会有一个 NE，如果没有 pure strategy NE，那么肯定有 mixed strategy NE。

<br>

## Best Responses & Nash Equilibrium

上面用 payoff matrix 来找 NE，下面将介绍 best response 的方法来寻找 NE。

### Pure strategy

假设 player A 的两个 action 分别为 $a_1^A$ 和 $a_2^A$，player B 的两个 action 分别为 $a_1^B$ 和 $a_2^B$，payoff matrix 写成类似效用函数的形式

- $U^A(a_1^A,a_1^B)=6$ and $U^B(a_1^A,a_1^B)=4$
- $U^A(a_1^A,a_2^B)=3$ and $U^B(a_1^A,a_2^B)=5$
- $U^A(a_2^A,a_1^B)=4$ and $U^B(a_2^A,a_1^B)=3$
- $U^A(a_2^A,a_2^B)=5$ and $U^B(a_2^A,a_2^B)=7$

如果 A 选择 $a_1^A$，B 的 best response 为 $a_2^B$；如果 A 选择 $a_2^A$，B 的 best response 为 $a_2^B$。

如果 B 选择 $a_1^B$，A 的 best response 为 $a_1^A$；如果 B 选择 $a_2^B$，B 的 best response 为 $a_2^A$。

第一幅图为 B 选择变化时，A 的 best response；第二幅图为 A 选择变化时，B 的 best response，注意横纵坐标不同，将两幅图放在一起，交点即为 NE，即 $(a_2^A,a_2^B)$

{% asset_img 15.png %}



### Mixed Strategy

假设 payoff matrix 如下

{% asset_img 16.png 500 500 %}

$\pi^A_1$ 为 A 选择 $a_1^A$ 的概率，$1-\pi^A_1$ 为 A 选择 $a_2^A$ 的概率；$\pi^B_1$ 为 B 选择 $a_1^B$ 的概率，$1-\pi^B_1$ 为 B 选择 $a_2^B$ 的概率。假设给定 $\pi_1^B$，我们可以得到 A 选择不同 action 下的平均 payoff

{% asset_img 17.png 400 400 %}

当给定 $\pi_1^B$ 时，A 的 best response 为

{% asset_img 18.png 400 400 %}

对应的 best response curve 为

{% asset_img 19.png 400 400 %}

同理，给定 $\pi_1^A$ 时，B 的情况

{% asset_img 20.png 400 400 %}

B 的 best response curve 为

{% asset_img 21.png 400 400 %}

找两个曲线的交点，即为 NE，有三个 NE，两个为 pure strategy NE，一个为 mixed strategy NE。

{% asset_img 22.png %}





<br>

