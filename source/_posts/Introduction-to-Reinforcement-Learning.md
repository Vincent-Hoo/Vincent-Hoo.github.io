---
title: Introduction to Reinforcement Learning
mathjax: true
date: 2019-08-24 16:22:19
tags:
---



> 课程来源：李宏毅2017机器学习课程
> 课程主页：http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML17.html
> 课程梗概：简要介绍强化学习的基本概念，以及强化学习两种基本的训练方法：policy-based approach 和 value-based approach。



<!--more-->



*preface*

AlphaGo 的成功使得强化学习进入了研究者的视野，强化学习是不同于监督学习和无监督学习之外的另外一种学习方式，它通过不断地探索，从经验中得到知识，从而使得模型的性能逐步提升。

<br>

## Basic Idea of Reinforcement Learning

强化学习有几个重要的组成部分：

- Agent
- Observation(state)
- Action
- Environment
- Reward

Agent 可以理解为整个模型，agent 看到 environment 中的一个 state，可以当作是一次 observation，之后做出反应，此为 action，该反应反馈到 environment 后，agent 会得到回馈 reward；接着 agent 又得到新的 observation，依次类推，直到终止条件。

举个简单的例子，agent 观察到一杯水，于是将其打翻，得到负面的反馈；下一个迭代，agent 观察到打翻掉的水，于是将其收拾好，此时就会得到正面的反馈。依次下去，agent 就会学到不可以打翻水。

{% asset_img 1.png 500 500 %}

{% asset_img 2.png 500 500 %}



**监督学习和强化学习的不同**

- 监督学习可以理解为学生向老师学习，老师代表的是 label，是永远的正确答案，拿围棋举例，监督学习告诉学生，当棋盘为 A 这样的时候，应该走这一步；当棋盘为 B 这样的时候，应该走那一步，但对于围棋以及电玩这种应用，显然不可能枚举所有的情况，也不可能有唯一的一个正确选项，这就催生了强化学习。
- 强化学习可以理解为从经验中学习，学生通过自己的探索，得出获胜的方法，而这其中有另外一个老师的角色，而这个老师只告诉学生，这样做好不好，即 reward。因此学生探索得越多（训练越多），经验越丰富，模型也就越好。



**强化学习的目标函数**

下面是 space invader 的游戏，黄色的为 alien，下面绿色的为太空船，左上角的为分数，游戏的目的是杀死所有的 alien。

先介绍一个概念 episode，它是 agent 完整的一次游戏（探索）过程，包括多个观察 s 以及动作 a。强化学习的目标函数是要最大化一次 episode 的累积 reward，而不是每一个 action 的 reward，假如目标函数是优化每一个 action 后的 reward，那么 agent 就会学到只射击外星人，而不会学到向左向右移动，因为只有射击才有 reward。

{% asset_img 3.png 600 600 %}

<br>

## 强化学习训练方法

主要分为两种方法：

- policy-based approach，学习一个 actor，即直接学习 agent
- value-based approach，学习一个 critic，评判 agent 的好坏

当然也有两者结合的方法。



### Policy-based Approach——Learning an Actor

actor 的输入为 observation，输出为可能 action 的概率，actor 记为 $\pi_{\theta}(s)$

{% asset_img 4.png 600 600 %}

上面说到强化学习的目标函数为一次 episode 的总 reward，但是由于随机性，即使是同样的 actor，同样的输入，得到的 episode 也会不一样，因此定义总 reward 的期望作为目标函数。

{% asset_img 5.png 600 600 %}

一次 episode 可以看成是一次 trajectory $\tau=\{s_1,a_1,r_1,s_2,a_2,r_2,...,s_T,a_T,r_T\}$，trajectory 的总 reward 记为 $R(\tau)=\sum_{t=1}^Tr_t$，固定 actor $\pi_{\theta}(s)$，得到某一次 trajectory 的概率为 $P(\tau|\theta)$，$\bar{R_{\theta}}$ 可以近似为 N 次 trajectory reward 的平均。

{% asset_img 6.png 500 500 %}

