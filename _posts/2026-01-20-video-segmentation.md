---
title: 视频分割技术
date: 2026-01-20
publish_display_date: 2026-01-20
excerpt: ""
categories: [Computer Vision]
tags: [视频分割, 视频语义分割, 视频实例分割, VisTR, 光流, 记忆网络]
layout: single
author_profile: true
---

## 一、视频语义分割

### 1、基于光流的特征传播

**特征对齐**：

设 $t$ 帧特征 $\mathbf{F}_t$，$t+1$ 帧特征 $\mathbf{F}_{t+1}$，光流场 $\mathbf{V}_{t \to t+1}$：

$$
\mathbf{F}_{t \to t+1} = \text{Warp}(\mathbf{F}_t, \mathbf{V}_{t \to t+1})
$$

其中 $\text{Warp}$ 为双线性采样操作，通过光流将前一帧特征对齐到当前帧坐标空间。

**自适应权重学习**：

$$
\mathbf{F}_{\text{agg}} = \sum_{i=-k}^{k} w_i \cdot \text{Warp}(\mathbf{F}_{t+i}, \mathbf{V}_{t+i \to t})
$$

权重 $w_i$ 通过可学习网络预测，反映各帧特征的质量及其与当前帧的相关性。

### 2、记忆网络方法

**STM（Spatio-Temporal Memory）**：

记忆矩阵 $\mathcal{M} = \{\mathbf{K}_i, \mathbf{V}_i\}_{i=1}^{N}$，键 $\mathbf{K}$ 存储特征，值 $\mathbf{V}$ 存储掩码。

**查询过程**：

(1) 相似度计算：

$$
\mathbf{S}_i = \mathbf{Q} \cdot \mathbf{K}_i^T
$$

(2) 注意力权重：

$$
\alpha_i = \text{softmax}(\mathbf{S}_i / \sqrt{d})
$$

(3) 读取：

$$
\mathbf{M}_{\text{out}} = \sum_{i=1}^{N} \alpha_i \cdot \mathbf{V}_i
$$

**更新策略**：

- **FIFO**：先进先出，保留最近的帧
- **基于重要性**：根据预测不确定性决定是否保留和更新记忆，保留信息量大的帧，丢弃冗余信息

## 二、视频实例分割（VIS）

### 1、跟踪-分割联合建模

**MaskTrack R-CNN**：

在 Mask R-CNN 基础上添加跟踪分支，预测相邻帧实例对应关系。

**匹配损失**：

$$
\mathcal{L}_{\text{match}} = -\sum_{(i,j) \in \mathcal{M}} \log \mathbf{P}_{ij}
$$

其中 $\mathbf{P}_{ij}$ 为匹配概率函数，$\mathcal{M}$ 为匹配对集合，通过最大化匹配概率实现跨帧实例关联。

### 2、时序一致性约束

**循环一致性损失**：

对于三帧序列 $\{I_t, I_{t+1}, I_{t+2}\}$：

$$
\mathcal{L}_{\text{cycle}} = \sum_{\mathbf{p}} \| \mathbf{p} - \text{Warp}(\text{Warp}(\mathbf{p}, \mathbf{V}_{t \to t+1}), \mathbf{V}_{t+1 \to t+2}) \|_2^2
$$

**时序平滑损失**：

$$
\mathcal{L}_{\text{smooth}} = \sum_{t} \sum_{\mathbf{p}} \| \mathbf{M}_t(\mathbf{p}) - \mathbf{M}_{t+1}(\mathbf{p} + \mathbf{V}_{t \to t+1}(\mathbf{p})) \|_2^2
$$

其中 $\mathbf{M}_t$ 为第 $t$ 帧的分割掩膜，该损失约束相邻帧间掩膜沿光流方向保持一致。

## 三、Transformer在视频分割中的应用

### 1、VisTR

将视频视为序列，使用 Transformer 编码器-解码器架构：

**时序位置编码**：

$$
\text{PE}(t, 2i) = \sin\left( \frac{t}{10000^{2i/d}} \right)
$$

$$
\text{PE}(t, 2i+1) = \cos\left( \frac{t}{10000^{2i/d}} \right)
$$

其中 $t$ 为帧索引，$i$ 为维度索引，$d$ 为编码维度。

**空间位置编码**：

$$
\text{PE}(\mathbf{p}, 2i) = \sin\left( \frac{\mathbf{p}}{10000^{2i/d}} \right)
$$

**实例查询学习**：

可学习查询向量 $\mathbf{Q} \in \mathbb{R}^{N_{\text{ins}} \times d}$，其中 $N_{\text{ins}}$ 为最大实例数，通过解码器与视频特征交互，每个查询对应一个实例的分割结果。

### 2、Deformable Attention for Video

可变形注意力减少计算量，仅在参考点附近采样：

$$
\text{DeformAttn}(\mathbf{z}_q, \mathbf{p}_q, \mathbf{x}) = \sum_{m=1}^{M} \mathbf{W}_m \sum_{k=1}^{K} A_{mqk} \cdot \mathbf{W}_m' \mathbf{x}(\mathbf{p}_q + \Delta \mathbf{p}_{mqk})
$$

其中 $\Delta \mathbf{p}_{mqk}$ 为可学习的偏移量，$M$ 为注意力头数，$K$ 为采样点数。

## 四、评估指标

### 1、视频分割质量（VSQ）

综合评估分割精度与时序一致性，通常包含：
- **区域相似度**：$m\text{IoU}$（平均交并比）
- **轮廓准确度**：$m\text{AP}$（平均精度）

### 2、时序一致性指标

**时间稳定性误差（TSE）**：

$$
\text{TSE} = \frac{1}{T-1} \sum_{t=1}^{T-1} \frac{| \mathbf{M}_t \setminus \text{Warp}(\mathbf{M}_{t+1}, \mathbf{V}_{t+1 \to t}) |}{|\mathbf{M}_t \cup \text{Warp}(\mathbf{M}_{t+1}, \mathbf{V}_{t+1 \to t})|}
$$

其中 $\setminus$ 为对称差，用于衡量相邻帧分割结果在光流对齐后的一致性。TSE 值越低，表示模型的时间稳定性越好。
