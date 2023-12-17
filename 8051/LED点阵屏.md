---
aliases: 
tags:
  - 51单片机
  - LED点阵屏
summary: 
create time: 2023-12-10T20:32:00
modify time: 2023-12-17T13:05:00
---
## 显示原理

### 8 * 8 LED 点阵模块

![[Pasted image 20231216182406.png]]

### 74HC595

![[Pasted image 20231216195404.png]]

![[Pasted image 20231216202202.png]]

1. OE：output enable 输入使能
2. RCLK：register clock 寄存器时钟：用与上升沿锁存
3. SRCLR：serial clear 串行清零
4. SRCLK：serial clock 串行时钟，用于上升沿移位
5. SER：serial register 串行数据 ，用于串行输入
6. QH：用于多片级联

[74HC595芯片中文资料](./attachments/74HC595芯片中文资料)

实现串行输入，并行输出。

## 8051 sfr and sbit

### sfr(special function register)

特殊功能寄存器声明，使用==寄存器变量名==代替==寄存器在寄存器堆中的编号==。

### sbit(special bit)

特殊位声明，使用==位变量名==代替==可位寻址寄存器在寄存器堆中的一个位==。

## code keyword

将动画数组存放到存放代码的flash中，默认数据存放置EEPROM中

### EEPROM

数据段存储器，存放全局变量，静态变量，存储空间==小==，初始化后可以修改，==可读可写==

### flash

代码段存储器，存放指令代码，存储空间==大==，初始化后不可以修改，==只读==

## 代码实现

### LED点阵屏显示图形

```c
#include <REGX52.H>
#include "Delay.h"

sbit RCK = P3^5; // RCLK
sbit SCK = P3^6; // SRCLK
sbit SER = P3^4; // SER

#define MATRIX_LED_PORT P0

/**
 * @brief  74HC595写入一个字节
 * @param  需要写入的字节
 * @retval 无
 */
void _74HC595_WriteByte(unsigned char Byte) {
	unsigned char i;
	for (i = 0; i < 8; ++i) {
		SER = Byte & (0x80 >> i);
		SCK = 1;
		SCK = 0;
	}
	RCK = 1;
	RCK = 0;
}

/**
 * @brief  LED点阵屏显示一列数据
 * @param  需要显示数据的列（范围0 - 7）
 * @param  需要显示的数据 （高位在上，1为亮，0为灭）
 * @retval 无
 */
void MatrixLED_ShowColumn(unsigned char Column,Data) {
	_74HC595_WriteByte(Data);
	MATRIX_LED_PORT = ~(0x80 >> Column);
	Delay(1); // 延时显示
	MATRIX_LED_PORT = 0xFF; // 位清零
}

void main() {
	SCK = 0; // 初始化上升沿移位
	RCK = 0; // 初始化上升沿锁存
	while (1) {
		MatrixLED_ShowColumn(0, 0x3C);
		MatrixLED_ShowColumn(1, 0x42);
		MatrixLED_ShowColumn(2, 0xA9);
		MatrixLED_ShowColumn(3, 0x85);
		MatrixLED_ShowColumn(4, 0x85);
		MatrixLED_ShowColumn(5, 0xA9);
		MatrixLED_ShowColumn(6, 0x42);
		MatrixLED_ShowColumn(7, 0x3C);
	}
}
```

### LED点阵屏显示动画

#### MatrixLED.h

```c
#ifndef __MATRIXLED_H__
#define __MATRIXLED_H__

void MatrixLED_Init();
void MatrixLED_ShowColumn(unsigned char Column,Data);

#endif // __MATRIXLED_H__
```

#### MatrixLED.c

```c
#include <REGX52.H>
#include "Delay.h"

sbit RCK = P3^5; // RCLK
sbit SCK = P3^6; // SRCLK
sbit SER = P3^4; // SER
#define MATRIX_LED_PORT P0

/**
 * @brief  74HC595写入一个字节
 * @param  需要写入的字节
 * @retval 无
 */
void _74HC595_WriteByte(unsigned char Byte) {
	unsigned char i;
	for (i = 0; i < 8; ++i) {
		SER = Byte & (0x80 >> i);
		SCK = 1;
		SCK = 0;
	}
	RCK = 1;
	RCK = 0;
}

/**
 * @brief  LED点阵屏初始化
 * @param  无
 * @retval 无
 */
void MatrixLED_Init() {
	SCK = 0; // 初始化上升沿移位
	RCK = 0; // 初始化上升沿锁存
}

/**
 * @brief  显示一列数据
 * @param  需要显示数据的列（范围0 - 7）
 * @param  需要显示的数据 （高位在上，1为亮，0为灭）
 * @retval 无
 */
void MatrixLED_ShowColumn(unsigned char Column,Data) {
	_74HC595_WriteByte(Data);
	MATRIX_LED_PORT = ~(0x80 >> Column);
	Delay(1); // 延时显示
	MATRIX_LED_PORT = 0xFF; // 位清零
}
```

#### LED点阵屏显示动画

```c
#include <REGX52.H>
#include "Delay.h"
#include "MatrixLED.h"

// code keywork : 将动画数组存放到存放代码的flash中，默认数据存放置EEPROM中
// EEPROM : 数据段存储器，存放全局变量，静态变量，存储空间小，初始化后不可以修改，只读
// flash : 代码段存储器，存放指令代码，存储空间大，初始化后可以修改，可读可写
unsigned char code Animation_1[] = {
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0xFF,0x08,0x08,0x08,0xFF,0x00,0x0E,0x15,0x15,0x15,0x0C,0x00,0xFE,0x01,0x02,0x00,
0xFE,0x01,0x02,0x00,0x0E,0x11,0x11,0x11,0x0E,0x00,0xFD,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
};

unsigned char code Animation_2[] = {
0x3C,0x42,0xA9,0x85,0x85,0xA9,0x42,0x3C,
0x3C,0x42,0xA5,0x85,0x85,0xA5,0x42,0x3C,
0x3C,0x42,0xA5,0x89,0x89,0xA5,0x42,0x3C,
};

void main() {
	unsigned char i = 0, offset = 0, count = 0;
	MatrixLED_Init();
	while (1) {
		for (i = 0; i < 8; ++i) {
			MatrixLED_ShowColumn(i, Animation_2[i + offset]);	
		}
		++count;
		if (count > 10) { // 扫描十遍，利用视觉他停留
			count = 0;
			offset += 8; // 流动字幕偏移为1，逐帧动画偏移为8
			if (offset > 16) { // 显示最后的图像
				offset = 0;
			}
		}
	}
}
```

#### 取字模软件设置

1. 设置图像宽高
2. 设置图像为纵向取模