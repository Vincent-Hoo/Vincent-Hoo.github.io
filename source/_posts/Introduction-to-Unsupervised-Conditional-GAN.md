---
title: Introduction to Unsupervised Conditional GAN
mathjax: true
date: 2019-08-19 11:58:18
tags:
---

> 课程来源：李宏毅2018生成对抗网络课程
> 课程主页：http://speech.ee.ntu.tw/~tlkagk/courses_MLDS18.html
> 文章梗概：本文降介绍非监督的条件GAN，只有两个数据域的数据，并不存在pair关系，其中会介绍最为经典的cycleGAN。



<!-- more -->

<br>

*preface*

之前讲到的 conditional generation 都是 supervised 的，即对于每一个输入都有对应的输出，但是有一些任务，比如风格迁移，从一个 domain 的图像转到另一个 domain 的图像，这种任务是非监督的，我们只有两个 domain 的多张图像，但是他们不存在 pair 的关系，下面来看下 GAN 如何解决这种问题。



## Domain transfer

cycleGAN 就是用来解决不同 domain 之间的互转，论文中，作者用了普通马和斑马的例子，将一匹普通马转成斑马，或将斑马“去斑”。

首先先看单向的cycleGAN，domain A 是斑马，domain B 是普通马，先通过一个生成器将 A 域的马转成 B 域，再将生成的 B 域的假马，通过另外一个生成器转回 A 域。这其中有两个损失

- 一是 cycle consistency，即重塑的 A 域的马要和原来 A 域的马要尽可能像。
- 二是判别损失，生成的 B 域的假马要通过判别器来判断是否真假。

{% asset_img 1.png 550 550 %}

而 cycleGAN 本质就是两个镜像对称的 GAN，形成一个环形网络，共有两个判别器，两个生成器，两个判别器用来判断是否为对应的 domain。

{% asset_img 2.png 550 550 %}

cycleGAN 用来解决两个 domain 之间的转换，当涉及多个 domain 之间的转换时，若每两个 domain 之间都用一个生成器来解决的话，那必定会产生巨大的开销，starGAN 用一个生成器和判别器就解决了多个 domain 之间的转换问题

{% asset_img 3.png 550 550 %}

生成器接收输入图像和目标域，生成假的图像后，再通过同一个生成器，将原域和假图像输入，得到重构图像，尽可能使得重构图像和原图接近；而判别器除了需要判别真假外，还需要另外做一个 domain classification。

<br>