---
aliases:
  - 串口
tags:
  - 51单片机
  - c
  - 串口
  - 通信
summary: 
create time: 2023-12-09T17:29:00
modify time: 2023-12-10T20:32:00
---
## 硬件电路

### USB转TTL串口

![[Pasted image 20231209201815.png]]

## 电平标准

电平标准是数据1和数据0的表达方式，是传输线缆中认为规定的电压与数据的对应关系。

1. 正常信号：使用相较于GND电压的差值表示数据
2. 差分信号：使用两线电压差表示数据，适用与远距离信号传输

## 接口及引脚的定义

1. 数据接口（线路）
2. 流控制接口（线路）
3. 电源接口（线路）

## 常见通信接口比较

|  名称  |       引脚定义       |   通信方式   |        特点        |
|:------:|:--------------------:|:------------:|:------------------:|
|  UART  |       TXD, RXD       | 全双工，异步 |     点对点通信     |
|  IIC   |       SCL, SDA       | 半双工，同步 |   可挂载多个设备   |
|  SPI   | SCLK, MOSI, MISO, CS | 全双工，同步 |   可挂载多个设备   |
| 1-Wire |          DQ          | 半双工，异步 |   可挂载多个设备   |
|  CAN   |                      |              | 差分信号，汽车使用 | 
|  USB   |                      |              |                    |

## 通信的基本概念

### 串行通信与并行通信

### 异步通信与同步通信

### 单工、半双工与全双工通信

### 通信速率

## 51单片机的UART

### 工作模式1（常用）

8位UART，波特率可变

#### 串口工作原理图（集成单片机内部）

![[Pasted image 20231209203318.png]]

SBUF：串口数据缓存寄存器，读写缓冲寄存器共享同一个地址（进行读写时分别使用不同物理寄存器），实现接受数据和传输数据时产生不同的中断信号，控制对该地址读或写时使用的实际物理寄存器。

### 串口相关中断系统

![[Interrupt System#中断结构]]

### 串口相关寄存器

![[Pasted image 20231209204845.png]]

#### SCON

#### PCON

## 串口通行代码实现

### 串口通行波特率配置代码

![[Pasted image 20231210184825.png]]

### 串口通信驱动代码

#### UART.h

```c
#ifndef __UART_H__
#define __UART_H__

void UART_Init();
void UART_SendByte(unsigned char Byte);

#endif // __UARE_H__
```

#### UART.c

```c
#include <REGX52.H>

/**
 * @brief  串口初始化 4800bps@12.000MHz
 * @param  无
 * @retval 无
 */
void UART_Init() {
// 串口通信控制寄存器SCON
	// 工作模式1：SM0 = 0 SM1 = 1
	// 接受使能 REN = 1
	SCON = 0x50;
// 电源控制寄存器PCON
	// 波特率加倍SMOD = 1
	PCON |= 0x80;		//使能波特率倍速位SMOD = 1	
// 配置定时器
	TMOD &= 0x0F;		//设置timer1模式
	TMOD |= 0x20;		//设置timer1模式为8位自动重装
	TL1 = 0xF3;		//设定定时初值
	TH1 = 0xF3;		//设定定时器重装值
	ET1 = 0;		//禁止定时器1中断
	TR1 = 1;		//启动定时器1
	EA = 1;		// 启动所有中断
	ES = 1;	   	// 启动串口中断
}

/**
 * @brief  串口向上位机发送一个字节的数据
 * @param  需要发送的一个字节数据
 * @retval 无
 */
void UART_SendByte(unsigned char Byte) {
	SBUF = Byte;
	while(TI == 0); // 等待数据传输完成，TI == 1
	TI = 0; // 手动将TI串口数据传输中断标志位重新置0
}


/* 串口接受数据中断函数模板
void UART_Routine() interrupt 4 {
	if (RI == 1) {

		RI = 0; // 手动将RI串口数据接受中断标志位重新置0
	}
}
*/
```

### 单片机向上位机发送数据

```c
#include <REGX52.H>
#include "Delay.h"
#include "UART.h"

unsigned char Sec;

void main() {
	UART_Init();
	while (1) {
		UART_SendByte(Sec);
		++Sec;
		Delay(1000);
	}
}
```

### 上位机向单片机发送数据控制LED

单片机通过接受数据中断信号R1 = 1，处理上位机发送过来的数据即上位机向SBUF写入的数据。

单片机串口接受数据中断号：4

![[Interrupt System#中断程序接口函数API]]

```c
#include <REGX52.H>
#include "Delay.h"
#include "UART.h"

unsigned char Sec;

void main() {
	UART_Init();
	while (1) {
		
	}
}

void UART_Routine() interrupt 4 {
	if (RI == 1) {
		P2 = ~SBUF;
		UART_SendByte(SBUF);
		RI = 0;
	}
}
```

### STC串口助手中数据显示模式

1. HEX模式/十六进制模式/二进制模式：以原始数据的形式显示
2. 文本模式/字符模式：以原始数据编码后的形式显示

