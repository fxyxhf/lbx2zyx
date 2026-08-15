---
title: 双目立体视觉技术
date: 2025-05-05
publish_display_date: 2025-05-05
excerpt: ""
categories: [3D Vision, Computer Vision]
tags: [双目视觉, 立体匹配, 极线几何, SGM, 深度学习]
layout: single
author_profile: true
---

双目视觉，亦称立体视觉，是一种基于多视几何理论的三维场景重建技术。其核心机制在于模拟生物双眼视差感知原理，通过两个空间分离的成像传感器获取场景的多视角观测数据，进而解算目标物体的三维空间坐标。本论述将从数学模型、关键技术及算法演进三个维度展开分析。

## 一、双目视觉的几何基础

### 1、极线几何

定义两相机光心 $\mathbf{C}_1$、$\mathbf{C}_2$ 构成的基线 $\mathbf{B}$，任一三维点 $\mathbf{P}$ 在两像平面投影点 $\mathbf{p}_1$、$\mathbf{p}_2$ 满足极线约束：

$$
\mathbf{p}_2^T \mathbf{F} \mathbf{p}_1 = 0
$$

其中 $\mathbf{F}$ 为基本矩阵，编码了相机的相对位姿参数，是极线几何的代数表达形式。

### 2、视差-深度转换模型

基于三角测量原理，深度 $Z$ 与视差 $d$ 的关系为：

$$
Z = \frac{B \cdot f}{d}
$$

其中 $B$ 为基线长度，$f$ 为相机焦距，$d = x_L - x_R$ 为水平视差。

### 3、立体校正

通过 Bouguet 算法将非共面图像对变换为共面行对准形式，将极线约束简化为水平扫描线搜索：

$$
\mathbf{R}_{\text{rect}} \cdot \mathbf{K}^{-1} \mathbf{p} = \mathbf{R}_{\text{rect}} \cdot \mathbf{K}^{-1} \mathbf{p}'
$$

其中 $\mathbf{R}_{\text{rect}}$ 为旋转矩阵，$\mathbf{K}$ 为内参矩阵，$\mathbf{p}$、$\mathbf{p}'$ 分别为校正前后的像素坐标。

## 二、立体匹配的核心算法体系

### 1、局部匹配方法

**代价函数构建**：

绝对差和（SAD）：

$$
C(x,y,d) = \sum_{(i,j) \in \mathcal{N}} | I_L(x+i, y+j) - I_R(x+d+i, y+j) |
$$

归一化互相关（NCC）：

$$
C(x,y,d) = \frac{ \sum (I_L - \bar{I}_L)(I_R - \bar{I}_R) }{ \sqrt{ \sum (I_L - \bar{I}_L)^2 \sum (I_R - \bar{I}_R)^2 } }
$$

**优化策略**：采用 Winner-Takes-All 策略选择最小代价视差。

### 2、全局优化方法

**能量函数定义**：

$$
E(d) = \sum_{\mathbf{p}} C(\mathbf{p}, d_\mathbf{p}) + \lambda \sum_{(\mathbf{p}, \mathbf{q}) \in \mathcal{N}} V(d_\mathbf{p}, d_\mathbf{q})
$$

其中 $V$ 为平滑项（如 Potts 模型），$\lambda$ 为平滑权重系数。

**求解算法**：

- **图割法**：通过最大流最小割理论求解
- **置信传播**：基于马尔可夫随机场迭代优化

### 3、半全局匹配（SGM）

沿 8/16 个路径方向聚合代价，构建聚合代价体 $\mathbf{S}$：

$$
\mathbf{S}(\mathbf{p}, d) = C(\mathbf{p}, d) + \sum_{\mathbf{r}} \min_{d'} \left( \mathbf{S}(\mathbf{p} - \mathbf{r}, d') + V(d, d') \right)
$$

参数 $P_1$、$P_2$ 控制视差连续性约束强度（$P_1 < P_2$）。

## 三、深度学习驱动的算法革新

### 1、端到端立体匹配网络

**特征提取**：

采用 Siamese 架构的 ResNet 或 VGG 网络提取多尺度特征，生成左右视图的特征图 $\mathbf{F}_L$、$\mathbf{F}_R$。

**代价体构建**：

在 4D 空间（$H \times W \times D \times C$）构建特征差异体：

$$
\mathbf{C}(x,y,d,c) = \mathbf{F}_L(x,y,c) - \mathbf{F}_R(x-d,y,c)
$$

或采用连接/点积方式进行特征融合。

**三维卷积聚合**：

通过 3D CNN 优化代价体，输出概率体 $\mathbf{P}(x,y,d)$，经 softmax 归一化后得到视差概率分布。

### 2、代表性网络架构

- **GC-Net**：引入可微 ArgMin 层实现亚像素精度
- **PSMNet**：金字塔池化模块增强上下文感知
- **RAFT-Stereo**：基于循环迭代的残差优化

## 四、技术挑战与应对策略

### 1、遮挡区域处理

**左右一致性检验**：

剔除无效视差：

$$
| d_L(x,y) - d_R(x - d_L(x,y), y) | \leq 1
$$

**深度学习填补**：采用 U-Net 结构进行空洞修复

### 2、弱纹理区域匹配

- **结构光辅助**：投射伪随机散斑（Kinect v1 方案）
- **多模态融合**：结合 ToF 或 LiDAR 稀疏深度先验

### 3、实时性优化

- **FPGA 加速**：基于流水线架构实现 Census 变换硬件加速
- **边缘计算**：部署轻量级网络（如 MobileStereoNet）

## 五、方法对比

| 方法类别 | 典型算法 | 精度 | 效率 | 适用场景 |
|----------|----------|------|------|----------|
| 局部匹配 | SAD、NCC | 中 | 高 | 纹理丰富场景 |
| 全局优化 | 图割、BP | 高 | 低 | 高精度需求 |
| 半全局 | SGM | 较高 | 中 | 工业/车载应用 |
| 深度学习 | PSMNet、RAFT | 高 | 中 | 复杂通用场景 |

## 六、前沿研究方向

1. **无监督立体匹配**：通过左右视图重构损失实现自监督训练
2. **事件相机立体匹配**：基于动态视觉传感器的异步匹配
3. **跨域泛化**：提升深度学习模型在不同数据集间的泛化能力
4. **轻量化网络设计**：面向移动端和嵌入式平台的实时立体匹配
