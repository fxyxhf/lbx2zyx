---
title: 基于Simulink的一阶/二阶测量系统时频域特性分析
date: 2023-08-26
publish_display_date: 2023-08-26
excerpt: ""
categories: [Measurement, Control System]
tags: [Simulink, 一阶系统, 二阶系统, 零极点, 上升时间, 带宽]
layout: single
author_profile: true
---

## 一、实验要求
在实际测量过程中，测量系统特性影响测量结果，因为测量结果等于被测量物理量的真值与测量系统单位冲激响应（特别是前端传感器特性）的卷积，测试测量系统的时域、频域特性对精准测量具有重要意义。

具体要求：
1. 利用 Simulink 仿真设计搭建一阶或二阶测量系统（测量系统通过查找资料自行确定）。
2. 设计测试系统方案，至少采用 2 种方法，测试测量系统的时域特性和频率特性。
3. 研究系统零极点参数对测量系统的上升时间以及系统带宽的影响。

## 二、技术方案
### 2.1 系统模型
本实验选取典型的一阶和二阶测量系统（传感器模型）进行仿真分析：

**一阶系统传递函数**：
- 低通：H(s) = 1 / (τ·s + 1)
- 高通：H(s) = τ·s / (τ·s + 1)

**二阶系统传递函数**：
- 低通：H(s) = ωn² / (s² + 2ξ·ωn·s + ωn²)
- 高通：H(s) = s² / (s² + 2ξ·ωn·s + ωn²)
- 带通：H(s) = 2ξ·ωn·s / (s² + 2ξ·ωn·s + ωn²)
- 带阻：H(s) = (s² + ωn²) / (s² + 2ξ·ωn·s + ωn²)

其中 τ 为时间常数，ξ 为阻尼比，ωn 为自然频率。

### 2.2 测试方法
采用以下两种方法测试系统特性：

**方法一：时域测试（冲激/阶跃响应法）**
- 输入信号：单位冲激信号或单位阶跃信号
- 观测输出响应，测量上升时间（10%~90%）、调节时间、超调量等时域指标
- 通过冲激响应的傅里叶变换获取系统频率特性

**方法二：频域测试（扫频/相关法）**
- 输入信号：不同频率的正弦扫频信号或白噪声
- 测量输出信号幅值衰减和相位变化，绘制波特图
- 提取系统带宽（-3dB 频率点）、谐振峰值等频域指标

### 2.3 零极点影响分析
通过改变系统零极点参数（τ、ξ、ωn），观察系统上升时间和带宽的变化规律，总结零极点分布对系统动态性能的影响。

## 三、实验程序
本实验基于 Simulink 进行仿真搭建，以下给出一个典型系统的测试代码框架（其他情况可自行修改）：
<img width="692" height="152" alt="image" src="https://github.com/user-attachments/assets/4b8fdc9e-0fcf-40dd-aaa1-c63a5741c865" />


## 四、实验结果

实验分别采用 e 指数信号和方波信号作为激励，对一阶和二阶系统进行仿真。系统零极点参数设置如下表所示：

| 系统类型 | 参数设置 |
|---------|---------|
| 一阶低通 | τ=1 |
| 一阶高通 | τ=1 |
| 二阶低通 | ξ=2, ωn=1；ξ=0.2, ωn=1 |
| 二阶高通 | ξ=0.2, ωn=1；ξ=1, ωn=1 |
| 二阶带通 | ξ=2, ωn=1 |
| 二阶带阻 | ξ=0.2, ωn=1 |

<img width="680" height="342" alt="image" src="https://github.com/user-attachments/assets/9de109e3-3244-4058-b605-c5e41ace0767" />

<img width="701" height="353" alt="image" src="https://github.com/user-attachments/assets/54ef9758-8b01-40f4-8348-713c2ba33452" />

<img width="690" height="341" alt="image" src="https://github.com/user-attachments/assets/26fbbefb-a325-4547-9230-747af76501e0" />

<img width="697" height="352" alt="image" src="https://github.com/user-attachments/assets/8df530c6-c0fe-4ce0-a1e9-33f347db8e52" />

<img width="703" height="363" alt="image" src="https://github.com/user-attachments/assets/15098735-4d84-4520-ad28-90d067b2170e" />

