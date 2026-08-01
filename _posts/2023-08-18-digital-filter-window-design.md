---
title: 窗函数法数字低通滤波器设计实验
date: 2023-08-18
publish_display_date: 2023-08-18
excerpt: ""
categories: [Signal Processing]
tags: [MATLAB, FIR滤波器, 窗函数, 频率响应, 线性相位]
layout: single
author_profile: true
---

## 一、实验目的
研究数字滤波器设计思想，理解数字频率与模拟频率的关系，掌握数字系统处理模拟信号的方法。掌握窗函数设计数字滤波器的方法，理解滤波器的线性相位重要意义。

## 二、实验要求
设计数字低通滤波器，截止频率 \(\omega_c = \pi/4\)，在不同窗口长度（\(N=15\)，\(N=33\)）下，分别求出系统函数 \(H(z)\)，通过幅频特性和相频特性，观察通带带宽和阻带带宽，总结窗口长度对滤波特性的影响。

## 三、技术方案
设计滤波器并分析的主要步骤：
1. 确定理想低通滤波器的单位脉冲响应 \(h_d(n)\)，其截止频率为 \(\omega_c\)，注意需将模拟频率对采样率归一化。
2. 根据阻带最小衰减选择窗函数类型（本实验采用三角窗、矩形窗、汉宁窗进行对比）。
3. 给定窗口长度 \(N\)，计算实际滤波器系数 \(h(n) = h_d(n) \cdot w(n)\)。
4. 利用 `freqz` 函数绘制滤波器的幅频特性和相频特性曲线。
5. 对比不同窗长和不同窗函数下的波特图，总结窗口长度对过渡带宽、阻带衰减等性能的影响。

本实验中，已知截止频率 \(\omega_c = \pi/4\)，窗口长度分别取 \(N=15\) 和 \(N=33\)，直接列出滤波器系数向量（通过 MATLAB 的 `fir1` 函数生成），然后绘制频率响应曲线进行比较。

## 四、实验程序
以下程序以三角窗为例，读者可自行将窗函数替换为 `rectwin`（矩形窗）或 `hann`（汉宁窗）进行对比。

```matlab
clc;
clear;
close all;

% 窗口长度 N=15
N1 = 15;
wc1 = pi/4;
hn1 = fir1(N1-1, wc1/pi, 'low', bartlett(N1));
figure(1), freqz(hn1, 1);
title('三角窗, N=15');

% 窗口长度 N=33
N2 = 33;
wc2 = pi/4;
hn2 = fir1(N2-1, wc2/pi, 'low', bartlett(N2));
figure(2), freqz(hn2, 1);
title('三角窗, N=33');

% 如需比较其他窗函数，可将 bartlett 替换为：
% rectwin(N) 或 hann(N) 等
```

## 五、实验结果
分别采用矩形窗、汉宁窗、三角窗，在窗口长度 \(N=15\) 和 \(N=33\) 下得到幅频响应和相频响应曲线（幅值以 dB 表示，相位以度表示）。主要观察结论如下：

<img width="667" height="356" alt="image" src="https://github.com/user-attachments/assets/2add9698-c331-4f4e-8453-65b0d8a5c719" />

<img width="679" height="355" alt="image" src="https://github.com/user-attachments/assets/6087f9e8-5443-42e7-b98a-e9eefd46ba58" />

<img width="715" height="377" alt="image" src="https://github.com/user-attachments/assets/03a7db65-5f4c-4cee-ac61-a53083b29f0f" />

<img width="708" height="371" alt="image" src="https://github.com/user-attachments/assets/d1eb0ff9-a411-4c48-b85a-18a4f8f69b97" />

<img width="613" height="320" alt="image" src="https://github.com/user-attachments/assets/90246eef-bfb1-46c9-8364-b48a5927a124" />

<img width="669" height="353" alt="image" src="https://github.com/user-attachments/assets/1a98fb5f-2b14-4dbe-9ab2-623a9f59c14b" />


1. **窗口长度的影响**（同一种窗函数）：
   - 当 \(N=15\) 时，过渡带较宽，阻带衰减较小，通带起伏较大。
   - 当 \(N=33\) 时，过渡带变窄，频率分辨率提高，阻带衰减增大，通带更平坦。
   - 结论：窗口长度 \(N\) 越大，滤波器过渡带越窄，频率分辨率越高，但计算量也随之增加。

2. **不同窗函数的影响**（相同 \(N\)）：
   - 矩形窗：主瓣最窄，过渡带最陡，但旁瓣峰值最高，阻带最小衰减最小（约 -13 dB），泄漏严重。
   - 汉宁窗：主瓣略宽于矩形窗，旁瓣衰减显著改善（约 -44 dB），阻带衰减较好。
   - 三角窗（巴特利特窗）：主瓣宽度与汉宁窗相近，旁瓣衰减略逊于汉宁窗（约 -25 dB）。
   - 结论：窗函数的选择需在过渡带宽度和阻带衰减之间权衡，无窗（矩形窗）虽分辨率高但阻带性能差，汉宁窗等兼顾了衰减和分辨率。

3. **线性相位特性**：
   - 所有设计的 FIR 滤波器均具有严格的线性相位（相频曲线为直线），这保证信号通过后不会产生相位失真，是 FIR 滤波器的重要优点。

综上，窗口长度和窗函数类型共同决定了数字滤波器的性能指标，实际设计中应根据系统对过渡带陡峭度和阻带衰减的具体要求来合理选择。
