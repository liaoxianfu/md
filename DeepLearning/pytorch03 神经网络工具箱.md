## 神经网络工具 nn

torch.nn 是为深度学习设计的模块，核心数据结构是Module，它是一个抽象的概念既可以表示Layer也可以表示多个Layer的神经网络。在实际中常见的是继承`nn.Moudle`,设计自己的网络结构。

### 设计全连接层

```python
import torch as t
import torch.nn as nn


class Linear(nn.Module):
    def __init__(self, input_shape, output_shape):
        super(Linear, self).__init__()
        # 随机初始化w b 参数
        self.w = nn.Parameter(t.randn(input_shape, output_shape), requires_grad=True)
        self.b = nn.Parameter(t.randn(output_shape), requires_grad=True)

    def forward(self, x: t.Tensor):
        x = x.mm(self.w)
        return x + self.b.expand_as(x)


if __name__ == '__main__':
    layer = Linear(4, 3)
    a = t.randn(1, 4)
    out = layer(a)
    print(out)
    # 收集参数
    for name, parameter in layer.named_parameters():
        print(name, "-----", parameter)
```

自定义层`Linear`必须继承`nn.Module`，并且在其构造函数中需调用`nn.Module`的构造函数，即`super(Linear, self).__init__()` 或`nn.Module.__init__(self)`，推荐使用第一种用法，尽管第二种写法更直观。

在构造函数`__init__`中必须自己定义可学习的参数，并封装成`Parameter`，如在本例中我们把`w`和`b`封装成`parameter`。`parameter`是一种特殊的`Tensor`，但其默认需要求导（requires_grad = True）

`forward`函数实现前向传播过程，其输入可以是一个或多个tensor。

无需写反向传播函数，nn.Module能够利用autograd自动实现反向传播，这点比Function简单许多。

使用时，直观上可将layer看成数学概念中的函数，调用layer(input)即可得到input对应的结果。它等价于`layers.__call__(input)`，在`__call__`函数中，主要调用的是 `layer.forward(x)`，另外还对钩子做了一些处理。所以在实际使用中应尽量使用`layer(x)`而不是使用`layer.forward(x)`，关于钩子技术将在下文讲解。

`Module`中的可学习参数可以通过`named_parameters()`或者`parameters()`返回迭代器，前者会给每个parameter都附上名字，使其更具有辨识度。

Module能够自动检测到自己的`Parameter`，并将其作为学习参数。除了`parameter`之外，Module还包含子`Module`，主Module能够递归查找子`Module`中的`parameter`。下面再来看看稍微复杂一点的网络，多层感知机。

```python
import torch as t
import torch.nn as nn


class Linear(nn.Module):
    def __init__(self, input_shape, output_shape):
        super(Linear, self).__init__()
        # 随机初始化w b 参数
        self.w = nn.Parameter(t.randn(input_shape, output_shape), requires_grad=True)
        self.b = nn.Parameter(t.randn(output_shape), requires_grad=True)

    def forward(self, x: t.Tensor):
        x = x.mm(self.w)
        return x + self.b.expand_as(x)


class Perceptron(nn.Module):
    def __init__(self, input_shape, hidden_shape, output_shape):
        super(Perceptron, self).__init__()
        # 多层layer
        self.layer1 = Linear(input_shape, hidden_shape)
        self.layer2 = Linear(hidden_shape, output_shape)

    def forward(self, x: t.Tensor):
        x = self.layer1(x)
        x = t.sigmoid(x)  # 激活函数
        x = self.layer2(x)
        return x


if __name__ == '__main__':
    layer = Perceptron(4, 10, 1)
    data = t.randn(1, 4)
    print(layer(data))
    for name,paramter in layer.named_parameters():
        print(name,"-----",paramter.size())

```

### 常用的神经网络层

为方便用户使用，PyTorch实现了神经网络中绝大多数的layer，这些layer都继承于nn.Module，封装了可学习参数`parameter`，并实现了forward函数，且很多都专门针对GPU运算进行了CuDNN优化，其速度和性能都十分优异。可以阅读官方文档：https://pytorch.org/docs/stable/nn.html

