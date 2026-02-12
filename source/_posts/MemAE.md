---
title: MemAE代码复现及自有数据集测试
date: 2026-02-12 15:08:53
tags:
---
## 简介
MemAE模型是典型的图像异常检测算法，利用的是自编码器的生成惯性，把有缺陷的图像还原为无缺陷图像（背景图）。  
进而，将有缺陷图与背景图进行对比，利用阈值找到变化较大的区域，即为找到的缺陷区域。  
这里先把作者的博客地址写出来，代码和论文也放这里，方便下载：  
博客：[MemAE介绍](https://donggong1.github.io/anomdec-memae.html)  
代码：[MemAE代码](https://github.com/donggong1/memae-anomaly-detection)  
论文：[MemAE论文](./MemAE/**.pdf)


## 复现
### 1.数据集下载

### 2.预处理代码转换
作者使用的是`matlab`进行数据的处理，但是我电脑上没有`matlab`，所以使用AI转换成了python。
贴出我的python预处理代码，分别对应`/matlab_script`中的同名`.m`文件：
- script_img_prep.py
```python
import os
import glob
from PIL import Image
import numpy as np
import scipy.io


def trans_img2img(inpath, outpath, opts):
    """Preprocess images: transform images to images"""

    # Create output directory if it doesn't exist
    os.makedirs(outpath, exist_ok=True)

    # Get list of image files
    img_files = glob.glob(os.path.join(inpath, f'*.{opts["img_type"]}'))

    if not img_files:
        print(f"No images found in {inpath}")
        return

    # Read first image to get dimensions
    with Image.open(img_files[0]) as img:
        W, H = img.size

    print(inpath)

    for i, img_file in enumerate(sorted(img_files), start=1):
        with Image.open(img_file) as img:
            # Convert to grayscale if needed
            if opts['is_gray'] and img.mode == 'RGB':
                img = img.convert('L')
            # Convert to RGB if needed
            elif not opts['is_gray'] and img.mode == 'L':
                img = img.convert('RGB')

            # Resize if output size specified
            if opts['outsize']:
                img = img.resize(opts['outsize'])

            # Format output filename with leading zeros
            out_filename = f"{i:03d}.jpg"
            img.save(os.path.join(outpath, out_filename))


def trans_img2label(inpath, idx, outpath):
    """For UCSD data frames - generate frame-level ground truth labels"""

    # Construct input path
    gt_in_path = os.path.join(inpath, f"Test{idx:03d}_gt/")
    print(gt_in_path)

    # Create directory if needed
    os.makedirs(gt_in_path, exist_ok=True)

    # Get list of BMP files
    file_list = sorted(glob.glob(os.path.join(gt_in_path, '*.bmp')))
    l = np.zeros(len(file_list))

    for j, file_path in enumerate(file_list):
        # Read image
        img = Image.open(file_path)
        img = np.array(img).astype(float) / 255.0  # Equivalent to im2double

        # Calculate sum of pixel values
        f = np.sum(img)

        # Binary classification based on threshold
        if f < 1:
            l[j] = 0
        else:
            l[j] = 1

    # Save as .mat file
    out_filename = os.path.join(outpath, f"Test{idx:03d}.mat")
    scipy.io.savemat(out_filename, {'l': l})

if __name__ == '__main__':
    # preprocessing UCSD data frames
    data_root_path = 'D://images/'
    in_path = os.path.join(data_root_path, 'UCSD_Anomaly_Dataset.v1p2/UCSDped2/')
    out_path = os.path.join(data_root_path, 'UCSD_Anomaly_Dataset.v1p2/processed/UCSD_P2_256/')

    os.makedirs(out_path,exist_ok=True)

    sub_dir_list = ['Train', 'Test']
    file_num_list = [16, 12]

    opts = {
        'is_gray': True,
        'maxs': 320,
        'outsize': [256, 256],  # output size
        # 'outsize': [112, 112],
        'img_type': 'tif'
    }

    for subdir_idx in range(len(sub_dir_list)):
        # Train, Test
        subdir_file_num = file_num_list[subdir_idx]
        subdir_name = sub_dir_list[subdir_idx]
        subdir_in_path = os.path.join(in_path, subdir_name + '/')
        subdir_out_path = os.path.join(out_path, subdir_name + '/')

        for i in range(1, subdir_file_num + 1):
            v_name = f"{subdir_name}{i:03d}"
            v_path = os.path.join(subdir_in_path, v_name + '/')
            v_out_path = os.path.join(subdir_out_path, v_name + '/')
            os.makedirs(v_out_path,exist_ok=True)
            print(v_path)
            print(v_out_path)
            trans_img2img(v_path, v_out_path, opts)

    # generate frame level gt labels only for Test
    gt_in_path = os.path.join(in_path, 'Test/')
    gt_out_path = os.path.join(out_path, 'Test_gt/')
    os.makedirs(gt_out_path,exist_ok=True)

    for i in range(1, file_num_list[1] + 1):
        trans_img2label(gt_in_path, i, gt_out_path)

```
- script_index_gen.py
```python
import os
import glob
import numpy as np
import scipy.io


def generate_clip_indexes():
    # Configuration
    data_root_path = 'D://images/'
    in_path = os.path.join(data_root_path, 'UCSD_Anomaly_Dataset.v1p2/processed/UCSD_P2_256/')

    frame_file_type = 'jpg'
    clip_len = 16  # Number of frames in a clip
    overlap_rate = 0  # Overlap percentage
    skip_step = 1  # Frame sampling step
    clip_rng = clip_len * skip_step - 1

    # Overlap backward shift options
    overlap_shift = clip_len - 1  # Full overlap (shift 1 one step)
    # overlap_shift = clip_len // 2  # Half overlap
    # overlap_shift = 10
    # overlap_shift = 5

    sub_dir_list = ['Train', 'Test']

    for sub_dir_name in sub_dir_list:
        print(sub_dir_name)
        sub_in_path = os.path.join(in_path, sub_dir_name + '/')
        idx_out_path = os.path.join(in_path, sub_dir_name + '_idx/')
        os.makedirs(idx_out_path, exist_ok=True)

        # Get video sequence directories
        v_dirs = glob.glob(os.path.join(sub_in_path, sub_dir_name + '*'))
        v_dirs = [d for d in v_dirs if os.path.isdir(d)]

        for v_dir in sorted(v_dirs):
            v_name = os.path.basename(v_dir)
            print(v_name)

            # Get frame files
            frame_files = glob.glob(os.path.join(v_dir, f'*.{frame_file_type}'))
            frame_num = len(frame_files)

            # Calculate start and end indices
            s_list = np.arange(0, frame_num, clip_rng + 1 - overlap_shift)
            e_list = s_list + clip_rng

            # Filter valid indices (where end doesn't exceed frame count)
            valid_mask = e_list < frame_num
            s_list = s_list[valid_mask]
            e_list = e_list[valid_mask]

            # Create output directory for this video
            video_sub_dir_out_path = os.path.join(idx_out_path, v_name + '/')
            os.makedirs(video_sub_dir_out_path, exist_ok=True)

            # Save index files for each clip
            for j, (s, e) in enumerate(zip(s_list, e_list), 1):
                idx = np.arange(s, e + 1, skip_step)

                # Save as .mat file
                out_file = os.path.join(video_sub_dir_out_path,
                                        f"{v_name}_i{j:03d}.mat")
                scipy.io.savemat(out_file, {'v_name': v_name, 'idx': idx + 1})  # +1 for MATLAB 1-based indexing


if __name__ == '__main__':
    generate_clip_indexes()

```
> 注意：数据集路径和名称需要根据实际位置修改。
- 运行完毕后，会在`\processed\UCSD_P2_256`下生成几个文件夹：  
[](./MemAE/img.png)
### 3.训练
！最好把reqruiment写上  
没有使用sh命令，直接运行的训练文件`script_training.py`。  
在此之前，需要修改配置文件`training_options.py`的默认值。
### 4.测试
可以使用步骤3的训练结果进行测试，也可以下载原作者提供的参数文件。  
同样，测试前对配置文件`testing_options.py`进行修改。  
运行`script_testing.py`
## todo
预处理的python代码  
2D数据转换