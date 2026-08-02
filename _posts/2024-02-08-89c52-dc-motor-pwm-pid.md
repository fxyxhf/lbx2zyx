---
title: 89C52单片机控制PWM输出的直流电机闭环调速系统设计
date: 2024-02-08
publish_display_date: 2024-02-08
excerpt: ""
categories: [Microcontroller, Control System]
tags: [89C52, PWM, 直流电机, PID控制, 闭环调速]
layout: single
author_profile: true
---

## 一、实验目的
1. 了解89C52单片机内部三个定时器/计数器0、1和2的工作原理；
2. 掌握基于89C52单片机定时器2的PWM波的输出和电机功率驱动原理，及其实现方法；
3. 掌握以89C52单片机为核心、基于PID控制的直流电机闭环调速系统的软、硬件设计方法。

## 二、实验内容
本实验以AT89C52为核心组成实时控制系统，使用PID控制。系统输入两个参考控制信号：转速给定信号 r(t) 和转速反馈信号 c(t)，通过PID程序的控制在单片机 P1.5 口输出 PWM 信号来控制达林顿管 TIP122，通过单片机内部的定时器调节 PWM 信号的占空比，直流电机的转速随之变化，从而实现对直流电机的转速控制。

具体信号路径：
- 从单片机的定时器 T0 口输入 250kHz 的给定信号，此信号作为直流电机的转速基准信号，在调节过程中保持不变；
- 单片机定时器 T1 口输入转速反馈信号，在电机稳速转动时反馈信号为 5kHz；
- 以上两个信号在PID调节过程中是比例环节的输入信号，PID程序通过比较二者的变化，当转速变化时启动PID控制，通过比例、积分、微分环节改变PWM波的占空比，对直流电机转速进行实时调节。

## 三、实验流程图

<img width="550" height="123" alt="image" src="https://github.com/user-attachments/assets/cefbc32b-aee9-4a5a-8340-1bd6fafe7908" />

<img width="479" height="194" alt="image" src="https://github.com/user-attachments/assets/106006fa-aa90-411b-8f55-f413f40d6922" />

### 系统框图
直流电机闭环调速系统由给定信号、反馈信号、PID控制器、PWM发生器、驱动电路和直流电机组成。单片机读取给定与反馈的差值，经PID运算后输出PWM占空比控制信号，驱动电机调速。

<img width="640" height="437" alt="image" src="https://github.com/user-attachments/assets/d198c24f-9f94-4889-941b-7bad6fee68e0" />

### 程序流程图
程序主要包括：
- 初始化定时器2用于PWM输出，定时器0和1用于测速；
- 主循环中等待定时器1中断（采样周期）；
- 中断服务程序中计算误差，执行PID运算，更新PWM占空比。

## 四、实验结果

<img width="498" height="295" alt="image" src="https://github.com/user-attachments/assets/e6c51f7a-9618-4ad8-a37e-98821915a29d" />

### 硬件电路图
硬件电路以AT89C52为核心，P1.5输出PWM信号经TIP122达林顿管驱动直流电机，转速反馈信号经整形后接入T1，给定信号由外部信号源经T0输入。

<img width="554" height="257" alt="image" src="https://github.com/user-attachments/assets/b5a7bf93-06f5-4df3-919f-ea588d521cc0" />

### 运行结果
系统上电后，电机根据给定转速启动，通过PID调节后转速稳定在设定值附近，示波器显示PWM波形占空比随负载变化自动调整，验证了闭环调速功能。

## 五、参考程序
以下为完整的89C52汇编程序，包含初始化、PID计算、PWM更新和中断服务：

