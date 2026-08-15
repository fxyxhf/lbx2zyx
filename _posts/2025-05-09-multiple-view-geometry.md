---
title: 多视图几何理论与方法
date: 2025-05-09
publish_display_date: 2025-05-09
excerpt: ""
categories: [Computer Vision, 3D Vision]
tags: [多视图几何, SfM, MVS, 光束法平差, 对极几何, 三维重建]
layout: single
author_profile: true
---

多视图几何是计算机视觉与摄影测量学的核心理论分支，旨在通过多个视角的图像观测，联合恢复三维场景结构及相机运动参数。其核心在于利用多视点间的几何约束关系（如极线约束、重投影一致性）构建数学优化问题，最终实现场景的三维重建与运动估计。以下从理论框架、数学模型、关键算法及前沿挑战四方面展开论述。

## 一、理论框架与数学基础

### 1、对极几何

定义两视图间的几何关系，核心为基础矩阵 $\mathbf{F}$，满足对极约束方程：

$$
\mathbf{x}'^T \mathbf{F} \mathbf{x} = 0
$$

其中 $\mathbf{x}$、$\mathbf{x}'$ 为对应点在两视图的齐次坐标。

本质矩阵 $\mathbf{E}$ 为归一化坐标系下的基础矩阵，满足：

$$
\mathbf{E} = [\mathbf{t}]_\times \mathbf{R}
$$

其中 $\mathbf{R}$、$\mathbf{t}$ 为两相机间的旋转与平移，$[\mathbf{t}]_\times$ 为反对称矩阵算子。

### 2、多视张量

**三视张量（Trifocal Tensor）**：

描述三视图间的几何约束，满足：

$$
\sum_{i} \mathbf{x}_i^T \mathbf{T}_i \mathbf{x}_i' = 0
$$

其中 $\mathbf{T}_i$ 对应三个相机的投影矩阵列向量。

**四视张量（Quadrifocal Tensor）**：

扩展至四视图，建立四线性约束关系。

### 3、投影模型与三角化

对于第 $i$ 个相机，三维点 $\mathbf{X}$ 的投影方程为：

$$
\lambda_i \mathbf{x}_i = \mathbf{K}_i \begin{bmatrix} \mathbf{R}_i & \mathbf{t}_i \end{bmatrix} \mathbf{X}
$$

三角化通过最小化重投影误差求解 $\mathbf{X}$：

$$
\min_{\mathbf{X}} \sum_{i=1}^{M} \| \mathbf{x}_i - \pi_i(\mathbf{X}) \|_2^2
$$

其中 $\pi_i$ 为第 $i$ 个相机的投影函数。

## 二、关键算法体系

### 1、结构从运动（SfM）

**增量式重建流程**：

1. 特征匹配（SIFT/SURF）
2. 两视图初始化（5点算法求解 $\mathbf{E}$）
3. 增量注册新视图（PnP 求解相机位姿）
4. 光束法平差全局优化：

$$
\min_{\{\mathbf{R}_i, \mathbf{t}_i\}, \{\mathbf{X}_j\}} \sum_{i=1}^{M} \sum_{j=1}^{N} \| \mathbf{x}_{ij} - \pi_i(\mathbf{X}_j) \|_2^2
$$

**全局式重建**：基于旋转平均与平移平均直接优化全局参数。

### 2、多视图立体匹配（MVS）

**代价体构建**：在离散深度/视差空间构建多视相似性测度：

$$
\mathbf{C}(\mathbf{p}, d) = \sum_{i=1}^{N} \rho \left( \mathbf{I}_i(\mathbf{H}_i(\mathbf{p}, d)) - \mathbf{I}_{\text{ref}}(\mathbf{p}) \right)
$$

其中 $\rho$ 为匹配代价函数（如 ZNCC），$\mathbf{H}_i$ 为单应性变换。

**深度图融合**：基于泊松表面重建或截断符号距离函数融合多视角深度图。

