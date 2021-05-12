---
layout: post
title: Transformer
date: 2021-5-11
author: 
tags: [cv, note]
comments: true
toc: true
pinned: False
---

<!-- more -->

# Transformer

## ViT（vision transformer）
Google在2020年
1. 直接把图像分成固定大小的patchs，然后通过线性变换得到patch embedding，这就类比NLP的words和word embedding
1. 由于transformer的输入就是a sequence of token embeddings，所以将图像的patch embeddings送入transformer后就能够进行特征提取从而分类了

