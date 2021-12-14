---
title: VIT预训练模型介绍
mathjax: true
date: 2021-12-14 21:12:40
tags:
---

Transformer 提出后，一大堆基于 transformer 的 NLP 预训练模型被提出，如 BERT、GPT 等，但 transformer 在 CV 领域上的应用却迟迟没能火起来，最近 VIT (vision transformer) 的提出，给 CV community 注入了一剂强心剂，再一次证明了 transformer 是现今最强的特征提取器，按照 NLP 的发展历程，要训练这种 data-hungry 的大规模网络，就必定需要预训练方法，用监督训练的方式肯定是行不通的，需要利用无标注的数据，采取自监督的方法进行预训练。这篇文章将介绍 BEIT 和 MAE 这两种预训练方法

>BEIT: BERT Pre-Training of Image Transformers
>Masked Autoencoders Are Scalable Vision Learners

<!--more-->

<br>

### VIT 介绍

transformer 擅长处理序列数据，因此 NLP 领域可以很自然地利用 transformer 来构造网络，但是 CV 领域处理的是图片视频这些三维的数据，它不具有序列性。要用 transformer 来处理图片数据，就必须得引入序列信息，而 VIT 采取了一种很简单的方法：将图片或特征图（先用 CNN 得到 feature map）切成一块块的 patch，就能构造出一个从左上角到右下角的序列，加入位置编码，引入分类 token，然后再将这个序列输入到 transformer 里面。transformer 的结构对比标准的 transformer 结构也做了一点修改，将 layernorm 的位置提前了。

{% asset_img vit.png 700 %}


VIT 整体的训练方式是采取 pre-train 再 finetune 的形式，跟 BERT 一样，VIT 也是有 base、large 和 huge 三种结构，对于这种 data-hungry 的网络，在小数据集（如 cifar10/100）上 train from scratch 肯定很差，因此才需要预训练再微调。VIT 采用的预训练方式是有监督的训练，从上面的网络结构图也能看出，它的预训练任务只能采取分类任务，因此作者采用了三个大分类数据集来做对比，分别是 ImageNet-1k (1.3M)、ImageNet-21k (14M) 和 JFT-18k (303M)，一个数据集比一个数据集大。

由于网络大，只采取监督训练不能完全训练好网络，因此实验结果总会出现，用 ImageNet-1k 预训练的 VIT-Large 比 VIT-Base 还要差的结果，只有用 JFT 这种超大数据集训练才能训得好，但这样对数据集的要求就太高了，因此需要利用自监督的训练方式来对 VIT 进行预训练。

<br>

### BEIT

BEIT 是参考 BERT 的方式对网络进行预训练的，BERT 是采用 MLM (masked language modeling) 的方式进行预训练，网络结构只用了 encoder，对输入的 sequence 进行随机 mask，然后让网络去预测 mask 掉的 token。

由于 NLP 的输入的句子是由一个个的 token 组成的，而 token 是从有限的集合中来的，也就是 vocabulary，但是 cv 里，将图片切成一个个 patch 后，这些 patch 不可能从一个有限的 vocabulary 里面得到，如果要按照 BERT 那一套预训练方式，首先需要将 patch 映射到 token，因此作者训练了一个 tokenizer 实现了这个功能，作者把这个叫做 visual token，然后其他的部分和 BERT 保持一致，mask 掉某些 patch，输入到网络，网络预测 masked patch 的 visual token。对于 masking 部分，作者 mask 掉 40% 的 patch，并且不是完全随机 mask，而是提出一种 block-wise 的 masking 方法，详细看论文。

预训练部分由于采取的是自监督的方式，因此并不需要像原始 VIT 那样需要那么大的监督数据集，作者只采用了 ImageNet-1k 就能取得比较好的结果了。

{% asset_img beit.png 700 %}

<br>

### MAE

MAE 代表 Masked Auto-Encoder，从名字上可以看出，它预训练的方式就是将图片 mask 掉，然后训练一个 auto-encoder 将其复原，这种结构在 cv 里的复原任务里经常能看到，但为什么现在才提出来呢，后面会详细说下 NLP 和 CV 领域的差别。

整体的结构如下图，首先将图片进行随机 mask，不同 BEIT 的是，MAE mask 的比例高达 70-80%，并且 BEIT 做的是预测 masked token，MAE 做的是恢复 masked patch，并且对于 encoder 的输入，没有输入 masked patch，这是为了保持与 finetune 阶段一致，减少预训练和 finetune 的 gap，其实 bert 采取 8:1:1 的比例 mask 掉 token，也是为了减少这个 gap 的，并且只输入 visible patch，也能减少很多的运算量；然后会将 encoder 的输出加上 mask token，再一同输入到 decoder 进行解码恢复，注意 encoder 和 decoder 都是 VIT 结构，但是 encoder 要比 decoder 大，这是因为 decoder 只用于预训练，finetune 阶段 decoder 将会被去掉，下游任务只会用到 encoder。

对比 BERT 和 MAE，我们可以发现，其实 BERT 更像是 MAE 的 decoder 部分，MAE 先通过 encoder 得到高级的语义信息，再和 mask token 拼接在一起，输入到 decoder，而 bert 输入是一个个的词，天然就是一种语义信息很高的 token，对比图片的一个个 patch（高冗余）。

{% asset_img mae.png 500 500 %}

作者思考过 cv 和 nlp 两个领域在预训练 masked 编码器的区别，主要有三个方面导致 transformer 当初在 cv 领域没火起来。

1. CNN 一直是 cv 领域的主流网络，因为图片这种数据适合卷积来操作，而这种 grid 的卷积操作不好加入序列信息，如位置编码或者 mask 信息，这种结构的 gap 直到 VIT 出现才被解决。
2. 文字和图片的信息密度不同，文字是一种高度语义化的信息，因此 bert 采取 MLM 这种方式，并且只要 mask 15% 就能够很好的充当预训练的功能；但是图片是一种高度冗余的信息，它可以很容易地通过邻近的像素去还原 mask，并不需要很多的高级语义，要解决这种问题，作者提出只要 mask 的比例足够大，也能学习到很多语义信息来充当预训练。
3. 对于 bert，预测 missing words 是一种高级的语义任务，encoder 的输出就已经有很多高级语义信息了，因此 bert 的 decoder 就只是一个简单的 MLP；而对于 MAE，它是要恢复 pixel，这个相对来说是不需要很多语义信息的，因此 MAE 专门引入了一个 decoder 来做这件事。



<br>