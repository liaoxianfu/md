### Deep Residual Learning for Image Recognition



Kaiming He  Xiangyu Zhang  Shaoqing Ren  Jian Sun
Microsoft Research
{kahe, v-xiangz, v-shren, jiansun}@microsoft.com



#### 摘要

越深的神经网络越难进行训练。我们提出了一个残差学习框架去减缓网络训练。这种在比之前使用的网络更深。我们显式地用形式表示这些layers作为带有引用layer的输入残差函数去替代没有引用的函数。我们提供了大量的能够被证实的证据来证明这些残差网络是更贱容易被优化的。而且可以利用更深的网络来提高准确度。在ImageNet数据集中我们评估了最深可达152层的残差网络--是VGG网络深度的8倍但是仍然具有更低的复杂性。整体上这些残差网络在ImageNet数据集上取得了3.57%的误差。这个结果赢得了ILSVRC 2015分类任务比赛的第一名。我们同样也提供了100和1000层对CIFAR-10的分析。



representations的深度对于许多视觉识别任务至关重要。我们仅使用非常深的representations就在COCO目标检测数据集上相对提高了28%。深度残差网络是我们在ILSVRC2015和COCO2015比赛的基础。我们还赢得了ImageNet detection, ImageNet localization, COCO detection, and COCO segmentation.比赛的第一名。



#### 1、介绍



深度卷积神经网络已经为图像分类带来了一系列的突破[22,21]。深层网络以端到端的多层方式自然地集成低/中/高级别特征[50]和分类器，并且特征的“级别”可以通过堆叠的层(深度)的数量来丰富。最近的证据[41，44]揭示了网络深度是至关重要的，在具有挑战性的ImageNet数据集[36]上的领先结果[41，44，13，16]都利用了深度为16[41]到30[16]的“非常深”[41]模型。许多其他重要的视觉识别任务也已经从非常深的模型中得到了益处。

![image-20200714150441336](D:\MarkDown\DeepLearning\img\image-20200714150441336.png)

图1  CIFAR-10具有20层和56层“平面”网络的训练误差(左)和测试误差(右)。网络越深，训练误差越大，因此测试误差越大。在ImageNet上出现了类似的现象，如图4所示。



在深度重要性的推动下，一个问题出现了：学习更好的网络是否像堆叠更多层一样容易？回答这个问题的一个障碍是一个臭名昭著的“梯度爆炸与梯度消失”问题【1,9】，它从一开始就阻止收敛。然而，这个问题在很大程度上已经通过normalized initialization[23，9，37，13]和中间归一化层[16]来解决，这使得具有数十层的网络能够开始收敛于具有反向传播的随机梯度下降(SGD)[22]。

当更深的网络能够开始收敛时，降级问题就暴露了出来：随着网络深度的增加，精度达到饱和(这可能并不令人惊讶)，然后迅速降级。出乎意料的是，这种退化并不是由过度拟合引起的，而且在适当深度的模型上增加更多的层会导致更高的训练误差，正如[11，42]中所报告的那样，并被我们的实验彻底验证。图1显示了一个典型的示例。

(训练精度的)下降表明并不是所有的系统都同样容易优化。让我们考虑一个较浅的结构和它的深层次的对应物，它在上面增加了更多的层。对于更深层的模型，有一种构造的解决方案：添加的层是身份映射，其他层是从学习的浅层模型复制而来的。这种构造解的存在表明，较深的模型应该不会比较浅的模型产生更高的训练误差。但是实验表明我们现在靠手工解决事不能找到一个解决方案是相比更好（或者说在一个可行的时间范围内做到这一点）。

![image-20200717112811673](D:\MarkDown\DeepLearning\img\image-20200717112811673.png)

在本文中，我们通过引入深度残差学习框架来解决退化问题。我们明确的使用这些陈去拟合一个残差mapping而不是希望通过一些堆叠层去直接得到下面的层。在形式上我们将下面的映射结果记为$\mathcal{H(x)}$我们将非线性的另一个映射为$\mathcal{F(x):=H(x)-x}$ 原始的映射就变为$\mathcal{F(x)+x}$。我们假设优化残差网络比原始的未进行引用的网络更加容易。在极端情况下，如果单位映射时最优的，那么残差网络的梯度降为0要比通过一堆非线性层来拟合单位映射容易得多。

$\mathcal{F(x)+x}$的式可以通过具有“捷径连接”的前馈神经网络来实现(图2)。Shortcut connections（快捷连接）[2, 34, 49] 是那些跳过一个或多个层的连接。在我们的例子中，Shortcut connections（快捷连接）只是执行身份映射，并且它们的输出被添加到堆叠层的输出(图2)。实际上快捷连接既不会增加额外参数，也不会增加计算复杂性。整个网络仍然可以由SGD使用反向传播进行端到端的训练，并且可以使用通用库(例如，Caffe[19])轻松实现，而无需修改解算器。

我们在ImageNet[36]上进行了全面的实验，以显示退化问题并评估我们的方法。结果表明：1)极深残差网络易于优化，但对应的“普通”网络(简单的堆叠层)在深度增加时表现出更高的训练误差；2)深残差网络可以很容易地从深度的大幅增加中获得精度提升，其结果明显好于以往的网络。在CIFAR-10数据集上也展示了显示的情况。这表明了我们的方法的优化难度和效果不仅仅在特定数据集上有效。我们在这个数据集上提供了超过100层的成功训练的模型，并探索了超过1000层的模型。在ImageNet分类数据集上我们通过非常深的残差网络取得了优秀的成绩。我们的152层剩余网是ImageNet上出现过的最深的网络，但其复杂度仍然低于VGG网[41]。我们总体上取得了在ImageNet测试集top-5 中==3.57%==的错误率额日期额赢得了*ILSVRC 2015 classification competition.*的第一名。极深的表示在其他识别任务上也有很好的泛化性能，并带领我们在ILSVRC和COCO 2015大赛中进一步获得了ImageNet检测、ImageNet本地化、COCO检测和COCO分割的第一名。这有力的证据表明，残差学习原理是通用的，我们期望它也适用于其他视觉和非视觉问题。





#### 2、 相关工作