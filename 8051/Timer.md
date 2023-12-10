---
aliases:
  - 定时器
tags:
  - 定时器
  - 51单片机
  - c
summary: 
create time: 2023-12-05T20:38:00
modify time:
---
## Introduction

CPU执行Delay函数时，CPU被阻塞，无法进行其他任务。

### location

51单片机的定时器属于单片机的内部资源，其电路的连接和运转均在单片机内部完成。

### feature

1. 用于计时系统，可实现软件计时，或者使程序间隔一个固定时间完成一项操作
2. 替代长时间的Delay，提高CPU的运行效率和处理速度
3. 多进程操作系统，时间片轮转
4. ...

### STC89C52定时器资源

定时器个数：3（T0、T1、T2），T0、T1兼容传统的C51单片机，T2是此型号增加的资源

## 定时器/计数器工作模式(与传统8051单片机兼容)

STC98C52的T0和T1的四种工作模式：

1. 模式0(13位定时器/计数器)
2. 模式1(16位定时器/计数器)（常用）
3. 模式2(8位自动重装模式)
4. 模式3(两个8位计数器)

### 模式1(16位定时器/计数器)

![[Pasted image 20231208161047.png]]

连接的相关中断系统

![[Interrupt System#中断结构]]

#### 脉冲信号产生来源

1. （T：timer）SYSclk(system clock)：系统时钟，即晶振周期，本开发板的晶振为12MHz。本开发板默认对系统时钟进行12T分频，分频后为1MHz，晶振周期为1us
2. （C：counter）T0 Pin：外部引脚

### 中断系统

[[Interrupt System]]

## 定时器/计数器0和1的相关寄存器

![[Pasted image 20231208164751.png]]

### TCON：定时器/计数器控制寄存器

### TMOD：定时器/计数器工作模式寄存器

### TL0、TL1、TH0、TL1：存储定时器/计数器内容


![[Interrupt System#^interrupt-register]]


## 使用中断程序代码

### 定时器/计数器0初始化函数

#### 详细注释代码

```c
void Timer0Init() {
// 设置控制寄存器TMOD TCON
	// TMOD 不可位寻址
	// GATE = 0; 不开启外部引脚控制
	// C/~T = 0; 工作在计时器模式
	// M1 = 0; M0 = 1; 工作在模式1 
	// TMOD = 0x01; 影响timer1设置
	TMOD &= 0xF0; // 保留高4位，清零低4位
	TMOD |= 0x01; // 设置低4位（timer0）

	// TCON 可位寻址
	// 中断溢出标志位初始化为0
	TF0 = 0;
	// 开启timer_0
	TR0 = 1;

// 设置通用寄存器TH0 TH0
// 脉冲频率：1MHz，计时器+1速度：1us
// TH0:TH1(16bits)：maximum = 65535，initial value = 64535
// 设置为每隔1ms触发一次中断
	TH0 = 64535 / 256;	// 取64535高位
	TL0 = 64535 % 256 + 1; 	// 取64535低位，65536时溢出

// 设置中断寄存器
	// 开启timer_0中断
	ET0 = 1;
	// 开启所有中断
	EA = 1;
	// 设置timer_0优先级为0
	PT0 = 0;
}

```

#### STC官方定时器/计数器代码

```c
void Timer0Init(void)		//1毫秒@12.000MHz
{
// 	AUXR &= 0x7F;		//定时器时钟12T模式 新版本代码
	TMOD &= 0xF0;		//设置定时器模式
	TMOD |= 0x01;		//设置定时器模式
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	TF0 = 0;		//清除TF0标志
	TR0 = 1;		//定时器0开始计时
	ET0 = 1; 	// 开启timer_0中断
	EA = 1; 	// 开启所有中断
	PT0 = 0; 	// 设置timer_0优先级为0
}
```

### 定时器/计数器0中断服务程序

- ! 中断服务程序不能执行过于复杂的任务，不可长时间处于中断函数内部。

```c
// timer0 中断服务程序模板（中断号：1）
void Timer0_Routine() interrupt 1 {
	static unsigned int T0Count;
	// 设施T0Count每个1ms加1
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	++T0Count;
	if (T0Count >= 1000) {
		T0Count = 0;
		// 中断服务程序功能实现代码
	}	
}
```

### 按键控制LED流水灯模式

```c
#include <REGX52.H>
#include "Timer0.h"
#include "Key.h"
#include <INTRINS.H>

unsigned char KeyNum, LEDMode;

void main() {
    P2 = 0xFE;
    Timer0Init();
    while (1) {
        KeyNum = Key();
        if (KeyNum) {
            if (KeyNum == 1) {
                ++LEDMode;
                if (LEDMode >= 2) {LEDMode = 0;}
            }
        }
    }
}

void Timer0_Routine() interrupt 1 {
	static unsigned int T0Count;
	// 设施T0Count每个1ms加1
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	++T0Count;
	if (T0Count >= 500) {
		T0Count = 0;
		if (LEDMode == 0) {
			P2 = _crol_(P2, 1);
		}
		if (LEDMode == 1) {
			P2 = _cror_(P2, 1);
		}
	}	
}
```

### 定时器时钟

```c
#include <REGX52.H>
#include "Delay.h"
#include "LCD1602.h"
#include "Timer0.h"

unsigned char Sec = 55, Min = 59, Hour = 23;

void main() {
	LCD_Init();
	Timer0Init();

	LCD_ShowString(1, 1, "Clock:");
	LCD_ShowString(2, 1, "  :  :");
	while (1) {
		LCD_ShowNum(2, 1, Hour, 2);
		LCD_ShowNum(2, 4, Min, 2);
		LCD_ShowNum(2, 7, Sec, 2);
	}
}

// timer0 中断服务程序模板（中断号：1）
void Timer0_Routine() interrupt 1 {
	static unsigned int T0Count;
	// 设施T0Count每个1ms加1
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	++T0Count;
	if (T0Count >= 1000) {
		T0Count = 0;
		++Sec;
		if (Sec >= 60) {
			Sec = 0;
			++Min;
			if (Min >= 60) {
				Min = 0;
				++Hour;
				if (Hour >= 24) {
					Hour = 0;
				}
			}
		}
	}	
}
```
