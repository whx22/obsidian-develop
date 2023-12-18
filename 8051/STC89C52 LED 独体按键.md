---
author: whx
title: STC89C52 LED 独体按键
time: 2023-11-21-Tuesday
tags:
  - mcu
  - 51单片机
  - LED
  - 独立按键
---
## STC89C52 介绍

[STC89Cxx中文参考手册](./attachments/STC89Cxx中文参考手册)

[普中-2&普中-3&普中-4开发板原理图](./attachments/普中-2&普中-3&普中-4开发板原理图)

初始化状态 ：端口（MCU内部寄存器）为高电平

通过控制MCU内部寄存器的值，实现控制外部IO设备（通过总线连接，驱动器（电路放大器））

### ISP(in-system programming)

在系统编程，在系统烧录。

过去：烧录可执行目标文件到单片机内部的flash中，需要使用专用的烧录器，从开发板取下单片机，放入专用的烧录器使用专用接口，固定单片机，进行烧录，烧录结束后，将单片机从新安装到开发板上，运行程序。

ISP：在系统上（开发板系统）直接烧录程序。PC机通过串口把可执行目标文件直接ISP到单片机内部flash中。

### 单片机工作过程

1. 编写代码
2. 编译生成可执行文件
3. 烧录（下载到单片机MCU内部的flash中（EEPROM））
4. 启动单片机，执行程序
## LED模块

![[Pasted image 20231122202021.png]]

高电平灭，低电平亮（原因：高电平（电压电流限制）驱动能力有限，低电平驱动能力强）

### 点亮LED

```c
#include <REGX52.H>

void main() {
	P2 = 0xFE; // 1111 1110
	while (1) {
	}
}
```

### LED闪烁

```c
#include <REGX52.H>
#include <INTRINS.H>

void Delay500ms() //@12.000MHz
{
	unsigned char i, j, k;

	_nop_(); // 空指令，头文件INTRINS.H
	i = 4;
	j = 205;
	k = 187;
	do
	{
		do
		{
			while (--k);
		} while (--j);
	} while (--i);
}

void main() {
	while (1) {
		P2 = 0xFE;
		Delay500ms();
		P2 = 0xFF;
		Delay500ms();
	}
}
```

### LED流水灯

```c
#include <REGX52.H>
#include <INTRINS.H>

// STC-ISP生成
void Delay500ms() //@12.000MHz
{
	unsigned char i, j, k;

	_nop_(); // 空指令，头文件INTRINS.H
	i = 4;
	j = 205;
	k = 187;
	do
	{
		do
		{
			while (--k);
		} while (--j);
	} while (--i);
}

// 接受参数，控制延时时间
void Delay1ms(unsigned int xms) //@12.000MHz
{
	unsigned char i, j;
	while (xms) {
		i = 2;
		j = 239;
		do {
			while (--j);
		} while (--i);
		xms--;
	}
}

void main() {
	while (1) {
		P2 = 0xFE; // 1111 1110
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xFD; // 1111 1101
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xFB; // 1111 1011
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xF7; // 1111 0111
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xEF; // 1110 1111
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xDF; // 1101 1111
		Delay500ms(); // or Delay1ms(500);
		P2 = 0xBF; // 1011 1111
		Delay500ms(); // or Delay1ms(500);
		P2 = 0x7F; // 0111 1111
		Delay500ms(); // or Delay1ms(500);
	}
}
```

## 独立按键

端口：P3

1. 按键没有按下：高电平（1）
2. 按键按下：低电平（0）

![[Pasted image 20231122180215.png]] ^key-diagram

### 独立按键控制LED亮灭

```c
#include <REGX52.H>

void main() {
	while (1) {
		if (P3_1 == 0) {
			P2_0 = 0;
		} else {
			P2_0 = 1;
		}
	}
}
```

### 按键的抖动

机械开关，在机械触点断开、闭合时，由于机械触点的弹性作用，产生的数字信号抖动。

![[Pasted image 20231121201524.png]]

### 解决方案

#### 硬件消抖

#### 软件消抖

延迟函数

### 独立按键控制LED状态

```c
#include <REGX52.H>

void Delay(unsigned int xms) {
	unsigned char i, j;
	while (xms--) {
		i = 2;
		j = 239;
		do {
			while (--j);
		} while (--i);
	}
}

void main() {
	while (1) {
		if (P3_1 == 0) {
			Delay(20); // 按下：按键按下瞬间消抖
			while (P3_1 == 0); // 检测按键处于按下状态
			Delay(20); // 松开：按键会弹瞬间消抖

			P2_0 = ~P2_0; // LED状态取反
		}
	}
}
```

### 独立按键控制LED显示二进制

```c
#include <REGX52.H>

void Delay(unsigned int xms) {
	unsigned char i, j;
	while (xms--) {
		i = 2;
		j = 239;
		do {
			while (--j);
		} while (--i);
	}
}

void main() {
	unsigned char LEDNum = 0;
	while (1) {
		if (P3_1 == 0) {
			Delay(20);
			while (P3_1 == 0);
			Delay(20);
			
			// P2 上电初始化 1111 1111 LED全灭
			
			LEDNum++;
			P2 = ~LEDNum;
		}
	}
}
```

### 独立按键控制LED移位

```c
#include <REGX52.H>

void Delay(unsigned int xms);

unsigned char LEDNum;

void main() {
	while (1) {
		if (P3_1 == 0) {
			Delay(20);
			while (P3_1 == 0);
			Delay(20);

			LEDNum++;
			if (LEDNum >= 8) {
				LEDNum = 0;
			}
			P2 = ~(0x01 << LEDNum);
		}
		if (P3_0 == 0) {
			Delay(20);
			while (P3_0 == 0);
			Delay(20);

			if (LEDNum == 0) {
				LEDNum = 7;
			} else {
				LEDNum--;
			}
			P2 = ~(0x01 << LEDNum);
		}
	}
}

void Delay(unsigned int xms) {
	unsigned char i, j;
	while (xms--) {
		i = 2;
		j = 239;
		do {
			while (--j);
		} while (--i);
	}
}
```