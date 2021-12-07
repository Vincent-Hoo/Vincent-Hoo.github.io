---
title: NLP预训练模型概况
mathjax: true
date: 2021-12-07 19:34:57
tags:
---



{% asset_img titan.png 700 700 %}
<!--more-->
<br>
*preview*
预训练模型在 CV 任务里面是非常常见的，通常是用 ImageNet 数据集在一个网络上进行预训练，利用庞大 ImageNet 数据集让网络尽可能学习到更多的通用语义信息，然后下游任务直接用预训练模型进行微调即可，这是 CV 领域非常成熟的一套方法。

但在 NLP 领域预训练模型迟迟没能面世的原因，可能在于 NLP 的任务多且杂，不同任务的网络设计差异比较大，因此无法做到很好的统一。但这只是我猜测的原因罢了，要说 NLP 的预训练模型，早年的词向量也算是一种预训练模型，把训练好的词向量直接用到下游任务上，但这种预训练模型的效果对比起 CV 领域的预训练模型，效果不太好罢了。直到 transformer 的出现，NLP 的预训练模型才大规模的发展，这篇文章总结一下各科技大公司所提出的 NLP 预训练模型。

### ELMO

讲 ELMO 前必须先讲下词向量，word2vec 等词向量有一个很严重的问题，就是无法区分多义词，比如 play 这个单词，十几种意思只能融合在一个词向量里面，这样的词向量无法应对所有的语境，虽然后面有一些方法解决多义词的问题，但是都没有很好地解决掉。

ELMO 是 embedding from language model 的简称，来自《Deep contextualized word representation》论文，它的基本思想是，通过训练一个双向的 LSTM 语言模型，然后得到三种 embedding，分别是最下层的单词 embedding，第一层 LSTM 输出的 embedding，以及第二层 LSTM 输出的 embedding，利用这三种 embedding 就能很好地解决多义词的问题，因为最原始的 embedding 经过语言模型后，会根据不同语境得到不同的语义信息。

ELMO 训练好后可以很容易的应用到下游任务，整体做法就和词向量作为预训练模型的方法一样，将句子输入到语言模型得到三个 embedding，然后再把这些 embedding 输入到下游任务里面。

{% asset_img elmo.jpeg 500 500 %}
<br>


### GPT

generative pre-training model，是 OpenAI 2018年提出的模型，利用 transformer 解决各种 NLP 任务，GPT 预训练采用的是单向的语言模型，即根据上文预测下一个可能出现的词，因此 GPT 采用的 transformer 也是单向的 transformer，即将 Encoder 中的 Self-Attention 替换成了 Masked Self-Attention，对于某个 token，只计算与前面 token 的 attention。

而训练的过程其实非常的简单，就是将句子 n 个词的词向量(第一个为 [start] )加上 Positional Encoding 后输入到 Transfromer 中，n 个输出分别预测该位置的下一个词([start] 预测句子中的第一个词，最后一个词的预测结果不用于语言模型的训练)。由于采用的是 masked self-attention，因此对于某个词的预测，他只能看到前面的词，这样保证了模型的合理性。

{% asset_img gpt.png 500 500 %}

训练好单向的语言模型后，就可以应用到下游任务了，运用少量的带标签数据对模型参数进行微调，上一步中最后一个词的输出我们没有用到，在这一步中就要使用这一个输出来作为下游监督学习的输入，将最后一个词的输出连接到一个 linear 层进行下游任务，不同下游任务的改造如下，因为 GPT 是一个单向的语言模型，整体的过程就好像我让它看完一整个句子，然后问它最后的结果，因此用的也是最后一个 token 的输出来连 linear 层。这一点在后面的 GPT 系列体现更加明显。

{% asset_img gpt-finetune.png 500 500 %}


一些题外话：由于 GPT 是一个单向的语言模型，因此它可以做生成任务，即随机给他一段句子，他能写出一段故事来，甚至是给一个新闻标题生成新闻，GPT 问世的时候曾生成过一篇 unicorn 的 fake news，因此现在都拿 unicorn 去表示 GPT。

<br>

### BERT

Bidirectional Encoder Representation from Transformer，是 Google 2018 年提出的模型，和 GPT 非常类似，将 GPT 的单向语言模型改成了双向的语言模型，即不用采用 masked self-attention，但在预训练的时候，不能再像 GPT 那样预测下一个词，这时候 BERT 采取的是一种 masked language model(MLM) 的预训练任务，本质上和 word2vec 的 cbow 方法一样，即一个句子随机 mask 一些词，然后利用上下文的词去预测出这个 masked 掉的词，因此有人就形象地这个任务称为完形填空。