<img width="687" height="355" alt="image" src="https://github.com/user-attachments/assets/0e262824-bc11-4fb4-ade8-8a2351b276d0" />

<img width="705" height="355" alt="image" src="https://github.com/user-attachments/assets/f00277d8-4f5c-4f19-a9c1-bd217938c45d" />

<img width="654" height="333" alt="image" src="https://github.com/user-attachments/assets/b574f9e9-7f1a-4bc6-94c6-c850977eb2ad" />

<img width="658" height="341" alt="image" src="https://github.com/user-attachments/assets/ca0f0bf0-f2ef-460a-ad56-e772b6f1fcf3" />

<img width="653" height="340" alt="image" src="https://github.com/user-attachments/assets/56b33b69-4091-4503-8635-e72c35438ffc" />

<img width="631" height="314" alt="image" src="https://github.com/user-attachments/assets/7946c438-40bc-43de-a30a-ffb12b4e76f3" />

<img width="622" height="311" alt="image" src="https://github.com/user-attachments/assets/3375b40a-a4c4-4699-916c-a9b96480b138" />

<img width="606" height="305" alt="image" src="https://github.com/user-attachments/assets/2785a516-4e76-4afd-9e1d-8ee3ca2a4545" />

<img width="647" height="313" alt="image" src="https://github.com/user-attachments/assets/2a09d04e-ead4-4060-856a-670815200510" />

<img width="646" height="330" alt="image" src="https://github.com/user-attachments/assets/72f3f380-66f2-48f5-a589-0032d8ed43ed" />

![Uploading image.png…]()

### 4.1 时域响应（e指数信号激励）
1. **一阶低通（τ=1）**：输出跟踪输入，存在指数上升过程，上升时间约为 2.2τ。
2. **一阶高通（τ=1）**：输出为输入的微分形式，信号快速衰减，稳态为零。
3. **二阶低通（ξ=2, ωn=1）**：过阻尼系统，响应无振荡，上升时间较长。
4. **二阶低通（ξ=0.2, ωn=1）**：欠阻尼系统，响应出现振荡和超调，上升时间短。
5. **二阶高通（ξ=0.2, ωn=1）**：输出包含高频成分，稳态趋于零，存在振荡。
6. **二阶高通（ξ=1, ωn=1）**：临界阻尼，响应平稳，无振荡。
7. **二阶带通（ξ=2, ωn=1）**：仅允许特定频带通过，输出呈衰减振荡。
8. **二阶带阻（ξ=0.2, ωn=1）**：抑制特定频带，输出高频和低频成分。

### 4.2 时域响应（方波信号激励）
方波信号含有丰富的谐波分量，更能体现系统的滤波特性：
1. **一阶低通**：输出为方波的平滑版本，上升沿和下降沿变缓（积分效应）。
2. **一阶高通**：输出为方波的尖峰脉冲（微分效应）。
3. **二阶低通（ξ=2）**：输出缓慢平滑，响应较慢。
4. **二阶低通（ξ=0.2）**：输出出现振铃效应，但响应速度较快。
5. **二阶高通（ξ=0.2）**：输出为方波边沿的振荡脉冲。
6. **二阶高通（ξ=1）**：输出为平滑的尖峰，无振荡。
7. **二阶带通**：输出为方波中特定频率成分的共振响应。
8. **二阶带阻**：输出保留了方波的直流和高频分量。

### 4.3 零极点对系统性能的影响

**零点的影响**：
- 零点使系统能够更快地响应输入信号，具有更短的上升时间。
- 零点可增加系统的带宽，提升高频响应能力。
- 零点位于原点附近时，高通特性明显。

**极点的影响**：
- 极点靠近原点或数量增多时，系统上升时间变长，响应速度减慢。
- 极点的阻尼比和自然频率决定了系统的超调量和振荡特性：
  - ξ < 1（欠阻尼）：上升时间短，但有超调和振荡。
  - ξ ≥ 1（临界阻尼或过阻尼）：无超调，但上升时间较长。
- 极点限制了系统能够传递的频率范围，极点靠近原点时带宽较低。

**结论**：系统的零点和极点的数量、位置对系统的上升时间和带宽有着显著影响。设计中需要根据实际测量需求在响应速度和稳定性之间进行权衡。
