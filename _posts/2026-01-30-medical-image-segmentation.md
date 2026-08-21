---
title: 医学图像分割技术
date: 2026-01-30
publish_display_date: 2026-01-30
excerpt: ""
categories: [Medical Imaging, Computer Vision]
tags: [医学图像分割, nnU-Net, 3D U-Net, 脑肿瘤分割, 半监督学习]
layout: single
author_profile: true
---

## 一、医学图像分割的独特挑战

医学图像分割面临三大核心挑战：数据异质性（不同设备、协议）、标注稀缺性（专家标注成本高）、解剖结构复杂性（器官边界模糊、形态多变）。

**数据预处理标准化**

Z-score标准化：

$$
I_{\text{norm}} = \frac{I - \mu}{\sigma}
$$

窗宽窗位调整（CT图像）：

$$
I_{\text{window}} = \begin{cases}
0 & I < L - W/2 \\
255 & I > L + W/2 \\
\frac{(I - (L - W/2))}{W} \times 255 & \text{otherwise}
\end{cases}
$$

其中 $L$ 为窗位，$W$ 为窗宽。

## 二、经典分割架构的医学改进

### 1、nnU-Net：自适应框架

nnU-Net 的核心创新在于自动配置系统：

(1) 数据指纹提取：自动分析数据集的图像尺寸、体素间距、模态数量等属性

(2) 自适应网络配置：根据图像间距计算各层级的下采样因子

(3) 动态参数调整：根据 GPU 内存自动确定批大小和 patch 大小，动态调整卷积核尺寸（3×3×3 或 3×3）

### 2、3D U-Net变体

**V-Net**：引入残差连接和 Dice 损失

$$
\mathcal{L}_{\text{Dice}} = 1 - \frac{2 \sum_{i} p_i g_i}{\sum_{i} p_i + \sum_{i} g_i}
$$

**层级特征融合**：

$$
\mathbf{F}_{\text{fused}} = \text{Concat}[\mathbf{F}_1, \mathbf{F}_2, \cdots, \mathbf{F}_L]
$$

## 三、特定器官分割技术

### 1、心脏分割（ACDC数据集）

**多阶段分割策略**：

(1) 定位阶段：使用轻量网络检测 ROI

(2) 精细分割：在 ROI 内进行全分辨率分割

(3) 后处理：利用心脏解剖约束（心室、心肌的拓扑关系）

**解剖约束损失**：

$$
\mathcal{L}_{\text{anatomy}} = \sum_{i} \| \mathbf{M}_i - \mathbf{M}_i^{\text{prior}} \|_2^2
$$

其中 $\mathbf{M}_i^{\text{prior}}$ 为解剖先验（如心室体积比应为 1:3）。

### 2、脑肿瘤分割（BraTS挑战）

**多模态融合**（T1、T1c、T2、FLAIR）：

$$
\mathbf{F} = \text{Fusion}(\mathbf{F}_{\text{T1}}, \mathbf{F}_{\text{T1c}}, \mathbf{F}_{\text{T2}}, \mathbf{F}_{\text{FLAIR}})
$$

**肿瘤子区域联合分割**：

- 整个肿瘤（WT）：水肿 + 增强 + 坏死
- 肿瘤核心（TC）：增强 + 坏死
- 增强肿瘤（ET）

**分层 Dice 损失**：

$$
\mathcal{L}_{\text{hierarchical}} = \sum_{r \in \{\text{WT}, \text{TC}, \text{ET}\}} \lambda_r \cdot \mathcal{L}_{\text{Dice}}^r
$$

## 四、弱监督与半监督分割

### 1、基于涂鸦的分割

**DeepCut 方法**：

将涂鸦作为种子约束：

$$
\mathcal{L}_{\text{seed}} = -\sum_{i \in \mathcal{S}} \log P(y_i \mid \mathbf{x}_i)
$$

**随机游走传播**：

基于图的标签传播：

$$
\mathbf{L} \mathbf{f} = \mathbf{b}
$$

其中 $\mathbf{L}$ 为图拉普拉斯矩阵，$\mathbf{b}$ 为边界条件（涂鸦点）。

### 2、一致性正则化半监督

**Mean Teacher 框架**：

- 学生网络：$\theta_s$，通过梯度下降更新
- 教师网络：$\theta_t = \alpha \theta_t + (1 - \alpha) \theta_s$

一致性损失：

$$
\mathcal{L}_{\text{cons}} = \sum_{i} \| \mathbf{P}_s(\mathbf{x}_i) - \mathbf{P}_t(\mathbf{x}_i) \|_2^2
$$

其中 $\mathbf{P}_s$ 为学生网络的预测概率，$\mathbf{P}_t$ 为教师网络的预测概率，通过约束两者一致性，利用无标注数据提升模型泛化能力。
