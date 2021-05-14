---
layout: post
title: Transformer
date: 2021-5-12
author: 
tags: [cv, note]
comments: true
toc: true
pinned: False
---

<!-- more -->

# Transformer


##

自注意力通过使用所有位置之间的成对亲和力来计算特征的加权总和，以捕获单个样本内的长期依赖性，从而更新每个位置的特征。

![](https://pic3.zhimg.com/v2-a1d3c1ac443534c7f5a1f5693d4ea04e_r.jpg)



## ViT（vision transformer）
Google在2020年
1. 直接把图像分成固定大小的patchs，然后通过线性变换得到patch embedding，这就类比NLP的words和word embedding
1. 由于transformer的输入就是a sequence of token embeddings，所以将图像的patch embeddings送入transformer后就能够进行特征提取从而分类了

