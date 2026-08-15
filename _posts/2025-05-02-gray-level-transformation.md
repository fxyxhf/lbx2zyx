---
title: 灰度变换方法
date: 2025-05-02
publish_display_date: 2025-05-02
excerpt: ""
categories: [Image Processing]
tags: [灰度变换, 图像增强, 对比度拉伸, MATLAB, 空间域处理]
layout: single
author_profile: true
---

## 一、空间域处理

### 基本表达式

图像空间域处理可表示为：

$$
g(x,y) = \mathcal{T} \left[ f(x,y) \right]
$$

其中 $f(x,y)$ 为输入图像，$g(x,y)$ 为输出图像，$\mathcal{T}$ 为在点 $(x,y)$ 的一个指定邻域上定义的、对 $f$ 进行处理的算子。

### 运算过程

从起点开始逐个像素移动，每个点作为中心构建一个正方形或矩形的邻域，$\mathcal{T}$ 算子作用在每个位置上，即可得到相应的输出 $g(x,y)$。

### 亮度变换函数

一种特殊情形：当邻域大小为 $1 \times 1$ 时，$g$ 在 $(x,y)$ 处的值仅由 $f$ 在该点的灰度值决定，此时 $\mathcal{T}$ 称为亮度变换函数或灰度变换函数，其表达式如下：

$$
s = T(r)
$$

其中 $r$ 表示图像 $f$ 在点 $(x,y)$ 的灰度，$s$ 表示图像 $g$ 在点 $(x,y)$ 的灰度。

## 二、imadjust 和 stretchlim 函数

### 1、imadjust 函数

`imadjust` 函数用于对灰度级图像进行灰度变换，其语法格式为：

```matlab
g = imadjust(f, [low_in high_in], [low_out high_out], gamma);
```

该函数将 `low_in` 和 `high_in` 之间的值映射到 `low_out` 和 `high_out` 之间，默认为 0-1。

**反转灰度**：若 `low_out` 大于 `high_out`，可实现输出灰度的反转，同样的效果还可以用 `imcomplement` 函数实现：

```matlab
% low_out大于high_out
g = imadjust(f, [0 1], [1 0]);

% imcomplement函数实现
g = imcomplement(f);
```

**gamma 参数**：对于 `gamma` 值的选取，默认为 1，即为线性映射；大于 1 时，映射被加权至较低/较暗的输出值；小于 1 时，映射被加权至较高/较亮的输出值。

**ROI 提取**：可以通过将小区域的灰度扩展到大区域的方式实现 ROI 的提取。

### 2、stretchlim 函数

使用 `stretchlim` 函数可实现自动设置 `low_in` 和 `high_in`，其语法为：

```matlab
Low_High = stretchlim(f);
```

`Low_High` 为一个双元素向量，对应 `low_in` 和 `high_in`，可实现对比度拉伸。

**与 imadjust 结合使用**：

```matlab
g = imadjust(f, stretchlim(f), [ ]);
```

**指定 tol 参数**：另一种格式为：

```matlab
Low_High = stretchlim(f, tol);
```

其中 `tol` 为一个两元素向量 `[low_frac high_frac]`，它指定将以低像素值和高像素值充满的图像部分。

- `tol` 默认值为 `[0.01 0.99]`
- 若 `tol` 为标量，则 `low_frac = tol`，`high_frac = 1 - tol`，将以低像素值和高像素值充满相等的部分
- 若 `tol = 0`，则 `Low_High = [min(f(:)) max(f(:))]`

## 三、对数及对比度拉伸变换

### 1、对数变换

对数变换表达式为：

$$
g = c \cdot \log(1 + f)
$$

其中 $c$ 为常数，$f$ 为浮点数。

**动态范围压缩**：对数变换的作用是压缩动态范围，若想压缩值出现在显示的完整范围内，以 8 比特为例，则有：

```matlab
gs = im2uint8(mat2gray(g));
```

其中 `mat2gray` 函数将值限定在区间 `[0,1]` 内，`im2uint8` 函数将值限定在区间 `[0,255]` 内，并转换为 `uint8` 形式。

### 2、对比度拉伸变换

**基本表达式**：对比度拉伸变换函数把窄范围的输入灰度扩展为宽范围的输出灰度，其表达式为：

$$
s = \frac{1}{1 + (m/r)^E}
$$

其中 $r$ 表示输入图像的灰度，$s$ 为输出图像的相应灰度，$E$ 用于控制该函数的斜率。当 $E \to \infty$ 时输出二值图像，此时受限函数称为阈值化/阈值处理函数。

**MATLAB 实现**：

```matlab
g = 1 ./ (1 + (m ./ f).^E);
```

**注意**：输出值不能超过 `[0,1]` 区间。

## 四、指定任意灰度变换

对于灰度映射，可以使用 `interp1` 函数，语法如下：

```matlab
g = interp1(z, T, f);
```

其中：
- `f` 为输入图像
- `g` 为输出图像
- `T` 为列向量，包含变换函数的值
- `z` 也为列向量，且长度与 `T` 相同

**z 的生成方式**：

```matlab
z = linspace(0, 1, numel(T))';
```

**映射过程**：对于 $f$ 中的一个像素值，`interp1` 首先寻找横坐标上的值 $z$，然后寻找（内插）$T$ 中的相应值，并将内插的值输出到 $g$ 中的相应像素位置。

## 五、技术对比

| 方法 | 表达式 | 适用场景 | 特点 |
|------|--------|----------|------|
| 线性变换 | $s = T(r)$ | 一般增强 | 简单直接 |
| imadjust | 参数化映射 | 灵活调整 | 支持gamma校正 |
| 对数变换 | $s = c \log(1+r)$ | 动态范围压缩 | 扩展暗区细节 |
| 对比度拉伸 | $s = 1/(1+(m/r)^E)$ | 对比度增强 | 支持阈值化 |
| interp1映射 | $s = \text{interp1}(z,T,r)$ | 任意映射 | 高度灵活 |

## 六、总结

灰度变换是图像增强中最基础也是最核心的空间域处理方法。通过合理选择变换函数，可以实现对比度拉伸、动态范围压缩、灰度反转等多种效果。MATLAB 提供了 `imadjust`、`stretchlim`、`interp1` 等丰富的函数支持，便于根据具体应用场景灵活实现各种灰度映射策略。
