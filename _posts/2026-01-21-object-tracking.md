---
title: 目标跟踪技术
date: 2026-01-21
publish_display_date: 2026-01-21
excerpt: ""
categories: [Computer Vision]
tags: [目标跟踪, 相关滤波, Siamese网络, 多目标跟踪, 卡尔曼滤波, DeepSORT]
layout: single
author_profile: true
---

## 一、任务定义与分类

### 1、任务形式化定义

给定视频序列 $\{I_1, I_2, \cdots, I_T\}$ 和初始帧目标状态 $\mathbf{b}_1$，目标跟踪旨在预测后续帧中的目标状态序列：

$$
\{\mathbf{b}_t\}_{t=2}^{T} = \arg\max_{\mathbf{b}_t} P(\mathbf{b}_t \mid \mathbf{b}_1, I_1, \cdots, I_t)
$$

其中 $\mathbf{b}_t$ 通常表示为边界框 $(x, y, w, h)$ 或分割掩膜。

### 2、分类体系

- **单目标跟踪（SOT）**：跟踪单个预定义目标
- **多目标跟踪（MOT）**：同时跟踪多个目标，需要处理目标出现/消失
- **视频目标分割（VOS）**：像素级跟踪，输出掩码而非边界框

## 二、核心跟踪框架与算法

### 1、判别式相关滤波（DCF）

基本原理：学习一个滤波器 $\mathbf{w}$，使其与目标特征 $\mathbf{f}$ 的相关响应最大：

$$
\min_{\mathbf{w}} \| \mathbf{f} \star \mathbf{w} - \mathbf{g} \|_2^2
$$

其中 $\mathbf{g}$ 为期望响应（通常为2D高斯函数），$\star$ 为相关操作。

**频域求解（MOSSE算法）**：

$$
\mathbf{W} = \frac{\mathbf{F} \odot \mathbf{G}^*}{\mathbf{F} \odot \mathbf{F}^* + \lambda}
$$

其中 $\mathbf{F}$、$\mathbf{G}$ 分别为 $\mathbf{f}$、$\mathbf{g}$ 的傅里叶变换，$\lambda$ 为正则化参数。

**在线更新**：

$$
\mathbf{W}_t = (1 - \eta) \mathbf{W}_{t-1} + \eta \mathbf{W}_{\text{current}}
$$

其中 $\eta$ 为学习率，控制模型更新速度。

### 2、Siamese网络跟踪

**SiamFC（全卷积Siamese网络）**：

$$
\mathbf{R} = \varphi(\mathbf{Z}) \star \varphi(\mathbf{X})
$$

其中 $\varphi$ 为特征提取网络，$\star$ 为互相关操作，$\varphi(\mathbf{Z})$ 为模板特征，$\varphi(\mathbf{X})$ 为搜索区域特征。

**响应图计算**：设模板特征 $\mathbf{F}_Z$，搜索区域特征 $\mathbf{F}_X$，输出响应图 $\mathbf{R}$：

$$
\mathbf{R} = \text{corr}(\mathbf{F}_Z, \mathbf{F}_X) = \mathcal{F}^{-1}(\mathcal{F}(\mathbf{F}_Z) \odot \mathcal{F}(\mathbf{F}_X)^*)
$$

### 3、Transformer跟踪

**TransT（Transformer Tracking）**：编码器-解码器架构处理模板和搜索区域特征。

**特征融合模块**：设模板特征 $\mathbf{F}_Z$，搜索特征 $\mathbf{F}_X$，交叉注意力机制：

$$
\text{CrossAttn}(\mathbf{F}_X, \mathbf{F}_Z) = \text{softmax}\left( \frac{\mathbf{Q}_X \mathbf{K}_Z^T}{\sqrt{d_k}} \right) \mathbf{V}_Z
$$

其中 $\mathbf{Q}_X = \mathbf{F}_X \mathbf{W}_Q$，$\mathbf{K}_Z = \mathbf{F}_Z \mathbf{W}_K$，$\mathbf{V}_Z = \mathbf{F}_Z \mathbf{W}_V$。