我们的目的是要最大化 $\bar{R_{\theta}}$，因此只要对 $\bar{R_{\theta}}$ 求梯度，再进行梯度上升更新 $\pi_{\theta}(\cdot)$ 即可，下面看一下梯度的计算过程。
$$
\nabla \bar{R_{\theta}}=\sum_{\tau}R(\tau)\nabla P(\tau|\theta)=\sum_{\tau}R(\tau)P(\tau|\theta)\frac{\nabla P(\tau|\theta)}{P(\tau|\theta)}
$$

$$
=\sum_\tau R(\tau)P(\tau|\theta) \nabla logP(\tau|\theta)
$$

$$
\approx \frac{1}{N}\sum_{n=1}^{N} R(\tau^n)\nabla logP(\tau^n|\theta)
$$

假设 $\tau=\{s_1,a_1,r_1,s_2,a_2,r_2,...,s_T,a_T,r_T\}$，那么 $\nabla logP(\tau|\theta)$ 是多少呢？
$$
P(\tau|\theta)=p(s_1)p(a_1|s_1,\theta)p(r_1,s_2|s_1,a_1)p(a_2|s_2,\theta)p(r_2,s_3|s_2,a_2)...
$$

$$
=p(s_1)\prod_{t=1}^{T}p(a_t|s_t,\theta)p(r_t,s_{t+1}|s_t,a_t)
$$

$$
\approx \prod_{t=1}^Tp(a_t|s_t,\theta)
$$

因此
$$
\nabla logP(\tau|\theta)=\sum_{t=1}^T\nabla logp(a_t|s_t,\theta)
$$
所以，一次迭代的总梯度为
$$
\nabla \bar{R_{\theta}} \approx \frac{1}{N}\sum_{n=1}^NR(\tau^n)\nabla logP(t^n|\theta)=\frac{1}{N}\sum_{n=1}^NR(\tau^n)\sum_{t=1}^{T_n}\nabla logp(a_t^n|s_t^n,\theta)
$$

$$
=\frac{1}{N}\sum_{n=1}^N\sum_{t=1}^{T_n}R(\tau^n)\nabla logp(a_t^n|s_t^n,\theta)
$$

前面提到，我们目标函数考虑的是累计的 reward，而不是某个 action 后的瞬间 reward，我们假设在 $\tau^n$ 中，机器看到某个状态 $s_t^n$ 后采取了 $a_t^n$，假如整体的 $R(\tau^n)$ 为正，那么模型就会调整 $\theta$ 从而使得 $p(a_t^n|s_t^n,\theta)$ 增大，反之亦然。

我们将强化学习的梯度和普通分类问题的梯度进行对比

- 强化学习：$\frac{1}{N}\sum_{n=1}^N\sum_{t=1}^{T_n}R(\tau^n)\nabla logp(a_t^n|s_t^n,\theta)$
- 分类问题：$\frac{1}{N}\sum_{n=1}^N\sum_{t=1}^{T_n}yi\nabla logp(a_t^n|s_t^n,\theta)$

其实对于 actor 来说，它就是一个分类器，给定一个 state，输出各个 action 的概率，而强化学习的目标函数只是在各个概率前，用 $R(\tau^n)$ 作为其权重而已。

### Value-based Approach——Learning a Critic

Critic 不会用来决定一个 action，但是给定一个 actor，它可以判断它的好坏，因此 actor 可以从 critic 中找出来，经典的算法就是 Q-learning。

State value function $V^\pi(s)$，输入一个 state，输出 cumulated reward，该函数取决于 actor $\pi$，当 actor 较强，输入一个 state，得到的 reward 可能比较大；反之 actor 较弱，reward 则较小。

#### How to estimate $V^\pi(s)$

**Monte-Carlo based approach**

critic 让 actor 自己玩游戏，当输入状态 s 后，观察 actor 能得到多少 reward。

{% asset_img 7.png 500 500 %}

**Temporal-difference approach**

有些应用的 episode 很长，或者甚至不会停止，那么我们只能用中间的一小部分来进行训练，假设只有一长串 episode 的一部分 $\{...,s_t,a_t,r_t,s_{t+1},...\}$，易证得：$V^\pi(s_t)-r_t=V^\pi(s_{t+1})$，

{% asset_img 8.png 500 500 %}



<br>