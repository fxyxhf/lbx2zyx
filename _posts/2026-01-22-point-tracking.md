---
title: 点跟踪技术
date: 2026-01-22
publish_display_date: 2026-01-22
excerpt: ""
categories: [Computer Vision]
tags: [点跟踪, 光流, KLT, RAFT, PIPs, 轨迹跟踪]
layout: single
author_profile: true
---

点跟踪是计算机视觉中的基础任务，旨在在视频序列中跟踪稀疏点的运动轨迹。与目标跟踪不同，点跟踪关注像素级或亚像素级的点对应关系，广泛应用于运动分析、三维重建、视频编辑等领域。本文系统阐述点跟踪的任务定义、传统方法、深度学习方法和多帧点跟踪技术。

## 一、任务定义与挑战

### 1、任务形式化

给定视频序列 $\{I_1, I_2, \cdots, I_T\}$ 和第一帧的 $N$ 个点集 $\{\mathbf{p}_1^1, \mathbf{p}_2^1, \cdots, \mathbf{p}_N^1\}$，目标是估计每个点在所有帧中的轨迹：

$$
\mathbf{p}_i^t = \text{Track}(\mathbf{p}_i^1, I_1, \cdots, I_t)
$$

其中 $\mathbf{p}_i^t$ 为第 $i$ 个点在第 $t$ 帧的位置，若点不可见则标记为 $\text{occluded}$。

### 2、关键挑战

**(1) 大位移**：点可能在帧间移动很大距离，超出局部搜索范围

**(2) 遮挡处理**：点可能被遮挡后重新出现，需要处理点的消失与重现

**(3) 点对应模糊**：相似外观点之间的混淆，导致错误匹配

**(4) 长序列跟踪**：误差累积导致轨迹漂移

## 二、传统点跟踪方法

### 1、光流法（Optical Flow）

**亮度恒定假设**：

$$
I(x + u, y + v, t + \Delta t) = I(x, y, t)
$$

**一阶泰勒展开**：

$$
I_x u + I_y v + I_t = 0
$$

其中 $I_x = \frac{\partial I}{\partial x}$，$I_y = \frac{\partial I}{\partial y}$，$I_t = \frac{\partial I}{\partial t}$。

**Lucas-Kanade方法**：

假设局部窗口内光流恒定，最小化：

$$
\min_{\mathbf{d}} \sum_{(x,y) \in \mathcal{W}} \| \nabla I(x,y) \cdot \mathbf{d} + I_t(x,y) \|_2^2
$$

**正规方程求解**：

$$
\mathbf{A}^T \mathbf{A} \mathbf{d} = \mathbf{A}^T \mathbf{b}
$$

其中 $\mathbf{A} = \begin{bmatrix} I_x(\mathbf{p}_1) & I_y(\mathbf{p}_1) \\ \vdots & \vdots \\ I_x(\mathbf{p}_n) & I_y(\mathbf{p}_n) \end{bmatrix}$，$\mathbf{b} = \begin{bmatrix} -I_t(\mathbf{p}_1) \\ \vdots \\ -I_t(\mathbf{p}_n) \end{bmatrix}$。

### 2、Kanade-Lucas-Tomasi（KLT）跟踪器

**特征点选择**：选择具有显著特征（角点）的点，通过计算结构张量特征值判断：

$$
\lambda_{\min} > \tau
$$

选择 $\lambda_{\min}$ 大于阈值的点作为特征点，确保点在空间上具有足够的变化信息。

**金字塔L-K**：

在多尺度金字塔上从粗到细计算光流，先在大尺度上估计粗略运动，再逐级细化到原始分辨率，以处理大位移。

## 三、深度学习方法

### 1、RAFT（Recurrent All-Pairs Field Transforms）

**核心架构**：

(1) **特征提取**：提取多尺度特征

(2) **相关性体积**：计算所有点对间的相似度

(3) **循环更新**：GRU迭代更新光流场

