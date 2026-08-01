---
title: 基于STM32F103的简易洗衣机控制系统设计
date: 2023-08-14
categories: STM32实验
tags: [STM32F103,嵌入式,控制系统]
---

## 一、实验要求
基于STM32F103ZET6编写程序，设计一个简易洗衣机控制系统，可以通过键盘设定洗衣时间和洗衣次数，可以显示洗涤、漂洗等时间，可以进行强洗和弱洗选择，可以进行水位检测模拟，并且洗完可以报警。

## 二、设计思路
首先针对第一个要求通过键盘设定洗衣时间和洗衣次数，我使用了游戏手柄来控制，而我将洗衣时间分为洗涤时间和漂洗时间，分别设定，二者的分和秒均可自由调节，而按下相应按键时洗衣次数也会增加。

对于第二个要求显示洗涤、漂洗等时间，我在第一个要求中已经实现了洗涤时间和漂洗时间的分别设定，在按下开启按键后，开始洗涤，洗涤时间开始倒计时，当洗涤结束后红灯亮进行提示，然后开始漂洗，漂洗时间开始倒计时。对于强洗和弱洗的模式选择，我依然通过游戏手柄上的按键来控制，首先我设置默认为强洗模式，按下按键次数为奇数时变更为弱洗模式，按下按键次数为偶数时不变，仍为强洗模式。

对于水位检测模拟，我设定10为临界状态，当水位超过10时，绿灯亮起，当按下触摸按键时，绿灯可以短暂熄灭，而水位值的设定我依然通过按键，初始值为零，依次加一。

在上述要求之外，我还设计了实时时间显示，通过RTC来显示年月日、时分秒、以及星期，同时由于usmart的功能，通过向串口调试助手发送相应参数值，可以校正当前时间，或者变更为其他时间。此外，我还设计了实时采集当前温度，并在LCD屏上显示。