## 三、目标状态估计与更新

### 1、卡尔曼滤波在跟踪中的应用

**状态向量**：$\mathbf{x} = [x, y, w, h, \dot{x}, \dot{y}, \dot{w}, \dot{h}]^T$

**状态转移矩阵**：

$$
\mathbf{F} = \begin{bmatrix}
1 & 0 & 0 & 0 & \Delta t & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & \Delta t & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 & \Delta t & 0 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & \Delta t \\
0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1
\end{bmatrix}
$$

**观测矩阵**（检测器提供观测 $\mathbf{z} = [x, y, w, h]^T$）：

$$
\mathbf{H} = \begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0
\end{bmatrix}
$$

### 2、模型在线更新策略

**模板更新机制**：线性插值更新：

$$
\mathbf{T}_t = (1 - \eta_t) \mathbf{T}_{t-1} + \eta_t \mathbf{T}_{\text{current}}
$$

其中 $\eta_t$ 为更新率，根据跟踪置信度 $c_t$ 自适应调整：

$$
\eta_t = \begin{cases} \eta_{\text{high}} & c_t > \tau_{\text{high}} \\ \eta_{\text{low}} & \tau_{\text{low}} < c_t \leq \tau_{\text{high}} \\ 0 & c_t \leq \tau_{\text{low}} \end{cases}
$$

## 四、多目标跟踪（MOT）

### 1、数据关联问题

**匈牙利算法**：最小化分配代价：

$$
\min_{\pi} \sum_{i=1}^{m} \sum_{j=1}^{n} c_{ij} \cdot \pi_{ij}
$$

其中代价矩阵 $\mathbf{C} \in \mathbb{R}^{m \times n}$，$\pi$ 为分配矩阵。

**外观相似度（余弦距离）**：

$$
d_{\text{app}}(i, j) = 1 - \frac{\mathbf{f}_i \cdot \mathbf{f}_j}{\|\mathbf{f}_i\|_2 \cdot \|\mathbf{f}_j\|_2}
$$

**运动相似度（马氏距离）**：

$$
d_{\text{mot}}(i, j) = (\mathbf{z}_j - \hat{\mathbf{x}}_i)^T \mathbf{S}_i^{-1} (\mathbf{z}_j - \hat{\mathbf{x}}_i)
$$

### 2、SORT / DeepSORT算法

**SORT**：卡尔曼滤波 + 匈牙利算法，基于IOU进行数据关联。

**DeepSORT增强**：
1. 添加外观特征匹配（使用CNN提取外观特征）
2. 级联匹配策略：优先匹配最近出现的轨迹
3. IOU匹配作为后备关联

## 五、评估指标

### 1、单目标跟踪

**(1) 精确度图（Precision Plot）**

中心位置误差小于阈值的帧占比：

$$
\text{Precision} = \frac{1}{T} \sum_{t=1}^{T} \mathbb{I}(\| \mathbf{p}_t - \mathbf{p}_t^{\text{gt}} \|_2 \leq \tau)
$$

**(2) 成功率图（Success Plot）**

重叠率大于阈值的帧占比：

$$
\text{Success} = \frac{1}{T} \sum_{t=1}^{T} \mathbb{I}(\text{IoU}(\mathbf{b}_t, \mathbf{b}_t^{\text{gt}}) \geq \tau)
$$

### 2、多目标跟踪

**MOTA（多目标跟踪准确度）**：

$$
\text{MOTA} = 1 - \frac{\text{FP} + \text{FN} + \text{IDSW}}{\text{GT}}
$$

其中 FP 为误报数，FN 为漏报数，IDSW 为身份切换次数，GT 为真值目标总数。

**IDF1（身份F1分数）**：

$$
\text{IDF1} = \frac{2 \cdot \text{IDTP}}{2 \cdot \text{IDTP} + \text{IDFP} + \text{IDFN}}
$$

IDF1 衡量跟踪器正确保持目标身份的能力，是评估多目标跟踪中身份稳定性的核心指标。
