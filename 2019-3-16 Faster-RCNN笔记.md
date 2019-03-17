---
title: 2019-3-16 Faster-RCNN笔记 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# Faster-RCNN
## 相关工作
### object proposals (对象提议,目标推荐)
```markdown
There is a large literature on object
proposal methods. Comprehensive surveys and com-
parisons of object proposal methods can be found in
[19], [20], [21]. Widely used object proposal methods
include those based on grouping super-pixels (e.g.,
Selective Search [4], CPMC [22], MCG [23]) and those
based on sliding windows (e.g., objectness in windows
[24], EdgeBoxes [6]). Object proposal methods were
adopted as external modules independent of the de-
tectors (e.g., Selective Search [4] object detectors, R-
CNN [5], and Fast R-CNN [2])
```
### Deep Networks for Object Detection 目标检测中的深度网络

R-CNN只相当于一个分类器,他并不进行对边界的预测和计算.因而,需要一个额外的边界框算法(bounding box regression)进行进行精炼(refine).对于边界框选定,曾经采用过全连接来对单目标进行识别,后来发展到使用卷积层识别多目标.

之后出现了一个算法杂糅了二者,论文对其介绍是:
The MultiBox methods [26], [27] gen-
erate region proposals from a network whose last
fully-connected layer simultaneously predicts mul-
tiple class-agnostic boxes, generalizing the “single-
box” fashion of OverFeat. These class-agnostic boxes
are used as proposals for R-CNN [5]. The MultiBox
proposal network is applied on a single image crop or
multiple large image crops (e.g., 224×224), in contrast
to our fully convolutional scheme.
# faster R-CNN
整体主要分为两个部分,一个模块(RPN)用来寻找目标(propose regions),另一部分(fast R-CNN)用来识别目标.如下图所示:
![enter description here](./images/1552743902254.png)
## Region Proposal Networks
> A Region Proposal Network (RPN) takes an image
(of any size) as input and outputs a set of rectangular
object proposals, each with an objectness score.

该网络负责将任意尺寸的图片作为输入并输出一堆表现出目标和背景之间差距度的子图片.目标的特性就是通过每个区域里目标和子区间之间的差距程度实现的.
同时,为了减少计算量,该算法将RPN网络与Fastrcnn网络的卷积模块合并到了一起(可以从图中看出来).
为了实现目标建议的工作,RPN网络采用的方式是将一个小的窗口(窗口的空间定义为N\*N)在经过了卷积模块之后取得的特征映射(feature map)上进行滑动.可以看出,每一个小的窗口都映射对应着一个低维度特征.之后将这个特征送入两个同胞(sibling)全连接层(一个是用来进行框回归(box regression)的,一个是用来进行框分类的(box classification))中.
对于177和228pixel值的ZF和VGG,论文中采用N=3,认为这时可以得到比较好的感受野()
### Anchors(锚,也就是RPN中作者的思路)
下图表示的是RPN网络大意.
![enter description here](./images/1552746493248.png)
如果假设在其中一个滑动窗口中有k个可疑目标,则对于Boxreg网络需要有4k个输出来表明其坐标,在BoxCls中需要有2K个输出来表明其分类结果(这是针对2分类问题而言的).作者将此联系称作"锚"(不太理解).
```
An anchor is centered at the sliding window
in question, and is associated with a scale and aspect
ratio (Figure 3, left). By default we use 3 scales and
3 aspect ratios, yielding k = 9 anchors at each sliding
position. For a convolutional feature map of a size
W × H (typically ∼2,400), there are W Hk anchors in
total.
```
一个锚位于一个滑动窗口的中心,并且它可由窗口的尺度(scale)和纵横比(aspect ratio)确定.
## Translation Invariant Anchors
此部分略.重点是在说这个算法有多好多好.
```
If one translates an object in an image,
the proposal should translate and the same function
should be able to predict the proposal in either lo-
cation.
```
## Multi-scale Anchors as regression Reference
此处是在讨论如何针对不同尺度的目标对象进行建模的问题.论文指出目前总共有两种思路:基于图像金字塔的和基于划窗的.前者是先将图片resize成几个不同尺寸的图片,之后进行特征提取.这种方式比较耗时.另一种思路是采用不同大小(或者aspect ratio)的划窗遍历整个feature map,这种方式多代表了不同尺寸的滤波器.
而作者的思路是这二者的融合.
```
As a comparison, our anchor-based method is built
on a pyramid of anchors, which is more cost-efficient.
Our method classifies and regresses bounding boxes
with reference to anchor boxes of multiple scales and
aspect ratios. It only relies on images and feature
maps of a single scale, and uses filters (sliding win-
dows on the feature map) of a single size. We show by
experiments the effects of this scheme for addressing
multiple scales and sizes (Table 8).
```
## Loss functions
解决的是如何训练RPN网络的问题,作者指出正确的anchor应该具有的特性:
> We as-
sign a positive label to two kinds of anchors: (i) the
anchor/anchors with the highest Intersection-over-
Union (IoU) overlap with a ground-truth box, or (ii) an
anchor that has an IoU overlap higher than 0.7 with5
any ground-truth box.

坏的anchor的特性:
> We
assign a negative label to a non-positive anchor if its
IoU ratio is lower than 0.3 for all ground-truth boxes.
Anchors that are neither positive nor negative do not
contribute to the training objective

由此可得损失函数的定义为
![enter description here](./images/1552828275906.png)
其中,i是在一个minibatch中的anchor的索引,p_i是该anchor成功预测了一个目标的可能性,t_i代表的是一个存储了boundingbox坐标信息的四维向量,N_cls和N_reg均为正则项,作者说后面会证明这个可以省略掉.
关于坐标,作者给出了下面的公式
![enter description here](./images/1552828713843.png)
可以看出这是比较常规的做法,其中无下标(例如x)指的是要被预测的box的横坐标,小标a表示的是anchor的量,上标\*代表的是先验值.

下面将阅读源码,并尝试自己进行修改.