## 三、主要代码
该代码以正点原子战舰开发板V3提供的例程为基础，此处给出`main.c`完整代码：
```c
#include "led.h"
#include "delay.h"
#include "key.h"
#include "sys.h"
#include "lcd.h"
#include "usart.h"	
#include "usmart.h"	 
#include "rtc.h" 
#include "joypad.h" 
#include "tpad.h"
#include "beep.h"
#include "tsensor.h"
 
 int main(void)
 {	 
 	u8 t=0;	
	u8 key,keyy;
	u8 cs=0;
	u8 sw=0;
	u8 x=0;
	u8 k;
	u8 dt1=0,dt2=0;u8 pt1=0,pt2=0;
	u16 temp1;u16 adcx;
	 
	delay_init();	    	 //延时函数初始化	  
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//设置中断优先级分组为组2：2位抢占优先级，2位响应优先级
	uart_init(115200);	 	//串口初始化为115200
 	LED_Init();			     //LED端口初始化
	LCD_Init();	
        KEY_Init();	 
	usmart_dev.init(SystemCoreClock/1000000);	//初始化USMART	
	RTC_Init();	  			//RTC初始化
	JOYPAD_Init();
	TPAD_Init(6);
	BEEP_Init();
	T_Adc_Init();
	 
	POINT_COLOR=RED;//设置字体为红色 
	LCD_ShowString(30,50,200,16,16,"washing  machine ");	
	LCD_ShowString(30,70,200,16,16,"Washing Time:         :");	 //洗涤时间
	LCD_ShowString(30,90,200,16,16,"Rinsing time:         :");   //漂洗时间
	LCD_ShowString(30,110,200,16,16,"Laundry frequency:");	     //洗衣次数
	LCD_ShowString(30,130,200,16,16,"Laundry mode:");            //洗衣模式
	LCD_ShowString(30,150,200,16,16,"water level:");             //水位
	LCD_ShowString(30,170,200,16,16,"current time");             //当前时间
	POINT_COLOR=BLUE;//设置字体为蓝色
	LCD_ShowString(30,190,200,16,16,"    -  -  ");	   
	LCD_ShowString(30,210,200,16,16,"   :   :   ");		    
	while(1)
	{	
		if(t!=calendar.sec)
		{
			t=calendar.sec;
			LCD_ShowNum(30,190,calendar.w_year,4,16);									  
			LCD_ShowNum(70,190,calendar.w_month,2,16);									  
			LCD_ShowNum(100,190,calendar.w_date,2,16);	 
			switch(calendar.week)
			{
				case 0:
					LCD_ShowString(30,230,200,16,16,"Sunday   ");
					break;
				case 1:
					LCD_ShowString(30,230,200,16,16,"Monday   ");
					break;
				case 2:
					LCD_ShowString(30,230,200,16,16,"Tuesday  ");
					break;
				case 3:
					LCD_ShowString(30,230,200,16,16,"Wednesday");
					break;
				case 4:
					LCD_ShowString(30,230,200,16,16,"Thursday ");
					break;
				case 5:
					LCD_ShowString(30,230,200,16,16,"Friday   ");
					break;
				case 6:
					LCD_ShowString(30,230,200,16,16,"Saturday ");
					break;  
			}
			LCD_ShowNum(30,210,calendar.hour,2,16);									  
			LCD_ShowNum(60,210,calendar.min,2,16);									  
			LCD_ShowNum(90,210,calendar.sec,2,16);
		}	
		delay_ms(10);	
 
		key=KEY_Scan(1);
		if(key)
		{	 				  
			switch(key)
			{
				case KEY0_PRES:;break;			   
				case KEY1_PRES:;break;	    
				case KEY2_PRES:break;	    
				case WKUP_PRES:k=0;break;		 				 
			}
			
		}
		else delay_ms(100);
		
		if(k==0)
		{
			printf("开始洗衣\r\n");
			if(dt1>0)  
			{
				dt2--;
			    if(dt2==0)  
				{	
				  dt2=59;dt1--;
				}
			}
			if(dt1==0) 
			{
				dt2--;
			    if(dt2==0) 
				{
					k=1;			        
				}
			}
		}
		if(k==1)
		{
			if(pt1>0)
			        {
				     pt2--;
			         if(pt2==0)  
				    {	
				     pt2=59;pt1--;
				    }
			        }
			        if(pt1==0) 
				    pt2--;
				    if(pt2==0)
				    {
						BEEP=1;
						k=2;
					}
		}
		if(k==2)
		{
			printf("洗衣完毕\r\n"); 
		}
		
								
		keyy=JOYPAD_Read();	
		if(keyy)
		{	 
				  
			switch(keyy)
			{	    
				case 16:dt1=dt1+1;break;	  //UP	  
				case 32:dt2=dt2+1;break;	  //DOWN   
				case 64:pt1=pt1+1;break;	  //LEFT    
				case 128:pt2=pt2+1;break;     //RIGHT		    
				case 4:x=x+1;break;		      //SELECT   
				case 8:cs=cs+1;break;	      //START	  
				case 2:sw=sw+1;break;	      //A		   					
				case 1:;break;	 	          //B
			}
			
		}
		else delay_ms(700);	 
		
		if(sw>10) LED1=0;
		if(TPAD_Scan(0))	
		{
		  LED1=!LED1;		
		} 
		
		if(x%2==0)	LCD_ShowString(130,130,200,16,16,"Forcedwashing");		    
		else        LCD_ShowString(130,130,200,16,16,"Weakwashing");
		
		temp1=Get_Temprate();	//得到温度值 
                LCD_ShowString(30,250,200,16,16,"TEMPERATE: 00.00C");	
                adcx=temp1/100;		
		LCD_ShowxNum(30+11*8,250,adcx,2,16,0);		//显示整数部分
		LCD_ShowxNum(30+14*8,250,temp1%100,2,16, 0X80);	//显示小数部分
		
		LCD_ShowNum(200,110,cs,2,16);									  
		LCD_ShowNum(200,150,sw,2,16);	
		LCD_ShowNum(180,70,dt1,2,16);
		LCD_ShowNum(210,70,dt2,2,16);
		LCD_ShowNum(180,90,pt1,2,16);	
		LCD_ShowNum(210,90,pt2,2,16);
	}
 }
```

## 四、实验结果
1. 洗衣机初始状态，从上至下依次为洗涤时间，漂洗时间，洗衣次数，洗衣模式（默认为强洗），水位，当前时间，当前温度。
2. 洗涤时间和漂洗时间设定界面，示例洗涤时间设定为1分2秒，漂洗时间设定为3分3秒。
3. 洗衣模式切换演示，可以由强洗切换为弱洗。
4. 洗衣次数调节界面，可通过按键增加洗衣次数。
5. 洗涤阶段倒计时，洗涤结束自动进入漂洗倒计时。
6. 漂洗阶段倒计时，漂洗全部完成后蜂鸣器报警提示洗衣结束。
7. 水位模拟检测：水位数值大于10时绿灯点亮；按下触摸按键，绿灯短暂熄灭，水位数值可通过按键递增调整。
