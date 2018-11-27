---
title: Introduction to Generative Adversarial Network
date: 2018-11-21 18:55:27
tags:
mathjax: true
---


> 课程来源：李宏毅2018生成对抗网络课程
> 课程主页：http://speech.ee.ntu.edu.tw/~tlkagk/courses_MLDS18.html
> 文章梗概：该文章分为四个部分：GAN basics, GAN as structured learning, can generator learn by itself, can discriminator generator.

<!-- more -->

<br>
*preface*

> Yann LeCun said adversarial training is the coolest thing since sliced bread.
> He also said GAN and its variation are the most interesting idea in the last 10 years in ML.

<br>

# Basic Idea of GAN

## Generator

GAN是用来生成东西的，如图片、文章等，因此GAN会有一个生成器generator，当输入一个随机的向量的时候，就会输出你想要的图片或文章。

{% asset_img 1.png 550 550 %}



上面所说的generator实际上就是一个神经网络，网络的输入是一个随机向量，输出是一个高维向量（图片），输入向量的每一个维度可能会代表不同的特征，改变不同维度的数值，会导致生成的图片的某一个特征的改变。

{% asset_img 2.png 500 500 %}

## Discriminator

GAN中另外一个重要的成分就是判别器discriminator，它用来判断生成的图片的质量，或者用来判断生成的图片是否符合要求。假设我们要生成二次元图片，那么判别器的输入就是一张图片，输出是一个score，score越大代表生成的图片越像二次元图片。

{% asset_img 3.png 550 550 %}

## Generator和Discriminator的关系

在Ian Goodfellow最开始的论文里面，他用假钞制造者和警察来类比generator和discriminator，假钞制造者想要制造出假钞来蒙骗警察，而警察是假钞的鉴定者，因此generator和discriminator之间互相对抗（这也是GAN叫adversarial的原因），并且不断地进化，到最后generator能制造出跟真钞一模一样的钞票，而discriminator也无法判断出真钞还是假钞。下图给出的是image generation的例子，第一代的generator可能生成的图片啥都不是，然后第二代稍微好一点，到最后就能生成二次元图片。

{% asset_img 4.png 550 550 %}

用另外一个视角去看GAN，我们可以把generator看作是学生，而discriminator看作是老师，老师看过很多的二次元图片，知道什么样的图片是好的，而学生学习画二次元图片，老师看学生画的图片，然后**反馈**给学生，让学生画出更好的画。

因此generator和discriminator亦敌亦友

{% asset_img 5.png 550 550 %}



## 训练流程

GAN是由两个network组成的，一个是generator，一个是discriminator，两个network需要单独训练，训练流程如下

1. 初始化generator和discriminator的参数
2. 固定住generator的参数，更新discriminator，discriminator是用来判断生成图片的好坏的，首先我们有一个真实的二次元图片dataset，discriminator实际上是一个二分类器，真实的二次元图片是class 1，生成的图片是class 0，discriminator需要给真实图片高分，而给生成的图片低分

    {% asset_img 6.png 550 550 %}

3. 固定住discriminator的参数，更新generator，generator的目标是生成二次元图片，由于discriminator对于生成的图片一般会给低分，而对于真实图片会给高分，因此我们需要训练generator使得它生成的图片尽可能地能够拿到高分，即蒙骗discriminator

    {% asset_img 7.png 550 550 %}

具体的算法流程如下

{% asset_img 8.png 550 550 %}

简单分析一下两个network的loss function，discriminator实际上是一个二分类器，因此loss function就是cross entropy，discriminator希望真实图片的score尽可能大，而生成图片的score尽可能小；而generator则是希望自己生成的图片被discriminator打的分越高越好。


<br>
# GAN as Structured Learning

machine learning就是需要找到一个函数能够把对应的X投影到Y上，分类如下

- classification：输出一个类别
- regression：输出一个scalar
- structured learning：输出一个sequence，matrix等，如机器翻译，语音识别，text-to-image等

structured learning难点在于

1. structured learning可以看作是one-shot learning，one-shot learning指的就是每一个样本只会出现一次，而在传统的分类问题，一个标签往往会对应很多个样本，而structured learning中如果把每一个输出都看作是一个类别的话，可能就会出现很多的类别，比如机器翻译中，一个Y可能就是一个class，而每个Y就可能只会出现一次，testset中的Y在trainset中更是没有出现。
2. output space会非常大，output space中的绝大多数的类别，更是没有X对应，这里举机器翻译的例子，假如是英译中，output space就是全部可能的中文句子，你不可能找到一个dataset，有所有中文句子的英文翻译。
3. 由于上面的两点，structured learning必须要学会创造新的东西。因为在test中要输出的正确答案，可能是在训练中一次都没见过的。
4. 由于structured learning中机器需要学会创造，因此机器需要学会规划，要有大局观。

而GAN的generator就是用bottom-up的方法学着component-by-component地生成object，而discriminator就是用top-down的方法，整体地评估这个object。


<br>
# 两个疑问

我们上面提到，GAN实际是由两个network组成的，一个是generator，一个是discriminator，generator学习生成object，discriminator给出评价，两者缺一不可。两个疑问是：generator能不能自己根据真实的data学习生成object？discriminator既然那么会评价，那么它能不能生成object？

