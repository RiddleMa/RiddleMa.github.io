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
我的template大小不固定，search也不固定。
针对template大小不固定的问题，我现在的做法是：
    1. 对template设定目标尺寸，进行缩放，记录缩放比例；
    2. 对search同时进行缩放，按照步骤1的缩放比例；
    3. search的大小不足