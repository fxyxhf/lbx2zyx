---
title: 遥感变化检测技术
date: 2026-01-27
publish_display_date: 2026-01-27
excerpt: ""
categories: [Remote Sensing, Computer Vision]
tags: [遥感变化检测, CVA, Siamese网络, 多时相分析, LSTM]
layout: single
author_profile: true
---

遥感变化检测是通过分析同一地区不同时相的遥感影像，识别地表覆盖类型或状态变化的过程，广泛应用于土地利用/覆盖变化监测、灾害评估、城市扩张分析等领域。本文系统阐述传统变化检测方法、深度学习变化检测方法及多时相变化检测技术。

## 一、传统变化检测方法

### 1、像素级变化检测

**差异图像生成**：

- 代数法：$\mathbf{D} = |\mathbf{I}_2 - \mathbf{I}_1|$
- 比值法：$\mathbf{D} = \frac{\mathbf{I}_2}{\mathbf{I}_1}$（或 $\log(\mathbf{I}_2) - \log(\mathbf{I}_1)$）

**变化阈值确定（OTSU算法）**：

最大化类间方差：

$$
\sigma_B^2 = \omega_0 \cdot (\mu_0 - \mu_T)^2 + \omega_1 \cdot (\mu_1 - \mu_T)^2
$$

最优阈值：$T^* = \arg\max_T \sigma_B^2(T)$，其中 $\omega_0$、$\omega_1$ 分别为背景和目标类的像素比例，$\mu_0$、$\mu_1$、$\mu_T$ 为对应类的灰度均值。

### 2、CVA（Change Vector Analysis）

**变化向量**：将多波段变化量构成向量：

$$
\Delta \mathbf{v} = \mathbf{v}_2 - \mathbf{v}_1
$$

**变化幅度**：衡量变化的强度：

$$
\|\Delta \mathbf{v}\| = \sqrt{ \sum_{b=1}^{B} (v_{2b} - v_{1b})^2 }
$$

**变化方向**：反映变化的类型：

$$
\theta = \arctan\left( \frac{\Delta v_2}{\Delta v_1} \right)
$$

## 二、深度学习变化检测

### 1、双时相编码器-解码器

**Siamese U-Net 架构**：

共享权重的双编码器：

$$
\mathbf{F}_1 = \text{Encoder}(\mathbf{I}_1), \quad \mathbf{F}_2 = \text{Encoder}(\mathbf{I}_2)
$$

差异特征：

$$
\mathbf{F}_{\text{diff}} = |\mathbf{F}_1 - \mathbf{F}_2|
$$

解码器：

$$
\hat{\mathbf{M}} = \text{Decoder}(\mathbf{F}_{\text{diff}})
$$

### 2、注意力增强的变化检测

**时空注意力模块**：

$$
\mathbf{F}_{\text{att}} = \mathbf{F}_{\text{diff}} \odot \text{Sigmoid}(\text{Conv}(\mathbf{F}_{\text{diff}}))
$$

通过注意力机制增强变化区域的响应，抑制伪变化干扰。

### 3、多尺度变化检测

**金字塔特征融合**：

$$
\mathbf{F}_{\text{fused}} = \sum_{s=1}^{S} w_s \cdot \text{UpSample}(\mathbf{F}_{\text{diff}}^s)
$$

## 三、多时相变化检测

### 1、时间序列分析

**LSTM-based 变化检测**：

隐状态更新：

$$
\mathbf{h}_t = \text{LSTM}(\mathbf{F}_t, \mathbf{h}_{t-1})
$$

变化概率：

$$
\mathbf{P}_t = \sigma(\text{MLP}(\mathbf{h}_t))
$$

### 2、Transformer时序建模

**时间位置编码**：

$$
\text{PE}(t, 2i) = \sin\left( \frac{t}{10000^{2i/d}} \right)
$$

$$
\text{PE}(t, 2i+1) = \cos\left( \frac{t}{10000^{2i/d}} \right)
$$

**时间注意力**：

$$
\mathbf{F}_{\text{out}} = \sum_{t=1}^{T} \alpha_t \cdot \mathbf{F}_t
$$

其中 $\alpha_t$ 通过注意力机制学习得到：

$$
\alpha_t = \text{softmax}\left( \frac{\mathbf{Q} \cdot \mathbf{K}_t^T}{\sqrt{d_k}} \right)
$$

## 总结与展望

遥感变化检测技术经历了从像素级差异分析到深度学习语义变化检测的演进。传统方法（CVA、OTSU）计算高效但鲁棒性有限，深度学习方法（Siamese U-Net、注意力机制）显著提升了检测精度和泛化能力，而基于 LSTM 和 Transformer 的多时相方法则为捕捉时序依赖提供了新的技术路径。未来趋势集中在弱监督变化检测、跨传感器变化检测及实时变化监测等方面。
