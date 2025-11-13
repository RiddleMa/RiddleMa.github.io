---
title: yolo_use1
date: 2025-11-13 17:59:26
tags:
---

# YOLO使用记录
## 简介
> YOLO太强大了，感觉根本不需要额外添加什么代码，就能完成基础的训练  
> 代价是：必须弄懂参数和返回值

## 数据集
找到了公开的半导体缺陷检测数据集，是光学显微镜拍摄的，图像的标注质量很高。  
这个网站提供的数据集很多，记录：[wafer defect](https://universe.roboflow.com/wafer-irhuv/wafer-defect-rv1vx/)  
支持下载的格式很多，直接使用YOLOv11格式

## 数据集格式
> YOLO对格式和config等文件要求很严格，所以要使用规定的格式。  
> 如果是自有数据集，需要按照格式进行转换。
- 文件目录：
- train
- test
- valid
- data.yaml  

其中，train, test, valid文件夹内目录：
- images
  - image1.jpg
  - image2.jpg
  - ......
- labels
  - image1.txt
  - image2.txt
  - ......  
  
data.yaml的格式示例如下：
``` commandline
train: ../train/images  # 指示训练集的路径
val: ../valid/images
test: ../test/images

nc: 7   # 类别数
names: ['BLOCK ETCH', 'COATING BAD', 'PARTICLE', 'PIQ PARTICLE', 'PO CONTAMINATION', 'SCRATCH', 'SEZ BURNT']    # 类别名
# 中文名称：块状蚀刻、涂层不良、颗粒、PIQ颗粒、PO污染、划痕、SEZ机台过烧（暂不清楚）

roboflow:   # 网站添加的附加信息
  workspace: wafer-irhuv
  project: wafer-defect-rv1vx
  version: 2
  license: CC BY 4.0
  url: https://universe.roboflow.com/wafer-irhuv/wafer-defect-rv1vx/dataset/2
```
### label格式
示例如下，每个标注为一行，  
第一个数字表示`标签类别`，后面的小数为归一化后的`数据坐标`,每个坐标有两个数值，顺序是`x1,y1,x2,y2....`
``` commandline
1 0.6975 0.3889583333333333 0.6810937499999999 0.3577083333333333 0.579375 0.31375 0.5634375 0.301875 0.516875 0.2779166666666667 0.4840625 0.2558333333333333 0.44749999999999995 0.23645833333333333 0.35890625 0.20770833333333333 0.28500000000000003 
0.19104166666666667 0.19125 0.17854166666666668 0.13093749999999998 0.17583333333333334 0.1159375 0.18104166666666668 0.09828125 0.19749999999999998 0.08796875 0.21416666666666667 0.0840625 0.23208333333333334 0.08203125 0.28 0.08937500000000001 
0.33208333333333334 0.11499999999999999 0.41791666666666666 0.15984375 0.5304166666666666 0.17890625 0.55875 0.19437500000000002 0.5697916666666667 0.21421874999999999 0.574375 0.275 0.5702083333333333 0.40578125 0.5685416666666666 0.49328125 0.574375
 0.649375 0.574375 0.688125 0.5664583333333333 0.7006249999999999 0.5608333333333333 0.71703125 0.5477083333333332 0.725625......
```

## 训练
训练就一行代码，但是方法里面可以添加很多参数，这个要参考官网文档[使用 Ultralytics YOLO 进行模型训练](https://docs.ultralytics.com/zh/modes/train/)
``` python
from ultralytics import YOLO
# model = YOLO("yolo11n-seg.pt")
# results = model.train(data="/data/mazerui/images/Wafer Defect.v2-final.yolov11/data.yaml", epochs=300, imgsz=512,batch=8,device=[0,1,2,3])    # visualize=True
```
简单写下现在用到的参数：
- epoch: 轮数
- imgsz: 图像缩放到的大小，长宽都为这个
- batch: batch size
- device: 指定GPU
- visualize: 可以使用tensorboard进行实时监控

## 结果
                 Class     Images  Instances      Box(P          R      mAP50  m
                   all        907        909      0.741      0.727      0.749      0.518       0.69      0.647      0.658      0.364
            BLOCK ETCH        181        181      0.821      0.713      0.817       0.62      0.826      0.707        0.8      0.576
           COATING BAD         69         69      0.571      0.599      0.517      0.387      0.509      0.482      0.411      0.222
              PARTICLE        119        119      0.774      0.866      0.826      0.543        0.8      0.843      0.848      0.524
          PIQ PARTICLE         86         86      0.733      0.826      0.782       0.47      0.738      0.779      0.751      0.377
      PO CONTAMINATION        110        111      0.684      0.586      0.686      0.483      0.522      0.396      0.417      0.205
               SCRATCH        322        323      0.832      0.831      0.898       0.68       0.79      0.771      0.784       0.34
             SEZ BURNT         20         20      0.771      0.673      0.718      0.445      0.643       0.55      0.598      0.307

> yolo11n的结果，结果也还行吧，后面补一下测试集