### 3、动态场景处理

**多体运动恢复**：引入运动分割分离独立运动目标。

**时空 BA**：联合优化结构与运动轨迹：

$$
\min \sum_{t=1}^{T} \sum_{i=1}^{M_t} \sum_{j=1}^{N_t} \| \mathbf{x}_{ij}(t) - \pi_i(\mathbf{X}_j(t)) \|_2^2
$$

## 三、数学工具与优化方法

### 1、李群与李代数

相机位姿 $\mathbf{T} \in \text{SE}(3)$ 参数化为李代数 $\boldsymbol{\xi} \in \mathfrak{se}(3)$：

$$
\mathbf{T}(\boldsymbol{\xi}) = \exp(\boldsymbol{\xi}^{\wedge})
$$

其中 $\boldsymbol{\xi}^{\wedge}$ 为生成元，$\boldsymbol{\xi}$ 为李代数坐标。

### 2、鲁棒估计理论

**RANSAC** 用于鲁棒估计基础矩阵：

$$
N = \frac{\log(1 - p)}{\log(1 - (1 - \epsilon)^s)}
$$

其中 $p$ 为置信度，$\epsilon$ 为内点率，$s$ 为样本大小。

**M-估计**：使用 Huber 损失函数抑制外点：

$$
\rho(e) = \begin{cases} \frac{1}{2} e^2 & |e| \leq \delta \\ \delta(|e| - \frac{1}{2}\delta) & \text{其他} \end{cases}
$$

### 3、非凸优化策略

**松弛技术**：半定规划求解旋转矩阵：

$$
\min_{\mathbf{R}} \| \mathbf{R} - \mathbf{R}_{\text{init}} \|_F^2 \quad \text{s.t.} \quad \mathbf{R} \in \text{SO}(3), \quad \mathbf{R} \succeq 0
$$

**分支定界法**：全局优化相机位姿初值。

## 四、前沿挑战与研究趋势

### 1、大规模场景优化

- **分布式 BA**：采用 Hadoop/Spark 实现参数分块优化，处理百万级点云
- **层次式 BA**：基于场景图分割为子问题迭代优化

### 2、深度学习融合

- **端到端 SfM**：利用 GNN 直接预测相机位姿与三维点
- **神经辐射场（NeRF）**：联合优化多视图几何与隐式辐射场：

$$
\mathcal{L}_{\text{NeRF}} = \sum_{\mathbf{r}} \| \hat{C}(\mathbf{r}) - C(\mathbf{r}) \|_2^2 + \lambda \sum_{\mathbf{p}} \| D_{\text{pred}}(\mathbf{p}) - D_{\text{MVS}}(\mathbf{p}) \|_2^2
$$

### 3、动态场景建模

- 时空联合优化：引入光流约束与物理运动先验
- 事件相机集成：利用事件流的高时间分辨率处理高速运动

### 4、鲁棒性与实时性

- 硬件加速：基于 FPGA/GPU 的 BA 并行化
- 增量式 SLAM：滑动窗口优化与边缘化

## 五、方法对比

| 方法 | 精度 | 效率 | 适用场景 | 关键挑战 |
|------|------|------|----------|----------|
| 增量式 SfM | 高 | 中 | 通用场景 | 累积误差 |
| 全局式 SfM | 中 | 高 | 大规模场景 | 旋转平均鲁棒性 |
| MVS | 高 | 低 | 稠密重建 | 弱纹理区域 |
| NeRF | 高 | 极低 | 高质量渲染 | 计算资源消耗 |

## 六、总结

多视图几何作为三维视觉的理论基石，经历了从经典几何约束建模到深度特征融合的演进。当前研究正朝着大规模高效优化、动态场景建模以及多模态数据融合等方向持续突破。未来发展趋势将集中在深度学习与传统几何方法的深度融合、实时性优化以及大规模场景的鲁棒处理上。
