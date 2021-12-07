---
title: Transformer简单介绍
mathjax: true
date: 2021-12-07 19:30:06
tags:
---

Transformer 是 Google 在 2017 年发表的 《Attention is all you need》论文中提出的，为了解决机器翻译等 seq2seq 任务中 RNN 不能并行化等问题，transformer 是一个完全由 self-attention 搭建起来的网络，完全抛弃了 CNN 和 RNN 等传统网络结构，从而实现了并行化，到目前为止已经完全取代 RNN 成为自然语言处理领域最好的特征提取器。

<!--more-->
<br>
### self-attention

上面提到 transformer 是由 self-attention 搭建起来的网络，这里就先介绍一下 self-attention 机制，NLP 任务的输入往往是一连串的文字，把它看成是一个由多个 token 组成的 sequence，那么 self-attention 想要做的事情就是计算 token 之间的相关性。

每一个 token 会由三个向量所表征，分别是 Q(query)、K(key) 和 V(value) 向量，当我们需要计算 token A 与其他 token 之间的相似度的时候，就用 token A 的 query 向量去和其他 token 的 key 向量去进行相似度计算，这里的相似度计算采用的是内积（因此 transformer 中的 **Q 向量 和 K 向量的维度必须一致**），内积后进行 softmax 操作得到与各个 token 之间的相似度，然后将相似度与各个 token 的 V 向量进行 weighted sum，最终得到 token A 的输出。其他 token 也是进行同样的操作得到各自的输出。

上面提到 transformer 可以并行化是因为，我们可以把上述的操作进行矩阵化，把 sequence 里的 token 进行拼接，我们可以得到三个矩阵 Q K V，下图表示的是两个 token，每个 token 的向量都是 3 维，计算出两个 token 的相似度，除以  $\sqrt d_k$ 是为了减小量级，加速收敛，然后进行 weighted sum 得到每个 token 的输出。

{% asset_img 1.jpeg 500 500 %}

<br>

### Multi-head Attention

上面提到的 self-attention 正是下图的 scaled dot-product attention，Mask 表示的是有目标性地选择 token 去进行 softmax 操作，因为为了方便训练，我们会设置一个超参是 maxlen，表示一个 sequence 的最大长度，少于这个长度的 sequence 会被后面填充特殊 [PAD]，在计算相似度的时候，这些 [PAD] 字符是没有意义的，所以我们要把它 mask 掉，计算真正有意义的 token 之间的相似度。

而下面右图的 multi-head attention 是 transformer 真正用到，所谓多头注意力机制类似 group convolution，将空间划分成多个子空间，然后再各个子空间里面计算 token 之间的相似度。比如一个 token 是由 512 维的 QKV 向量组成，那么将这些向量划成 8 个头，每个头为 64 份，然后分别对每一份计算 scaled dot-product attention，再把输出结果合并即可，这样做的目的是为了细粒度化，因为特征空间的每一个维度可能都对应着不同的信息，如果对整一个大的空间去进行相似度，可能会不准确，所以将空间切分进行细粒度的相似度计算。

{% asset_img 2.png 500 500 %}

<br>

### positional encoding

上面提到的 self-attention 是可以进行矩阵操作的，因此这也是它可以并行化的原因，但是这同时也失去了时序的信息，我们把一个 sequence 打乱顺序输入，得到的结果会是一样的，因为上面的 self-attention 无法捕捉 token 的位置信息，因此我们需要在词嵌入的基础上加入位置的 embedding。

transformer 论文提出的位置编码如下，pos 表示第几个 token，i 表示的是 token 的第几个维度

{% asset_img 3.png 500 500 %}

作者之所以设置这样的位置编码，是利用了 sin 和 cos 函数的性质，下一个位置的编码向量可以由前面的编码向量线性表示，等价于以一种非常容易学会的方式告诉了网络单词之间的绝对位置，让模型能够轻松学习到相对位置信息。

<br>

### Network Structure

