---
title: 医学目标检测技术
date: 2026-02-03
publish_display_date: 2026-02-03
excerpt: ""
categories: [Medical Imaging, Computer Vision]
tags: [医学目标检测, RetinaNet, CenterNet, 细胞检测, Focal Loss]
layout: single
author_profile: true
---

## 一、医学检测的特殊性

医学目标检测的特点：目标尺寸差异大（从几毫米的微钙化到几十厘米的器官）、对比度低、形态不规则。

**医学锚框设计**

解剖学先验锚框：根据目标解剖尺寸设计锚框，例如肺结节检测中锚框尺寸设为 5-30mm，肝脏检测中设为 50-200mm，通过统计训练集中目标尺寸分布确定锚框尺度。

## 二、先进检测架构

### 1、RetinaNet医学改进

**Focal Loss改进**：

$$
\text{FL}(p_t) = -\alpha_t (1 - p_t)^\gamma \log(p_t)
$$

医学场景调优：$\alpha = 0.75$，$\gamma = 2$

**特征金字塔优化**：

医学图像需要更深的特征融合，通过增加自顶向下的路径层数，强化小目标特征的传递。

### 2、CenterNet用于医学检测

**热图回归**：

对于每个目标，预测中心点热图：

$$
\mathbf{Y}_{xyc} = \exp\left( -\frac{(x - \tilde{x})^2 + (y - \tilde{y})^2}{2\sigma^2} \right)
$$

其中 $\sigma$ 与目标尺寸相关，控制热图高斯核的扩散范围。

**3D CenterNet扩展**：

预测3D热图：$\mathbf{Y} \in \mathbb{R}^{H \times W \times D \times C}$

## 三、特定应用：细胞检测

### 1、密集细胞检测

**Adaptive NMS**：

针对密集细胞调整 NMS 阈值：

$$
\tau_{\text{adaptive}} = \tau_{\text{base}} + \delta \cdot (1 - \frac{d_i}{\bar{d}})
$$

其中 $d_i$ 为第 $i$ 个检测与最近邻的距离，$\bar{d}$ 为平均距离，在细胞密集区域自适应降低阈值以保留更多检测。

### 2、细胞分类与计数

**检测+分类联合**：

网络输出：$(\text{位置}, \text{大小}, \text{类别概率})$

**细胞计数回归**：

$$
\mathcal{L}_{\text{count}} = \| \hat{N} - N_{\text{gt}} \|_2^2
```

## 总结与展望

医学目标检测在临床辅助诊断中具有重要价值，其核心挑战在于目标的低对比度、大尺度变化及密集分布。基于解剖先验的锚框设计、改进的Focal Loss和CenterNet等关键方法显著提升了检测精度，而针对细胞等密集目标的Adaptive NMS进一步优化了检测召回率。未来趋势将朝向3D医学检测、多器官联合检测及大模型驱动的少样本医学检测等方向发展。
