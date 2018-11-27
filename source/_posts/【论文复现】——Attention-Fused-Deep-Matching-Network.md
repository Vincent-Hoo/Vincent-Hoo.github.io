---
title: 【论文复现】——Attention-Fused Deep Matching Network
date: 2018-11-05 21:15:55
tags:
mathjax: True
---

> title: Attention-Fused Deep Matching Network for Natural Language Inference
> conference: 2018, IJCAI
> authors: Chaoqun Duan, Lei Cui, etc

<!-- more -->
<br>
# Natural Language Inference

NLI是NLP中一个比较核心的任务，主要用来判断两个句子是否有一定的关联，两个句子分为称为premise sentence和hopothesis sentence，NLI通常被看为是一个分类任务，经典的数据集包括SNLI、MultiNLI和Quora，这个分类任务的标签为entailment、contradiction和neural，一般基于神经网络的NLI模型包含三个层：encoder层、matching层、prediction层。

- encoder层：给定两个句子 $p = (p_1, ..., p_m)$，$q = (q_1, ..., q_n)$，经过embedding之后为 $p = (e_{p_1}, ..., e_{p_m})$，$q = (e_{q_1}, ..., e_{q_n})$，encoder层就是将两个句子分别通过一个Bi-LSTM进行压缩，得到 $H_p = (h_{p_1}, ..., h_{p_m})$，$H_q = (h_{q_1}, ..., h_{q_n})$，$H_p$ 和 $H_q$ 作为句子 $p$ 和 $q$ 的encoded representation。
- matching层：匹配层将两个句子联系起来，$g()$ 是一个匹配函数，$V_p = g(H_p, H_q)$，$V_q = g(H_q, H_p)$，得到新的representation
- prediction层：这一层将两个句子 $V_p$ 和 $V_q$ 各自的重要信息提取出来，提取方法采用pooling的方法，分别用GlobalMaxPooling和AveragedPooling进行提取，提取后的信息进行拼接后接入全连接网络进行预测。 

<br>

# Attention-Fused Deep Matching Network(AFDMN)

作者提出了一个基于注意力机制的匹配网络，来得到更好的interaction效果，注意力机制可以计算一个词与另一个句子的相关性，即attention weight。作者提出的网络结构大致如下，作者提出的matching层分为四个部分：cross attention层，fusion层，self attention层，fusion层，一个matching layer成为一个computational block，整个matching layer可以有多个blocks。

{% asset_img afdmn.JPG 600 300 %}

1. cross attention
   给定两个句子来自上一个block的representation，$H_p^{t-1} = (h_{p_1}^{t-1}, ..., h_{p_m}^{t-1})$ 和 $H_q^{t-1} = (h_{q_1}^{t-1}, ..., h_{q_n}^{t-1})$，首先计算一个相关性矩阵 $A$，$A_{i,j}$ 表示 句子 $p$ 的第 $i$ 个词和句子 $q$ 的第 $j$ 个词的相关性，$<a, b>$ 表示内积。

   $$ A_{i,j}^t = h_{p_i}^{t-1} W h_{q_j}^{t-1} + <U_l^t, h_{p_i}^{t-1}> + <U_r^t, h_{q_j}^{t-1}>$$ 

   $A$ 矩阵的第 $i$ 行做softmax就可以得到句子 $p$ 的第 $i$ 个词对于句子 $q$ 的attention weight，
   $A$ 矩阵的第 $j$ 列做softmax就可以得到句子 $q$ 的第 $j$ 个词对于句子 $p$ 的attention weight。

   $$a_{p_i}^t = softmax(A_{i:}^t)，a_{q_j}^t = softmax(A_{:j}^t)$$

   然后句子 $p$ 可以用句子 $q$ 和attention weight来表示，同理句子 $q$。
   $$\tilde{h}_{p_i}^t = H_q^{t-1} \cdot a_{p_i}^t，\tilde{h}_{q_j}^t = H_p^{t-1} \cdot a_{q_j}^t$$

2. fusion for cross attention
   对于句子 $p$，我们有两个representation，一个是原始的表示，一个是经过cross attention用句子 $q$ 表示的句子 $p$；同理句子 $q$；fusion层将这两种表示拼接，再经过Bi-LSTM
   $$\bar{f}_{p_i}^t = [h_{p_i}^t;\tilde{h}_{p_i}^t;h_{p_i}^t - \tilde{h}_{p_i}^t;h_{p_i}^t \odot \tilde{h}_{p_i}^t]$$

   $$\tilde{f}_{p_i}^t = Relu(W_f^t \bar{f}_{p_i}^t) + b_f^t$$

   $$f_{p_i}^t = BiLSTM(\tilde{f}_p^t)$$