- 构造函数的参数，如nn.Linear(in_features, out_features, bias)，需关注这三个参数的作用。
- 属性、可学习参数和子module。如nn.Linear中有`weight`和`bias`两个可学习参数，不包含子module。
- 输入输出的形状，如nn.linear的输入形状是(N, input_features)，输出为(N，output_features)，N是batch_size。

除了卷积层和池化层，深度学习中还将常用到以下几个层：

- Linear：全连接层。
- BatchNorm：批规范化层，分为1D、2D和3D。除了标准的BatchNorm之外，还有在风格迁移中常用到的InstanceNorm层。
- Dropout：dropout层，用来防止过拟合，同样分为1D、2D和3D。 下面通过例子来说明它们的使用。
- 

```python
# 输入 batch_size=2，维度3
input = t.randn(2, 3)
linear = nn.Linear(3, 4)
h = linear(input)

# 4 channel，初始化标准差为4，均值为0
bn = nn.BatchNorm1d(4)
bn.weight.data = t.ones(4) * 4
bn.bias.data = t.zeros(4)

bn_out = bn(h)
# 注意输出的均值和方差
# 方差是标准差的平方，计算无偏方差分母会减1
# 使用unbiased=False 分母不减1
bn_out.mean(0), bn_out.var(0, unbiased=False)

# 每个元素以0.5的概率舍弃
dropout = nn.Dropout(0.5)
o = dropout(bn_out)
o # 有一半左右的数变为0
```

### 激活函数

Pytorch实现了常见的激活函数  http://pytorch.org/docs/nn.html#non-linear-activations。激活函数可以当成单独的layer进行使用。



### 前馈神经网络

在以上的例子中，基本上都是将每一层的输出直接作为下一层的输入，这种网络称为前馈传播网络（feedforward neural network）。对于此类网络如果每次都写复杂的forward函数会有些麻烦，在此就有两种简化方式，ModuleList和Sequential。其中Sequential是一个特殊的module，它包含几个子Module，前向传播时会将输入一层接一层的传递下去。ModuleList也是一个特殊的module，可以包含几个子module，可以像用list一样使用它，但不能直接把输入传给ModuleList。

###  损失函数

在深度学习中要用到各种各样的损失函数（loss function），这些损失函数可看作是一种特殊的layer，PyTorch也将这些损失函数实现为`nn.Module`的子类。然而在实际使用中通常将这些loss function专门提取出来，和主模型互相独立。

文档地址 http://pytorch.org/docs/nn.html#loss-functions

### 优化器

PyTorch将深度学习中常用的优化方法全部封装在`torch.optim`中，其设计十分灵活，能够很方便的扩展成自定义的优化方法。

所有的优化方法都是继承基类`optim.Optimizer`，并实现了自己的优化步骤。下面就以最基本的优化方法——随机梯度下降法（SGD）举例说明。这里需重点掌握：

- 优化方法的基本使用方法
- 如何对模型的不同部分设置不同的学习率
- 如何调整学习率

```python
from torch import  optim
optimizer = optim.SGD(params=net.parameters(), lr=1)
optimizer.zero_grad() # 梯度清零，等价于net.zero_grad()

input = t.randn(1, 3, 32, 32)
output = net(input)
output.backward(output) # fake backward

optimizer.step() # 执行优化
```

设置不同的学习率

```python
# 为不同子网络设置不同的学习率，在finetune中经常用到
# 如果对某个参数不指定学习率，就使用最外层的默认学习率
optimizer =optim.SGD([
                {'params': net.features.parameters()}, # 学习率为1e-5
                {'params': net.classifier.parameters(), 'lr': 1e-2}
            ], lr=1e-5)
optimizer
```

### 保存参数

在PyTorch中保存模型十分简单，所有的Module对象都具有state_dict()函数，返回当前Module所有的状态数据。将这些状态数据保存后，下次使用模型时即可利用`model.load_state_dict()`函数将状态加载进来。优化器（optimizer）也有类似的机制，不过一般并不需要保存优化器的运行状态。

```python
# 保存模型
t.save(net.state_dict(), 'net.pth')

# 加载已保存的模型
net2 = Net()
net2.load_state_dict(t.load('net.pth'))
```

### 使用GPU训练

将Module放在GPU上运行也十分简单，只需两步：

- model = model.cuda()：将模型的所有参数转存到GPU
- input.cuda()：将输入数据也放置到GPU上

