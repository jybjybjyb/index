---
layout: post
title: Medicine segmentation
date: 2021-5-10
author: 
tags: [cv, note]
comments: true
toc: true
pinned: False
---


[toc]
<!-- more -->

# 医学图像语义分割

## FCN网络

![](https://pic1.zhimg.com/v2-8922c306fba615f26f2b891dae12d808_r.jpg)


![](https://pic4.zhimg.com/v2-19f47f55b9f1e2779b8cbcdef25d10d7_r.jpg)

此前的基于神经网络的图像语义分割网络是利用以待分类像素点为中心的图像块来预测中心像素的标签，一般用CNN+FC的策略构建网络，显然这种方式无法利用图像的全局上下文信息，而且逐像素推理速度很低；

而FCN网络舍弃全连接层FC，全部用卷积层构建网络，通过转置卷积以及不同层特征融合的策略，使得网络输出直接是输入图像的预测mask，效率和精度得到大幅度提升。

1. 全卷积网络（不含fc层）
1. 转置卷积deconv（反卷积）
1. 不同层特征图跳跃连接（相加）


## SegNet

![](https://pic4.zhimg.com/v2-e51555742cb704ad7db664ed3cc168f3_b.jpg)

1. FCN通过将特征图deconv得到的结果与**Encoder**对应大小的特征图相加得到上采样结果
1. 而SegNet用Encoder部分maxpool的索引进行Decoder部分的上采样




## UNet



![](https://pic1.zhimg.com/v2-0edfb2beb9c9799bdcaea3f3ef1440a4_r.jpg)



1. UNet网络由U通道和短接通道（skip-connection）组成，U通道类似于SegNet的编解码结构，其中编码部分（contracting path）进行特征提取和捕获上下文信息，解码部分（expanding path）用解码特征图来预测像素标签。
1. 短接通道提高了模型精度并解决了梯度消失问题，特别要注意的是短接通道特征图与上采用特征图是拼接而不是相加（不同于FCN）。


**unet 可以2D 或者 3D卷积**


### 2D unet


![2dunet](https://pic3.zhimg.com/80/v2-cf0ead284f68d91a42cd3908cf793956_1440w.jpg)

### 3D unet

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

#### Patch Size and Batch Size

在三维医学图像的处理中，显存不足是经常容易遇到的问题。而训练过程中，显存占用与模型的输入patch size，batch size以及网络的结构相关。然而，patch size越大，所需要的模型越大。因此，在抉择时，主要在patch size与batch size之间进行权衡。nnUNet中优先考虑patch size，保证模型能基于更多的信息来进行推理。但batch size最小值应该为2，需要保证训练过程中优化过程的鲁棒性。最后，在保证patch size的前提下，如果显存有多余的，再对应增加batch size。因为batch size大部分情况都比较小，所以之前固定设计中没有用BN而使用Instance Norm。

而最优patch size的决定，采用迭代的方式进行。将batch size先固定为2，然后设定一个较大的patch size初始值，再根据当前patch size来设计网络结构。之后根据显存占用情况，不断的迭代调整patch size。

#### target spacing

target spacing指的是数据集在经过重采样之后，每张图像所具有的相同的spacing值，即每张图像的每个体素所代表的实际物理空间大小。




## VNet

1. 与U-Net类似，不同在于该架构增加了跳跃连接，并
1. 用3D操作物替换了2D操作以处理3D图像（volumetric image）。 并且
1. 针对广泛使用的细分指标进行优化 Dice Loss。

![3dvnet](https://pic1.zhimg.com/80/v2-0ff38da4316fcb48db94f7bc95d27ed4_1440w.jpg)

![](https://pic2.zhimg.com/v2-20cd291def56efd5d634c283389ec805_r.jpg)

## FC-DenseNet (百层提拉米苏网络)

1. 该网络最简单的版本是由向下过渡的两个下采样路径和向上过渡的两个上采样路径组成。
1. 包含两个水平跳跃连接，将来自下采样路径的特征图与上采样路径中的相应特征图拼接在一起。
1. 融合DenseNet与U-Net网络（从信息交流的角度看，密集连接确实要比残差结构更强大）


![](https://pic3.zhimg.com/v2-ab666adaef42c9c27bad749d0d5b38a2_r.jpg)

## 未完待续