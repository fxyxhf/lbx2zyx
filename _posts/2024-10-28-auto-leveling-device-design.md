---
title: 自动调平装置设计
date: 2024-10-28
publish_display_date: 2024-10-28
excerpt: ""
categories: [Embedded, Control System]
tags: [STM32, MPU6050, 自动调平, PID, 姿态解算]
layout: single
author_profile: true
---

## 一、方案设计

### 1.1 任务分析
本课题的设计任务是利用姿态传感器MPU6050设计并制作一套自动调平装置，由圆盘、姿态传感器、可以调节高度的三个脚组成。并对姿态传感器的数据进行解算，设计控制算法通过STM32控制电机进行平台调节。

### 1.2 原理设计

#### 1.2.1 MPU6050原理

MPU6050是整合性六轴运动处理模块，它可以实时获取运动物体在三维坐标系内的偏转角度，其中横滚角roll为绕X轴偏转的角度，俯仰角pitch为绕Y轴偏转的角度，航向角yaw为绕Z轴偏转的角度。

<img width="478" height="623" alt="image" src="https://github.com/user-attachments/assets/87add357-649b-4215-8544-64022382fe38" />

MPU6050外观如图1所示，其中较为重要的是SCL和SDA两个管脚，它们是连接MCU的I2C接口，MCU通过这个I2C接口来控制MPU6050，此时MPU6050作为一个I2C从机设备进行通信。

**MPU6050的驱动流程：**
1. 初始化I2C接口：初始化MPU6050的SDA和SCL
2. 复位MPU6050：电源管理寄存器1(0X6B)的Bit7写1，复位后设置电源管理寄存器1为0X00
3. 设置满量程范围：陀螺仪±2000dps，加速度传感器±2g
4. 设置其他参数：关闭中断和AUX I2C接口，关闭所有FIFO通道
5. 配置系统时钟源并使能传感器：选择x轴陀螺PLL作为时钟源

<img width="506" height="593" alt="image" src="https://github.com/user-attachments/assets/c87f5a31-cb92-42a6-a9df-e8229174f6a4" />

<img width="535" height="633" alt="image" src="https://github.com/user-attachments/assets/f810cfb1-b016-4d25-aa40-48a8a284eb70" />

<img width="1176" height="275" alt="image" src="https://github.com/user-attachments/assets/20075439-a7a0-4b44-a4d0-82926224753a" />

对于姿态解算问题，MPU6050自带了DMP（数字运动处理器），可以将加速度传感器和角速度传感器的原始数据直接转换成四元数输出，得到四元数之后即可计算出欧拉角。DMP输出的四元数是q30格式的，需要先除以2^30转换为浮点数，再计算欧拉角。

#### 1.2.2 调平算法

<img width="417" height="297" alt="image" src="https://github.com/user-attachments/assets/968a97e5-338f-42db-8fde-63dbb8636ba2" />

实时检测当前姿态，将俯仰角和横滚角与水平位置进行对比（精度优于1°，即在-1°到1°范围内可认为完成调平），当满足要求时声光提示，不满足时根据姿态调整电机运动实现调平。具体策略为根据俯仰角和横滚角的数值，控制电动推杆上升或下降。

#### 1.2.3 按键控制

<img width="933" height="337" alt="image" src="https://github.com/user-attachments/assets/4c25514a-6124-4608-98d8-85197acdc292" />

利用矩阵键盘切换不同界面显示以及进行模式选择：
- 按键13：显示课题题目、姓名学号以及当前时间
- 按键16：退出
- 按键14：显示温度以及三个欧拉角
- 按键1-3：控制电机1（上升、下降、暂停）
- 按键5-7：控制电机2
- 按键9-11：控制电机3
- 按键12：设置模式
- 按键15：自动调平

**管脚分配：**

| 功能 | 管脚 |
|------|------|
| LED | PC13 |
| 蜂鸣器 | PA3 |
| 矩阵键盘行 | PB8-PB11 |
| 矩阵键盘列 | PB12-PB15 |
| MPU6050 SCL | PB6 |
| MPU6050 SDA | PB7 |
| OLED SCL | PB6 |
| OLED SDA | PB7 |
| L298 IN1-IN6 | PA0-PA5 |

### 1.3 结构方案设计

