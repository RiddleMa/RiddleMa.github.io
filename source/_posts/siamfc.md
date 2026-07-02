---
title: 使用孪生网络进行图像模板匹配
date: 2026-06-29 17:15:38
tags: 
  - patternMatch
  - CV
---
## 简介
模板匹配是使用一个固定的模板图像，在另一张图像上寻找与模板图像相同的图像，以进行位置对齐。
模板图像为template image，待匹配的图像称为search image，通常search比template大。
传统模板匹配方法有基于空间和时间的方法。

## 改进及思考
### 1. 一种做法
我的template大小不固定，search也不固定。
假设要固定template和search的长宽，template-->152，search-->512
针对template大小不固定的问题，我现在的做法是：
    1. 裁剪template，保证template为正方形；
    2. 如果template边长大于152，设定目标尺寸，进行缩放，对search同时进行缩放；
    3. 如果template边长小于152，padding到152。
这种做法的好处是，模型接收固定长度的图像。
### 2. 另一种做法
不对图像进行resize，只做裁剪；
对模型进行修改，使得其能够接收不固定长度图像；
经过验证，这种方法效果很差，原因未知；

