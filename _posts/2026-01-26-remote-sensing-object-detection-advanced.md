---
title: 遥感目标检测技术
date: 2026-01-26
publish_display_date: 2026-01-26
excerpt: ""
categories: [Remote Sensing, Computer Vision]
tags: [遥感目标检测, 旋转目标检测, RoI Transformer, 小目标检测, 多任务学习]
layout: single
author_profile: true
---

遥感目标检测是遥感图像解译的核心任务之一。与自然图像不同，遥感图像中的目标具有任意方向、尺度差异大、背景复杂等特点，特别是目标的旋转任意性对检测算法提出了特殊要求。本文系统阐述旋转目标检测的数学基础、先进检测架构及多任务联合检测方法。

## 一、旋转目标检测的数学基础

### 1、旋转框表示法对比

**OpenCV 格式**：$(x_c, y_c, w, h, \theta)$，$\theta \in [-90^\circ, 0^\circ]$

**长边定义法**：$(x_c, y_c, w, h, \theta)$，$\theta \in [-90^\circ, 90^\circ]$

**角度周期性处理**：

$$
\theta_{\text{norm}} = \begin{cases}
\theta + 180^\circ & \text{if } \theta < -90^\circ \\
\theta - 180^\circ & \text{if } \theta \geq 90^\circ \\
\theta & \text{otherwise}
\end{cases}
$$

### 2、旋转IoU计算

**多边形相交面积计算**：
1. 将旋转矩形转换为4个角点
2. 使用 Sutherland-Hodgman 算法求多边形交集
3. 鞋带公式计算面积：

$$
\text{Area} = \frac{1}{2} \left| \sum_{i=1}^{n} (x_i y_{i+1} - x_{i+1} y_i) \right|
$$

**近似方法（rbbox方法）**：

$$
\text{IoU}_{\text{approx}} = \frac{\text{Area}_{\text{intersection}}}{\text{Area}_1 + \text{Area}_2 - \text{Area}_{\text{intersection}}}
$$

## 二、先进检测架构

### 1、RoI Transformer

水平 RoI 到旋转 RoI 的转换过程：

$$
\Delta \theta, \Delta x, \Delta y, \Delta w, \Delta h = \text{RegHead}(\mathbf{F}_{\text{hroi}})
$$

$$
\mathbf{b}_{\text{r}} = \mathbf{b}_{\text{h}} + \Delta \mathbf{b}
$$

其中 $\mathbf{F}_{\text{hroi}}$ 为水平 RoI 池化提取的特征，通过回归头预测旋转参数，将水平 RoI 自适应地转换为旋转 RoI。

### 2、密集小目标检测

**特征金字塔优化（AugFPN）**：
1. 一致性监督：所有尺度特征对齐
2. 残差特征增强
3. 软选择机制融合多尺度特征

**小目标注意力模块**：

$$
\mathbf{F}_{\text{enhanced}} = \mathbf{F} + \gamma \cdot \text{Conv}(\text{Sigmoid}(\mathbf{F}_{\text{small}}) \odot \mathbf{F})
$$

其中 $\mathbf{F}_{\text{small}}$ 为小目标检测分支的特征，$\gamma$ 为可学习的缩放因子，通过注意力机制增强小目标特征响应。

## 三、多任务联合检测

### 1、检测与分割联合

**Mask R-CNN 的旋转扩展**：

- 旋转 RoI Align + 旋转掩码头

联合损失函数：

$$
\mathcal{L} = \mathcal{L}_{\text{cls}} + \mathcal{L}_{\text{reg}} + \mathcal{L}_{\text{mask}}
$$

### 2、多时相检测

**时序一致性约束**：

$$
\mathcal{L}_{\text{temporal}} = \sum_{t=1}^{T-1} \| f_t(I_t) - f_{t+1}(I_{t+1}) \|_{\text{smooth}}
$$

通过约束相邻时相检测结果的一致性，利用时序信息提升检测的稳定性和准确性。
