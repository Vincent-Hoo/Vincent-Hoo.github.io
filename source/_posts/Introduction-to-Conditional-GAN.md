---
title: Introduction to Conditional GAN
mathjax: true
date: 2019-08-19 11:57:15
tags:
---

> 课程来源：李宏毅2018生成对抗网络课程
> 课程主页：http://speech.ee.ntu.tw/~tlkagk/courses_MLDS18.html
> 文章梗概：本文介绍了条件对抗生成网络，与一般的GAN不同，条件GAN的输入并不是随机向量，而是条件。



<!-- more -->

<br>

*preface*

在最原始的 GAN 的里面，生成器接收一个随机的 noise，然后随机地生成一张图片，再交给判别器去判断该生成图片是否 realistic；假如现在我们不想让生成器随机地生成图片，而是根据我们的输入对应的生成图片，那就需要在生成器的输入加入额外的元素。



## Text-to-Image

传统监督的 text-to-image 的方法，需要找到 text 和 image 的 pair，然后训练一个网络使得生成的 image 和 ground true 的 image 越像越好。而 GAN 的方法则是在原有 GAN 的基础上加入对应的 text。

{% asset_img 3.png 550 550 %}

而判别器除了要判别出生成图和真实图之外，还要判断真实图和文字是否 match，因此对于判别器的二分类问题，negative examples 有两种，一种是 text 和 generated image，另一种是 text 和真实图片不 match。

{% asset_img 1.png 550 550 %}

而对于判别器的架构，有两种架构：

- 将条件 c 和 object 一起输入到一个网络里面进行判断，输出一个分数。
- 将两种 negative example 分开，一个网络判断是否 realistic，另一个网络判断是否 match。

{% asset_img 2.png 550 550 %}



*In Conclusion*
ConditionalGAN 简单来说就是在生成器和判别器的输入分别加入 condition。







<br>