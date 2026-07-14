---
title: 增量学习的方法总结
date: 2026-02-12 15:09:28
tags:
---
## 背景
- SAM是一种基于预训练的图像大模型，使用该模型能够对图像相关的任务进行迁移使用。
针对特殊领域的图像分割任务，SamAdapte被发明出来用来对专用数据的分割任务进行加强。
- 在这个模型的应用过程中，频繁出现新的数据，这些数据的特征与训练数据的特征不尽相同。
导致在第一批数据上训练的模型参数，在后面批次数据上的表现性能不佳。
需要找到一种方法，能够同时对新旧数据具有良好的表现。
- 针对这个问题，最简单的解决办法就是把新旧数据都放入训练集中，同时参与训练。
但是训练的成本太高了，原训练数据集已经很大了，再一批批地加入新数据，会严重增加训练成本，需要的时间也非常长。
- 所以怎么同时保证兼容性和高效性？

---
## 一、基础方案：混合重放（Replay）+ 增量微调
这是最易落地的方案，核心是每次加入新数据时，从旧数据中采样一部分与新数据混合训练，避免模型完全遗忘旧数据。
### 1. 具体实现步骤

1. **数据管理**：维护一个 “旧数据缓存池”，保存第一批（及后续）数据的代表性样本（无需全量，按类别 / 分布采样）；
2. **混合训练**：新数据到来时，将新数据与缓存池中的旧数据按比例（如 7:3）混合，用 SamAdapter 继续微调；
3. **参数更新**：沿用 SamAdapter 的低秩适配（LoRA/Adapter）机制，仅更新适配层参数，冻结 SAM 主干，减少遗忘风险。

### 2. 代码示例
```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, ConcatDataset
from sam_adapter import SamAdapter  # 假设你已有的SamAdapter封装类

class IncrementalSamAdapter:
    def __init__(self, sam_adapter_model, cache_size=1000):
        self.model = sam_adapter_model  # 预训练好的SamAdapter模型
        self.old_data_cache = []  # 旧数据缓存池
        self.cache_size = cache_size  # 缓存池最大容量
        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        self.model.to(self.device)

    def update_cache(self, old_dataset):
        """更新旧数据缓存池（按类别均匀采样，避免类别失衡）"""
        # 简单采样逻辑：随机选cache_size个样本，可替换为按类别分层采样
        sample_indices = torch.randperm(len(old_dataset))[:self.cache_size]
        self.old_data_cache = [old_dataset[i] for i in sample_indices]

    def train_with_new_data(self, new_dataset, lr=1e-4, epochs=5, mix_ratio=0.3):
        """
        增量训练：混合旧数据缓存和新数据
        Args:
            new_dataset: 新批次数据
            lr: 学习率（增量训练建议更小）
            epochs: 训练轮数
            mix_ratio: 旧数据占总训练数据的比例
        """
        # 1. 构建混合数据集
        if len(self.old_data_cache) > 0:
            # 按比例采样旧数据
            old_sample_num = int(len(new_dataset) * mix_ratio / (1 - mix_ratio))
            old_sample_indices = torch.randperm(len(self.old_data_cache))[:old_sample_num]
            old_samples = [self.old_data_cache[i] for i in old_sample_indices]
            
            # 拼接新旧数据
            mixed_dataset = ConcatDataset([old_samples, new_dataset])
        else:
            # 首次训练：仅用新数据，同时缓存
            mixed_dataset = new_dataset
            self.update_cache(new_dataset)

        # 2. 构建数据加载器
        train_loader = DataLoader(mixed_dataset, batch_size=8, shuffle=True)

        # 3. 增量训练（仅更新Adapter层，冻结SAM主干）
        optimizer = torch.optim.Adam(self.model.adapter_parameters(), lr=lr)
        criterion = nn.CrossEntropyLoss()  # 根据你的任务替换损失函数

        self.model.train()
        for epoch in range(epochs):
            total_loss = 0.0
            for batch in train_loader:
                images, labels = batch
                images, labels = images.to(self.device), labels.to(self.device)

                # 前向传播
                outputs = self.model(images)
                loss = criterion(outputs, labels)

                # 反向传播+更新
                optimizer.zero_grad()
                loss.backward()
                optimizer.step()

                total_loss += loss.item()

            print(f"Epoch {epoch+1}/{epochs}, Loss: {total_loss/len(train_loader):.4f}")

        # 4. 更新缓存池（将新数据加入缓存，保持缓存代表性）
        self.update_cache(ConcatDataset([self.old_data_cache, new_dataset]))

# ---------------------- 使用示例 ----------------------
# 1. 初始化SamAdapter模型（假设已加载预训练权重）
base_sam_adapter = SamAdapter(pretrained_path="sam_vit_h.pth")

# 2. 初始化增量学习封装类
incremental_model = IncrementalSamAdapter(base_sam_adapter, cache_size=2000)

# 3. 第一批数据训练
first_dataset = YourFirstDataset()  # 替换为你的第一批数据集
incremental_model.train_with_new_data(first_dataset)

# 4. 第二批新数据训练（自动混合第一批数据）
second_dataset = YourSecondDataset()  # 替换为你的第二批数据集
incremental_model.train_with_new_data(second_dataset, lr=5e-5)

# 5. 后续新数据重复步骤4即可

```

