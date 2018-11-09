---
title: Coursera深度学习课程第五课第三周————序列模型及注意力机制
date: 2018-11-05 10:41:47
mathjax: True
tags:
---


{% asset_img certificate.png 400 10 %}
该课程讲述序列模型，如机器翻译，语音识别，以及注意力机制将如何提升序列模型的性能。

<!-- more -->
<br>
## Seq2Seq模型

seq2seq模型常用于机器翻译，语音识别，输入和输出都是序列

### encoder-decoder model

以机器翻译作为例子，encoder是一个RNN模型，输入句子，然后encoder将这个句子编码，输出一个向量，叫做context vector或者encoding representation，然后这个context vector作为decoder的隐藏层输入，<SOS>作为decoder的第一个输入，得到第一个输出词，再将这个输出词作为输入，得到第二个输出词，以此类推。

{% asset_img encoder_decoder_model.png 500 250 %}

image captioning同样也是生成序列的任务，给定一张图片，需要生成一段描述图片的文字，只是这里encoding representation用CNN来实现。

{% asset_img image_captioning.png 500 250 %}
<br>
### Machine Translation vs Language Model

Language Model 是用来评判一个句子的合理性的，它需要最大化概率 $P(y^{<1>}, y^{<2>}, ..., y^{T_y})$

{% asset_img language_model.png 500 200 %}

 而Machine Translation采用encoder-decoder的模型，decoder的部分和language model是一模一样的，差别在于language model的隐藏层输入是0向量，而machine translation的隐藏层输入是encoder的输出，因此machine translation又可以作为 conditional language model，条件语言模型，它也是最大化句子的概率，但是这个概率是在输入句子是x的前提下，$P(y^{<1>}, y^{<2>}, ..., y^{T_y} |  x^{<1>}, x^{<2>}, ..., x^{T_x})$   

{% asset_img machine_translation_model.png 500 200 %}

当我们训练完一个机器翻译模型之后，输入一个句子，它的输出并不一定是固定的，输出的结果会在一定范围内变化，但是差别不大，且一般翻译都是正确的，那么我们怎样才能找到最好的翻译呢？有人提出用贪心去搜索，先选择概率最大的第一个词，再基于第一个词，选择概率最大的第二个词，然而我们整体的目标是要最大化整句话的概率，而非每一次一个词的概率，所以贪心搜索就会导致以下情况会选择第二个句子，因为前三个词 $P(Jane/is/visiting) < P(Jane/is/going)$ 

> Jane is visiting Africa in September
> Jane is going to be visiting Africa in September


<br>
### Beam Search

greedy search每一次只选择一个candidate作为下一次搜索的前提，Beam Search不同之处在于有一个Beam Width的参数，每一次会选择Beam Width个candidate，假如Beam Width等于3，那么decoder的第一步会选择3个first word candidate，比如是in，jane，September，然后将decoder复制3份，分别以encoding vector和这3个first word candidate作为前提，继续搜索3个second word candidate，然后发现 in september，jane is，jane visits是最有可能的3个candidate，这样就把September开头的可能翻译全部去掉，然后以这3个作为输入，继续搜索

{% asset_img beam_search1.png 600 300 %}

{% asset_img beam_search2.png 600 300 %}
<br>
## Attention Mechanism

attention机制最早提出是在机器翻译领域，传统的encoder-decoder模型在生成每一个输出词的时候，认为每一个输入词都是同等重要的，但是事实上，我们在翻译的时候，我们只会关注整个句子的某一部分去翻译出某一个词，而不是整个句子，因此对于输出词，每一个输入词的重要性是不一样的，attention机制就是计算输出词与每一个输入词的相关性，相关性可以理解为一个权重，$\alpha^{<2,1>}$ 表示第二个输出词与第一个输入词的关系，通过权重可以计算出一个context vector，作为decoder每一轮的输入

{% asset_img attention1.png 600 300 %}

权重的计算一般采用一个简单神经网络去实现，输入是decoder上一个状态的隐藏层和encoder的某一个输入词的隐藏层，然后神经网络输出一个权重，再把权重进行softmax的归一化

{% asset_img attention2.png 600 300 %}


下面左图是加入注意力机制的网络图，右图是计算注意力机制的图。
{% asset_img attention3.png %}



<br>