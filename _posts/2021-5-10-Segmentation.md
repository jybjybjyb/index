---
layout: post
title: Segmentation
date: 2021-5-10
author: 
tags: [cv, note]
comments: true
toc: true
pinned: False
---

<!-- more -->

# 分割


![](https://pic2.zhimg.com/v2-9330a39fb41f28b242070be5ed6bb83d_r.jpg)


- 图像分割可以分为两类：语义分割（Semantic Segmentation）和实例分割（Instance Segmentation）
- 语义分割只是简单地对图像中各个像素点分类
- 实例分割更进一步，需要区分开不同物体，这更加困难，从一定意义上来说，实例分割更像是语义分割加检测。    
- CNN 分类模型全连接得最终分类，但是中间的特征图位置信息丢失。
- 因为我们需要给出图像不同位置的分类概率，特征图过小时会损失位置信息。
- 然而，下采样必不可少。下采样层对于提升感受野非常重要，这样高层特征语义更丰富，而且对于分割来说较大的感受野也至关重要；另外的一个现实问题，没有下采样层，特征图一直保持原始大小，计算量是非常大的。

## 流派奥义

![](https://pic1.zhimg.com/v2-a7b7d8c875dca06839a669d950aa541c_r.jpg)

- 全卷积网络 FCN，此处有误，FCN第一次提出就有反卷积
- EncoderDecoder 结构，其中 Encoder 就是下采样模块，负责特征提取，而 Decoder 是上采样模块（通过插值，转置卷积等方式），负责恢复特征图大小，一般两个模块是对称的，经典的网络如 U-Net
- DilatedFCN，主要是通过空洞卷积（Atrous Convolution）来减少下采样率但是又可以保证感受野，如DeepLab

### DeepLab

![](https://pic1.zhimg.com/v2-9949c7916b7d8fcf2b6a99e22f416c28_r.jpg)

- Encoder 的主体是带有空洞卷积的 DCNN，可以采用常用的分类网络如 ResNet
- 金字塔池化模块（Atrous Spatial Pyramid Pooling, ASPP)，引入多尺度信息
- v3 + 引入了 Decoder 模块，其将底层特征与高层特征进一步融合，提升分割边界准确度

#### DilatedFCN

![](https://pic1.zhimg.com/v2-2fd4385e4698bdf45134148fe88e1edc_r.jpg)


- 修改分类网络的后面 block，用空洞卷积来替换 stride=2 的下采样层
- 加上 ASPP 结构



#### 空间金字塔池化（ASPP）

![](https://pic2.zhimg.com/v2-bfa712ca4922174106c72153d9de2d2d_r.jpg)

ASPP 模块主要包含以下几个部分： 
1. 一个 1×1 卷积层，以及三个 3x3 的空洞卷积，对于 output_stride=16，其 rate 为 (6, 12, 18) ，若 output_stride=8，rate 加倍（这些卷积层的输出 channel 数均为 256，并且含有 BN 层）；
1. 一个全局平均池化层得到 image-level 特征，然后送入 1x1 卷积层（输出 256 个 channel），并双线性插值到原始大小
1. 将（1）和（2）得到的 4 个不同尺度的特征在 channel 维度 concat 在一起，然后送入 1x1 的卷积进行融合并得到 256-channel 的新特征。

#### Decoder

![](https://pic2.zhimg.com/v2-fa76d61cfeeab3302a330663cc420639_r.jpg)

1. 首先将 encoder 得到的特征双线性插值得到 4x 的特征，然后与 encoder 中对应大小的低级特征 concat，如 ResNet 中的 Conv2 层
1. 由于 encoder 得到的特征数只有 256，而低级特征维度可能会很高，为了防止 encoder 得到的高级特征被弱化，先采用 1x1 卷积对低级特征进行降维（paper 中输出维度为 48）。
1. 两个特征 concat 后，再采用 3x3 卷积进一步融合特征，最后再双线性插值得到与原始图片相同大小的分割预测。

#### 空洞卷积（Atrous Convolution）

![](https://pic4.zhimg.com/v2-debbe96d9a61bdd07d31e10b6bc4a077_r.jpg)

- 在不改变特征图大小的同时控制感受野，这有利于提取多尺度信息
- rate（r）控制着感受野的大小，r 越大感受野越大。通常的 CNN 分类网络的 output_stride=32，若希望 DilatedFCN 的 output_stride=16，只需要将最后一个下采样层的 stride 设置为 1，并且后面所有卷积层的 r 设置为 2，这样保证感受野没有发生变化





### 全卷积网络 FCN


![](https://pic1.zhimg.com/v2-8922c306fba615f26f2b891dae12d808_r.jpg)


![](https://pic4.zhimg.com/v2-19f47f55b9f1e2779b8cbcdef25d10d7_r.jpg)


- 早期的方法是直接把分类网络用来对图像中的所有像素进行单独的分类，通过“滑窗”的方法获得整个图像的分割结果。一般用CNN+FC的策略构建网络，显然这种方式无法利用图像的全局上下文信息，而且逐像素推理速度很低；

- Long 等人[34]提出了全卷积网络（fully convolutional network, FCN）

- 在分类网络的末尾一般使用全连接层，全连接层的输入特征的大小是固定的，并且全连接层使特征完全丢失了原来的空间信息。

- 将全连接层替换成一般的卷积层使网络能够输出 2D 空间热图（heatmap），实现端对端的训练和预测。


特点：
1. 全卷积网络（不含fc层）
1. 转置卷积==deconv==（反卷积）
1. 不同层特征图跳跃连接（相加）




### UNet



![](https://pic1.zhimg.com/v2-0edfb2beb9c9799bdcaea3f3ef1440a4_r.jpg)



1. UNet网络由U通道和短接通道（skip-connection）组成，U通道类似于SegNet的编解码结构，其中编码部分（contracting path）进行特征提取和捕获上下文信息，解码部分（expanding path）用解码特征图来预测像素标签。
1. 短接通道提高了模型精度并解决了梯度消失问题，特别要注意的是短接通道特征图与上采用特征图是拼接而不是相加（不同于FCN）。


**unet 可以2D 或者 3D卷积**


#### 2D unet


![2dunet](https://pic3.zhimg.com/80/v2-cf0ead284f68d91a42cd3908cf793956_1440w.jpg)

#### 3D unet

两次卷积 + BN
```
nn.Conv3d(in_channels, inter_channels, 3, stride=1, padding=1),
nn.ReLU(True),
nn.Conv3d(inter_channels, out_channels, 3, stride=1, padding=1),
nn.ReLU(True)

if batch_norm:
    layers.insert(1, nn.BatchNorm3d(inter_channels))
    layers.insert(len(layers)-1, nn.BatchNorm3d(out_channels))
```

maxpool encode
```
nn.MaxPo
ol3d(2, stride=2)
```

反卷积或上采样 decode
```
nn.Upsample(scale_factor=2, mode='nearest')
nn.ConvTranspose3d(in_channels, in_channels, 2, stride=2)
```



![3dunet](https://pic2.zhimg.com/v2-72333983b558cda040f3d700f51e9a4d_r.jpg)

1. 网络第一层的feature map通道数为32，每下采样一次，通道数加倍，最大值为320。
1. 网络的基本的block设置为conv-instance norm-leaky ReLU。在上下采样的每个stage中都重复两次。
1. 下采样使用stride为2的卷积实现，上采样使用transposed convolution实现。
1. 上采样路中，除了最底下的两层外，都会有deep supervision。






### VNet

1. 与U-Net类似，不同在于该架构增加了跳跃连接，并
1. 用3D操作物替换了2D操作以处理3D图像（volumetric image）。 并且
1. 针对广泛使用的细分指标进行优化 Dice Loss。

![3dvnet](https://pic1.zhimg.com/80/v2-0ff38da4316fcb48db94f7bc95d27ed4_1440w.jpg)

![](https://pic2.zhimg.com/v2-20cd291def56efd5d634c283389ec805_r.jpg)



### VoxResNet



### SegNet

![](https://pic4.zhimg.com/v2-e51555742cb704ad7db664ed3cc168f3_b.jpg)

1. FCN通过将特征图deconv得到的结果与**Encoder**对应大小的特征图相加得到上采样结果
1. 而SegNet用Encoder部分maxpool的索引进行Decoder部分的上采样



### FC-DenseNet (百层提拉米苏网络)

1. 该网络最简单的版本是由向下过渡的两个下采样路径和向上过渡的两个上采样路径组成。
1. 包含两个水平跳跃连接，将来自下采样路径的特征图与上采样路径中的相应特征图拼接在一起。
1. 融合DenseNet与U-Net网络（从信息交流的角度看，密集连接确实要比残差结构更强大）


![](https://pic3.zhimg.com/v2-ab666adaef42c9c27bad749d0d5b38a2_r.jpg)

## 评价标准

### Dice

- Dice值可以综合的衡量预测值和真实值之间的重叠率
 
### Haus
- 豪斯多夫距离（Hausdorff distance，记为 Haus）作为辅助评判标准，Haus 是度量空间中任意两个集合之间的一种距离，可以用来描述两个点集之间的相似度。

- 因为假阳性小区域会对 Haus 产生较大的影响，所以在分割任务中经常使用 Haus95 而非 Haus 距离， Haus95 是预测结果和真值标签之间的距离的 95% 分位数，在此任务中 Haus95比 Haus 更具参考价值。



$ s $