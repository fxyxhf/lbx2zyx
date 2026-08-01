---
title: "STM32F103 模拟篮球计分器设计"
date: 2023-08-14
publish_display_date: 2023-08-14
excerpt: ""
categories: [Embedded]
tags: [STM32, LCD, 红外遥控, 嵌入式综合设计, ADC温度采集]
layout: single
author_profile: true
---

## 一、实验要求
基于STM32F103ZET6编写程序，实现模拟篮球计分器。设备能够显示比赛总时间（分钟、秒、十分之一秒）；实时展示两队比分；支持按键控制计时暂停，暂停状态下比分正常显示；两支队伍均可实现加1、加2、加3分操作。

拓展附加功能：支持比分减分，单次三分操作触发LED指示灯；支持一键切换LCD屏幕背景色；支持比赛提前终止，终止后自动对比比分并显示获胜队伍；集成片上温度采集，LCD实时显示温度，同时通过串口向上位机输出温度数值。

## 二、设计思路
比赛计时分为两种模式，由独立按键切换：正常速率计时与0.1倍速计时。0.1倍速计时将十分之一秒计数器按照每秒自增1实现，正常计时模式按照标准0.1秒步进计数。

比分信息依靠正点原子LCD驱动函数`LCD_ShowString()`、`LCD_ShowxNum()`完成显示，合理配置坐标参数，避免文字、数字重叠错位，保证界面布局整齐。

计时启停与模式切换由开发板独立按键控制。定义状态变量`t`区分工作模式：`t=0`为0.1倍速计时，`t=1`暂停计时，`t=2`正常速率计时，`t=3`终止比赛。通过多分支`if...else if`语句根据`t`的值执行对应逻辑；暂停模式下跳过计时累加，时间保持冻结，但其余功能正常运行。

战舰V3开发板仅提供4路独立按键，无法承载全部计分操作，因此引入红外遥控器完成比分增减。采用`switch...case`匹配不同遥控键值，实现两队加分、减分逻辑。

额外拓展功能说明：
1. 减分功能：防止遥控误触造成比分错误；
2. 三分指示灯：任意队伍得到3分时点亮板载LED；
3. LCD背景切换：遥控器指定按键触发`LCD_Clear()`更换屏幕底色；
4. 比赛终止：计时清零，自动比较两队分数，在屏幕输出比赛结果；
5. 片内温度ADC采集，LCD实时展示温度，并周期性通过串口向上位机发送温度数据。