```assembly
ORG 0000H
START: SJMP START1
		ORG 001BH
T10: AJMP T100
	ORG 002BH
T2: AJMP T20
	ORG 0030H
START1: CLR P1.5      ; PWM=0
		SETB 78H        ; SET P1.5 SIGN
		MOV IE,#10100000B  ; T2 REQ
		SETB IP.5       ; T2 INTERRVPT PRIORITY
		MOV 0C8H,#00000000B  ; T2CON
		MOV 0CDH,#0D8H   ; TH2
		MOV 0CCH,#0F0H   ; TL2
		MOV 0CBH,#0D8H   ; RCAP2H
		MOV 0CAH,#0F0H   ; RCAP2L
		MOV 31H,#0D8H    ;
		MOV 30H,#0F0H    ; PWM = 1, 10000
		MOV 33H,#0D8H    ;
		MOV 32H,#0F0H    ; PWM = 0, 10000
		NOP
		MOV TMOD,#65H    ; T1 MODE 2, T0 MODE 1
		MOV TH0,#00
		MOV TL0,#00
		MOV TH1,#244     ;
		MOV TL1,#244     ; 12*1000/300 ms = 40 ms
		MOV 41H,#03H     ;
		MOV 40H,#84H     ; 600
		NOP
		MOV 71H,#00
		MOV 70H,#24
		MOV 72H,#01      ; KP1,KP2; KP1=24, KP2=1, DIV 2
		MOV 75H,#00
		MOV 74H,#16
		MOV 76H,#05      ; KI1, KI2; KI1=16, KI2=5, DIV 32
		MOV 79H,#00
		MOV 78H,#00
		MOV 7AH,#05      ; KD1, KD2; KD1=0, KD2=5, DIV 32
		MOV 7FH,#00
		MOV 7EH,#00
		MOV 7DH,#00
		MOV 7CH,#00
		MOV 35H,#00
		MOV 34H,#00
		MOV 37H,#00
		MOV 36H,#00
		MOV 39H,#00
		MOV 38H,#00      ; e(k)
		MOV 3BH,#00
		MOV 3AH,#00      ; e(k-1)
		MOV 3DH,#00
		MOV 3CH,#00      ; e(k-2)
		MOV 59H,#00
		MOV 58H,#00      ; e(k)-e(k-1)
		MOV 5BH,#00
		MOV 5AH,#00      ; e(k)
		MOV 5DH,#00
		MOV 5CH,#00      ; e(k)-2e(k-1)+e(k-2)
		MOV 4BH,#00H
		MOV 4AH,#00H
		MOV 49H,#00H
		MOV 48H,#00H     ; KP1/KP2 [e(k)-e(k-1)]
		MOV 4FH,#00H
		MOV 4EH,#00H
		MOV 4DH,#00H
		MOV 4CH,#00H     ; KI1/KI2 e(k)
		MOV 53H,#00H
		MOV 52H,#00H
		MOV 51H,#00H
		MOV 50H,#00H     ; KD1/KD2 [e(k)-2e(k-1)+e(k-2)]
		MOV 57H,#00H
		MOV 56H,#00H
		MOV 55H,#0D8H
		MOV 54H,#0F0H    ; n(k-1)=10000
		MOV 61H,#00
		MOV 60H,#00
START2: SETB P1.5
		CLR 78H          ; CLEAR P1.5 SIGN
		SETB 0CAH         ; SET TR2
		MOV R2,#60
		LCALL DELAY       ; DELAY FOR 3S
		SETB TR0          ; SET TR0
		SETB TR1          ; SET TR1
		SETB IE.3         ; T1 INTERRUPT ENABLE
HERE:	SJMP HERE
		ORG 0200H
T100:	PUSH ACC
		PUSH B
		PUSH PSW
		CPL P1.4          ; 测试点
T11:	MOV A,TH0
		MOV B,TL0
		CJNE A,TH0,T11
		PUSH ACC
		PUSH B
		CLR C
		XCH A,B
		SUBB A,34H
		MOV 36H,A
		XCH A,B
		SUBB A,35H
		MOV 37H,A
		POP 34H
		POP 35H
		CLR C
		MOV A,40H
		SUBB A,36H
		MOV 38H,A
		MOV 60H,A
		MOV A,41H
		SUBB A,37H
		MOV 39H,A
		MOV 61H,A
		JNC PT
NT:	SETB 7CH
		MOV A,60H
		CPL A
		ADD A,#01H
		MOV 60H,A
		MOV A,61H
		CPL A
		ADDC A,#00H
		MOV 61H,A        ; |e(k)|
		SJMP PTT
PT:	CLR 7CH
PTT:	CLR C
		MOV A,60H
		SUBB A,#200
		MOV A,61H
		SUBB A,#00
		JNC PTTT         ; 500 < SAMPLE < 70000 < SAMPLE < 700
		SJMP PID
PTTT:	LJMP BANG
PID:	CPL P1.2          ; 测试点
		CLR C
		MOV A,38H
		SUBB A,3AH
		MOV 58H,A
		MOV A,39H
		SUBB A,3BH
		MOV 59H,A        ; e(k)-e(k-1)
		MOV 5AH,38H
		MOV 5BH,39H      ; e(k)
		CLR C
		MOV A,3CH
		SUBB A,3AH
		MOV 5CH,A
		MOV A,3DH
		SUBB A,3BH
		MOV 5DH,A
		MOV A,5CH
		ADD A,58H
		MOV 5CH,A
		MOV A,5DH
		ADDC A,59H
		MOV 5DH,A        ; e(k)-2e(k-1)+e(k-2)
		MOV R0,#58H
		MOV R1,#70H
		LCALL LOAD
		MOV R0,#48H
		LCALL MUL3       ; KP1[e(k)-e(k-1)]
		MOV 7FH,4BH
		MOV 7EH,4AH
		MOV 7DH,49H
		MOV 7CH,48H
		MOV R0,72H
		LCALL DIVV
		MOV 4BH,7FH
		MOV 4AH,7EH
		MOV 49H,7DH
		MOV 48H,7CH      ; KP1/KP2 [e(k)-e(k-1)]
		MOV R0,#5AH
		MOV R1,#74H
		LCALL LOAD
		MOV R0,#4CH
		LCALL MUL3       ; KI1 e(k)
		MOV 7FH,4FH
		MOV 7EH,4EH
		MOV 7DH,4DH
		MOV 7CH,4CH
		MOV R0,76H
		LCALL DIVV
		MOV 4FH,7FH
		MOV 4EH,7EH
		MOV 4DH,7DH
		MOV 4CH,7CH      ; KI1/KI2 e(k)
		MOV R0,#5CH
		MOV R1,#78H
		LCALL LOAD
		MOV R0,#50H
		LCALL MUL3       ; KD1 [e(k)-2e(k-1)+e(k-2)]
		MOV 7FH,53H
		MOV 7EH,52H
		MOV 7DH,51H
		MOV 7CH,50H
		MOV R0,7AH
		LCALL DIVV
		MOV 53H,7FH
		MOV 52H,7EH
		MOV 51H,7DH
		MOV 50H,7CH      ; KD1/KD2 [e(k)-2e(k-1)+e(k-2)]
		MOV R0,#54H
		MOV R1,#48H
		MOV R2,#03
		LCALL ADD4       ; n(k-1)+KP[e(k)-e(k-1)]
		MOV R0,#54H
		MOV R1,#4CH
		MOV R2,#03
		LCALL ADD4       ; n(k-1)+KP[...]+KIe(k)
		MOV R0,#54H
		MOV R1,#50H
		MOV R2,#03
		LCALL ADD4       ; n(k)=n(k-1)+KP+KI+KD
		MOV A,57H
		CJNE A,#00,BJ
		MOV A,56H
		CJNE A,#00,BJ
		SJMP NOR
BJ:	CPL P1.6
NOR:	CLR IE.5           ; CLOSE T2 REQ
		MOV 31H,55H
		MOV 30H,54H      ; WIDTH of PWM-1
		CLR C
		MOV A,#0E0H
		SUBB A,30H
		MOV 32H,A
		MOV A,#0B1H
		SUBB A,31H
		MOV 33H,A
		SETB IE.5        ; OPEN T2 REQ
		MOV 3CH,3AH
		MOV 3DH,3BH      ; STORE NEW e(k-2)
		MOV 3AH,38H
		MOV 3BH,39H      ; STORE NEW e(k-1)
		SJMP T1RE
BANG:	CPL P1.3          ; 测试点
		JB 7CH,T0_FAST
T0_SLOW: MOV 31H,#0F8H
		MOV 30H,#30H     ; COUNTER=2000
		MOV 33H,#0B9H
		MOV 32H,#0B0H    ; L BYTE, COUNTER-18000
		SJMP T1RE0
T0_FAST: MOV 31H,#0B9H
		MOV 30H,#0B0H    ; H BYTE, COUNTER-18000
		MOV 33H,#0F8H
		MOV 32H,#30H     ; L BYTE, COUNTER=2000
T1RE0:	MOV 39H,#00
		MOV 38H,#00      ; e(K)=0
		MOV 3BH,#00
		MOV 3AH,#00      ; e(K-1)=0
		MOV 3DH,#00
		MOV 3CH,#00      ; e(K-2)=0
		MOV 57H,#00H
		MOV 56H,#00H
		MOV 55H,#0D8H
		MOV 54H,#0F0H    ; n(k-1)=10000
T1RE:	NOP
T1RE1:	POP PSW
		POP B
		POP ACC
		RETI
		ORG 0400H
T20:	CLR 0CFH           ; CLR TF2
		JNB 78H,CPWM
SPWM:	SETB P1.5          ; PWM=1
		MOV 0CBH,33H      ; PRESENT PWM=1
		MOV 0CAH,32H      ; NEXT PWM-0 WIDTH
		CLR 78H
		SJMP T20R
CPWM:	CLR P1.5           ; PWM=0
		MOV 0CBH,31H      ; PRESENT PWM=0
		MOV 0CAH,30H      ; NEXT PWM-1 WIDTH
		SETB 78H
T20R:	RETI
		ORG 0500H
LOAD:	MOV A,@R0
		MOV R6,A
		INC R0
		MOV A,@R0
		MOV R7,A
		MOV A,@R1
		MOV R4,A
		INC R1
		MOV A,@R1
		MOV R5,A
		RET
		ORG 0600H
MUL3:	MOV A,R7
		RLC A
		MOV 7AH,C
		MOV A,R5
		RLC A
		MOV 79H,C
		ANL C,7AH
		JC POSI
		MOV C,7AH
		ORL C,79H
		SJMP SIGN
POSI:	CPL C
SIGN:	MOV 7BH,C
		MOV A,R7
		JB ACC.7,CPL11
MUL01:	MOV A,R5
		JB ACC.7,CPL22
MUL10:	LCALL MUL1
		JB 7BH,CPL33
		SJMP MUL11
CPL11:	MOV A,R6
		CPL A
		ADD A,#01H
		MOV R6,A
		MOV A,R7
		CPL A
		ADDC A,#00H
		MOV R7,A
		SJMP MUL01
CPL22:	MOV A,R4
		CPL A
		ADD A,#01H
		MOV R4,A
		MOV A,R5
		CPL A
		ADDC A,#00H
		MOV R5,A
		SJMP MUL10
CPL33:	DEC R0
		DEC R0
		DEC R0
		MOV R2,#03H
CPL2:	MOV A,@R0
		CPL A
		ADD A,#01H
		MOV @R0,A
NEXT:	INC R0
		MOV A,@R0
		CPL A
		ADDC A,#00H
		MOV @R0,A
		DJNZ R2,NEXT
MUL11:	RET
		ORG 0700H
MUL1:	MOV A,R6
		MOV B,R4
		MUL AB
		MOV @R0,A
		MOV R3,B
		MOV A,R4
		MOV B,R7
		MUL AB
		ADD A,R3
		MOV R3,A
		MOV A,B
		ADDC A,#00
		MOV R2,A
		MOV A,R6
		MOV B,R5
		MUL AB
		ADD A,R3
		INC R0
		MOV @R0,A
		MOV R1,#00
		MOV A,R2
		ADDC A,B
		MOV R2,A
		JNC LAST
		INC R1
LAST:	MOV A,R7
		MOV B,R5
		MUL AB
		ADD A,R2
		INC R0
		MOV @R0,A
		MOV A,B
		ADDC A,R1
		INC R0
		MOV @R0,A
		RET
		ORG 0800H
ADD4:	MOV A,@R0
		ADD A,@R1
		MOV @R0,A
ADD41:	INC R0
		INC R1
		MOV A,@R0
		ADDC A,@R1
		MOV @R0,A
		DJNZ R2,ADD41
		RET
		ORG 0900H
SUB4:	CLR C
		MOV A,@R0
		SUBB A,@R1
		MOV @R0,A
SUB41:	INC R0
		INC R1
		MOV A,@R0
		SUBB A,@R1
		MOV @R0,A
		DJNZ R2,SUB41
		RET
		ORG 0A00H
DIVV:	MOV A,7FH
		JB ACC.7,N
		LCALL DIVV1
		SJMP DRE
N:		LCALL BUM
		LCALL DIVV1
		LCALL BUM
DRE:	RET
		ORG 0B00H
DIVV1:	CLR C
		MOV A,7FH
		RRC A
		MOV 7FH,A
		MOV A,7EH
		RRC A
		MOV 7EH,A
		MOV A,7DH
		RRC A
		MOV 7DH,A
		MOV A,7CH
		RRC A
		MOV 7CH,A
		DJNZ R0,DIVV1
		RET
		ORG 0C00H
BUM:	MOV A,7FH
		CPL A
		MOV 7FH,A
		MOV A,7EH
		CPL A
		MOV 7EH,A
		MOV A,7DH
		CPL A
		MOV 7DH,A
		MOV A,7CH
		CPL A
		MOV 7CH,A
		MOV A,7CH
		ADD A,#01H
		MOV 7CH,A
		MOV A,7DH
		ADDC A,#00H
		MOV 7DH,A
		MOV A,7EH
		ADDC A,#00H
		MOV 7EH,A
		MOV A,7FH
		ADDC A,#00H
		MOV 7FH,A
		RET
		ORG 0D00H
DELAY:	PUSH 02H
DELY1:	PUSH 02H
DELY2:	PUSH 02H
DELY3:	DJNZ 02H,DELY3
		POP 02H
		DJNZ 02H,DELY2
		POP 02H
		DJNZ 02H,DELY1
		POP 02H
		DJNZ 02H,DELAY
		RET
		END
```

## 六、实验分析
通过实验调试，系统实现了以下功能：
- 定时器2产生频率为约 10kHz 的PWM波，占空比由 PID 输出量动态调整；
- 定时器0测量给定信号频率（250kHz），定时器1测量反馈信号频率（5kHz），二者之差作为误差输入；
- PID 程序采用位置式算法，参数 KP=24/1=24，KI=16/5=3.2，KD=0/5=0（实际实验中可调整为非零值以改善动态响应）；
- 加入积分分离（Bang-Bang控制）防止积分饱和，当误差大于阈值时直接输出最大/最小占空比，加速响应。

实验结果表明，电机在空载和加载时均能稳定运行，转速波动小，动态响应快，证明了基于89C52单片机采用定时器2实现PWM和PID算法的闭环调速方案的可行性。