<img width="888" height="511" alt="image" src="https://github.com/user-attachments/assets/13860cf3-0992-45bf-b8c4-2d3e4717afb3" />

圆盘平台用亚克力加工，直径16厘米，厚度5毫米。可调节高度的三个脚由电动推杆组成，推杆的伸缩由电机控制，实现高度的精确调节，推杆底部设计为通用螺纹接口。MPU6050模块使用胶带粘贴在预定位置。

### 1.4 仿真设计

<img width="1210" height="847" alt="image" src="https://github.com/user-attachments/assets/46b5e977-ddd2-4107-bc3d-2c1f7e89c199" />

<img width="1210" height="841" alt="image" src="https://github.com/user-attachments/assets/1205ff70-08c8-4e09-8773-fdcb9fafd3c7" />

在拟定好初步方案及编写部分程序后，在Proteus中进行了仿真。由于软件中没有MPU6050传感器，即使下载导入模型也不能进行仿真，因此用按键来代替，按下不同按键对应的角度加一，按下最后一个按键角度清零，电机转动，同时声光提示。

### 1.5 系统框图

<img width="393" height="217" alt="image" src="https://github.com/user-attachments/assets/a5c7e6a0-d765-4607-b871-7b89a9a2bec0" />

<img width="650" height="361" alt="image" src="https://github.com/user-attachments/assets/49ce3aa6-1f5e-4372-bce3-cd16d9b7b816" />

该系统以STM32F103C8T6为核心，通过L298驱动电动推杆，进而改变角度，通过MPU6050解算欧拉角，并以此作为反馈控制电机运动实现自动调平，同时有矩阵键盘和OLED实现人机交互，LED和蜂鸣器实现调平后的声光提示。

## 二、硬件电路设计

### 2.1 矩阵键盘

<img width="665" height="446" alt="image" src="https://github.com/user-attachments/assets/3ec62563-fe3f-42ff-85e2-69c56ec2d949" />

将16个按键排成4行4列，共8根线连接到单片机的PB8-PB15引脚，配置行为输出模式，列为输入模式，通过行列扫描检测按键状态。

### 2.2 声光提示

<img width="753" height="672" alt="image" src="https://github.com/user-attachments/assets/b2922b38-e24d-4859-b2fb-b282d3189491" />

- LED指示灯连接PC13引脚，低电平点亮
- 蜂鸣器模块连接PA3引脚，高电平驱动

### 2.3 MPU6050及OLED

<img width="402" height="403" alt="image" src="https://github.com/user-attachments/assets/50898ae2-877d-4099-bd17-d38f1eff320b" />

- MPU6050使用I2C接口：SCL→PB6，SDA→PB7

<img width="303" height="155" alt="image" src="https://github.com/user-attachments/assets/c30a6d71-0a8d-4114-8a8c-e4773daba6ec" />

- OLED使用四线I2C接口：SCL→PB6，SDA→PB7

### 2.4 L298及电机部分

<img width="659" height="254" alt="image" src="https://github.com/user-attachments/assets/d7553dad-2191-4662-a9b1-674a0ea4a18e" />

<img width="405" height="115" alt="image" src="https://github.com/user-attachments/assets/b81eff49-507c-4b43-9e6e-d26089e9fb6d" />

电动推杆有红黑两条线，红接高黑接低推杆上升，红接低黑接高推杆下降。L298N使用12V供电，输入端接STM32的IO口，输出端接电机。

### 2.5 总览

<img width="1042" height="804" alt="image" src="https://github.com/user-attachments/assets/c02e32e6-b4f5-4913-a34c-f700be4c4ead" />

经上述操作，完成了硬件电路各部分设计，为便于后续操作及实验验证，利用Fritzing软件制作了整体接线示意图，如图15，图中可以清楚地看到电源模块、L298N、电机、蜂鸣器模块、MCU、OLED、矩阵键盘、MPU6050等模块，将电动推杆、MPU6050安装在圆盘平台上后硬件工作即全部完成。

## 三、测试结果及分析

### 3.1 主界面

<img width="266" height="200" alt="image" src="https://github.com/user-attachments/assets/7ed81917-1b99-4c36-84ea-56cfdbf11aa6" />

按下13键进入主界面，显示课题题目、学号、姓名、年月日、星期、时间（实时更新）。