3. self attention
   cross attention是计算两个句子的词的相关性，self attention同理，但是是计算一个句子内部句子之间的相关性，同样计算完attention weight后，可以通过weight算出一个新的representation，跟cross attention唯一不同的是，weight的计算直接采取内积的方式

4. fusion for self attention
   跟上面的fusion一样


<br>
# PyTorch实现

## cross attention
实现的时候需要注意的是：
1. weight的前半部分实际上是一个简单的神经网络，后半部分的 $U$ 是一个向量，是一个参数，因此要用 `nn.Parameter()`
2. 能量的后半部分计算比较复杂，下面提供了两个版本，分别是vectorized的和non-vectorized。

```python
class CrossAttn_Layer(nn.Module):
    def __init__(self, hidden_size):
        super(CrossAttn_Layer, self).__init__()
        self.hidden_size = hidden_size
        self.W = nn.Linear(2*hidden_size, 2*hidden_size)
        self.Ul = nn.Parameter(torch.FloatTensor(0.02*(np.random.rand(2*hidden_size)-0.5)))
        self.Ur = nn.Parameter(torch.FloatTensor(0.02*(np.random.rand(2*hidden_size)-0.5)))
        
    def forward(self, hp, hq):
        '''
        Args:
            hp: representation of the 1st sentence from the previous block
                dim: [batch_size, max_length_1, 2*hidden_size]
            hq: representation of the 2nd sentence from the previous block
                dim: dim: [batch_size, max_length_2, 2*hidden_size]
        Returns:
            output_after_crossAttn_1: [batch_size, max_length_1, 2*hidden_size]
            output_after_crossAttn_2: [batch_size, max_length_2, 2*hidden_size]
        '''
        
        batch_size = hp.size()[0]
        sentence_length1 = hp.size()[1]
        sentence_length2 = hq.size()[1]
        
        attn_energy = hp.bmm(self.W(hq).transpose(1,2))    
        energy1 = torch.mm(hp.contiguous().view(-1, 2*self.hidden_size), self.Ul.unsqueeze(1)).view(batch_size,sentence_length1, 1).expand(batch_size,sentence_length1, sentence_length2)       
        energy2 = torch.mm(hq.contiguous().view(-1, 2*self.hidden_size), self.Ur.unsqueeze(1)).view(batch_size, 1, sentence_length2).expand(batch_size,sentence_length1, sentence_length2) 
        attn_energy = attn_energy + energy1 + energy2

        
# =============================================================================
#         for b in range(batch_size):
#             for m in range(sentence_length1):
#                 for n in range(sentence_length2):
#                     attn_energy[b, m, n] += self.Ul.dot(hp[b, m, :]) + self.Ur.dot(hq[b, n, :])
#         
# =============================================================================        
        
        
        
        assert(attn_energy.size() == (batch_size, sentence_length1, sentence_length2))
        
        attn_weights_p = F.softmax(attn_energy, dim=2)
        attn_weights_q = F.softmax(attn_energy, dim=1)
        output_after_crossAttn_1 = attn_weights_p.bmm(hq)
        output_after_crossAttn_2 = attn_weights_q.transpose(1,2).bmm(hp)
        
        assert(hp.size() == output_after_crossAttn_1.size())
        assert(hq.size() == output_after_crossAttn_2.size())
         
        return output_after_crossAttn_1, output_after_crossAttn_2
```


<br>
## self attention

实现细节：
1. attention weight matrix不需要用for循环，直接用矩阵乘法实现内积
2. softmax的维度需要注意

```python
class SelfAttn_Layer(nn.Module):
    def __init__(self):
        super(SelfAttn_Layer, self).__init__()
    
    
    def forward(self, output_after_fusion):
        '''
        Args:
            output_after_fusion: [batch_size, max_length, 2*hidden_size]
        Returns:
            output_after_selfAttn: [batch_size, max_length, 2*hidden_size]
        '''
        
        similarity_matrix = output_after_fusion.bmm(output_after_fusion.transpose(1,2))
        attn_weight = F.softmax(similarity_matrix, dim=2)
        output_after_selfAttn = attn_weight.bmm(output_after_fusion)
        assert(output_after_fusion.size() == output_after_selfAttn.size())
        return output_after_selfAttn
```







<br>