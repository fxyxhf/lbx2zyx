---
title: 视频目标检测技术
date: 2025-12-31
publish_display_date: 2025-12-31
excerpt: ""
categories: [Computer Vision, Object Detection]
tags: [视频目标检测, 时序特征聚合, 光流, 卡尔曼滤波, SELSA]
layout: single
author_profile: true
---

## 一、核心任务定义

视频目标检测在连续视频帧中检测并跟踪目标，关键创新在于利用时序相关性提升性能。不同于图像检测的独立处理，视频目标检测考虑帧间的运动连续性，通过跨帧信息传递增强检测的准确性与稳定性。

## 二、核心技术原理

### 1、时序特征传播模型

对于第 $t$ 帧的特征图 $F_t$，通过运动估计传播到 $t+\Delta t$ 帧：

$$
F_{t+\Delta t}(\mathbf{p}) = \text{Warp}(F_t, \mathbf{v}_{t \to t+\Delta t}(\mathbf{p}))
$$

其中 $\mathbf{v}_{t \to t+\Delta t}$ 为光流向量，$\mathbf{p}$ 为像素位置，$\text{Warp}$ 表示基于光流的特征扭曲操作。

### 2、时序特征聚合（FGFA方法）

FGFA的加权聚合公式为：

$$
F_{\text{agg}}(\mathbf{p}) = \sum_{i=-k}^{k} w_i(\mathbf{p}) \cdot \text{Warp}(F_{t+i}, \mathbf{v}_{t+i \to t})
$$

权重 $w_i$ 通过可学习网络计算，反映相邻帧的质量和相关性，实现对特征的自适应加权融合。

### 3、卡尔曼滤波在跟踪-检测中的应用

对于目标状态向量 $\mathbf{x} = [x, y, \dot{x}, \dot{y}]^T$：

**预测步骤**：

$$
\hat{\mathbf{x}}_{k|k-1} = \mathbf{F} \cdot \hat{\mathbf{x}}_{k-1|k-1}
$$

$$
\mathbf{P}_{k|k-1} = \mathbf{F} \cdot \mathbf{P}_{k-1|k-1} \cdot \mathbf{F}^T + \mathbf{Q}
$$

**更新步骤**：

$$
\mathbf{K}_k = \mathbf{P}_{k|k-1} \cdot \mathbf{H}^T \cdot (\mathbf{H} \cdot \mathbf{P}_{k|k-1} \cdot \mathbf{H}^T + \mathbf{R})^{-1}
$$

$$
\hat{\mathbf{x}}_{k|k} = \hat{\mathbf{x}}_{k|k-1} + \mathbf{K}_k \cdot (\mathbf{z}_k - \mathbf{H} \cdot \hat{\mathbf{x}}_{k|k-1})
$$

其中 $\mathbf{z}_k$ 为当前帧检测结果，$\mathbf{F}$ 为状态转移矩阵，$\mathbf{Q}$ 为过程噪声协方差，$\mathbf{H}$ 为观测矩阵，$\mathbf{R}$ 为观测噪声协方差。

## 三、关键网络架构示例：SELSA

SELSA的自注意力机制定义如下：

设视频序列特征为 $\{\mathbf{f}_1, \mathbf{f}_2, \cdots, \mathbf{f}_T\}$，查询为当前帧特征 $\mathbf{q}$：

$$
\mathbf{q} = \mathbf{W}_q \cdot \mathbf{f}_t
$$

$$
\mathbf{k}_i = \mathbf{W}_k \cdot \mathbf{f}_i, \quad \mathbf{v}_i = \mathbf{W}_v \cdot \mathbf{f}_i
$$

$$
\text{Attention}(\mathbf{q}, \{\mathbf{k}_i\}, \{\mathbf{v}_i\}) = \sum_{i=1}^{T} \text{softmax} \left( \frac{\mathbf{q} \cdot \mathbf{k}_i}{\sqrt{d}} \right) \cdot \mathbf{v}_i
$$

这种跨帧的语义聚合显著提升了对遮挡和模糊的鲁棒性，使模型能够在目标被遮挡或运动模糊时仍保持稳定的检测性能。

## 四、训练技巧与损失函数

一致性损失确保相邻帧检测稳定：

$$
\mathcal{L}_{\text{cons}} = \sum_{t} \| \text{Det}(\mathbf{f}_t) - \text{Warp}(\text{Det}(\mathbf{f}_{t+1}), \mathbf{v}) \|_2^2
$$

总损失函数为：

$$
\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{cls}} + \lambda_1 \mathcal{L}_{\text{reg}} + \lambda_2 \mathcal{L}_{\text{cons}}
$$

其中 $\mathcal{L}_{\text{cls}}$ 为分类损失，$\mathcal{L}_{\text{reg}}$ 为回归损失，$\lambda_1$、$\lambda_2$ 为权重系数。

## 五、方法对比

| 方法 | 核心机制 | 优点 | 局限 |
|------|----------|------|------|
| 单帧检测 | 逐帧独立检测 | 实现简单 | 忽略时序信息 |
| FGFA | 光流引导特征聚合 | 性能提升显著 | 光流估计误差累积 |
| SELSA | 序列级语义聚合 | 遮挡鲁棒性强 | 计算开销大 |
| 卡尔曼滤波 | 线性状态估计 | 实时性好 | 非线性场景受限 |

## 六、前沿研究方向

1. **端到端视频目标检测**：将检测与跟踪统一为联合优化框架
2. **轻量化时序建模**：面向移动端和边缘设备的实时VOD模型
3. **长时序依赖建模**：基于Transformer的长程上下文建模
4. **自监督学习**：利用视频本身的时间一致性进行自监督预训练
