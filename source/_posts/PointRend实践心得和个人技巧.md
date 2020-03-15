---
title: PointRend 实践心得
date: 2020-03-14 22:30:30
tags:
- 技术
- 语义分割
---

  本篇主要记录关于最近比较火的一篇kaiming大神挂名的论文：PointRend: Image Segmentation as Rendering在实际使用过程中的一些心得技巧。[论文链接-arxiv](https://arxiv.org/pdf/1912.08193.pdf)
<!-- more -->
#### Intro
&ensp;2个月前接到新的项目需要对某场景进行语义分割，目前大多数semantic segmentation和instance segmentation的模型在实际生产环境中的表现并未达到可用水平，主要因素来自于performance，inference speed，labeling等，而且各大benchmark榜上以mIoU为主要的评价维度很多时候并不能准确地评价算法模型在使用场景中的可用性程度。恰好碰上这篇论文，于是花了数天时间把这个模块加到原本的模型后面跑了下训练，发现效果出奇的好。该算法代码已经集成在detectron2的开源库里面。

#### Method
&ensp;首先介绍一下论文内容。读完文章后不难会发现，该算法实质上与文章中描述的Rendering Algorithm并没有太大的关系，个人认为甚至这是作者的障眼法，反而原文开头部分对算法的描述似乎更精准：
> a module that performs point-based segmentation predictions at adaptively selected locations based on an iterative subdivision algorithm.

&ensp;对于实际应用场景而言，如文章中所述，目前在精度上的瓶颈来自于边缘位置分类的稳定性(参考下图)，这种瓶颈并不是因为网络本身的特征提取能力不足，而是在于网络连续的down/up sample带来的信息丢失，所以单纯的增加参数量已经不是可取之道。
![p0](pointrend_0.png)
&ensp;我个人理解该文章是借鉴了objects detection的思路，把segmentation也做成2 stage来提高精度，所以也能应用在instance segmentation任务。其本质可以说是在特征层面针对样本边缘难分点的OHEM，它的精妙之处在于区别开了training和inference的过程，在保证了inference的速度同时也提高了分类精度。

![p1](pointrend_1.png) ![p2](pointrend_2.png)
&ensp;算法过程如上两图，training过程中，该模块在网络最后的输出层上面随机取点(称为coarse prediction layer),再抽取网络backbone中仍然还带有比较丰富feature信息的一层(原文中叫fine-grained features)同样位置的点。之后把两种采样点的特征concat起来后，经过一个mlp处理(detectron2中只使用一个1x1的conv层)然后把结果替代原来的点作为该尺度下的输出，最后通过在多次上采样并重复这个过程到输出尺度。
&ensp;此外，loss function需要加入了一项points loss来对前面的mlp优化。而在inference过程中,把training的随机取点过程替代为在整个feature map上取topk个最难分的点(即在输出feathre map中取最大的两个channel的值最接近的topk点)，topk点的数量可以根据性能要求来调整，原文建议每一层抽取了14*14个点。文章还探讨了不同采样点方案对结果的影响，具体参考原文。

#### Experiment
&ensp;原文在COCO数据集上，与mask rcnn的结果进行对比，虽然结果的mIoU提升了只有1-2个点，但是在边缘位置是肉眼可见的提升。

![p3](pointrend_3.png)

&ensp;而在我自己的任务中，简单的二类分割任务在加入此模块后mIoU从95.4%提高至98.8%，更重要的是，over sampling 方法使场景分类稳定性大大提高。而在网络结构上,detectron2的代码中只使用一个1x1 conv作为模块的mlp，个人觉得觉得可以再后接了一层Elu(对于有速度要求的任务，可尝试用elu替换bn+Relu，效果往往不差)。其次个人发现把backbone结果先上采样层到fine-grained层同尺度作为coarse output再与fine-grained concat起来才开始迭代，比起原文中从最高层直接来concat收敛速度会快很多，猜测是由于concat两边的channel数相近能使mlp比较“均匀”地学习到特征。

#### Conclusion
&ensp;个人觉得这个方法以后可以成为segmentation任务的一个common trick，而从rcnn到deeplabv3，语义分割已经达到了80分可用水平，但要真正大规模在在工业应用还有很多工作，下一步会尝试下加入self attention看看两者能否互相产生些新的效果。









