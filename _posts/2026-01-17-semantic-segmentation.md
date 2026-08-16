---
title: 语义分割技术
date: 2026-01-17
publish_display_date: 2026-01-17
excerpt: ""
categories: [Computer Vision]
tags: [语义分割, FCN, U-Net, 空洞卷积, ASPP, 注意力机制]
layout: single
author_profile: true
---

## 一、全卷积网络（FCN）架构

### 1、卷积化过程

传统 CNN 的全连接层转换为 $1 \times 1$ 卷积：

假设全连接层权重 $\mathbf{W} \in \mathbb{R}^{M \times N}$，输入特征图 $\mathbf{F} \in \mathbb{R}^{H \times W \times C}$：

- **全连接**：$\mathbf{y} = \mathbf{W} \cdot \text{vec}(\mathbf{F})$
- **卷积化**：$\mathbf{W}$ 重塑为 $\mathbf{W}' \in \mathbb{R}^{1 \times 1 \times C \times M}$，输出 $\mathbf{Y} \in \mathbb{R}^{H \times W \times M}$

通过卷积化操作，FCN 能够接收任意尺寸的输入图像并输出对应尺寸的密集预测图。

### 2、转置卷积（Transposed Convolution）

输入特征图 $\mathbf{F} \in \mathbb{R}^{H_{\text{in}} \times W_{\text{in}} \times C}$，卷积核 $\mathbf{K} \in \mathbb{R}^{k \times k}$。

输出尺寸计算公式为：

$$
H_{\text{out}} = (H_{\text{in}} - 1) \times s + k - 2p
$$

$$
W_{\text{out}} = (W_{\text{in}} - 1) \times s + k - 2p
$$

其中 $s$ 为步长，$p$ 为填充。

前向传播公式为：

$$
\mathbf{Y} = \mathbf{W}^T \mathbf{Z}
$$

其中 $\mathbf{W}$ 为卷积矩阵，$\mathbf{Z}$ 为输入展平的向量。转置卷积通过可学习的参数实现特征图的空间上采样，是 FCN 实现端到端密集预测的关键操作。

## 二、U-Net系列架构演进

### 1、原始U-Net

编码器-解码器结构，跳跃连接将编码器特征与解码器特征拼接：

$$
\mathbf{F}_d^l = \text{Conv}\left( \text{Concat}\left[ \text{UpSample}(\mathbf{F}_d^{l+1}), \mathbf{F}_e^l \right] \right)
$$

**深度监督**：多尺度损失函数：

$$
\mathcal{L} = \sum_{l=1}^{L} \lambda_l \cdot \mathcal{L}_{\text{CE}}(\mathbf{P}_l, \mathbf{G})
$$

其中 $L$ 为监督层数，$\mathbf{P}_l$ 为第 $l$ 层的预测结果，$\mathbf{G}$ 为真值掩膜，$\lambda_l$ 为各层损失权重。

### 2、U-Net变体改进

**(1) Attention U-Net**

引入注意力门控机制，自动聚焦于目标区域：

$$
\alpha = \sigma\left( \mathbf{W}_1^T \mathbf{F}_e + \mathbf{W}_2^T \text{UpSample}(\mathbf{F}_d) + b \right)
$$

$$
\mathbf{F}_{\text{att}} = \alpha \odot \mathbf{F}_e
$$

其中 $\mathbf{F}_e$ 为编码器特征，$\mathbf{F}_d$ 为解码器特征，$\odot$ 为逐元素相乘。注意力门控抑制不相关区域的响应，增强目标区域的特征表达。

**(2) U²-Net**

嵌套 U 型结构，每个块内部也是 U 型结构，实现多尺度特征提取与聚合。

## 三、上下文建模技术

### 1、空洞卷积（Dilated Convolution）

**标准卷积**：

$$
\mathbf{y}[i] = \sum_{k=1}^{K} \mathbf{x}[i + k] \cdot \mathbf{w}[k]
$$

**空洞卷积**：

$$
\mathbf{y}[i] = \sum_{k=1}^{K} \mathbf{x}[i + r \cdot k] \cdot \mathbf{w}[k]
$$

其中 $r$ 为空洞率。空洞卷积在不增加参数量的前提下扩大感受野。

**感受野递推公式**：

$$
\text{RF}_l = \text{RF}_{l-1} + (k - 1) \times \prod_{i=1}^{l-1} s_i \times r_l
$$

其中 $\text{RF}_l$ 为第 $l$ 层的感受野，$s_i$ 为第 $i$ 层的步长，$r_l$ 为第 $l$ 层的空洞率。

### 2、ASPP（Atrous Spatial Pyramid Pooling）

并行多个不同空洞率的卷积以捕获多尺度上下文信息：

$$
\mathbf{F}_{\text{out}} = \text{Concat}\left[ \text{Conv}_{r=1}(\mathbf{F}), \text{Conv}_{r=6}(\mathbf{F}), \text{Conv}_{r=12}(\mathbf{F}), \text{Conv}_{r=18}(\mathbf{F}), \text{GAP}(\mathbf{F}) \right]
$$

其中 $r$ 为空洞率，$\text{GAP}$ 为全局平均池化。ASPP 通过并行多分支结构，在不增加计算量的情况下有效捕获不同尺度目标的上下文信息。

### 3、自注意力机制（Non-local）

自注意力机制实现全局上下文建模：

$$
\mathbf{y}_i = \frac{1}{\mathcal{C}(\mathbf{x})} \sum_{\forall j} f(\mathbf{x}_i, \mathbf{x}_j) \cdot g(\mathbf{x}_j)
$$

**常用实现**（多头注意力）：

$$
\mathbf{Y} = \text{softmax}\left( \frac{\mathbf{Q} \mathbf{K}^T}{\sqrt{d_k}} \right) \mathbf{V}
$$

其中 $\mathbf{Q}, \mathbf{K}, \mathbf{V}$ 由输入特征图经线性变换得到：

$$
\mathbf{Q} = \mathbf{X} \mathbf{W}_Q, \quad \mathbf{K} = \mathbf{X} \mathbf{W}_K, \quad \mathbf{V} = \mathbf{X} \mathbf{W}_V
$$

自注意力机制能够建立任意像素对之间的长程依赖关系，有效捕获全局上下文信息。

### 4、损失函数设计

**(1) 类别不平衡处理**

**Focal Loss**：

$$
\text{FL}(p_t) = -\alpha_t (1 - p_t)^\gamma \log(p_t)
$$

其中：

$$
p_t = \begin{cases} p & \text{if } y = 1 \\ 1 - p & \text{otherwise} \end{cases}
$$

$p$ 为模型预测概率，$\gamma$ 为聚焦参数（通常取 2）。Focal Loss 通过降低易分类样本的损失权重，迫使模型关注难分类样本。

**Tversky Loss**：

$$
\text{TL} = 1 - \frac{\text{TP}}{\text{TP} + \alpha \cdot \text{FN} + \beta \cdot \text{FP}}
$$

通常设 $\alpha = 0.7$，$\beta = 0.3$，强调召回率（FN 惩罚更大），适用于类别不平衡的分割任务。

**(2) 边界感知损失**

**Boundary Loss**：

$$
\mathcal{L}_{\text{boundary}} = \sum_{\mathbf{p} \in \Omega} \phi_G(\mathbf{p}) \cdot S_\theta(\mathbf{p})
$$

其中 $\phi_G$ 为边界距离变换图，$S_\theta$ 为预测的概率图。该损失函数在边界区域施加更高权重，引导网络生成更加精确的目标边界。
