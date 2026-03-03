---
title: 频域滤波使用的注意事项
date: 2026-03-03 09:42:40
tags:
---
## 简介
频域滤波主要是利用傅里叶变换，对图像的高频或低频信息进行增强或者削弱，实现无法在空域中达到的图像处理效果。

实际使用这一方法的过程中，发现了一些问题，记录一下。

## 频域滤波的一般步骤
1. 获取原图像
2. 填充零，填充到尺寸为2的整数倍
3. 二维傅利叶变换
4. 频域中心化，把四角的低频移动到图像中心
5. 构造频域滤波器掩膜，低通、高通、带通、高斯、巴特沃斯
6. 把频域图与掩膜相乘，逐点相乘
7. 反中心化
8. 逆傅利叶变换

- 简化方法
可以简化频域中心化方法，在傅里叶变换前对图像乘以(-1)^(x+y),实现和频域中心化同样的方法。

## 问题一：图像值域变化
使用频域滤波是将图像的一部分信号消除或者削弱，所以最终的图像灰度值会比原图要小。
在c++中，逆傅利叶变换时如果不添加参数`cv::DFT_SCALE`，得到的图像数值会是真实值的 
M×N倍,否则结果会严重失真。
- python
```python
ifft_img = np.fft.ifft2(fft_ishift)  # 2D-逆FFT，得到复数矩阵
img_real = np.real(ifft_img)  # 取实部
```
- C++
```C++
dft(fft_ishift, ifft_img, DFT_INVERSE | cv::DFT_SCALE | cv::DFT_REAL_OUTPUT);
```

## 问题二：掩膜差异
掩膜的生成方式也可能有错误。
以圆形掩膜为例，在图像长宽都为偶数的情况下，以**图像绝对中心**作为圆心，绘制圆形掩膜。

<div align="center">
  <img src="/frequency-domain-filtering/mask.png" width="150" alt="图1 ImageJ标准mask">
  <p>图1 ImageJ标准mask</p>
</div>

一个错误的做法是，使用**图像中心的像素**作为圆心，这种做法在图像长宽为奇数时没问题。但是傅利叶变换一般都把图像padding为偶数，所以这种做法会和Imagej结果有轻微的不同。

<div align="center">
  <img src="/frequency-domain-filtering/mask2.png" width="150" alt="图1 ImageJ标准mask">
  <p>图2 通常的错误做法mask</p>
</div>



