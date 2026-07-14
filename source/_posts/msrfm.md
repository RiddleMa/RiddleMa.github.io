---
title: Achieving the Defect Transfer Detection of  Semiconductor Wafer by a Novel Prototype  Learning-Based Semantic Segmentation Network
date: 2026-02-12 15:09:09
tags:
---
## 简介
论文地址：[Achieving the Defect Transfer Detection of  Semiconductor...](https://ieeexplore.ieee.org/abstract/document/10330004)  
这篇文章是半导体缺陷检测的论文，主要用到的方法是自编码器(AE, AutoEncoder)和原型学习（PL, Prototype Learning）。  
作者来自地大，发表的论文都是关于半导体缺陷检测的。  
文章中提到的优点有：
- 提出了multiscale residual fusion module (MSRFM)方法，经过精心设计，将前景和背景信息解耦，迫使网络更加关注缺陷特征。
- 提出了一种新的半导体晶片缺陷转移检测策略，避免了不同图案晶片中重复缺陷图像采集和标记的问题。在两种类型的晶圆上，所提出的网络在现实半导体生产线上的平均交集超过联合（mIoU）可以达到83.49%和80.12%。
- 设计了一个额外的自约束来调节原型计算过程，可用于其他PL网络。  


其他问题

## 概念
**原型学习**：传统的端对端分割方法问题有：
- 对数据样本的数量需求很大
- 泛化性能差，图像域发生变化则可能性能变得很差
原型学习解决的就是这两个问题，PANet是经典的原型学习论文，其主要思路是：
- 在训练过程中就构建一个基准原型，在预测过程中，将预测图像与原型进行比较，就能根据与原型的距离来生成分割结果
- 解决的是Few-shot问题，即使用少量的样本进行学习  

**自编码器**：  

## 具体方法
### MemAE记忆自编码器



## 问题
不知道loss怎么加起来的