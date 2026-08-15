---
title: 点云降采样方法
date: 2025-04-19
publish_display_date: 2025-04-19
excerpt: ""
categories: [3D Vision, Point Cloud Processing]
tags: [点云, 降采样, 体素滤波, 特征保持, 三维数据处理]
layout: single
author_profile: true
---

点云稀疏化处理是三维数据处理的关键预处理环节，其核心在于保持原始几何特征的同时降低数据冗余度。本文系统阐述四种典型下采样方法的数学原理及技术特性。

## 一、均匀降采样（Uniform Sampling）

基于空间均匀性准则的系统采样策略，通过设定采样间隔阈值 $$d$$，在欧氏空间内建立规则采样网格。

该方法可保证点云分布均匀性，但会导致高曲率区域特征丢失率增加。MATLAB代码如下：

```matlab
indices = 1:step:ptCloud.Count;
ptUniform = select(ptCloud, indices);
```

## 二、随机降采样（Random Sampling）

采用蒙特卡洛方法进行概率稀疏化，其中 $$r$$ 为保留概率系数。虽然时间复杂度仅为 $$O(n)$$，但其采样结果具有随机性。MATLAB代码如下：

```matlab
ptRandom = pcdownsample(ptCloud, 'random', ratio);
```

## 三、体素降采样（Voxel Grid Sampling）

基于空间离散化理论构建体素化表征，每个体素单元 $$V_{ijk}$$ 保留质心点：

$$
\mathbf{p}_{ijk} = \frac{1}{|V_{ijk}|} \sum_{\mathbf{p} \in V_{ijk}} \mathbf{p}
$$

MATLAB代码如下：

```matlab
ptVoxel = pcdownsample(ptCloud, 'gridAverage', voxelSize);
```

## 四、特征保持降采样（Feature-aware Sampling）

基于微分几何的曲率加权采样模型，通过构建概率质量函数：

$$
P(\mathbf{p}_i) = \frac{|\mathcal{H}(\mathbf{p}_i)|^\alpha}{\sum_{j=1}^{N} |\mathcal{H}(\mathbf{p}_j)|^\alpha}
$$

其中 $$\mathcal{H}$$ 为点云Hessian矩阵，$$\alpha \in [1,3]$$ 可控制特征保留强度。

MATLAB代码如下：

```matlab
% 计算曲率（基于局部PCA）
[~, ~, curvatures] = pca(ptCloud.Location);
lambda3 = curvatures(:,3);       % 最小特征值
curvature = lambda3 ./ sum(curvatures,2);  % 标准化曲率

% 概率抽样
prob = curvature / sum(curvature);
indices = randsample(ptCloud.Count, round(ptCloud.Count*ratio), true, prob);   
ptCurvature = select(ptCloud, indices);
```

## 五、技术对比与选型建议

**计算效率排序（由高到低）：**
随机采样 > 体素采样 > 均匀采样 > 特征采样

**特征保持能力排序（由高到低）：**
特征采样 > 体素采样 > 均匀采样 > 随机采样

**工业推荐：**
- 体素采样在逆向工程中应用最广，平衡了效率与精度
- 特征采样多用于精密检测场景，对细节保留要求较高
- 随机采样适用于对精度要求不高的快速预览场景
- 均匀采样适用于点云分布均匀、曲率变化平缓的简单场景
