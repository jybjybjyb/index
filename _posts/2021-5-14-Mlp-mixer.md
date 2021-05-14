---
layout: post
title: Mlp-mixer
date: 2021-5-14
author: 
tags: [cv, note]
comments: true
toc: true
pinned: False
---

<!-- more -->


# Mlp-mixer

## 构架

下图 1 描述了 Mixer 的宏观架构，它以一系列图像块的线性投影（输入的形状为 patches × channels）作为输入，先将输入图片拆分为 patch，通过 Per-patch Fully-connected 将每个 patch 转换为 feature embedding，接着馈入 N 个 Mixer Layer，最后通过 Fully-connected 进行分类。

![](https://pic2.zhimg.com/v2-391f4f65b4e534dd56b9662a2b4e20c9_r.jpg)

- Mixer 架构采用两种不同类型的 MLP 层：channel-mixing MLP 和 token-mixing MLP。

- channel-mixing MLP 允许不同通道之间进行通信，token-mixing MLP 允许不同空间位置（token）之间进行通信。这两种类型的层交替执行以促进两个维度间的信息交互。

- Mixer 架构可以看做是一个特殊的 CNN，使用 1×1 卷积进行 channel mixing，同时全感受野和参数共享的的单通道深度卷积进行 token mixing。

## 设计思想

- Mixer 架构的设计思想是**清楚地**将按位置（channel-mixing）操作 (i) 和跨位置（token-mixing）操作 (ii) 分开，两种操作都通过 MLP 来实现。

- Mixer 由大小相同的多个层组成。每个层由 2 个 MLP 块组成，其中，第一个块是 token-mixing MLP 块，第二个是 channel-mixing MLP 块。每个 MLP 块包含两个全连接层，以及一个单独应用于其输入数据张量的每一行的非线性层。

- Mixer 中的每个层（初始 patch 投影层除外）都采用相同大小的输入，这种「各向同性（isotropic）」的设计与使用固定宽度的 Transformer 或其他域中的深度 RNN 大致相似。

- 这不同于大多数具有金字塔结构的 CNN，即较深的层具有较低分辨率的输入，但是有较多通道（channel）。

- 除了 MLP 层，Mixer 还使用其他标准架构组件：跳远连接（skip-connection）和层归一化。此外，和 ViT 不同，Mixer 不使用位置嵌入，因为 token-mixing MLP 对输入 token 的顺序很敏感，因此能够学会表征位置。最后，Mixer 将标准分类头与全局平均池化层配合使用，随后使用线性分类器。




## 1*1 卷积

![](https://pic4.zhimg.com/80/v2-cf42a8eb42fa902347a619c6e4878840_1440w.jpg?source=3af55fa1)

1. 调节通道数
1. 增加非线性，可以在保持特征图尺度不变的（即不改变）的前提下大幅增加非线性特性
1. 跨通道信息，通道间的信息交互。
1. 减少参数，

