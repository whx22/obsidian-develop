---
aliases: 
tags:
  - 51单片机
  - 直流电机
  - c
summary: 
create time: 2023-12-25T09:33:00
modify time:
---
## Introduction

直流电机是指能将直流电能转换成机械能（直流电动机）或将机械能转换成直流电能（直流发电机）的旋转电机。

它是能实现直流电能和机械能互相转换的电机。

当它作电动机运行时是直流电动机，将电能转换为机械能；

作发电机运行时是直流发电机，将机械能转换为电能。

## 电机驱动电路

### 大功率器件直接驱动

![[Pasted image 20231225101342.png]]

![[Pasted image 20231225101359.png]]

D1：续流二极管，保护电路，消除电路断开瞬间电感产生的高电压（形成回路）

电路中存在感性原件：电路具有升压特性

### H桥驱动

可知控制电机的正反转

![[Pasted image 20231225101332.png]]

## PWM

PWM 是 Pulse Width Modulation 的缩写，中文意思就是脉冲宽度调制，简称脉宽调制。

==惯性系统==中，它是利用微处理器的数字输出来对模拟电路进行控制的一种非常有效的技术，其控制简单、灵活和动态响应好等优点而成为电力电子技术最广泛应用的控制方式，其应用领域包括测量，通信，功率控制与变换，电动机控制、伺服控制、调光、开关电源，甚至某些音频放大器。

其实我们也可以这样理解，PWM 是一种==对模拟信号电平进行数字编码==的方法。通过高分辨率计数器的使用，方波的占空比被调制用来对一个具体模拟信号的电平进行编码。

PWM 信号仍然是数字的，因为在给定的任何时刻，满幅值的直流供电要么完全有(ON)，要么完全无(OFF)。

电压或电流源是以一种通(ON)或断(OFF)的重复脉冲序列被加到模拟负载上去的。

通的时候即是直流供电被加到负载上的时候，断的时候即是供电被断开的时候。

只要带宽足够，任何模拟值都可以使用 PWM 进行编码。

![[Pasted image 20231225103337.png]]

从图中可以看到，上图 a 是一个正弦波即模拟信号，b 是一个数字脉冲波形即数字信号。

我们知道在计算机系统中只能识别是 1 和 0，对于 51 单片机芯片，要么输出高电平（5V），要么输出低电平（0），假如要输出 1.5V 的电压，那么就必须通过相应的处理，比如本章所要讲解的 PWM 输出，其实从上图也可以看到，只要保证数字信号脉宽足够就可以使用 PWM 进行编码，从而输出1.5V 的电压。

$频率 = 1 / T_s$

$占空比 = T_{on} / T_s$

$精度 = 占空比变化的步距$

### 定时器产生PWM方法

![[Pasted image 20231225125659.png]]

## 代码

### LED呼吸灯

```c
#include <REGX52.H>

sbit LED = P2^0;

void Delay(unsigned int t) {
  while (t--); // 此处t--和--t效果不同
}

void main() {
  unsigned char Time, i;
  while (1) {
    for (Time = 0; Time < 100; ++Time) { // 亮度变化(灭到亮)
      for (i = 0; i < 20; ++i) { // 延时
        LED = 0;  // LED亮
        Delay(Time);
        LED = 1;  // LED灭
        Delay(100 - Time);
      }
    }
    for (Time = 100; Time > 0; --Time) { // 亮度变化（亮到灭）
      for (i = 0; i < 20; ++i) { // 延时
        LED = 0; // LED亮
        Delay(Time);
        LED = 1; // LED灭
        Delay(100 - Time);
      }
    }
  }
}
```

### 直流电机调速

```c
#include <REGX52.H>
#include "Delay.h"
#include "Timer0.h"
#include "Nixie.h" // 非中断扫描
#include "Key.h" // 非中断扫描

sbit Motor = P1^0;
sbit LED = P2^0;

unsigned char Counter, Compare, KeyNum, Speed;

void main() {
  Timer0_Init();
  while (1) {
    KeyNum = Key();
    if (KeyNum == 1) {
      ++Speed;
      Speed %= 4;
      if (Speed == 0) { Compare = 0; }
      if (Speed == 1) { Compare = 50; }
      if (Speed == 2) { Compare = 75; }
      if (Speed == 3) { Compare = 100; }
    }
    Nixie(1, Speed);
  }
}

/**
 * @brief 每隔100us进入一次中断处理函数 
 *  
 */
void Timer0_Routine() interrupt 1 {
	TL0 = 0x9C;		//设置定时初值
	TH0 = 0xFF;		//设置定时初值
  ++Counter;
  Counter %= 100;
  if (Counter < Compare) {
    Motor = 1; // 电机工作
  } else {
    Motor = 0; // 电机不工作
  }
}
```