## 三、实验主要代码
该代码基于正点原子战舰V3开发板标准例程修改，仅展示`main.c`核心代码：
```c
#include "led.h"
#include "delay.h"
#include "key.h"
#include "sys.h"
#include "lcd.h"
#include "usart.h"
#include "remote.h"
#include "tsensor.h"
 
int main(void)
{			  
	u16 min=0;u16 s=0;u16 ss=0;
	u8 key,keyy;
	u8 t;
	u8 c=0;u8 a=0;u8 b=0;
	u8 x=0;
	u16 times;
	u16 temp1;u16 adcx;
	
	delay_init();	    	    
	uart_init(115200);	
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2); 	
 	LED_Init();			     
	LCD_Init(); 
	KEY_Init();
	Remote_Init();  
    T_Adc_Init();	
	
    while(1) 
    {	
	key=Remote_Scan();	
	if(key)
	{	 
		switch(key)
		{
			case 0:;break;			   
			case 162:;break;	    
			case 98:;break;	    
			case 2:;break;		 
			case 226:;break;		  
			case 194:;break;	   
			case 34:;break;		  
			case 224:;break;		  
			case 168:;break;		   
			case 144:;break;		    
			case 104:a=a+1;break;		       //1队加1分  
			case 152:a=a+2;break;	           //1队加2分
			case 176:a=a+3;LED0=0;break;	   //1队加3分，红灯亮 
			case 48:b=b+1;break;		       //2队加1分
			case 24:b=b+2;break;		       //2队加2分
			case 122:b=b+3;LED0=0;break;       //2队加3分，红灯亮		  
			case 16:a=a-1;break;			   //1队减1分				
			case 56:b=b-1;break;	           //2队减1分
			case 90:x++;if(x==9)x=0;;break;    //切换背景颜色
			case 66:;break;
			case 82:;break;		 
		}
	}
	else delay_ms(700);	 
		
	switch(x)                                  //切换背景颜色
	{
		case 0:LCD_Clear(WHITE);break;
		case 1:LCD_Clear(BLACK);break;
		case 2:LCD_Clear(MAGENTA);break;
		case 3:LCD_Clear(CYAN);break;
		case 4:LCD_Clear(YELLOW);break;
		case 5:LCD_Clear(BRRED);break;
		case 6:LCD_Clear(GRAY);break;
		case 7:LCD_Clear(LGRAY);break;
		case 8:LCD_Clear(BROWN);break;
	}
	POINT_COLOR=RED; 
	LCD_ShowString(10,50,200,16,16,"min:"); 
    LCD_ShowString(10,65,200,16,16,"s:"); 
	LCD_ShowString(10,80,200,16,16,"ss:"); 
	LCD_ShowString(30,100,200,16,16,"1  vs  2");		
		
		
	POINT_COLOR=BLUE;
	LCD_ShowxNum(50,50,min,4,16,0);
    LCD_ShowxNum(50,65,s,4,16,0);
	LCD_ShowxNum(50,80,ss,4,16,0);
		
	keyy=KEY_Scan(0);
	if(keyy)
	{
		switch(keyy)
		{
			case KEY1_PRES:t=1;break;     //暂停键
			case WKUP_PRES:t=0;break;     //0.1倍速计时
			case KEY0_PRES:t=2;break;     //正常计时
			case KEY2_PRES:t=3;break;     //终止比赛
		}
	}
		
	if(t==0)
	{
		ss++;
	    if(ss==10)
	    {
		 ss=0;
		 s++;
	    }
	    if(s==60)
	    {
		 s=0;
		 min++;
	    }
	    if(min==48)
	    {
		 min=0;
	    }
		 delay_ms(100);		
        }
		else if(t==1)
		{ }
		else if(t==2)
		{
			ss=0;
			ss=ss+10;
	        if(ss==10)
	        {
			    ss=0;
			    s++;
	        }
	        if(s==60)
	        {
			    s=0;
			    min++;
	        }
	        if(min==48)
	        {
			    min=0;
	        }
		    delay_ms(100);
		}
		else if(t==3)
		{
			ss=0;s=0;min=0;
			if(a>b) LCD_ShowString(30,180,200,16,16,"1 is the winner");
			else if(a<b) LCD_ShowString(30,180,200,16,16,"2 is the winner");
			else LCD_ShowString(30,180,200,16,16,"1=2");
		}
						
	POINT_COLOR=GREEN;
	LCD_ShowxNum(10,120,a,4,16,0);
	LCD_ShowxNum(60,120,b,4,16,0);
		
	temp1=Get_Temprate();	//得到温度值 
	if(temp1<0)
	{
		temp1=-temp1;
		LCD_ShowString(30+10*8,140,16,16,16,"-");	//显示负号
	}else LCD_ShowString(30+10*8,140,16,16,16," ");	//无符号	
    LCD_ShowString(30,140,200,16,16,"TEMPERATE: 00.00C");	
    adcx=temp1/100;		
	LCD_ShowxNum(30+11*8,140,adcx,2,16,0);		//显示整数部分
	LCD_ShowxNum(30+14*8,140,temp1%100,2,16, 0X80);	//显示小数部分
		
	if(USART_RX_STA&0x8000)
	{					   
			
	}else
	{
		times++;
		if(times%5000==0)
		{
			printf("篮球计分器\r\n");
			printf("综合设计\r\n\r\n");
		}
		if(times%10==0) printf("\r\n%f\r\n",(double)(adcx));  
		delay_ms(10);   
	}
}
}
```

## 四、实验结果
1. 上电初始界面：时分秒全部清零，两队比分0:0，持续采集并显示片内温度；
2. 遥控操作1队+1分，界面比分更新为1:0；
3. 在1:0基础上1队+2分，比分更新为3:0；
4. 在3:0基础上1队+3分，比分更新为6:0，同时开发板红灯点亮；
5. 在6:0基础上2队+1分，比分更新6:1；
6. 在6:1基础上2队+2分，比分更新6:3；
7. 在6:3基础上2队+3分，比分更新6:6；
8. 遥控器按键切换，循环切换LCD多种背景颜色；
9. 在6:6比分下执行1队减1分，比分5:6；再次执行2队减1分，比分5:5；
10. 独立按键切换至正常计时模式、0.1倍速计时模式，时间按照对应速率递增；
11. 按下暂停键，计时冻结，温度采集、比分显示功能不受影响；
12. 按下终止比赛按键：计时清零。比分相等时屏幕输出`1=2`；1队分数更高输出`1 is the winner`；2队分数更高输出`2 is the winner`；
13. 串口调试助手持续接收单片机上传的实时温度数值。

## 五、注意事项
1. 红外遥控器距离、角度会影响按键识别，遥控扫描加入延时消抖，避免连续多次触发；
2. 比分变量无上限保护，正式使用可增加阈值限制，防止数值溢出；
3. LCD坐标参数修改时注意字体宽度，避免文字重叠；
4. 串口波特率固定115200，上位机调试软件参数必须与程序保持一致；
5. 三分触发LED点亮后代码未设计自动熄灭逻辑，如需自动熄灭可增加延时翻转IO。