## Can generator learn by itself?

generator是一个输入随机向量，输出object的网络，如果需要学习出这样一个网络，按照监督学习的方法，我们首先需要构造数据集，随机生成X向量对应一张图片Y，但是这里面的问题在于如何去生成每一个图片Y所对应的随机向量X，我们在随机生成向量的时候，很有可能两张非常类似的图片对应了两个非常不一样的X，这样会使得network很难收敛，因为network无法满足让两个非常不一样的X，输出非常接近的Y，而上面提到GAN的generator的输入向量在某种程度上，每个维度是对应某一种特征的，类似于word embedding，因此我们不能随机地生成输入向量X。

由于输入向量在某种程度上表征着生成的图像，因此我们可以用encoder来生成每个图片的code，在训练auto-encoder的时候，我们不难发现，decoder实际上就是我们所要的generator，decoder接收一个code作为输入，然后输出code所对应的图片，这时候我们就能随机生成一些code，然后用decoder来生成图片。

{% asset_img 9.png 550 550 %}

但是auto-encoder训练出来的decoder可能不够鲁棒，因此用VAE(variational auto-encoder)来训练，使得decoder能够允许一定程度上的noise

{% asset_img 10.png 550 550 %}



似乎VAE训练的decoder能够作为generator，然后生成图片，不需要discriminator， 然而我们在训练auto-encoder的时候，我们是希望decoder的输出和输入的图片越接近越好，而一般评判接近与否是用两个图片的Euclidean distance来判断，但是这种评判标准是否合理呢？比如下面的例子，auto-encoder会更加倾向于前两个输出，因为整体的error较少，但是不符合我们的要求，这也意味着auto-encoder的generator它希望做到的是copy，它并不能学到精髓，这也意味着我们需要更加复杂的评判规则，去衡量生成的图片是否和原图片一样好，这也就是discriminator存在的必要性了。

{% asset_img 11.png 550 550 %}

由于auto-encoder是一个fully-connected network，它很难捕获到neuron之间的关系，neuron之间都是相互独立的，但是在生成图片的这件事上，component之间的correlation很重要，而一般network的架构很难将component之间的关系考虑进去，因此就要更大的网络。比如下面的例子中，Layer L是decoder最后一层，每个neuron代表一个pixel，第二个neuron想生成一滴颜色，它想第一个neuron和它一起生成，但是它无法控制它旁边的neuron，因为他们都是相互独立的。

{% asset_img 12.png 550 550 %}

## Can discriminator generate

 相对于generator，discriminator能够从全局去判断生成的图片好与坏，可以用一个convolution layer轻松检测component之间的关系。

而假设我们有一个训练好的discriminator，它能够判断什么样的图片是好的，那么我们可以通过以下的式子让discriminator生成好的图片。从全局中搜索出能让discriminator打高分的图片。

$$\tilde{x} = arg max_{x\in X} D(x)$$ 

我们姑且不去考虑，这个搜索能不能实现，而关心discriminator应该如何训练，我们有的数据集是真实的图片，即positive example，训练一个discriminator，除了要有positive example，还需要negative example，那么我们该如何生成negative example呢？下面的演算法能够实现discriminator的训练

一开始随机生成一些noise image作为negative example，然后训练discriminator，然后用这个discriminator去生成一些高分图片作为下一轮的negative example，然后再次训练，直到收敛。

{% asset_img 13.png 550 550 %}

下面这个图反映了discriminator如何一步步地收敛，一开始随机生成的数据在左边，真实的数据是中间，第一代discriminator能够把左边图片的评分压低，中间图片的评分拉高，但是这两个区域外的数据，discriminator无法知道该打高分还是低分，因此右边区域的分数可能甚至比真实数据还高；然后用这个discriminator去生成高分图片作为新的negative example，所以在下一轮迭代就会把这一部分的数据的评分压低。不断迭代，最终评分高的部分就只有真实数据。

{% asset_img 14.png 550 550 %}

这个演算法能够使得discriminator产生好的照片，但是问题在于，上面的那个argmax的方程并不容易解决，如果需要解决，就需要引入额外的约束，这又会影响网络的性能。

## Generator vs Discriminator

- generator
  - pros：擅长生成图片
  - cons：不能学到component之间的关系，只能浮于表面

- discriminator

  - pros：考虑大局
  - cons：生成图片基本不可能，argmax problem不容易解决

- generator + discriminator

  - discriminator的问题在于argmax problem无法解决，即找不到高评分的图片，即找不到下一轮的negative example来进行继续训练。而generator的存在刚好解决了discriminator的这个问题，generator就是努力学习生成能让discriminator评分高的图片，这样generator就可以生成高评分的图片，来作为下一轮discriminator的negative example，或者理解为generator就是学习解决这个argmax的problem。

    $$G \rightarrow \tilde{x} \equiv \tilde{x} = argmax_{x \in X} D(x) $$

  - generator的问题在于，它生成的图片是component-by-component的，因此不能考虑到大局观，而discriminator的存在就是从大局上给generator反馈，而不再是VAE的Euclidean distance。

  - 因此generator和discriminator相互促进，各自解决了对方的问题。



<br>