但是，直接将大量的词替换为 [MASK] 标签可能会造成一些问题，模型可能会认为只需要预测 [MASK] 相应的输出就行，其他位置的输出就无所谓。同时Fine-Tuning阶段的输入数据中并没有 [MASK] 标签，也有数据分布不同的问题。为了减轻这样训练带来的影响，BERT采用了如下的方式：

1. 输入数据中随机选择 15% 的词用于预测，这 15% 的词中，
2. 80% 的词向量输入时被替换为 [MASK] 
3. 10% 的词的词向量在输入时被替换为其他词的词向量
4. 另外 10% 保持不动

BERT 还提出了另外一种预训练方式 NSP(next sentence prediction)，即预测两个句子是否是连着的两句话，与 MLM 同时进行，组成多任务预训练。这种预训练的方式就是往 Transformer 中输入连续的两个句子，左边的句子前面加上一个 [CLS] 标签，两个句子之间使用 [SEP] 标签予以区分，[CLS] 的输出后连一个 linear 层，用来判断两个句子之间是否是连续上下文关系。而其他 masked 的 token 后面也接 linear 层来预测原来被 masked 掉的词。

{% asset_img bert-1.png 700 %}

为了区分两个句子的前后关系，BERT除了加入了Positional Encoding之外，还两外加入了一个在预训练时需要学习的 Segment Embedding 来区分两个句子。这样一来，BERT的输入就由词向量、位置向量、段向量三个部分相加组成。

{% asset_img bert-2.png 500 500 %}


BERT 的 Fine-Tuning 阶段和 GPT 没有太大区别。将分类预测用的输出向量从 GPT 的最后一个词的输出位置改为了句子开头 [CLS] 的位置了。不同的任务Fine-Tuning的示意图如下：

{% asset_img bert-3.png 700 %}

总结一下：BERT 借鉴了 ELMO、GPT、CBOW 等方法，本身没有太多的创新，更像是集大成者，但普适性是真的好，在各种任务上都达到了 SOTA 的水平，BERT 和 GPT 给整个 NLP 领域开了个好头，从此之后的预训练模型越做越大，训练的语料越来越多。

<br>

### GPT-2

GPT-2 沿用 GPT 的思路，依旧是单向的语言模型，即上文推断下文，但实现上完全抛弃 fine-tune 阶段，下游任务转为无监督的方式，要实现这个目标，GPT-2 用了一个更大更深的 transformer，堆叠了 48 层，且用了更大的且更高质量的语料库去预训练这个模型。下游任务无监督的方式是指不需要再用特定的任务数据集去微调，直接输入任务的描述以及问题，GPT-2 就会自动生成答案。举个例子，比如机器翻译任务，直接输入“请将下列中文翻译成英文，中文：我爱你，英文：”，然后 GPT-2 就会用它强大的语言模型，预测后面输出的字是 “ I Love You”，这种强大的能力得益于预训练模型的强大，以及语料库的丰富，这就好比如果一个人博览群书，自然就很轻松地就能完成翻译、问答、摘要等任务。

GPT-2 就好比一个很通用的语言模型，他学习到的知识可能是非常通用的，这才使他能够实现 zero-shot learning（不经过微调直接进行无监督），从 ELMO 到 GPT，再到 BERT，再到 GPT-2、GPT-3，模型规模越发的恐怖，甚至都无法公开因为太大了，如今的发展趋势可能慢慢地从专用模型往通用模型去转变，从少量标签数据到大量高质量无标签数据转变。






*references*

1. [从Word Embedding到Bert模型——自然语言处理预训练技术发展史](https://mp.weixin.qq.com/s/IgzOTLsj691mGA5TibLfcQ)
2. [Transformer结构及其应用详解--GPT、BERT、MT-DNN、GPT-2](https://zhuanlan.zhihu.com/p/69290203)
3. [bert代码](https://github.com/graykode/nlp-tutorial/blob/master/5-2.BERT/BERT.py)
4. [bert代码解读](https://www.bilibili.com/video/BV1Kb4y187G6?spm_id_from=333.999.0.0)
5. [李宏毅2021机器学习课程](https://www.bilibili.com/video/BV1Wv411h7kN?p=55)



