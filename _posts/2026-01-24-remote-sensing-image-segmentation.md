---
title: 遥感图像分割技术
date: 2026-01-24
publish_display_date: 2026-01-24
excerpt: ""
categories: [Remote Sensing, Computer Vision]
tags: [遥感图像分割, GEOBIA, 多尺度分割, HRCNet, 实例分割]
layout: single
author_profile: true
---

遥感图像分割是遥感影像分析的核心任务之一，旨在将遥感图像划分为具有语义意义的区域。与自然图像不同，遥感图像具有尺度变化大、地物类型多样、光谱信息丰富等特点，对分割算法提出了特殊要求。本文系统阐述面向对象分割、深度学习方法及实例分割在遥感中的应用。

## 一、面向对象分割（GEOBIA）

### 1、多尺度分割算法

**分形网络演化方法（FNEA）**：

区域合并的异质性准则：

$$
f = w_{\text{color}} \cdot h_{\text{color}} + w_{\text{shape}} \cdot h_{\text{shape}}
$$

其中形状异质性：

$$
h_{\text{shape}} = w_{\text{compact}} \cdot h_{\text{compact}} + w_{\text{smooth}} \cdot h_{\text{smooth}}
$$

- **紧凑度**：$h_{\text{compact}} = \frac{P}{2\sqrt{\pi A}}$，衡量对象的紧凑程度
- **平滑度**：$h_{\text{smooth}} = \frac{P}{B}$，其中 $B$ 为对象边界框周长

### 2、对象特征提取

**几何特征**：

- 面积：$A = \sum_{i} \text{pixel}_i$
- 长宽比：$R = \frac{L}{W}$
- 形状指数：$\text{SI} = \frac{P}{4\sqrt{A}}$

**纹理特征（GLCM）**：

- 对比度：$\text{Contrast} = \sum_{i,j} (i-j)^2 \cdot \mathbf{P}(i,j)$
- 熵：$\text{Entropy} = -\sum_{i,j} \mathbf{P}(i,j) \cdot \log \mathbf{P}(i,j)$

## 二、深度学习分割方法

### 1、针对高分影像的改进

**HRCNet（高分辨率上下文网络）**：

保持高分辨率特征的同时融合多尺度上下文：

$$
\mathbf{F}_{\text{out}} = \text{Fusion}\left( \mathbf{F}_{\text{high}}, \mathbf{F}_{\text{mid}}, \mathbf{F}_{\text{low}} \right)
$$

**多尺度注意力机制**：

$$
\mathbf{M} = \sigma\left( \text{MLP}\left( \text{GAP}(\mathbf{F}) \right) + \text{MLP}\left( \text{GMP}(\mathbf{F}) \right) \right)
$$

其中 GAP 为全局平均池化，GMP 为全局最大池化，通过两者结合增强特征的判别能力。

### 2、多光谱分割

**波段注意力机制**：

$$
\mathbf{F}_{\text{out}} = \sum_{b=1}^{B} \alpha_b \cdot \mathbf{F}_b
$$

其中权重 $\alpha_b$ 通过学习得到，使网络能够自适应地关注对当前分割任务更重要的光谱波段。

## 三、实例分割在遥感中的应用

### 1、建筑物实例分割

**轮廓感知损失**：

$$
\mathcal{L}_{\text{contour}} = \sum_{i} \| \mathbf{E}_{\text{pred}} - \mathbf{E}_{\text{gt}} \|_2^2
$$

其中边缘图通过 Sobel 算子提取：

$$
\mathbf{E} = \sqrt{(\mathbf{G}_x \star \mathbf{I})^2 + (\mathbf{G}_y \star \mathbf{I})^2}
$$

### 2、连接组件后处理

**形态学优化**：

(1) 闭运算填充空洞：$\mathbf{M}_{\text{closed}} = (\mathbf{M} \oplus \mathbf{K}) \ominus \mathbf{K}$

(2) 开运算去除噪声：$\mathbf{M}_{\text{opened}} = (\mathbf{M} \ominus \mathbf{K}) \oplus \mathbf{K}$

(3) 最小外接矩形拟合

通过形态学操作优化分割结果的几何完整性，消除孔洞和噪点。
