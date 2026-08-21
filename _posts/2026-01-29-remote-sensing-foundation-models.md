---
title: 遥感大模型技术
date: 2026-01-29
publish_display_date: 2026-01-29
excerpt: ""
categories: [Remote Sensing, Artificial Intelligence]
tags: [遥感大模型, 预训练模型, 多模态学习, LoRA, 开放词汇检测]
layout: single
author_profile: true
---

近年来，大模型技术已从自然语言处理和通用计算机视觉领域延伸至遥感领域。遥感大模型通过在海量遥感数据上进行预训练，学习通用的遥感影像表征，进而通过微调适配各类下游任务，显著降低了遥感应用开发成本并提升了模型泛化能力。本文系统阐述遥感预训练模型、大模型微调策略、应用范式及多模态大模型。

## 一、遥感预训练模型

### 1、遥感视觉基础模型

**RemoteCLIP**：遥感域对比语言-图像预训练

- 图像编码器：Vision Transformer
- 文本编码器：BERT

对比学习损失：

$$
\mathcal{L}_{\text{CLIP}} = -\frac{1}{N} \sum_{i=1}^{N} \log \frac{\exp(\text{sim}(\mathbf{v}_i, \mathbf{t}_i) / \tau)}{\sum_{j=1}^{N} \exp(\text{sim}(\mathbf{v}_i, \mathbf{t}_j) / \tau)}
$$

其中 $\text{sim}(\mathbf{v}, \mathbf{t}) = \frac{\mathbf{v} \cdot \mathbf{t}}{\|\mathbf{v}\|_2 \cdot \|\mathbf{t}\|_2}$ 为余弦相似度，$\tau$ 为温度参数。

### 2、多模态预训练

**光谱-空间-时间联合建模**：

输入：$\mathbf{X} = [\mathbf{I}_1, \mathbf{I}_2, \cdots, \mathbf{I}_T]$

编码：$\mathbf{F} = \text{Encoder}(\mathbf{X})$

**掩码图像建模（MIM）**：

随机掩码输入 patches，预测被掩码部分：

$$
\mathcal{L}_{\text{MIM}} = \sum_{i \in \mathcal{M}} \| \mathbf{p}_i - \mathbf{x}_i \|_2^2
$$

## 二、大模型微调策略

### 1、参数高效微调

**LoRA（Low-Rank Adaptation）**：

$$
\mathbf{W}' = \mathbf{W} + \Delta \mathbf{W} = \mathbf{W} + \mathbf{B} \mathbf{A}
$$

其中 $\mathbf{A} \in \mathbb{R}^{r \times d}$，$\mathbf{B} \in \mathbb{R}^{d \times r}$，$r \ll d$。仅训练 $\mathbf{A}$、$\mathbf{B}$ 参数，冻结原始权重 $\mathbf{W}$。

**Adapter 模块**：

$$
\mathbf{F}_{\text{out}} = \mathbf{F}_{\text{in}} + \text{MLP}(\text{LN}(\mathbf{F}_{\text{in}}))
$$

### 2、提示学习

**VPT（Visual Prompt Tuning）**：

可学习的提示 tokens：$\mathbf{P} = [\mathbf{p}_1, \mathbf{p}_2, \cdots, \mathbf{p}_m]$

输入：$\mathbf{Z} = [\mathbf{P}, \mathbf{E}]$，其中 $\mathbf{E}$ 为 patch embedding

## 三、大模型应用范式

### 1、少样本学习

基于提示的少样本分类：

(1) 构建文本提示："a satellite image of [CLASS]"

(2) 计算图像特征与文本特征的相似度

(3) 最近邻分类

$$
\hat{y} = \arg\max_{c} \text{sim}(\mathbf{f}_{\text{img}}, \mathbf{f}_{\text{text}}^{(c)})
$$

### 2、开放词汇检测

**GLIP 的遥感适配**：

区域-文本对齐损失：

$$
\mathcal{L}_{\text{align}} = \sum_{i} \sum_{j} \ell_{\text{CE}}( \text{sim}(\mathbf{r}_i, \mathbf{t}_j) )
$$

## 四、多模态大模型

### 1、光学-SAR融合模型

**跨模态对比学习**：

- 正样本对：同一地区的光学和 SAR 图像
- 负样本对：不同地区的图像

损失函数：

$$
\mathcal{L}_{\text{cross}} = -\log \frac{\exp(\text{sim}(\mathbf{f}_{\text{opt}}, \mathbf{f}_{\text{SAR}}^+) / \tau)}{\sum_{k} \exp(\text{sim}(\mathbf{f}_{\text{opt}}, \mathbf{f}_{\text{SAR}}^k) / \tau)}
$$

### 2、地理知识增强

**GeoGPT**：融合地理知识的大语言模型

- 输入：图像特征 + 地理位置编码 + 时间编码
- 输出：地理描述、变化分析、灾害评估

地理位置编码（GeoHash）：

将经纬度编码为二进制字符串，实现位置信息的紧凑表示。