transformer 的整体结构如下，和 seq2seq 模型结构一样，分为 encoder 部分和 decoder 部分，输入的 sequence 首先会对每个 token 进行词嵌入再加入位置编码得到输入的 sequence，然后经过 N=6 个的 encoder 得到输出。然后编码器的输出会输入到每一个的 decoder 里面，而 decoder 同样会有输入，这里详细说下训练和预测两个环节的差异。

transformer 在预测的时候，decoder 是串行的，比如机器翻译任务，输入中文句子到 encoder 得到输出，然后把 encoder 的输出和一个 [START] 特殊字符同时 输入到 decoder，表示解码任务的开始，然后解码器输出第一个翻译的 token A，然后将 [START] 和 token A 输入到 decoder 进行第二轮的解码，得到第二个翻译的 token B，依此类推，直到输出的 token 为 [END]。

Transformer 在训练的时候，decoder 是并行的，意味着只进行一轮的解码，假如翻译的句子是“我爱你”，那么翻译的结果应该是“I Love You”，此时 decoder 的输入为 [START] I Love You，decoder 的 ground-true 输出应该是 I Love You [END]，表示我输入 [START] 的时候应该输出 I，输入 I 的时候应该输出 Love。这种做法叫做 teacher-forcing，让训练过程接近预测过程。

{% asset_img 4.png 500 500 %}

#### encoder

编码器共有 N=6 个，每一个的结构都一样，由 multi-head attention 和 feed forward 两部分组成，add & Norm 分别表示残差连接和 LayerNorm，每一个编码器的输入为上一个编码器的输出，输入之后经过矩阵变换分别得到 QKV 三个矩阵，这里采用的是 self-attention，因此输入都是一致的，和后面的解码器区分开来，解码器的 KV 来自于最后一层编码器的输出，Q 来自于解码器 masked 多头注意力的输出。

layernorm 是对特征空间进行归一化，即对 512 维进行归一化，区别于 batchnorm，是对于每一维的特征在 batch 维度进行归一化。之所以不用 bn，因为 bn 对特征每一个维度进行归一化，显然不同 batch（不同sequence）的某一个特征维度并没有关联，所以 NLP 领域多用 layernorm

#### decoder

decoder 的输入有编码器最后一层的输出还有额外的经过位置编码的输入，结构由 maksed 多头注意力模块、encoder-decoder 注意力机制和前向反馈网络组成。

**masked multi-head attention**

这里要注意的是 **masked**，为什么要强调这个 masked 呢？是因为训练过程要模拟预测过程，即我在预测某一个词的时候我是不知道后面的词的，比如输入是 [START] I Love You，对于 Love，我只知道 [START] 和 I，因此在计算相似度的时候，后面的词要全部 mask 掉，包括 pad 的，只计算与前面 token 的相似度后 softmax。

**encoder-decoder attention**

这个区别于 self-attention，是计算输入的 token 和输出的某个 token 之间的 attention，因此 Q 向量来自于 masked 多头注意力的输出，KV 向量来自最后一层 encoder 的输出

#### classifier

decoder 的输出维度为 [batch, sequence_length, d_ff]，然后需要经过一个 linear 层得到 [batch, sequence_length, vocab_size]，因为本质上是一个分类任务，因此 softmax 后与 GT 计算 cross entropy



*reference*

1. https://zhuanlan.zhihu.com/p/308301901
2. https://zhuanlan.zhihu.com/p/44731789
3. [transformer pytorch 实现](https://github.com/graykode/nlp-tutorial/blob/master/5-1.Transformer/Transformer.py)，注意 mask 的实现
4. [上面代码的解读](https://www.bilibili.com/video/BV1dR4y1E7aL?spm_id_from=333.999.0.0)
5. [一些面试常问的transformer的问题](https://github.com/DA-southampton/NLP_ability/blob/master/%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E5%A4%84%E7%90%86/Transformer/%E7%AD%94%E6%A1%88%E5%90%88%E8%BE%91.md)

