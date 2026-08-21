---
title: 医学图像分类技术
date: 2026-02-06
publish_display_date: 2026-02-06
excerpt: ""
categories: [Medical Imaging, Computer Vision]
tags: [医学图像分类, Vision Transformer, 多实例学习, 不确定性估计]
layout: single
author_profile: true
---

## 一、疾病分类任务

### 1、二分类：良恶性分类

**乳腺癌病理分类**：

- 输入：组织病理图像
- 输出：良性 / 恶性

损失函数：

$$
\mathcal{L}_{\text{BCE}} = -\frac{1}{N} \sum_{i=1}^{N} \left[ y_i \log(p_i) + (1 - y_i) \log(1 - p_i) \right]
$$

### 2、多分类：疾病分级

**糖尿病视网膜病变分级（5类）**：

| 等级 | 描述 |
|------|------|
| 0 | 无视网膜病变 |
| 1 | 轻度非增殖性 |
| 2 | 中度非增殖性 |
| 3 | 重度非增殖性 |
| 4 | 增殖性 |

**有序分类损失**：

$$
\mathcal{L}_{\text{ord}} = -\sum_{i=1}^{N} \sum_{k=1}^{K} \mathbb{I}(y_i \geq k) \log(\sigma(s_i - b_k)) + \mathbb{I}(y_i < k) \log(1 - \sigma(s_i - b_k))
$$

其中 $s_i$ 为模型输出，$b_k$ 为可学习的阈值参数，该损失利用类别的序数关系进行建模。

## 二、先进分类架构

### 1、Vision Transformer医学应用

**ViT医学变体**：

将医学图像划分为 3D patches：

$$
\mathbf{X}_{\text{patch}} = \text{PatchEmbed}(\mathbf{I}) \in \mathbb{R}^{N \times D}
$$

### 2、多实例学习分类

**注意力池化MIL**：

对于全切片图像分类：

$$
\mathbf{z} = \sum_{k=1}^{K} a_k \cdot \mathbf{f}_k
$$

其中：

$$
a_k = \frac{\exp(\mathbf{w}^T \tanh(\mathbf{V} \mathbf{f}_k))}{\sum_{j=1}^{K} \exp(\mathbf{w}^T \tanh(\mathbf{V} \mathbf{f}_j))}
$$

## 三、不确定性估计

### 1、蒙特卡洛 Dropout

在测试时启用 Dropout，进行 $T$ 次前向传播：

$$
\hat{y} = \frac{1}{T} \sum_{t=1}^{T} f_{\theta_t}(\mathbf{x})
$$

$$
\sigma^2 = \frac{1}{T} \sum_{t=1}^{T} (f_{\theta_t}(\mathbf{x}) - \hat{y})^2
$$

### 2、集成学习

多模型集成：

$$
\hat{y} = \frac{1}{M} \sum_{m=1}^{M} f_{\theta_m}(\mathbf{x})
$$

不确定性：

$$
\sigma^2 = \frac{1}{M} \sum_{m=1}^{M} (f_{\theta_m}(\mathbf{x}) - \hat{y})^2
$$

## 总结与展望

医学图像分类是计算机辅助诊断的核心任务，涵盖良恶性二分类和疾病分级等多分类场景。Vision Transformer 通过自注意力机制捕获全局上下文，MIL 有效解决了全切片图像仅具有图像级标签的弱监督问题，而 Monte Carlo Dropout 和集成学习为临床决策提供了可靠的不确定性估计。未来趋势将朝向大模型预训练、少样本学习及多模态融合等方向发展。
