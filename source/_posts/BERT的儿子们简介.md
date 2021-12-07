---
title: BERT的儿子们简介
mathjax: true
date: 2021-12-07 19:45:40
tags:
---



Bert 的面世带动了 NLP 预训练模型的蓬勃发展，同时也衍生了很多的 bert 的变种，这里我们介绍一下 bert 的这些变种模型，如 ALBERT，fastbert，tinybert等，看下他们都是做了哪些优化的。

<!--more-->
<br>



### RoBERTa

>RoBERTa: A Robustly Optimized BERT Pretraining Approach

roberta 在结构上和 bert 完全一样，只是通过一些其他的方式证明 bert 还没完全训练完，性能还能提升，与 bert 比较，roberta 在预训练的时候做了以下改进

1：动态 mask：对比 bert 的 mask，稍微好一点
   - bert 的训练数据在训练前就全部构造好了，因此对于一个 sample，它的 mask 的位置是固定的，每个 epoch 都是一样
   - 而 roberta 构造训练数据是在数据输入模型前，这样能保证每个 epoch 的 sample，mask 的位置不同
{% asset_img roberta-1.png 500 500 %}


2： 去掉 NSP 任务，并且更改数据输入格式为全部填充可以跨越多个文档
- 结论就是 NSP 任务没什么用，ALBERT 也验证了并换成难度更大的 SOP 任务
- roberta 试验了四种输入格式
  - segment-pair + NSP：就是 bert 的输入格式
  - sentence-pair + NSP：输入的是一对句子，前后都是单个句子；对比 segment-pair，这个效果更差，因为两句句子太短了，对比一大段的两段文字，前后关系不好判断。
  - full-sentence：如果输入的最大长度为 512，那么就是尽量选择 512 长度的连续句子。如果跨 document 了，就在中间加上一个特殊分隔符。无 NSP。实验使用了这个，因为能够固定 batch size 的大小。
  - doc-sentence：输入和 full-sentence 类似，但不能跨两个 document

{% asset_img roberta-2.png 500 500 %}


3：更多的数据，更大的 batch size，更多的步数，更长的训练时间
   - 数据：bert：16G，roberta：160G
   - batchsize：bert：256，roberta：8k

<br>

### ALBERT

>  ALBERT: A Lite BERT For Self-Supervised Learning Of Language Representations

从论文题目看，albert 是主打 lightweight 的，对比 bert 主要有下面几点改进

1. Factorized embedding parameterization：bert 的 embedding 层，每个词向量的大小 E 都和 transformer 隐藏层 H 一样，$E = H = 768$，因此词表占了很大的一个参数量，因此 albert 将 E 和 H 分开，令 $E < H$，通过一个矩阵进行映射即可
2. Cross-layer parameter sharing：为了减少网络的参数量，albert 对不同层的参数进行共享，参数共享有三种方式：只共享 feed-forward network 的参数、只共享 attention 的参数、共享全部参数。albert 默认是共享全部参数的
3. Inter-sentence coherence loss：去掉 NSP 任务，改做 sentence-order-prediction（SOP）任务，NSP 任务比较简单，模型更多的会通过两个句子的 topic 是否一致来判断，而不是通过句子间的关系来判断，而 SOP 任务直接判断两个句子是正的还是反的，并且两个句子来自同一个文档，topic 保持一致。

作者实验了以下几种网络参数，通过共享所有层的参数，参数量大幅度下降

{% asset_img albert-1.png  %}

其实 Albert 只能做到减少参数量，并不能很好的加快推理速度，因为网络还是那么深，对比 bert-large 和 albert-large，他们唯一不同就在于 embedding 层变小了，但就这一点不同，就有 1.7 倍的加速了；另外我们可以看出共享参数是会掉精度的，这也很好理解

{% asset_img albert-2.png  %}

改变 embedding size，对于 bert 和 albert 都有一定影响，128 效果最好

{% asset_img albert-3.png %}

共享不同参数，可以看出 FFN 比较重要，attention 没那么重要，但考虑到压缩量，作者还是选择全部共享

{% asset_img albert-4.png  %}

SOP 任务训练可以解决 NSP 任务，但 NSP 任务训练不可以解决 SOP 任务

{% asset_img albert-5.png  %}

最后，作者尝试了用更多的数据（roberta的训练数据集），训练更长时间，都能得到提升，并且作者去掉 dropout 也能提升，因为模型还没到过拟合的程度，因此不需要 dropout。

albert 能做到好的结果，得益于 更大的 H，更多的数据，SOP 任务，dropout 的去除，将这些 trick 应用到 bert 上，bert 也能得到提升。

<br>

### BERT Distillation

>Distilling Task-Specific Knowledge from BERT into Simple Neural Networks.
>Patient Knowledge Distillation for BERT Model Compression.
>DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter.
>TinyBERT: Distilling BERT for Natural Language Understanding.
>MobileBERT: a Compact Task-Agnostic BERT for Resource-Limited Devices.
>MINILM: Deep Self-Attention Distillation for Task-Agnostic Compression of Pre-Trained Transformers.
>FastBERT: a Self-distilling BERT with Adaptive Inference Time.

上面蒸馏方法的对比如下表，可以看出主要的差别在于