### 3.2 MPU6050界面

<img width="355" height="264" alt="image" src="https://github.com/user-attachments/assets/4cec9349-a8b6-401f-8c8d-886acda9f36d" />

按下14键进入，显示温度、俯仰角、横滚角、航向角，数据实时更新。

### 3.3 电机运动界面

<img width="299" height="232" alt="image" src="https://github.com/user-attachments/assets/70839975-dfb0-4943-8585-d316fb242f27" />

<img width="300" height="217" alt="image" src="https://github.com/user-attachments/assets/9282f6de-ac18-45f7-b8fc-0fcbdc482bf6" />

<img width="294" height="222" alt="image" src="https://github.com/user-attachments/assets/1ed29a12-3c71-4cee-9c8b-940777655ac3" />

- 按键1-3：控制电机1上升/下降/暂停
- 按键5-7：控制电机2
- 按键9-11：控制电机3

### 3.4 自动调平界面

<img width="319" height="225" alt="image" src="https://github.com/user-attachments/assets/5c446d1d-b1f4-4ddb-8fb7-f2a163801308" />

按下15键进入自动调平模式，系统自动检测并调整平台至水平状态，精度优于±1°，调平完成后声光提示。

### 3.5 设置角度界面

<img width="292" height="230" alt="image" src="https://github.com/user-attachments/assets/c60f3857-cbc9-41ad-a8f3-823926214686" />

按下12键进入，设置期望角度后系统自动调节到目标角度，精度±1°以内。

## 四、作品照片及说明

### 4.1 硬件部分
主要硬件包括：ST-Link、矩阵键盘、STM32F103C8T6、OLED、L298N、蜂鸣器、MPU6050、圆盘平台、电动推杆、电源模块等。

### 4.2 软件部分

<img width="758" height="394" alt="image" src="https://github.com/user-attachments/assets/f27e8bd2-01c9-46d5-8542-9f4d96cdd1e1" />

<img width="784" height="241" alt="image" src="https://github.com/user-attachments/assets/a6253974-dac5-49b9-b923-1c3743d660f8" />

<img width="424" height="217" alt="image" src="https://github.com/user-attachments/assets/28e83f7f-b7bc-48ac-a092-f5991882eeb9" />

<img width="394" height="223" alt="image" src="https://github.com/user-attachments/assets/536218ad-828c-4520-912d-0d337a55e355" />

<img width="393" height="241" alt="image" src="https://github.com/user-attachments/assets/67a44a29-29ba-422f-9c2d-7a944e29523b" />

<img width="523" height="183" alt="image" src="https://github.com/user-attachments/assets/ac461df4-3a0e-48a3-aaf7-e356eb8a799e" />

<img width="209" height="141" alt="image" src="https://github.com/user-attachments/assets/8c63660e-2914-4091-a7d5-5de5a3f4b72b" />

<img width="601" height="407" alt="image" src="https://github.com/user-attachments/assets/4b90dc10-43c9-476f-a913-a7d6a75e878f" />

<img width="588" height="407" alt="image" src="https://github.com/user-attachments/assets/ac4d8ece-bf04-4ef4-8455-edf730a6207a" />

<img width="398" height="294" alt="image" src="https://github.com/user-attachments/assets/bc483cf7-3b0e-499d-aacc-ea3a7aa6737a" />

<img width="863" height="228" alt="image" src="https://github.com/user-attachments/assets/848ef806-746d-400e-bac6-01586dcf8677" />

主要头文件及功能：

| 头文件 | 功能 |
|--------|------|
| led.h | LED控制 |
| key.h | 矩阵键盘扫描 |
| mpu6050.h | MPU6050初始化和数据读取 |
| oled.h | OLED显示 |
| motor.h | 电机控制 |
| beep.h | 蜂鸣器控制 |
| rtc.h | 实时时钟 |

## 五、结论

本设计成功实现了基于STM32F103C8T6和MPU6050的自动调平装置。系统通过MPU6050实时获取平台姿态，采用DMP姿态解算获得高精度欧拉角，通过控制三个电动推杆实现平台的自动调平，精度优于±1°。同时设计了丰富的人机交互界面，包括主界面显示、姿态数据显示、手动电机控制、自动调平和角度设置等功能，达到了设计要求。