### 3. 关键细节说明
- **缓存池大小**：无需缓存全量旧数据，建议按 “新数据量 × 混合比例 /(1 - 混合比例)” 设置，避免内存溢出；
- **学习率**：增量训练的学习率建议比首次训练小（如 5e-5），减少对旧知识的覆盖；
- **混合比例**：旧数据占比建议 20%-40%，比例过高会增加训练成本，过低则易遗忘。

## 二、进阶方案：弹性权重巩固（EWC）+ Adapter
如果混合重放的内存成本过高（如数据量极大），可采用弹性权重巩固（EWC） 机制：在学习新数据时，对旧数据中重要的模型参数添加正则化约束，阻止其大幅变化，从而保留旧知识。
### 1. 核心逻辑
1. 训练第一批数据后，计算模型参数在旧数据上的 “重要性”（参数的 Fisher 信息矩阵）；
2. 训练新数据时，损失函数加入 “参数偏离惩罚项”：对重要参数，限制其与旧数据训练后的取值偏差；
3. 结合 Adapter 机制，仅对 Adapter 层参数应用 EWC 约束，进一步降低计算成本。
### 2. 核心代码片段（EWC 损失）
```python
def compute_fisher_information(model, dataset, num_batches=10):
    """计算Fisher信息矩阵（衡量参数重要性）"""
    fisher = {n: torch.zeros_like(p) for n, p in model.adapter_parameters().items()}
    model.eval()
    loader = DataLoader(dataset, batch_size=8, shuffle=True)
    
    for i, batch in enumerate(loader):
        if i >= num_batches:
            break
        images, labels = batch
        images, labels = images.to(model.device), labels.to(model.device)
        
        model.zero_grad()
        outputs = model(images)
        loss = nn.CrossEntropyLoss()(outputs, labels)
        loss.backward()
        
        # 累积梯度平方（近似Fisher信息）
        for n, p in model.adapter_parameters().items():
            if p.grad is not None:
                fisher[n] += (p.grad **2) / num_batches
    return fisher

def ewc_loss(model, old_params, fisher, lambda_ewc=1e3):
    """EWC正则化损失：惩罚参数偏离旧值"""
    loss = 0.0
    for n, p in model.adapter_parameters().items():
        loss += (fisher[n] * (p - old_params[n])** 2).sum()
    return lambda_ewc * loss

# 训练新数据时，损失函数改为：
total_loss = criterion(outputs, labels) + ewc_loss(model, old_params, fisher)
```

