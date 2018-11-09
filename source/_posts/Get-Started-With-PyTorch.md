---
title: Get Started With PyTorch
date: 2018-11-05 18:01:46
tags:
---


{% asset_img header.jpg 600 300 %}
该文章介绍PyTorch的基本数据结构，以及如何用PyTorch搭建一个神经网络

<!-- more -->

*introduction*

> PyTorch is a Python-based scientific computing package targeted at two sets of audiences: 
>
> - A replacement for NumPy to use the power of GPUs
> - a deep learning research platform that provides maximum flexibility and speed

PyTorch对比与Tensorflow最大的特点在于它建立的神经网络是动态的，而Tensorflow建立的计算图是静态的，这使得Tensorflow在RNN上会有一点被动。


<br>
# PyTorch的数据基础

正如官方所说，PyTorch是用来替代Numpy的，自然它就会继承Numpy的一些特点，torch自称是神经网络界的Numpy，因为它能将torch产生的tensor放在GPU中加速计算，正如Numpy可以将array放在CPU中加速运算。

## Tensor

torch最基本的数据形式就是tensor，是一个多维度的张量，torch中的tensor与Numpy中的array具有很多的相似性。
- 两者在创建上基本一致
- 两者可以通过函数相互转换
- 两者都有相同的数学运算

```python
import numpy as np
import torch

# 创建
numpy_data = np.array([[1,2,3], [4,5,6]])
torch_data_long = torch.LongTensor([[1,2,3], [4,5,6]])
torch_data_float = torch.FloatTensor([[1,2,3], [4,5,6]])

# 相互转换
numpy_data = torch_data_long.numpy()
torch_data = torch.from_numpy(numpy_data)

# 查看维度
numpy_shape = numpy_data.shape
torch_shape = torch_data.size()

# 矩阵相乘
numpy_mul = np.dot(numpy_data, numpy_data.T)
torch_mul = torch.mm(torch_data, torch_data.transpose(0,1))
```

**Note: ** `torch.dot()`只能针对于一维tensor，即内积；而`np.dot()`可以用来做矩阵乘法

下面列举一些常见的API

```python
# 张量转置，a、b为相互交换位置的维度
torch_data.transpose(a,b)

# 张量压缩和扩张，压缩的意思是指：将维度为1的维度去掉，如(3,1,4) -> (3,4)；而扩张就是正好相反
torch_data.squeeze(axe)
torch_data.unsqueeze(axe)

# 张量reshape
torch_data.view(axe1, axe2, ...)

# 张量批乘法，batch matrix multiplication，在神经网络里面，一般采取批训练，
#因此数据大小一般是(batch, a, b)，bmm会无视batch维度进行乘法，
#(batch, a, b) × (batch, b, c) -> (batch, a, c)
torch_data.bmm(torch_data1)
```


<br>
## Variable

`Variable`来自`torch.autograd`，Variable是神经网络的基础，它好比神经网络的参数，会变化。而实际上Variable描述的是一个计算的操作，多个Variable之间就构成一个计算图。Variable的属性如下：

- data：Tensor类型，Variable中存放的数据
- grad：梯度，由于Variable之间会通过数学运算进行相连，如神经网络，构成一个计算图，在反向传递的时候，就会计算计算图中每一个节点的梯度，并存放到grad里
- creator：创建这个Variable的操作，如加法、矩阵乘法等

Tensor和Variable区别在于：tensor只是一个数据形式，而Variable是tensor的container，是变量，会变化

**Note：** 最新版的torch里面，已经模糊掉了Variable的概念了，因此tensor和Variable已经没有区别，`torch.tensor`和`torch.autograd.Variable`已经是同一个类了，tensor可以追踪历史。


<br>
# PyTorch搭建神经网络

pytorch搭建和训练神经网络的方式，对比Keras来说，是复杂了一点点，因为Keras是比较high-level的封装，因此有些东西也无法实现，下面讲述一下如何搭建一个神经网络。

## 搭建神经网络

对于小型的网络，可以用一个Sequential将网络框住从而实现搭建，Sequential就好比Keras里面的序贯模型，顺序执行每一层。`torch.nn` 封装了所有的神经网络层，包括全连接层，卷积层，池化层等。

```python
net = torch.nn.Sequential(torch.nn.Linear(40, 20),
                          torch.nn.Dropout(0.8),
                          torch.nn.ReLu(),
                          torch.nn.Linear(20, 5),
                          torch.nn.Softmax()
                         )
```

而传统的方法是，通过创建一个继承`torch.nn.Module`的类，来实现搭建工作。`torch.nn.functional` 封装了所有神经网络中的激活函数，注意区分 `torch.nn.ReLu()` 和 `torch.nn.functinoal.relu(x)`，前者是一个层，后者是一个函数。