**相关性计算**：

设特征图 $\mathbf{F}_1 \in \mathbb{R}^{H \times W \times D}$，$\mathbf{F}_2 \in \mathbb{R}^{H \times W \times D}$，相关性体积 $\mathbf{C} \in \mathbb{R}^{H \times W \times H \times W}$：

$$
\mathbf{C}(i,j,k,l) = \langle \mathbf{F}_1(i,j), \mathbf{F}_2(k,l) \rangle
$$

**光流迭代更新**：

$$
\mathbf{o}^{(t+1)} = \mathbf{o}^{(t)} + \Delta \mathbf{o}
$$

$$
\mathbf{f}^{(t+1)} = \mathbf{f}^{(t)} + \mathbf{o}^{(t+1)}
$$

其中 $\mathbf{o}^{(t)}$ 为第 $t$ 次迭代的光流增量，$\mathbf{f}^{(t)}$ 为累积光流。

### 2、PIPs（Persistent Independent Particles）

**轨迹表示**：将点表示为持续独立的粒子，每个粒子独立跟踪其轨迹。

**轨迹预测网络**：

输入：以点为中心的图像块序列 $\{\mathbf{P}_i\}_{i=1}^{T}$

输出：未来帧的轨迹坐标 $\{\hat{\mathbf{p}}_t\}_{t=1}^{T}$ 和可见性 $\{\hat{v}_t\}_{t=1}^{T}$

**损失函数**：

$$
\mathcal{L} = \sum_{t=1}^{T} \| \hat{\mathbf{p}}_t - \mathbf{p}_t^{\text{gt}} \|_2^2 \cdot \mathbb{I}(v_t^{\text{gt}} = 1) + \text{BCE}(\hat{v}_t, v_t^{\text{gt}})
$$

## 四、多帧点跟踪

### 1、基于图的轨迹关联

构建完全图 $\mathcal{G} = (\mathcal{V}, \mathcal{E})$，其中：

- 顶点 $\mathcal{V}$：第 $t$ 帧第 $i$ 个检测点 $v_{t,i}$
- 边权重 $w_{e}$：表示关联概率

**轨迹形成**：通过求解最小成本流问题：

$$
\min_{\mathbf{x}} \sum_{e \in \mathcal{E}} c_e \cdot x_e
$$

其中 $x_e \in \{0,1\}$ 表示边是否激活，$c_e$ 为关联代价。

### 2、轨迹平滑与滤波

**样条插值**：对轨迹进行平滑处理

设轨迹点 $\{\mathbf{p}_1, \mathbf{p}_2, \cdots, \mathbf{p}_T\}$，拟合三次样条曲线：

$$
\mathbf{S}(t) = \sum_{i=1}^{T} \mathbf{c}_i B_i(t)
$$

满足连续性条件：$\mathbf{S}$ 连续，$\mathbf{S}'$ 连续。

## 五、评估指标

**端点误差（EPE）**：

$$
\text{EPE} = \frac{1}{N} \sum_{i=1}^{N} \| \mathbf{p}_i^{\text{pred}} - \mathbf{p}_i^{\text{gt}} \|_2
$$

**跟踪准确率（TA）**：

$$
\text{TA} = \frac{1}{N} \sum_{i=1}^{N} \mathbb{I}(\| \mathbf{p}_i^{\text{pred}} - \mathbf{p}_i^{\text{gt}} \|_2 < \tau)
$$

其中 $\tau$ 为误差阈值，通常取 3-5 像素。

**轨迹完整性（TI）**：

$$
\text{TI} = \frac{1}{N} \sum_{i=1}^{N} \frac{T_i^{\text{tracked}}}{T_i^{\text{total}}}
$$

其中 $T_i^{\text{tracked}}$ 为第 $i$ 个轨迹成功跟踪的帧数，$T_i^{\text{total}}$ 为该轨迹存在的总帧数。
