## pytorch tensor操作



### 创建tensor

![image-20200530104149056](D:\MarkDown\DeepLearning\img\image-20200530104149056.png)

创建自定形状的tensor

```python
torch.Tensor(2,3) #传入的是形状数据
```

创建指定数据的tensor

```python
# 用list的数据创建tensor 不建议使用 
b = t.Tensor([[1,2,3],[4,5,6]])
```

将tensor转换为python的list

```python
b.tolist() # 把tensor转为list
```

使用tensor方法创建指定数据的tensor

```python
t.tensor([[1,2,3],[4,5,6]])
```

tensor.size()返回的是torch.Size对象是tuple的子类

```python
b_size = b.size()
b_size # torch.Size([2, 3])
# 创建一个和b形状一样的tensor
c = t.Tensor(b_size)
```

函数使用

```python
t.arange(1, 6, 2) # tensor([1, 3, 5])
t.linspace(1, 10, 3) #tensor([ 1.0000,  5.5000, 10.0000])
t.randn(2, 3, device=t.device('cpu')) #标准分布
t.randperm(5) # 长度为5的随机排列 shuffle
```

### 操作tensor

调整tensor形状

```python
a = t.arange(0, 6)
a.view(2, 3)

'''
tensor([[0, 1, 2],
        [3, 4, 5]])
'''
```

```python
b = a.view(-1, 2) # 当某一维为-1的时候，会自动计算它的大小
b.shape #torch.Size([3, 2])
```

索引操作

与操作python list几乎没有差距

```python
a = t.randn(3, 4)
# a.size() 等价 a.shape
a.shape
```

行操作

```python
a[0] # 第0行(下标从0开始)
```

列操作

```python
a[:, 0] # 第0列
```

元素操作

```python
a[0][2] # 第0行第2个元素，等价于a[0, 2]
a[0, -1] # 第0行最后一个元素
a[:2] # 前两行
a[:,:2] #前两列
a[:2, 0:2] # 前两行，第0,1列
```

### Tensor类型

![image-20200530111308851](D:\MarkDown\DeepLearning\img\image-20200530111308851.png)

Tensor有不同的数据类型，如上图所示，每种类型分别对应有CPU和GPU版本(HalfTensor除外)。默认的tensor是FloatTensor，可通过`t.set_default_tensor_type` 来修改默认tensor类型(如果默认类型为GPU tensor，则所有操作都将在GPU上进行)。

```python
# 设置默认tensor，注意参数是字符串 参数见上图
t.set_default_tensor_type('torch.DoubleTensor')
```

### 逐元素操作

![image-20200530111907039](D:\MarkDown\DeepLearning\img\image-20200530111907039.png)

这部分操作会对tensor的每一个元素(point-wise，又名element-wise)进行操作，此类操作的输入与输出形状一致。常用的操作如上图所示。

### Tensor与Numpy相互转换

Numpy转换为Tensor



```python
import numpy as np
import torch as t
a = np.ones([2, 3],dtype=np.float32)
b = t.from_numpy(a)
# 或者
b = t.Tensor(a) # 也可以直接将numpy对象传入Tensor
```

**Numpy与Tensor共享内存**

```python
a[0, 1]=100
b

'''
tensor([[  1., 100.,   1.],
        [  1.,   1.,   1.]])
'''
```

Tensor转Numpy

```python
c = b.numpy() # a, b, c三个对象共享内存
c
'''
array([[  1., 100.,   1.],
       [  1.,   1.,   1.]], dtype=float32)
'''
```

**当numpy的数据类型和Tensor的类型不一样的时候，数据会被复制，不会共享内存。**

```python
a = np.ones([2, 3])
# 注意和上面的a的区别（dtype不是float32）
a.dtype
```

````python
b = t.Tensor(a) # 此处进行拷贝，不共享内存
b.dtype # torch.float32
````

```python
c = t.from_numpy(a) # 注意c的类型（DoubleTensor）
c

'''
tensor([[1., 1., 1.],
        [1., 1., 1.]], dtype=torch.float64)
'''
```

```python
a[0, 1] = 100
b # b与a不共享内存，所以即使a改变了，b也不变
c # c与a共享内存
```

在进行函数的参数传递的时候因为是引用传递，所以修改的也是原来的函数。

### CPU/GPU切换

tensor可以很随意的在gpu/cpu上传输。使用`tensor.cuda(device_id)`或者`tensor.cpu()`。另外一个更通用的方法是`tensor.to(device)`。

默认使用的是CPU

```python
a = t.randn(3, 4)
a.device 
# device(type='cpu')
```

判断是否能够使用GPU

```python
if t.cuda.is_available():
    print("yes")
    a = t.randn(3,4, device=t.device('cuda:0'))
    # 等价于
    # a.t.randn(3,4).cuda(0)
    # 但是前者更快
    print(a.device)
```

指定运行设备

```python
device = t.device('cuda:0')
a.to(device)
```

数据持久化

Tensor的保存和加载十分的简单，使用t.save和t.load即可完成相应的功能。在save/load时可指定使用的`pickle`模块，在load时还可将GPU tensor映射到CPU或其它GPU上。

```python
b = t.load('a.pth')
print(b) # 加载的数据也存储在gpu上 在序列化的时候也存储了device的信息

'''
tensor([[ 1.1949, -1.6040,  1.0460,  0.4037],
        [ 0.6720,  0.3330,  0.2608, -0.9307],
        [-1.1393,  0.6878,  0.1714,  0.7111]], device='cuda:0')
'''
```

去除device信息

```python
def f(storage,loc):
#     print(storage)
    print(loc)
    return storage
    
c = t.load('a.pth',map_location=f)
print(c)

'''
返回的时候 没有返回社别的额信息 会默认使用cpu
cuda:0
tensor([[ 1.1949, -1.6040,  1.0460,  0.4037],
        [ 0.6720,  0.3330,  0.2608, -0.9307],
        [-1.1393,  0.6878,  0.1714,  0.7111]])
'''

```

### 向量化

在科学计算中应尽量避免Python原生的for循环，使用向量化操作，也就是调用api进行操作。

```python
# 使用原生的for循环进行操作
def for_loop_add(x, y):
    result = []
    for i,j in zip(x, y):
        result.append(i + j)
    return t.Tensor(result)
# 使用api进行操作
x = t.zeros(100)
y = t.ones(100)
```

利用jupyter notebook进行测试

```python
%timeit -n 100 for_loop_add(x, y)
%timeit -n 100 x + y
```

> ```
> 1.19 ms ± 31.4 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
> The slowest run took 4.94 times longer than the fastest. This could mean that an intermediate result is being cached.
> 6.49 µs ± 4.29 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
> ```