自定义的类里面，`__init__` 函数和 `forward` 函数是必须要重载的，初始化函数定义网络中的参数，以及可以定义一些网络层；`forward` 函数定义前向传递的过程，返回输出值。

```python
import torch.nn.functional as F
class Net(torch.nn.module):
    def __init__(self, n_feature, n_hidden, n_output):
        super(Net, self).__init__()
        self.hidden_layer = torch.nn.Linear(n_feature, n_hidden)
        self.output_layer = torch.nn.Linear(n_hidden, n_output)
        
    def forward(self, x):
        x = F.relu(self.hidden_layer(x))
        y_preds = F.softmax(self.output_layer(x))
        return y_preds
    
net = Net(40, 20, 5)
```


<br>
## 训练神经网络

训练神经网络的步骤基本是

1. 定义损失函数和优化器
2. 迭代进行前向传递，以及误差求导，反向传递更新参数

在PyTorch里面，这些都需要自己去实现，不像Keras，只需要一个函数即可。优化器封装在`torch.optim` 里面，常用的有SGD、Adam，我们需要将网络里面需要更新的变量传给优化器，`net.parameters()` 返回网络中的Variable；损失函数常用的有MSE、CrossEntropy；

每一个epoch的步骤：

1. 计算网络输出值，然后计算loss，注意loss本身也是一个Variable；
2. `optimizer.zero_grad()` 是将网络中的变量的grad都清零；
3. `loss.backward()` 是进行反向传递，算出计算图里面每个节点的梯度；
4. `optimizer.step` 是更新网络中的所有变量。

```python
optimizer = torch.optim.SGD(net.parameters(), lr=0.1)
loss_func = torch.nn.MSELoss()

for epoch in range(100):
    y_preds = net(x)
    loss = loss_func(y_preds, y)
    # 下面的三步是训练网络必须有的
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

训练的过程实际上就是不断更新网络中变量参数的过程，训练完成后，可以将网络保存下来，保存的方式有两种，一种是将整个网络连带训练好的参数一起保存下来，另一种是只保留训练好的参数，这种方法我们在load的时候，需要先构造好一个完全一模一样的网络，然后再将参数load进去

```python
# save
torch.save(net, path1)
torch.save(net.state_dict(), path2)

# load
net = torch.load(path1)
net1 = Net(40, 20, 5)
net1.load_state_dict(torch.load(path2))
```


<br>
## 搭建高级神经网络

所谓高级的神经网络，无非就是CNN和RNN，具体搭建方法和上面一样，只是网络的层改变了而已，具体的网络层参数可以参考官方文档。而另外forward函数里面可以进行很多操作，很多网络结构并不是序贯模型，因此forward里面可以进行各种数学运算，如计算attention weight等。

由于CNN和RNN已经很普遍了，我在这里展示一下auto-encoder的实现，auto-encoder是一个无监督的模型，将输入数据进行encoder压缩，得到encoded representation，再通过decoder解压

```python
class AutoEncoder(nn.Module):
    def __init__(self):
        super(AutoEncoder, self).__init__()

        # 压缩
        self.encoder = nn.Sequential(
            nn.Linear(28*28, 128),
            nn.Tanh(),
            nn.Linear(128, 64),
            nn.Tanh(),
            nn.Linear(64, 12),
            nn.Tanh(),
            nn.Linear(12, 3)   # 压缩成3个特征, 进行 3D 图像可视化
        )
        # 解压
        self.decoder = nn.Sequential(
            nn.Linear(3, 12),
            nn.Tanh(),
            nn.Linear(12, 64),
            nn.Tanh(),
            nn.Linear(64, 128),
            nn.Tanh(),
            nn.Linear(128, 28*28),
            nn.Sigmoid()      
        )

    def forward(self, x):
        encoded = self.encoder(x)
        decoded = self.decoder(encoded)
        return encoded, decoded

autoencoder = AutoEncoder()
optimizer = torch.optim.Adam(autoencoder.parameters(), lr=LR)
loss_func = nn.MSELoss()

for epoch in range(EPOCH):
    for step, (x, b_label) in enumerate(train_loader):
        b_x = x.view(-1, 28*28)   # batch x, shape (batch, 28*28)
        b_y = x.view(-1, 28*28)   # batch y, shape (batch, 28*28)

        encoded, decoded = autoencoder(b_x)

        loss = loss_func(decoded, b_y)      
        optimizer.zero_grad()               
        loss.backward()                    
        optimizer.step()                    
```


<br>
# References

1. [Official PyTorch Tutorials](https://pytorch.org/tutorials/)，官方有很多用torch实现的例子，包括图像和文字方面的应用，以及GAN和RL。
2. [Morvan PyTorch Tutorials](https://morvanzhou.github.io/tutorials/machine-learning/torch/)



<br>