1. KD 的阶段是在预训练阶段还是微调阶段
2. KD 的位置，从最后的输出 logit 到中间特征层，和 CV 的 KD 一样
3. 是否用教师网络来初始化学生网络


{% asset_img distill-table.png  %}



#### Distilled BiLSTM

教师网络是先将 BERT-large 微调到具体任务，学生网络是 BiLSTM，将教师网络的 logit 输出蒸馏到学生网络的 logit，这里应该是用的教师网络的 [CLS] 的那个logit 输出，距离函数直接用的 MSE，针对不同输入的任务，都是将 lstm 两边的输出拼接然后接 softmax 得到任务的 logit。最终的损失函数为 ground-true label 的 CE loss + 教师网络和学生网络 logit 之间的 MSE（作者发现 MSE 比 CE 好）。

另外因为 KD 是在 fine-tune 阶段，所以做了 data augmentation。

{% asset_img distill-bilstm.png 500 500 %}


#### BERT-PKD

同样是 finetune 阶段的 KD，将 BERT-BASE 在下游任务上进行 finetune 得到教师网络，学生网络同样是 BERT，但是深度比教师网络浅，比如 3 或6 层，因为只是深度减少了，所以可以用教师网络来初始化学生网络，损失函数为下游任务的 ground-true CE loss + 教师网络和学生网络 logit CE loss + 中间层的蒸馏，中间层蒸馏也不是每个 token 都比较，只蒸馏 [CLS]，用的是 MSE loss，中间层的选择有两种，分别是 PKD-last 和 PKD-skip，skip 的结果好一点

{% asset_img bert-pkd.png 500 500 %}


#### DistilledBert

前面两个 KD 的工作都是在 finetune 阶段，学生网络从教师网络中学习到的都是 task-specific 的知识，DistilledBert 选择预训练阶段进行蒸馏，也叫做通用蒸馏（general distillation）。

- 教师网络是 BERT-base，学生网络为 6 层 bert，采取 PKD-skip 的方式初始化
- 训练方式采取 roberta 的方式，去掉 NSP 任务，采取动态 mask，但训练数据和 bert 保持一致，16G
- 由于只有 MLM 任务，因此损失函数有 MLM loss，教师网络-学生网络最后一层的交叉熵，除此之外，作者加了一个 cosine embedding loss，比较最后一层 hidden（softmax 层之前）



#### Tinybert

将前面所有工作混在一起，既进行预训练阶段的蒸馏，也进行微调阶段的蒸馏，蒸馏的位置包括输出 logit、中间层特征（attention map，hidden state）、word embedding。

整体的流程如下，首先有一个教师网络 bert-base，和一个没训练的学生网络，这里不拿教师网络去初始化，因为学生网络的隐藏层和教师网络可能不一样，初始化不了，然后用教师网络进行 general distillation 得到 general tinybert，然后将教师网络在下游任务中进行 finetune，再拿这个 finetune 的教师网络去进行 task-specific 的蒸馏得到 finetune tinybert（有数据增广）。

{% asset_img tinybert-1.png 500 500 %}


损失函数如下，注意 general distillation 没有 logit 的 loss，中间层的蒸馏采用 pkd-skip 的方式对应。

1. 输出层 logit 采取 CE loss
2. 中间层 attention map 蒸馏（MSE），用的是未 softmax 之前的 map
3. 中间层 hidden state 蒸馏（MSE），由于维度不同，会加一个映射矩阵
4. embedding 蒸馏（MSE），同样维度不同，也会加一个映射矩阵

{% asset_img tinybert-2.png 700 700 %}

#### miniLM

这篇论文的方法是最简洁的，也提出了一种新的知识——value-value 矩阵，attention map 是 K 和 V 点乘得到的，而 value-value 矩阵是将 V 矩阵和 V 矩阵的转置进行点乘，然后进行蒸馏；考虑到教师网络和学生网络层数、维度等都不同，所以只蒸馏最后一层，距离度量采取的是 KL 散度；最后引入助教制度，先蒸馏到助教，助教再蒸馏到学生。

性能的话，还要比 tinybert 和 distilledbert 要好不少，miniLM 只进行 general distillation，简洁效果又好，比 tinybert 蒸馏一大堆性价比高太多了

{% asset_img miniLM.png 700 700 %}



#### mobileBert

个人最不喜欢的一篇文章，结构复杂，虽说像 mobilenet 那样引入 bottleneck 的结构，但整体的流程太复杂，不推荐。



#### FastBert

fastbert 参考了 CV 领域里面的 self-distillation 和 adaptive inference 的策略，由于是自蒸馏，因此教师网络就是学生网络，具体流程是：先拿一个预训练好的 bert 模型，在下游任务中进行 finetune 好，然后在每一层 transformer 后加入一个 classifier，这些 classifier 叫做 student classifier，最后一层的 classifier（finetune 好的）叫做 teacher classifier，然后就是自蒸馏的部分，teacher classifier 的输出分别蒸馏给每一个 student classifier，用的是 KL 散度。

adaptive inference 部分就是对于一个样本，每经过一层 transformer，就经过一个 student classifier，然后输出的 logit，计算不确定性，不确定性用 logit 的 entropy 来表示，如果不确定性低于阈值，就停止往下过网络，反之继续过网络。

{% asset_img fastbert.png 700 700 %}
