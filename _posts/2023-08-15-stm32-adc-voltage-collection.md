---
title: 基于STM32F103的ADC电压采集实验
date: 2023-08-15
publish_display_date: 2023-08-15
excerpt: ""
categories: [Embedded]
tags: [STM32, ADC, 串口, 电压采集]
layout: single
author_profile: true
---

## 一、实验目的
基于STM32F103ZET6编写程序，通过ADC1的通道1（PA1）读取外部电压值（0~3.3V），并将读取的电压值通过串口调试助手实时显示，同时LCD屏上同步显示原始ADC值和电压值。

## 二、设计思路
正点原子战舰V3开发板自带的ADC例程已实现了在LCD上显示转换结果，但无法在PC端通过串口查看。本实验在保留LCD显示的基础上，增加串口输出功能，利用USART1将采集到的电压值以浮点数格式发送至串口调试助手，便于远程监控和数据分析。

硬件连接：将待测电压源（0~3.3V）连接至PA1引脚，开发板通过USB串口与PC通信。软件上配置ADC1为独立模式、单次转换、软件触发，采用DMA或查询方式读取转换结果，本实验使用查询方式，通过多次采样取平均以提高稳定性。

## 三、主要代码
该代码以正点原子STM32F103战舰V3的ADC例程为基础，在`main.c`中增加了串口打印功能，并针对通道1（PA1）进行配置。以下为`main.c`完整代码：

```c
#include "led.h"
#include "delay.h"
#include "key.h"
#include "sys.h"
#include "lcd.h"
#include "usart.h"	 
#include "tsensor.h"
#include "adc.h"
 
int main(void)
{	 	
    u16 times=0;
    u16 adcx;
    float temp;
    
    delay_init();                                     // 延时函数初始化
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);   // 中断优先级分组
    uart_init(115200);                                // 串口初始化，波特率115200
    LED_Init();                                       // LED初始化
    LCD_Init();                                       // LCD初始化
    ADC_Config_Init();                                // ADC初始化（通道1）
    KEY_Init();                                       // 按键初始化
	 
    POINT_COLOR=RED; 
    LCD_ShowString(60,50,200,16,16,"WarShip STM32");
    LCD_ShowString(60,70,200,16,16,"ADC TEST");
    LCD_ShowString(60,90,200,16,16,"xhsbq");
    LCD_ShowString(30,110,200,16,16,"2023/8/15");
    POINT_COLOR=BLUE; 
    LCD_ShowString(60,130,200,16,16,"ADC_CH1_VAL:");   // 显示ADC原始值
    LCD_ShowString(60,150,200,16,16,"ADC_CH1_VOL:0.000V"); // 显示电压值
    
    while(1)
    {
        // 获取ADC通道1的10次采样平均值
        adcx = Get_Adc_Average(ADC_Channel_1, 10);
        LCD_ShowxNum(156,130,adcx,4,16,0);              // 显示原始值（0~4095）
        
        // 计算电压值（3.3V参考）
        temp = (float)adcx * (3.3 / 4096);
        adcx = temp;                                    // 整数部分
        LCD_ShowxNum(156,150,adcx,1,16,0);              // 显示电压整数位
        temp -= adcx;
        temp *= 1000;
        LCD_ShowxNum(172,150,temp,3,16,0X80);           // 显示电压小数位（3位）

        // 串口发送数据
        if(USART_RX_STA & 0x8000)   // 若接收到数据，则处理（本例未使用）
        {					   
            // 可添加串口命令解析
        }
        else
        {
            times++;
            // 每5秒打印一次提示信息
            if(times % 5000 == 0)
            {
                printf("\r\n战舰STM32开发板\r\n");
                printf("xhsbq\r\n\r\n");
            }
            // 每300ms打印一次当前电压值
            if(times % 30 == 0)
            {
                printf("\r\n电压值: %.3f V\r\n", temp + adcx); // 恢复完整电压
            }
            delay_ms(10);   
        }
    }
}
```

**说明**：  
- `ADC_Config_Init()` 函数内部配置了ADC1的通道1（PA1），采用常规通道、软件触发、12位分辨率。  
- `Get_Adc_Average()` 执行多次转换取平均，以抑制噪声。  
- 串口每隔约300ms输出一次电压值（`times%30`，延时10ms，共300ms），同时每5秒输出一次标题信息。

## 四、实验结果
1. **硬件连接**：将可调直流电源或电位器分压电路输出端连接至开发板的PA1引脚，GND共地。

2. **LCD显示**：上电后，LCD屏正确显示“ADC_CH1_VAL”和“ADC_CH1_VOL”，且数值随输入电压变化实时更新。

3. **串口调试助手**：打开串口助手，设置波特率115200、8位数据、1位停止位、无校验。可

4. **稳定性**：多次采样平均使得读数抖动较小，LCD与串口显示一致，验证了ADC采集和串口通信功能的正确性。
