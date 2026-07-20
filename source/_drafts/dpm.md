---
title: DPM的算法（Deep Pattern Match）
date: 2026-07-20 18:24:42
tags:
---
## 简介
基于深度学习的pattern match算法研究
pattern match中文名叫模板匹配，更专业的名字应该是template match。pattern更倾向于机器学习的模式，在半导体领域的意义更偏向于晶圆上的图案。但是由于项目习惯，还是使用PTM和DPM表示模板匹配和深度模板匹配。
## 传统的模板匹配算法
传统的模板匹配算法包括空域和频域匹配算法。
空域匹配算法是采用滑动窗口的办法，用模板在搜索图上进行滑动，计算窗口区域两个图像的像素相似性，直到滑动完毕后根据最大值确定匹配位置坐标。
### 空域匹配
空域匹配可以参考OpenCV的matchTemplate算法：
[OpenCV  matchTemplate](https://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/imgproc/histograms/template_matching/template_matching.html)

空域匹配的核心是相关性计算方法，matchTemplate提供的相关性计算有：
1. CV_TM_SQDIFF
2. CV_TM_SQDIFF_NORMED
3. CV_TM_CCORR
4. CV_TM_CCORR_NORMED
5. CV_TM_CCOEFF
6. CV_TM_CCOEFF_NORMED
`_NORMED`是对计算结果进行了归一化处理。

为了提高计算速度，可以采用金字塔的计算方式，即：对图像和模板进行下采样，从下到上进行最佳位置的寻找；下采样的作用是减少了匹配的步数，下采样的图中获得粗匹配位置后，最终在原图上对应匹配位置附近进行匹配，获取最终的匹配坐标。
下采样的次数可以控制，如果下采样倍数为2，一次下采样后图像宽度减少1/2，计算量减少为原来的1/4。多次下后，图像的宽度逐渐减少，形成一个倒金子塔形的结构，这种方法就叫做金字塔加速。

### 频域匹配
频域匹配我需要学习后总结。




## 训练阶段


## 预测阶段
- 