---
title: 基于x86微处理器PWM控制的直流电机微机接口设计
date: 2024-02-08
publish_display_date: 2024-02-08
excerpt: ""
categories: [Microcomputer, Control System]
tags: [x86, PWM, 直流电机, PID控制, 微机接口]
layout: single
author_profile: true
---

## 一、实验目的
1. 了解直流电机闭环调速的方法；
2. 掌握PID控制规律及算法；
3. 了解计算机在控制系统中的应用。

## 二、实验内容

<img width="711" height="508" alt="image" src="https://github.com/user-attachments/assets/7778944b-0cbd-47ff-8b10-8d5d489c47ac" />

1. 连接实验线路（直流电机驱动板与x86微机接口板）；

<img width="777" height="212" alt="image" src="https://github.com/user-attachments/assets/c65e9792-b59e-4113-ac8f-feb4a06e7caf" />

<img width="505" height="321" alt="image" src="https://github.com/user-attachments/assets/4efa7bc1-53a1-408c-9da7-1c715d614628" />

<img width="824" height="540" alt="image" src="https://github.com/user-attachments/assets/ae859d36-b931-4bc0-9f95-dd857654cdd5" />

2. 参考流程图，编写实验程序，检查无误后编译、链接并装入系统；
3. 启动86专用图形界面，运行程序，观察电机转速及示波器上给定值与反馈值的波形；
4. 暂停程序运行，根据实验波形分析直流电机的响应特性；

<img width="703" height="629" alt="image" src="https://github.com/user-attachments/assets/c0309afa-ebdb-456a-81fa-3f428f25f4d4" />

5. 改变PID参数（IBAND、KPP、KII、KDD）的值，观察其响应特性，选择一组较好的控制参数。

## 三、实验结果

### 3.1 不同PID参数下的响应特性

以下为不同参数组合下电机转速的响应特性对比：

<img width="1015" height="487" alt="image" src="https://github.com/user-attachments/assets/1b126915-8bb6-4840-80c7-05379060a8bf" />


### 3.2 波形分析

<img width="978" height="397" alt="image" src="https://github.com/user-attachments/assets/74b9f832-de50-4293-8191-8a5a1e6dd3f4" />

**例程波形**：电机启动时转速快速上升，出现约25%的超调，随后经过约1.5s的调节过程趋于稳定。稳态时存在小幅波动，但整体满足基本控制要求。

<img width="1006" height="365" alt="image" src="https://github.com/user-attachments/assets/75ef19b5-d2f2-4907-bf1e-713e38b46029" />

**去掉IBAND后波形**：由于积分分离阈值取消，积分项在全范围内作用，导致启动阶段积分饱和，超调增大至约35%，稳定时间延长至约2.0s，响应性能明显下降。

<img width="962" height="372" alt="image" src="https://github.com/user-attachments/assets/8ef5c55a-1aab-458d-b584-da2d23ab33ab" />

**自测较好参数波形（IBAND=0060H，KPP=1150H，KII=002FH，KDD=0030H）**：
- 超调量约12%，显著降低
- 稳定时间约0.8s，响应迅速
- 稳态波动小，接近预设转速
- 综合性能优于例程参数

### 3.3 参数调节分析

通过研究例程程序代码和查阅相关资料，PID各参数的作用总结如下：

**比例系数（KPP）**：主要作用是提高系统的调节速度，使电机更快地响应并快速接近预设参数值。如果KPP设置过大，系统可能产生振荡甚至不稳定。

**积分系数（KII）**：目的是减小稳态误差，使系统在预设速度附近稳定，最终消除误差。如果KII设置过大，系统稳定性减弱，可能无法达到预设速度。积分分离阈值（IBAND）的作用是在误差较大时暂停积分作用，防止积分饱和引起的超调。

**微分系数（KDD）**：具有前瞻性的控制作用，在系统出现大偏差之前预测变化趋势，提前施加控制作用，从而提高系统的相对稳定性，改善动态性能。

**调节顺序建议**：
1. 首先调整KPP，使电机速度能够迅速接近设定值；
2. 确定适当的KPP后，再调整KII和IBAND，确保速度在设定范围内稳定；
3. 最后进行KDD的调整，以减小系统动态偏差；
4. 如发现响应不符合预期，可返回微调前一个参数，直至获得稳定且符合预期的结果。

## 四、结论

本实验基于x86微处理器，通过PWM控制实现了直流电机的PID闭环调速。实验结果表明：
- PID控制器能有效实现直流电机的闭环调速，积分分离措施对抑制超调具有显著效果；
- 经过逐项调节，最终选取参数IBAND=0060H、KPP=1150H、KII=002FH、KDD=0030H，在该参数下电机转速稳定时间约为0.8s，超调量约12%，稳态误差小，满足控制要求；
- 通过参数整定实践，加深了对PID各环节作用的理解，掌握了微机控制系统的软硬件调试方法。
