---
title: "STM32F103 串行通信控制LED实验"
date: 2023-08-13
publish_display_date: 2023-08-13  
excerpt: ""
categories: [Embedded]
tags: [STM32, USART, 串口通信, 嵌入式]
layout: single
author_profile: true
---

## 一、实验目的
基于STM32F103ZET6编写程序，在串口调试助手中输入`$START$`时绿灯点亮；输入`$STOP$`时绿灯熄灭。

## 二、实验主要代码
该代码基于正点原子STM32F103战舰V3串口实验例程修改，下方仅展示`main.c`核心代码：
```c
#include "led.h"
#include "delay.h"
#include "key.h"
#include "sys.h"
#include "usart.h"
#include "string.h"

int main(void)
{
    u8 t;
    u8 len;
    delay_init();         //延时函数初始化 
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2); // 设置NVIC中断分组2
    uart_init(115200);    //串口初始化，波特率115200
    LED_Init();           //LED端口初始化
    KEY_Init();           //初始化按键硬件接口
    
    while(1)
    {
        if(USART_RX_STA&0x8000)
        {
            len=USART_RX_STA&0x3f; 
            printf("\r\n 您发送的消息为:\r\n\r\n");
            for(t=0;t<len;t++)
            {
                USART_SendData(USART1, USART_RX_BUF[t]); 
                if(strncmp(USART_RX_BUF,"$STOP$",6)==0) 
                {
                    LED1=1;
                }
                else if(strncmp(USART_RX_BUF,"$START$",7)==0)  
                {
                    LED1=0;
                }
                while(USART_GetFlagStatus(USART1,USART_FLAG_TC)!=SET);      
            }
            printf("\r\n\r\n"); //插入换行
            USART_RX_STA=0;     
        }
    }
}
```

程序逻辑说明：
首先引入硬件驱动头文件，依次调用初始化函数：延时初始化、中断优先级分组、串口初始化、LED初始化、按键初始化。
主循环内检测接收状态标志位 `USART_RX_STA&0x8000`，该标志在串口中断接收到换行符`0x0A`后置1，代表一帧数据接收完成。
`USART_RX_STA&0x3f` 获取本次接收的数据长度，通过for循环将收到的数据回传至上位机。

核心逻辑依靠 `strncmp` 字符串限定长度比较函数：
```c
int strncmp ( const char * str1, const char * str2, size_t n );
```
最多比较前`n`个字节，匹配成功返回0。
>⚠️区分：`strcmp()`会完整比较整条字符串，无长度限制，本实验场景不推荐使用。

匹配到`$START$`点亮绿灯，匹配`$STOP$`熄灭绿灯，处理完成后清空接收标志位，等待下一次指令。

## 三、实验结果
1. 发送`$START$` → 绿灯点亮
2. 发送`$STOP$` → 绿灯熄灭
重复发送指令可以反复切换LED状态。

## 四、注意事项
1. 串口调试助手需要选择正确COM口，波特率设置为 **115200**，与程序保持一致；
2. 程序依靠**回车换行符`0x0A`判定接收结束**，**必须开启【发送新行】**；
在XCOM调试助手中勾选「发送新行」，软件会在发送文本后自动追加回车换行，否则单片机无法识别指令；
3. 指令严格区分字符，不要额外添加空格，保证发送内容为纯 `$START$`、`$STOP$`。
