---
title: 正弦幅度调制-解调系统仿真设计与频率特性分析
date: 2023-08-26
publish_display_date: 2023-08-26
excerpt: ""
categories: [Signal Processing, Communication]
tags: [MATLAB, AM调制, 解调, 低通滤波器, 频谱分析]
layout: single
author_profile: true
---

## 一、实验要求
利用乘法器与低通滤波器，设计调制解调系统。设计的仿真系统，能够对低频输入信号（如三角波信号等）与高频载波相乘进行调制，然后再与高频载波相乘，通过低通滤波器，从而实现解调，恢复原信号。

## 二、技术方案

### 2.1 调制解调原理
**幅度调制（AM）** 的基本原理是将低频基带信号 \( m(t) \) 与高频载波 \( c(t) = \cos(\omega_c t) \) 相乘，得到已调信号：
\[
s(t) = m(t) \cdot \cos(\omega_c t)
\]

**解调** 采用相干解调方式，将已调信号与同频同相的本地载波相乘：
\[
s(t) \cdot \cos(\omega_c t) = m(t) \cdot \cos^2(\omega_c t) = \frac{m(t)}{2} + \frac{m(t)}{2} \cos(2\omega_c t)
\]

上式包含基带分量 \( m(t)/2 \) 和二倍频分量 \( m(t)\cos(2\omega_c t)/2 \)，通过低通滤波器滤除高频分量后，即可恢复原信号 \( m(t)/2 \)（幅度减半，可通过放大补偿）。

### 2.2 系统参数设置
- 基带信号：三角波，频率 \( f_m = 5 \) Hz
- 载波信号：正弦波，频率 \( f_c = 200 \) Hz
- 采样频率：\( f_s = 5000 \) Hz
- 低通滤波器：FIR 低通，截止频率 \( f_{cut} = 30 \) Hz
- 滤波器设计方法：窗函数法（汉明窗）

### 2.3 实现步骤
1. 生成低频三角波信号 \( m(t) \)。
2. 生成高频载波信号 \( c(t) = \cos(2\pi f_c t) \)。
3. 调制：\( s(t) = m(t) \cdot c(t) \)。
4. 解调：\( r(t) = s(t) \cdot c(t) \)。
5. 低通滤波：使用 `filter` 函数滤除高频分量，提取基带信号。
6. 绘制各阶段时域波形及频谱，验证系统性能。

## 三、实验程序

<img width="691" height="295" alt="image" src="https://github.com/user-attachments/assets/3360f705-5554-45e5-a61b-6acd789f413d" />

## 四、实验结果

### 4.1 时域波形分析

<img width="710" height="315" alt="image" src="https://github.com/user-attachments/assets/4b89f719-0504-43ae-8ba7-55ffb4a9a02c" />

1. **原始信号**：三角波信号，频率 5Hz，时域呈对称三角形状，频谱集中在 5Hz 及其奇次谐波处。

<img width="706" height="317" alt="image" src="https://github.com/user-attachments/assets/d7ff41d8-d2b8-4da1-8f77-ac77fe7fb399" />

2. **调制后信号**：三角波包络在高频载波（200Hz）上振荡，表现为载波幅度随基带信号变化，高频成分被搬移至载波频率附近（200Hz 处）。

<img width="708" height="291" alt="image" src="https://github.com/user-attachments/assets/5a43f13a-b461-4e25-b3d1-2c4f0d5bcab2" />

3. **解调后信号（滤波前）**：包含基带分量（0~15Hz）和以 2fc=400Hz 为中心的高频分量，时域波形呈现高频振荡叠加在基带之上。

4. **恢复信号**：经过低通滤波器后，高频分量被滤除，时域波形与原始三角波基本重合，仅存在少量滤波引起的边缘失真（由于滤波器非理想特性），幅度已补偿至原始水平。

### 4.2 频谱分析

<img width="714" height="325" alt="image" src="https://github.com/user-attachments/assets/b3ebc19b-de02-422c-a175-f041f0fba357" />

1. **滤波前谱线**：频谱中存在明显的 0Hz 附近基带分量和 400Hz 附近高频分量，幅值相近，验证了理论推导 \( r(t) = m(t)/2 + m(t)\cos(2\omega_c t)/2 \)。

<img width="714" height="326" alt="image" src="https://github.com/user-attachments/assets/73940bf9-8ae5-4170-890d-81e4c4bc6c4c" />

2. **滤波后谱线**：400Hz 高频分量被有效衰减（衰减量取决于滤波器阻带衰减），基带分量完整保留，频谱与原始信号一致，证明解调成功。

### 4.3 结论

- 基于乘法器与低通滤波器的相干解调方案能够有效实现正弦幅度调制信号的解调。
- 低通滤波器的截止频率必须大于基带信号最高频率且小于 2fc，以保证有效提取基带分量并滤除高频分量。
- 滤波器阶数越高，过渡带越陡峭，滤波效果越好，但会引入较大的群延迟，实际应用中需在性能和实时性之间权衡。
- 本系统对三角波信号实现了良好的恢复，验证了幅度调制-解调系统的工作原理。
