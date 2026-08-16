---
title: 遥感目标检测技术
date: 2026-01-01
publish_display_date: 2026-01-01
excerpt: ""
categories: [Computer Vision, Remote Sensing]
tags: [遥感目标检测, 旋转目标检测, R³Det, RoI Transformer, 特征金字塔]
layout: single
author_profile: true
---

## 一、独特挑战的数学描述

### 1、尺度分布建模

遥感图像中目标尺度遵循长尾分布：

$$
P(s) \propto \frac{1}{s^\alpha}
$$

其中 $s$ 为目标像素面积，$\alpha$ 为分布参数。这要求检测器在多个尺度上都具有良好的检测性能，以应对从微小目标到大型目标的尺度变化。

### 2、旋转表示方法

旋转框的两种表示法：

**五参数表示法**：

$$
\mathbf{b} = (x, y, w, h, \theta)
$$

其中 $(x, y)$ 为中心点坐标，$w$ 为宽度，$h$ 为高度，$\theta$ 为旋转角度。

**长边定义法**：

将旋转角 $\theta$ 定义为长边与水平轴的夹角，取值范围通常为 $[-90^\circ, 90^\circ)$ 或 $[-180^\circ, 180^\circ)$。

旋转 IoU 计算复杂，常用近似方法：

$$
\text{RotatedIoU} \approx \frac{\text{Area}(\text{Polygon}_A \cap \text{Polygon}_B)}{\text{Area}(\text{Polygon}_A \cup \text{Polygon}_B)}
$$

实际计算通过多边形相交面积实现，需计算两个旋转矩形之间的多边形交集，涉及线段交点计算、多边形裁剪等几何操作，计算复杂度远高于水平框 IoU。

## 二、关键技术详解

### 1、旋转RPN（RRPN）

对于每个锚点，生成多个旋转角度的锚框：

$$
\mathcal{A} = \{ (x_a, y_a, w_a, h_a, \theta_a) \mid \theta_a \in \Theta \}
$$

其中 $\Theta$ 为预定义的旋转角度集合（如 6 个或 12 个角度），通过在每个空间位置部署多角度锚框，实现旋转目标的全方位覆盖。

### 2、旋转RoI对齐（RoI Transformer）

将水平 RoI 转换为旋转 RoI 的学习过程：

**(1) 水平RoI特征提取**：

$$
\mathbf{F}_{\text{h}} = \text{RoIPool}(\mathbf{F}, \mathbf{b}_{\text{h}})
$$

**(2) 通过回归头预测旋转参数**：

$$
\Delta \mathbf{b} = \text{RegHead}(\mathbf{F}_{\text{h}})
$$

$$
\mathbf{b}_{\text{r}} = \mathbf{b}_{\text{h}} + \Delta \mathbf{b}
$$

**(3) 旋转RoI特征提取**：

$$
\mathbf{F}_{\text{r}} = \text{RoIAlign}(\mathbf{F}, \mathbf{b}_{\text{r}})
$$

RoI Transformer 通过可学习的变换将水平 RoI 自适应地转换为旋转 RoI，使后续的分类和回归头能够处理对齐后的旋转特征。

## 三、先进方法解析：R³Det

R³Det（Refined Rotation Detection）采用三阶段精炼策略：

**(1) 初步检测**：生成粗旋转框

**(2) 特征精炼**：对齐特征到精炼后的旋转框

**(3) 迭代优化**：多次精炼提升精度

**特征精炼模块的数学描述**：

设初始旋转框 $\mathbf{b}^{(0)}$，特征图 $\mathbf{F}$，第 $t$ 次精炼：

$$
\mathbf{b}^{(t)} = \mathbf{b}^{(t-1)} + \Delta \mathbf{b}^{(t)}
$$

其中 $\Delta \mathbf{b}^{(t)}$ 由精炼模块预测：

$$
\Delta \mathbf{b}^{(t)} = \text{RefineHead}(\text{RoIAlign}(\mathbf{F}, \mathbf{b}^{(t-1)}))
$$

通常 2-3 次迭代即可收敛，每次迭代都通过特征对齐和边界框回归逐步逼近真实目标边界。

## 四、多尺度特征金字塔优化

针对遥感图像的 ASPP 变体：

$$
\mathbf{F}_{\text{ASPP}} = \text{Concat} \left( \text{Conv}_{d=1}(\mathbf{F}), \text{Conv}_{d=6}(\mathbf{F}), \text{Conv}_{d=12}(\mathbf{F}), \text{Conv}_{d=18}(\mathbf{F}), \text{GAP}(\mathbf{F}) \right)
$$

其中 $d$ 为空洞率，$\text{GAP}$ 为全局平均池化。通过不同空洞率的并行卷积，ASPP 能够在不增加参数量的情况下有效扩大感受野，捕捉遥感图像中多尺度目标的上下文信息。

在实际遥感场景中，该结构常与特征金字塔网络（FPN）结合使用，形成多层级、多感受野的复合特征提取架构，以应对遥感图像中尺度差异大、背景复杂等挑战。
