---
title: SAM的原理和使用方法
date: 2026-02-12 14:57:45
tags:
---
> SAM已经更新到SAM3了，这里是SAM2的使用介绍
> SAM2更新过一版，最终版本是SAM2.1版
> SAM3的使用方法类似，后面研究下；

SAM2自带三个接口文件：
- build_sam，模型构建器
- sam2_image_predictor，交互式分割预测
- automatic_mask_generator，全自动掩码生成器  

现在分别介绍：

## build_sam
- 模型构建函数，用于对模型进行初始化；
- 可以指定配置文件`sam2.1_hiera_t.yaml`和参数文件`sam2.1_hiera_tiny.pt`；

## sam2_image_predictor
- 交互式预测器，需要用户提供 “提示信息”（点、框）来定位目标。
```python
from sam2.sam2_image_predictor import SAM2ImagePredictor

# 初始化模型
sam2_model = build_sam2(
    config_file="sam2_configs/sam2_hiera_l.yaml",  # 模型配置文件
    checkpoint="sam2_hiera_large.pt",             # 预训练权重
    device=torch.device("cuda" if torch.cuda.is_available() else "cpu")
)

# 初始化预测器（传入已构建的 SAM2 模型）
predictor = SAM2ImagePredictor(sam2_model)

# 加载图片并编码
image = cv2.imread("test.jpg")
predictor.set_image(image)

# 输入提示：比如在 (x=100, y=200) 点一个前景点（目标位置）
input_point = np.array([[100, 200]])
input_label = np.array([1])  # 1=前景，0=背景

# 生成掩码
masks, scores, logits = predictor.predict(
    point_coords=input_point,
    point_labels=input_label,
    box = None,     # 也可以指定锚框
    multimask_output=False,  # 只返回最优掩码
)
```
- 这里支持两种形式的指令，`point_coords`为点，`box`为锚框，也可以一起使用；
- `point_coords`的大小为(M,2)，`box`的大小为(1,4)；
- `point_labels`确定点选的是前景/背景，0的话就是反选；
- 同时使用两种标记的方式
  - 框定位目标大致范围 + 点标记目标核心区域（如分割人物时，框住全身 + 点标记脸部）；
  - 框定位目标 + 点标记背景（label=0），排除干扰区域。

## automatic_mask_generator
基于 SAM2 模型的全自动掩码生成器，无需用户提示，自动遍历图片并分割所有可见物体。
```python
from sam2.sam2_automatic_mask_generator import SAM2AutomaticMaskGenerator

# 初始化自动掩码生成器
mask_generator = SAM2AutomaticMaskGenerator(
    sam2_model,
    points_per_side=32,  # 每边采样点数（越多分割越细，速度越慢）
    pred_iou_thresh=0.86, # 过滤低IOU掩码
    stability_score_thresh=0.92,  # 过滤不稳定掩码
)

# 全自动生成所有物体的掩码
masks = mask_generator.generate(image)

# 输出结果：masks 是列表，每个元素包含 mask（掩码）、bbox（边界框）、score（置信度）等
print(f"自动分割出 {len(masks)} 个物体")
```
- 这种方法原理是使用采样点进行逐点预测，最后合并所有分割的结果；
- 采样点越多，速度越慢（实测很慢）；
