---
title: 遥感图像处理技术
date: 2026-01-23
publish_display_date: 2026-01-23
excerpt: ""
categories: [Remote Sensing, Image Processing]
tags: [遥感, 辐射校正, 图像融合, CLAHE, 去云]
layout: single
author_profile: true
---

## 一、遥感数据预处理全流程

### 1、辐射校正模型

**传感器辐射定标**：将 DN 值转换为辐射亮度：

$$
L_{\lambda} = \text{Gain} \times \text{DN} + \text{Bias}
$$

**大气校正（6S模型）**：

地表反射率 $\rho$ 计算：

$$
\rho = \frac{\pi \cdot (L_{\text{sat}} - L_{\text{path}})}{\tau \cdot E_0 \cdot \cos(\theta_s)}
$$

其中：

- $L_{\text{sat}}$：卫星接收辐射
- $L_{\text{path}}$：大气程辐射
- $E_0$：大气层外太阳辐照度
- $\tau$：大气透射率
- $\theta_s$：太阳天顶角

### 2、几何校正与配准

**仿射变换模型**：

$$
\begin{bmatrix} x' \\ y' \end{bmatrix} = \begin{bmatrix} a & b \\ c & d \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} + \begin{bmatrix} e \\ f \end{bmatrix}
$$

**多项式校正（二阶）**：

$$
x' = a_0 + a_1 x + a_2 y + a_3 x^2 + a_4 xy + a_5 y^2
$$

$$
y' = b_0 + b_1 x + b_2 y + b_3 x^2 + b_4 xy + b_5 y^2
$$

**配准误差评估（RMSE）**：

$$
\text{RMSE} = \sqrt{ \frac{1}{N} \sum_{i=1}^{N} \left[ (x_i' - \hat{x}_i')^2 + (y_i' - \hat{y}_i')^2 \right] }
$$

## 二、图像融合技术

### 1、多光谱与全色融合

**Gram-Schmidt 变换融合**：

(1) 对多光谱波段模拟低分辨率全色图像：

$$
\mathbf{P}_{\text{syn}} = \sum_{i=1}^{M} \alpha_i \cdot \mathbf{MS}_i
$$

(2) 对模拟全色与真实全色进行 Gram-Schmidt 变换

(3) 用变换后的第一分量替换为高分辨率全色

(4) 逆变换得到融合结果

**PCA 融合的数学基础**：

设多光谱图像矩阵 $\mathbf{X} \in \mathbb{R}^{N \times M}$，协方差矩阵：

$$
\mathbf{C} = \frac{1}{N} \mathbf{X}^T \mathbf{X}
$$

特征值分解：$\mathbf{C} = \mathbf{V} \boldsymbol{\Lambda} \mathbf{V}^T$

第一主成分与全色图像替换：

$$
\mathbf{Y}_{\text{fused}} = \mathbf{V} \cdot \text{Replace}(\mathbf{PC}_1, \mathbf{PAN})
$$

### 2、深度学习融合方法

**PGFNet（渐进式梯度融合网络）**：

多尺度特征提取与渐进融合：

$$
\mathbf{F}_{\text{fused}} = \sum_{i=1}^{L} w_i \cdot \mathbf{F}_i
$$

其中 GFF（梯度特征融合）模块：

$$
\mathbf{F}_{\text{GFF}} = \text{Conv}(\text{Concat}[\nabla \mathbf{F}_1, \nabla \mathbf{F}_2, \cdots, \nabla \mathbf{F}_L])
$$

## 三、图像增强算法

### 1、对比度增强

**自适应直方图均衡化（CLAHE）**：

局部直方图裁剪与重分布：

$$
H'(k) = \begin{cases} H(k) & H(k) \leq \tau \\ \tau & H(k) > \tau \end{cases}
$$

其中 $\tau$ 为裁剪限制参数（通常取 2-3），通过限制直方图的高度避免过度增强噪声。

### 2、去云算法

**多时相云修复**：

基于时间序列的云去除：

$$
\hat{I}(x,y,t) = \sum_{i=1}^{T} w_i \cdot I(x,y,t_i)
$$

其中 $\text{CloudMask}$ 为云掩码，$w_i$ 为时间权重（近期权重更高）。

**深度学习去云（CloudGAN）**：

生成器损失函数：

$$
\mathcal{L}_G = \mathcal{L}_{\text{MSE}} + \lambda \mathcal{L}_{\text{perc}} + \beta \mathcal{L}_{\text{adv}}
$$

感知损失使用 VGG 特征：

$$
\mathcal{L}_{\text{perc}} = \sum_{l} \| \phi_l(\hat{I}) - \phi_l(I_{\text{gt}}) \|_2^2
$$

其中 $\phi_l$ 为预训练 VGG 网络第 $l$ 层的特征图。
