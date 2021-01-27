本周在上周的基础上进一步学习了FCN网络的内容，了解了其细节部分，理解了上采样的实现以及利用pytorch实现了FCN的网络模型。



## FCN网络模型解析

作者采用了Alex Net VGG16 GoogleNet分类网络作为基础网络模型进行测试。最好的是VGG16 上周已经介绍了具体情况，这里不再进行赘述。具体情况见下表。

![image-20200627133834073](D:\MarkDown\DeepLearning\img\image-20200627133834073.png)

作者使用的是VGG16做为预训练模型进行改造训练。

基础FCN

![image-20200627184005195](D:\MarkDown\DeepLearning\img\image-20200627184005195.png)



基础的FCN将VGG16的全连接层变为全卷积层，以输入的图为（224,224）为例子，经过VGG16卷积过后的大小为（7,7）那么就采用了卷积核为（7,7）大小的卷积核进行卷积。输出的大小通道为4096，此时的特征图为（4099,1,1），再使用（1,1）的卷积核进行卷积输出为（4096,1,1）大小 再进行上采样恢复到输入图的大小。实现了像素级别的分类。

但是基础的FCN由于进行了5次下采样，丢失了许多的细节信息，为了解决这些问题，作者将heatmap与之前卷积层数据进行相加补充丢失的细节信息。其中分为FCN32是、FCN16s、FCN8s，32、16、8代表上采样进行扩大的倍数 或者说在前面进行下采样进行maxpooing后的倍数。这里以FCN8s为例进行说明：



![FCN8](D:\MarkDown\DeepLearning\img\FCN8.png)



如上图所示，在进行基础的FCN卷积时不仅保存了最后的数据，也保存了pool3的数据和pool4的数据，通过转置卷积（逆卷积）进行上采样，使得pool3、pool4和最后输出的数据进行想加，最后再进行上采样扩大8倍，扩充到与输入图片的大小保持相等的状态。其他的类似。



核心代码 ：

```python
class FCN8(nn.Module):
    def __init__(self, pretrained_net, n_class): # pretrained_net 是进行修改后的VGG16代码 返回的是包括了5次maxpooling后数据的字典数据
        super().__init__()
        self.n_class = n_class
        self.dropout = nn.Dropout2d()

        self.pretrained_net = pretrained_net
        self.relu = nn.ReLU(inplace=True)
        self.fc6 = nn.Conv2d(512, 4096, 5)
        self.fc7 = nn.Conv2d(4096, 4096, 1)
        self.score_fr = nn.Conv2d(4096, n_class, 1)
        self.tr_conv_fr1 = nn.ConvTranspose2d(n_class, 512, kernel_size=4, stride=2, dilation=3)
        self.tr_conv_fr2 = nn.ConvTranspose2d(512, 256, kernel_size=3, stride=2, padding=1, dilation=1,output_padding=1)
        self.tr_conv_fr3 = nn.ConvTranspose2d(256, n_class, kernel_size=4, stride=8, padding=1, dilation=3)
        self.bn1 = nn.BatchNorm2d(512)
        self.bn2 = nn.BatchNorm2d(256)

    def forward(self, x):
        output = self.pretrained_net(x)
        x5 = output['x5']
        x4 = output['x4']
        x3 = output['x3']
        x5 = self.relu(self.fc6(x5))
        x5 = self.dropout(x5)
        x5 = self.relu(self.fc7(x5))
        score_fr = self.score_fr(x5)
        score = self.bn1(self.relu(self.tr_conv_fr1(score_fr)))
        score += x4
        score = self.bn2(self.relu(self.tr_conv_fr2(score)))
        score += x3
        score = self.tr_conv_fr3(score)
        return score
```



FCN的主要创新点就是利用了上采样实现了像素点级别的识别。下面是论文中提出的两种上采样的方式（逆卷积和双线性插值）。论文指出使用逆卷积效果要比双线性插值算法要好。逆卷积需要进行参数学习,双线性插值算法不需要。


###  逆卷积ConvTranspose2d

参考 https://blog.csdn.net/qq_27261889/article/details/86304061

在生成图像中，我们需要不断扩大图像的尺寸，目前在深度学习中ConvTranspose2d时其中的一个方法。convTranspose2d 是pytorch里的函数名字。论文中，可以称为fractionally-strided convolutions， 也有的称为deconvolutions,但是不建议用后一个，因为这个实际上是一个特殊的正向卷积，不是‘逆卷’积。



