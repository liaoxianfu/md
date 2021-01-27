## 自动求导



PyTorch在autograd模块中实现了计算图的相关功能，autograd中的核心数据结构是Variable。从v0.4版本起，Variable和Tensor合并。我们可以认为需要求导(requires_grad)的tensor即Variable. autograd记录对tensor的操作记录用来构建计算图。

Variable提供了大部分tensor支持的函数，但其不支持部分`inplace`函数，因这些函数会修改tensor自身，而在反向传播中，variable需要缓存原来的tensor来计算反向传播梯度。如果想要计算各个Variable的梯度，只需调用根节点variable的`backward`方法，autograd会自动沿着计算图反向传播，计算每一个叶子节点的梯度。

### 求导

指定参数可以进行求解梯度

```python
#在创建tensor的时候指定requires_grad
a = t.randn(3,4, requires_grad=True)
# 或者
a = t.randn(3,4).requires_grad_()
# 或者
a = t.randn(3,4)
a.requires_grad=True
a
```

假设进行梯度求解 $d = sum(c=(a+b))$ 

也就是

```python
a = t.randn(3,4, requires_grad=True) # a允许求导
b = t.zeros(3,4).requires_grad_() # b允许求导
c = a+b # 没有指定c能够求导
d = c.sum() 
d.backward() #进行反向传播
```

```python
d # d还是一个requires_grad=True的tensor,对它的操作需要慎重
d.requires_grad # True
```

```python
a.grad
```

```
tensor([[1., 1., 1., 1.],
        [1., 1., 1., 1.],
        [1., 1., 1., 1.]])
```

```python
# 此处虽然没有指定c需要求导，但c依赖于a，而a需要求导，
# 因此c的requires_grad属性会自动设为True
a.requires_grad, b.requires_grad, c.requires_grad
```

```python
# 由用户创建的variable属于叶子节点，对应的grad_fn是None
a.is_leaf, b.is_leaf, c.is_leaf

# (True, True, False)
```

求导 
$$
y=x^2*e^x
$$
求导结果
$$
{dy \over dx}=2x *e^x + x^2 * e^x
$$

```python
def f(x):
    '''计算y'''
    y = x**2 * t.exp(x)
    return y

def gradf(x):
    '''手动求导函数'''
    dx = 2*x*t.exp(x) + x**2*t.exp(x)
    return dx
```

```python
x = t.randn(3,4, requires_grad = True)
y = f(x)
y
'''
tensor([[1.7010e-03, 1.7171e+00, 2.9237e-01, 1.1170e-01],
        [1.4809e-03, 2.6344e+01, 1.3164e-01, 4.0763e+00],
        [3.2292e-01, 2.2965e+01, 5.3342e-01, 5.3551e-01]],
       grad_fn=<MulBackward0>)
'''
```

手动计算梯度

```python
gradf(x) 
tensor([[-0.0791,  5.7351, -0.4285,  0.8841],
        [-0.0740, 53.4620, -0.4462, 11.2205],
        [-0.4075, 47.4507, -0.0703, -0.0598]], grad_fn=<AddBackward0>)
```

自动求导

```python
y.backward(t.ones(y.size())) # gradient形状与y一致
x.grad
'''
tensor([[-0.0791,  5.7351, -0.4285,  0.8841],
        [-0.0740, 53.4620, -0.4462, 11.2205],
        [-0.4075, 47.4507, -0.0703, -0.0598]])
'''
```

### 计算图

PyTorch中`autograd`的底层采用了计算图，计算图是一种特殊的有向无环图（DAG），用于记录算子与变量之间的关系。一般用矩形表示算子，椭圆形表示变量。如表达式$𝐳 = 𝐰𝐱 + 𝐛 $可分解为$𝐲 = 𝐰𝐱$和$𝐳 = 𝐲 + 𝐛$，其计算图如下所示，图中`MUL`，`ADD`都是算子，𝐰，𝐱，𝐛即变量。

![](D:\MarkDown\DeepLearning\img\com_graph.svg)

对$z=wx+b$进行求导

$${\partial z \over \partial b} = 1,\space {\partial z \over \partial y} = 1\\
{\partial y \over \partial w }= x,{\partial y \over \partial x}= w\\
{\partial z \over \partial x}= {\partial z \over \partial y} {\partial y \over \partial x}=1 * w\\
{\partial z \over \partial w}= {\partial z \over \partial y} {\partial y \over \partial w}=1 * x\\$$



在反向传播中非叶子节点的导数计算完成后立即被清空如果想要查看这些变量可以使用使用autograd.grad函数或者hook

```python
# 第一种方法：使用grad获取中间变量的梯度
x = t.ones(3, requires_grad=True)
w = t.rand(3, requires_grad=True)
y = x * w
z = y.sum()
# z对y的梯度，隐式调用backward()
print(t.autograd.grad(z, y))
print(t.autograd.grad(z,x))
# print(t.autograd.grad(z,w))
```

```python
# 第二种方法：使用hook
# hook是一个函数，输入是梯度，不应该有返回值
def variable_hook(grad):
    print('y的梯度：',grad)

x = t.ones(3, requires_grad=True)
w = t.rand(3, requires_grad=True)
y = x * w
# 注册hook
hook_handle = y.register_hook(variable_hook)
z = y.sum()
z.backward()

# 除非你每次都要用hook，否则用完之后记得移除hook
hook_handle.remove()
```

### 扩展autograd



自定义autograd实现反向求导

```python
class Mul(Function):

    @staticmethod
    def forward(ctx, w, x, b, x_requires_grad = True):
        ctx.x_requires_grad = x_requires_grad
        ctx.save_for_backward(w,x)
        output = w * x + b
        return output

    @staticmethod
    def backward(ctx, grad_output):
        w,x = ctx.saved_tensors
        grad_w = grad_output * x
        if ctx.x_requires_grad:
            grad_x = grad_output * w
        else:
            grad_x = None
        grad_b = grad_output * 1
        return grad_w, grad_x, grad_b, None
```



自定义的Function需要继承autograd.Function，没有构造函数`__init__`，forward和backward函数都是静态方法

backward函数的输出和forward函数的输入一一对应，backward函数的输入和forward函数的输出一一对应

backward函数的grad_output参数即t.autograd.backward中的`grad_variables`

如果某一个输入不需要求导，直接返回None，如forward中的输入参数x_requires_grad显然无法对它求导，直接返回None即可

反向传播可能需要利用前向传播的某些中间结果，需要进行保存，否则前向传播结束后这些对象即被释放