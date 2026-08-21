---
title: 医学异常检测技术
date: 2026-02-01
publish_display_date: 2026-02-01
excerpt: ""
categories: [Medical Imaging, Computer Vision]
tags: [医学异常检测, 无监督学习, 弱监督学习, 肺结节检测, AnoGAN]
layout: single
author_profile: true
---

## 一、无监督异常检测

### 1、重建误差方法

**Autoencoder异常检测**：

训练 Autoencoder 在正常数据上最小化重建误差：

$$
\mathcal{L}_{\text{AE}} = \| \mathbf{x} - \text{Decoder}(\text{Encoder}(\mathbf{x})) \|_2^2
$$

异常分数：$s = \| \mathbf{x} - \hat{\mathbf{x}} \|_2^2$

**变分自编码器（VAE）**：

引入分布约束，使隐变量服从标准正态分布：

$$
\mathcal{L}_{\text{VAE}} = \| \mathbf{x} - \hat{\mathbf{x}} \|_2^2 + \text{KL}(\mathcal{N}(\mu, \sigma^2) \| \mathcal{N}(0, 1))
$$

其中 $\text{KL}$ 散度项约束隐变量分布，鼓励学习有意义的潜在表征。

### 2、生成对抗网络（GAN）

**AnoGAN**：

训练生成器 $G$ 和判别器 $D$ 仅使用正常数据。

异常检测时，寻找最优隐变量 $\mathbf{z}^*$：

$$
\mathbf{z}^* = \arg\min_{\mathbf{z}} \| \mathbf{x} - G(\mathbf{z}) \|_2^2 + \lambda \| \mathbf{f}(\mathbf{x}) - \mathbf{f}(G(\mathbf{z})) \|_2^2
$$

其中 $\mathbf{f}$ 为判别器中间层特征。异常分数：

$$
s = \| \mathbf{x} - G(\mathbf{z}^*) \|_2^2 + \lambda \| \mathbf{f}(\mathbf{x}) - \mathbf{f}(G(\mathbf{z}^*)) \|_2^2
$$

## 二、弱监督异常检测

### 多实例学习（MIL）

图像级标签用于像素级检测：

将图像划分为多个 patch，视为包（bag）：
- 正包：至少一个 patch 包含异常
- 负包：所有 patch 正常

**注意力 MIL**：

$$
\mathbf{z} = \sum_{k=1}^{K} a_k \cdot \mathbf{f}_k
$$

其中注意力权重 $a_k$ 为：

$$
a_k = \frac{\exp(\mathbf{w}^T \tanh(\mathbf{V} \mathbf{f}_k^T))}{\sum_{j} \exp(\mathbf{w}^T \tanh(\mathbf{V} \mathbf{f}_j^T))}
$$

异常定位：注意力权重 $a_k$ 指示异常区域，权重越大的 patch 被认为是异常的可能性越高。

## 三、特定应用：肺结节检测

### 1、假阳性减少

**3D上下文感知分类**：

以候选结节为中心，提取多尺度特征：

(1) 结节 patch（16×16×16 mm³）

(2) 局部上下文（32×32×32 mm³）

(3) 全局上下文（64×64×64 mm³）

**集成分类器**：

$$
P_{\text{final}} = \text{MLP}\left( \text{Concat}[\mathbf{f}_1, \mathbf{f}_2, \mathbf{f}_3] \right)
$$

### 2、生长性分析

**时间序列特征**：

对于两次检查的同一结节：

$$
\Delta \mathbf{f} = \mathbf{f}_{\text{time2}} - \mathbf{f}_{\text{time1}}
$$

**恶性度评分**：

$$
\text{Score} = \text{Sigmoid}\left( \mathbf{w}^T [\mathbf{f}_{\text{current}}, \Delta \mathbf{f}] + b \right)
$$

## 总结与展望

医学异常检测因正常样本丰富、异常样本稀缺且类型多样，无监督和弱监督方法成为主流范式。Autoencoder 和 GAN 通过重建误差或生成对抗实现异常检测，MIL 利用图像级标签降低标注成本，而在肺结节等特定任务中，3D多尺度特征提取和时间序列分析有效提升了检测精度。未来趋势将朝向自监督预训练、跨域迁移学习及联邦学习下的隐私保护异常检测等方向演进。