假设有一张图片A经过卷积从A变成了B 那么逆卷积就是从B在变成A。

举例如下:

**卷积**

有一张特征图为`4*4*channels_in` channels_in为输入的通道数

卷积核为`3*3` 步长为1

那么输出为`(4-3+1)*（4-3+1）*channels_out`  channels_out为输出的通道数

![卷积操作](D:\MarkDown\DeepLearning\img\conv.jpg)

**逆卷积**

当给一个特征图a, 以及给定的卷积核设置，我们分为三步进行逆卷积操作：

第一步：对输入的特征图a进行一些变换，得到新的特征图a’

第二步：求新的卷积核设置，得到新的卷积核设置，后面都会用右上角加撇点的方式区分

第三步：用新的卷积核在新的特征图上做**常规的卷积**，得到的结果就是逆卷积的结果，就是我们要求的结果。



特征图a  Height width

输入的卷积核为 kenel的size为$Size$ 步长为$Stride$ padding 为$padding$

新的特征图为
$$
Height'= Height+(Stride-1)*(Height-1)\\
Width'= Width+(Stride-1)*(Width-1)
$$
第一步中怎么变换？

我们在新的特征图上添加插值(interpolation)

插值插入的值时0

插在哪里？ 在原来的高度上每两个相邻的中间插上$Stride-1$列0 我们知道对于输入为$Height$的特征图有$Height-1$个位置。

新的卷积核$Stride'=1$这个会保持不变，卷积核与之前的保持不变仍然为$Size$ padding 为 $Size -padding-1$

推算出的公式为
$$
Height_{out} =(Height-1)*Stride-2padding+Size
$$


推导:
$$
卷积计算公式：
Height_{out} =\frac{Height_{in}+2padding'-kernel\_size'+1}{Stirde'}\\
Height'= Height+(Stride-1)*(Height-1)
\\
padding' = (Size-padding-1)
\\
kernel_size' = Size
\\
Stride' = 1
\\
带入计算可得 Height_{out} =(Height-1)*Stride-2padding+Size
$$



Pytorch api

![image-20200627171631012](D:\MarkDown\DeepLearning\img\image-20200627171631012.png)





### 双线性插值上采样

单线性上采样就是知道了两个点的值并将两个点连成一条直线来确定中间点的值，假设现在有两个点$(x_1,y_1)、(x_2,y_2)$连成一条直线。双线性插值是一个二维坐标系，因此需要找到四个点进行确定中心点坐标。

![在这里插入图片描述](D:\MarkDown\DeepLearning\img\20190819145111124.png)



计算公式如下：

![img](D:\MarkDown\DeepLearning\img\b1.jpg)

实现代码如下:

```python

def resize(src, new_size):
    dst_w, dst_h = new_size # 目标图像宽高
    src_h, src_w = src.shape[:2] # 源图像宽高
    if src_h == dst_h and src_w == dst_w:
        return src.copy()
    scale_x = float(src_w) / dst_w # x缩放比例
    scale_y = float(src_h) / dst_h # y缩放比例

    # 遍历目标图像，插值
    dst = np.zeros((dst_h, dst_w, 3), dtype=np.uint8)
    for n in range(3): # 对channel循环
        for dst_y in range(dst_h): # 对height循环
            for dst_x in range(dst_w): # 对width循环
                # 目标在源上的坐标
                # src_x = (dst_x + 0.5) * scale_x - 0.5
                src_x = dst_x*scale_x
                # src_y = (dst_y + 0.5) * scale_y - 0.5
                src_y = dst_y * scale_y
                # 计算在源图上四个近邻点的位置
                src_x_0 = int(np.floor(src_x))
                src_y_0 = int(np.floor(src_y))
                src_x_1 = min(src_x_0 + 1, src_w - 1)
                src_y_1 = min(src_y_0 + 1, src_h - 1)

                # 双线性插值
                value0 = (src_x_1 - src_x) * src[src_y_0, src_x_0, n] + (src_x - src_x_0) * src[src_y_0, src_x_1, n]
                value1 = (src_x_1 - src_x) * src[src_y_1, src_x_0, n] + (src_x - src_x_0) * src[src_y_1, src_x_1, n]
                dst[dst_y, dst_x, n] = int((src_y_1 - src_y) * value0 + (src_y - src_y_0) * value1)
    return dst


```

