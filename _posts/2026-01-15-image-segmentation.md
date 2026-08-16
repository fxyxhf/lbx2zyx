---
title: 图像分割技术
date: 2026-01-15
publish_display_date: 2026-01-15
excerpt: ""
categories: [Computer Vision]
tags: [图像分割, Graph Cut, 水平集, 分水岭, 评估指标]
layout: single
author_profile: true
---

## 一、分割任务的数学定义

图像分割任务可形式化为将图像 $I$ 划分为 $K$ 个互不重叠的区域 $\{R_1, R_2, \cdots, R_K\}$：

$$
\bigcup_{i=1}^{K} R_i = I, \quad R_i \cap R_j = \emptyset \quad (i \neq j)
$$

每个区域满足同质性准则：对于任意像素 $\mathbf{p} \in R_i$，有 $F(\mathbf{p}) = \text{true}$，其中 $F$ 为特征函数，用于判断像素是否属于同一区域。

## 二、经典分割算法

### 1、基于图论的Graph Cut

将图像建模为图 $\mathcal{G} = (\mathcal{V}, \mathcal{E})$，其中：

- $\mathcal{V}$ 为像素节点，包含两个特殊终端节点：源点 $s$（前景）和汇点 $t$（背景）
- $\mathcal{E}$ 为相邻像素间的边，权重 $w_{ij}$ 表示像素 $i$ 与 $j$ 之间的相似度

**能量函数最小化**：

$$
E(\mathbf{f}) = \sum_{i \in \mathcal{V}} D_i(f_i) + \lambda \sum_{(i,j) \in \mathcal{E}} V_{ij}(f_i, f_j)
$$

其中：

- $D_i(f_i)$：数据项，衡量像素 $i$ 分配标签 $f_i$ 的代价
- $V_{ij}(f_i, f_j)$：平滑项，鼓励相邻像素标签一致
- $\lambda$：平衡参数

**最小割/最大流求解**：使用 Ford-Fulkerson 算法或 Boykov-Kolmogorov 算法在多项式时间内求解最优分割。

### 2、水平集方法

用隐函数 $\phi(x,y,t)$ 表示演化曲线，零水平集对应轮廓：

$$
\Gamma(t) = \{ (x,y) \mid \phi(x,y,t) = 0 \}
$$

**演化方程**：

$$
\frac{\partial \phi}{\partial t} = -F \cdot |\nabla \phi|
$$

其中 $F$ 为速度函数，包含多项驱动项：

- **曲率项**：$\alpha \cdot \kappa \cdot |\nabla \phi|$，保持轮廓平滑
- **图像梯度项**：$\beta \cdot g(I) \cdot |\nabla \phi|$，吸引轮廓到边缘
- **区域项**：$\gamma \cdot (I - c_1)^2 - (I - c_2)^2$，基于区域统计信息驱动演化

水平集方法天然支持拓扑变化，无需显式处理轮廓分裂与合并。

### 3、分水岭算法

将图像梯度视为地形图，从局部极小值开始注水：

$$
\text{Label}(\mathbf{p}) = \begin{cases} \text{WS} & \text{如果像素在分水岭上} \\ k & \text{如果像素属于第 } k \text{ 个区域} \end{cases}
$$

**距离变换分水岭**：对二值图像，计算距离变换：

$$
D(\mathbf{p}) = \min_{\mathbf{q} \in \mathbf{B}} \| \mathbf{p} - \mathbf{q} \|_2
$$

其中 $\mathbf{B}$ 为背景像素集。距离变换将二值图像转换为距离图，然后对距离图应用分水岭算法，有效避免过分割问题。

## 三、评估指标体系

### 1、区域相似度度量

**Jaccard系数（IoU）**：

$$
\text{IoU} = \frac{|\mathbf{P} \cap \mathbf{G}|}{|\mathbf{P} \cup \mathbf{G}|} = \frac{\text{TP}}{\text{TP} + \text{FP} + \text{FN}}
$$

**Dice系数**：

$$
\text{Dice} = \frac{2|\mathbf{P} \cap \mathbf{G}|}{|\mathbf{P}| + |\mathbf{G}|} = \frac{2 \cdot \text{TP}}{2 \cdot \text{TP} + \text{FP} + \text{FN}}
$$

IoU 和 Dice 系数取值范围均为 $[0, 1]$，值越大表示分割结果与真值重叠度越高。Dice 系数对正类预测更为敏感，在类别不平衡时比 IoU 更稳定。

### 2、边界准确度度量

**Hausdorff距离**：

$$
d_H(\mathbf{P}, \mathbf{G}) = \max \left\{ \sup_{\mathbf{p} \in \mathbf{P}} \inf_{\mathbf{g} \in \mathbf{G}} \| \mathbf{p} - \mathbf{g} \|_2, \; \sup_{\mathbf{g} \in \mathbf{G}} \inf_{\mathbf{p} \in \mathbf{P}} \| \mathbf{g} - \mathbf{p} \|_2 \right\}
$$

Hausdorff 距离衡量两个边界集合之间的最大不匹配程度，对极端离群点敏感，适合评估分割边界的最大误差。

**平均对称表面距离（ASSD）**：

$$
d_{\text{ASSD}}(\mathbf{P}, \mathbf{G}) = \frac{1}{|\mathbf{P}| + |\mathbf{G}|} \left( \sum_{\mathbf{p} \in \mathbf{P}} \min_{\mathbf{g} \in \mathbf{G}} \| \mathbf{p} - \mathbf{g} \|_2 + \sum_{\mathbf{g} \in \mathbf{G}} \min_{\mathbf{p} \in \mathbf{P}} \| \mathbf{g} - \mathbf{p} \|_2 \right)
$$

ASSD 衡量两个边界集合的平均距离，相比 Hausdorff 距离对噪声更鲁棒，能更好地反映分割边界的整体贴合程度。
