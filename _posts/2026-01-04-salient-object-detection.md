---
title: 显著性目标检测技术
date: 2026-01-04
publish_display_date: 2026-01-04
excerpt: ""
categories: [Computer Vision]
tags: [显著性检测, U²-Net, 弱监督学习, 谱残差, 视觉注意]
layout: single
author_profile: true
---

## 一、显著性建模的理论基础

### 1、中心-周围对比度

经典的 Itti-Koch 模型通过多尺度特征对比来模拟视觉注意机制：

$$
S(\mathbf{p}) = \sum_{s=1}^{N} \sum_{c \in \mathcal{C}} \left\| \mathbf{F}_c(\mathbf{p}, s) - \mathbf{F}_c(\mathbf{p}, s+1) \right\|_2
$$

其中 $\mathbf{F}_c(\mathbf{p}, s)$ 为尺度 $s$ 下特征 $c$ 在位置 $\mathbf{p}$ 的值，$\mathcal{C} = \{\text{颜色}, \text{亮度}, \text{方向}\}$ 为特征通道集合，$N$ 通常取 2 或 3。该模型通过跨尺度差分来检测显著性区域，体现中心与周围之间的特征差异。

### 2、频域分析方法

基于谱残差的显著性检测方法通过频域变换分离场景中的异常成分：

对图像 $I(x,y)$ 进行傅里叶变换：

$$
\mathcal{F}(I) = \mathcal{F}(I)
$$

计算幅度谱 $A(f)$ 和相位谱 $P(f)$：

$$
A(f) = |\mathcal{F}(I)|, \quad P(f) = \mathcal{F}(I) / A(f)
$$

对数幅度谱 $\mathcal{L}(f)$ 为：

$$
\mathcal{L}(f) = \log(A(f))
$$

谱残差 $R(f)$ 定义为对数幅度谱与其均值滤波结果之差：

$$
R(f) = \mathcal{L}(f) - \text{MeanFilter}(\mathcal{L}(f))
$$

其中 $\text{MeanFilter}$ 为均值滤波器。最终显著性图 $S(x)$ 通过逆傅里叶变换得到：

$$
S(x) = \mathcal{F}^{-1}\left( \exp(R(f) + j \cdot P(f)) \right)
$$

该方法计算效率高，适用于快速显著性检测场景。

## 二、深度显著性网络架构

### 1、U²-Net：嵌套U型结构

U²-Net 的核心创新在于在每个层级都使用 U 型块（U-block）作为基本构建单元：

$$
\mathbf{F}_{\text{out}} = \mathcal{U}\left( \mathbf{F}_{\text{in}} \right)
$$

其中 $\mathcal{U}$ 表示特征融合与编码-解码操作，每个 U-block 包含 5-6 个卷积层，形成嵌套的 U 型结构：

$$
\mathbf{F} = \text{Concat}\left( \text{Encode}(\mathbf{F}_{\text{in}}), \text{Decode}(\mathbf{F}_{\text{enc}}) \right)
$$

**损失函数设计**：

$$
\mathcal{L} = \sum_{i=1}^{M} \lambda_i \cdot \ell(\mathbf{S}_i, \mathbf{G})
$$

其中 $M$ 为中间监督层数，通常 $M = 6$，$\mathbf{S}_i$ 为第 $i$ 层输出的显著性图，$\mathbf{G}$ 为真值掩膜，$\lambda_i$ 为各层损失的权重系数。这种多层次深度监督机制使网络能够同时捕获局部细节与全局语义信息。

### 2、边缘感知损失

结合边界预测的联合损失函数：

$$
\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{sal}} + \lambda \cdot \mathcal{L}_{\text{edge}}
$$

其中边缘损失通常使用 Dice 损失：

$$
\mathcal{L}_{\text{edge}} = 1 - \frac{2 \sum_{i} \mathbf{E}_i \cdot \mathbf{G}_i^{\text{edge}}}{\sum_{i} \mathbf{E}_i + \sum_{i} \mathbf{G}_i^{\text{edge}} + \epsilon}
$$

$\mathbf{E}_i$ 为预测的边缘图，$\mathbf{G}_i^{\text{edge}}$ 为边缘真值，$\epsilon$ 为平滑项防止除零。通过显式建模目标边界信息，边缘感知损失能够引导网络生成轮廓更清晰、定位更精确的显著性图。

## 三、弱监督学习方法

使用图像级标签训练显著性检测器，无需像素级标注：

**(1) CAM生成初始种子**

通过分类网络生成类别激活图（Class Activation Mapping）：

$$
\mathbf{CAM}(x, y) = \sum_{k} w_k \cdot \mathbf{F}_k(x, y)
$$

其中 $w_k$ 为第 $k$ 个通道的分类权重，$\mathbf{F}_k$ 为最后一层卷积特征图。CAM 提供目标位置的粗略响应，作为显著性初始种子。

**(2) 迭代精炼**

通过条件随机场（CRF）优化显著性图：

$$
E(\mathbf{S}) = \sum_{i} \phi_u(s_i) + \sum_{(i,j) \in \mathcal{E}} \phi_p(s_i, s_j)
$$

其中一元势函数 $\phi_u(s_i)$ 由 CAM 初始种子指导：

$$
\phi_u(s_i) = -\log P(s_i)
$$

成对势函数 $\phi_p(s_i, s_j)$ 为：

$$
\phi_p(s_i, s_j) = \mu(s_i, s_j) \cdot \exp\left( -\frac{\|\mathbf{p}_i - \mathbf{p}_j\|_2^2}{2\sigma_p^2} - \frac{\|\mathbf{I}_i - \mathbf{I}_j\|_2^2}{2\sigma_I^2} \right)
$$

其中 $\mu$ 为标签兼容性函数，$\mathbf{p}_i$ 为像素位置，$\mathbf{I}_i$ 为像素颜色值。通过 CRF 迭代优化，CAM 的粗略响应被逐步精炼为高质量的像素级显著性预测。

**(3) 迭代优化策略**：将 CRF 优化后的结果作为新的伪标签，重新训练显著性网络，重复若干次，实现自步学习与性能提升。