## 三、最优方案：多任务 Adapter + 动态路由
如果后续会持续加入多批次 / 多领域数据，可采用多任务 Adapter 架构：为每一批数据训练一个独立的 Adapter 分支，推理时通过 “动态路由” 选择适配的分支（或融合多分支输出），从根本上避免遗忘。
### 1. 核心思路
1. 为每一批新数据创建一个独立的 Adapter 层（如 LoRA 层），保存在模型中；
2. 训练时仅更新当前批次对应的 Adapter 分支，不改动其他分支和主干；
3. 推理时：
   - 若输入数据可归属到某一批次，直接用对应 Adapter；
   - 若无法归属，融合所有 Adapter 分支的输出（如加权平均）。
### 2. 核心代码示例
```python
class MultiAdapterSam(nn.Module):
    def __init__(self, base_sam):
        super().__init__()
        self.base_sam = base_sam  # 冻结的SAM主干
        self.adapters = nn.ModuleDict()  # 存储各批次的Adapter：key=批次ID，value=Adapter层
        self.current_adapter_id = None

    def add_adapter(self, adapter_id):
        """为新批次数据添加独立的Adapter"""
        if adapter_id not in self.adapters:
            # 创建新的Adapter（以LoRA为例）
            new_adapter = LoRAAdapter(dim=self.base_sam.feat_dim, r=8)  # 替换为你的Adapter实现
            self.adapters[adapter_id] = new_adapter

    def set_active_adapter(self, adapter_id):
        """设置当前训练/推理的Adapter"""
        self.current_adapter_id = adapter_id

    def forward(self, x):
        # 1. SAM主干提取特征
        feat = self.base_sam.extract_feat(x)
        
        # 2. 适配层处理
        if self.current_adapter_id is not None and self.current_adapter_id in self.adapters:
            # 使用指定的Adapter
            feat = self.adapters[self.current_adapter_id](feat)
        else:
            # 融合所有Adapter输出（推理时适配全量数据）
            adapter_outputs = [adapter(feat) for adapter in self.adapters.values()]
            feat = torch.stack(adapter_outputs).mean(dim=0)  # 简单平均，可替换为加权融合
        
        # 3. 预测头输出
        output = self.base_sam.head(feat)
        return output

# ---------------------- 使用示例 ----------------------
# 1. 初始化多Adapter模型
base_sam = load_pretrained_sam("sam_vit_h.pth")
multi_adapter_model = MultiAdapterSam(base_sam)

# 2. 第一批数据训练
multi_adapter_model.add_adapter(adapter_id="batch_1")
multi_adapter_model.set_active_adapter("batch_1")
train_model(multi_adapter_model, first_dataset)  # 仅训练batch_1对应的Adapter

# 3. 第二批数据训练
multi_adapter_model.add_adapter(adapter_id="batch_2")
multi_adapter_model.set_active_adapter("batch_2")
train_model(multi_adapter_model, second_dataset)  # 仅训练batch_2对应的Adapter

# 4. 推理（自动融合所有Adapter，适配全量数据）
multi_adapter_model.set_active_adapter(None)
preds = multi_adapter_model(test_images)
```

---
## 总结
核心方案选择如下：
1. **快速落地**：优先用「混合重放 + 增量微调」，代码改动小，仅需维护旧数据缓存池，适合中小规模数据；
2. **内存受限**：选择「EWC+Adapter」，无需缓存大量旧数据，通过参数正则化减少遗忘；
3. **长期扩展**：推荐「多任务 Adapter + 动态路由」，为每批数据独立训练 Adapter，无遗忘风险，适配持续新增的数据。
## 关键注意点
- 无论哪种方案，都需**冻结SAM 主干**，仅更新 Adapter 层，既降低计算成本，也减少旧知识遗忘；
- 增量训练的学习率需逐步降低（如每次新数据训练时学习率减半），避免参数震荡；
- 缓存池 / Adapter 分支需定期清理冗余数据 / 分支，避免内存 / 参数膨胀。