---
aliases:
  - 矩阵键盘
tags:
  - 51单片机
  - c
  - 矩阵键盘
summary: 
create time: 2023-12-04T20:14:00
modify time: 2023-12-05T20:38:00
---
## Introduction

键盘中按键数量较多时，为减少I/O口占用，将键盘排列成矩阵形式。采用逐行或逐列的“扫描”，可以读出任意按键的状态。

![[Pasted image 20231205173550.png]]

对比独立按键

![[STC89C52 LED 独体按键#^key-diagram]]

## 扫描

### 输出扫描

数码管扫描：为实现多个数码管同时显示的效果，使用输出扫描（快速换位输出）和人眼视觉停留。 

### 输入扫描

矩阵键盘：为实现所有按键同时检测的效果，使用按行或按列快速输入扫描和人类反应力。

## 准双向口输出配置

弱上拉模型，I/O端口读取数据时，为低电平有效。

准双向口输出类型可用作输出和输入功能而不需重新配置口线输出状态。这是因为当口线
输出为1时驱动能力很弱，允许外部装置将其拉低。当引脚输出为低时，它的驱动能力很强，
可吸收相当大的电流。准双向口有3个上拉晶体管适应不同的需要。

## 输入扫描矩阵键盘

### MatrixKey.h

```c
#ifndef __MATRIXKEY_H__
#define __MATRIXKEY_H__

unsigned char MatrixKey();

#endif // __MATRIXKEY_H__
```

### MatrixKey.c

```c
#include <REGX52.H>
#include "Delay.h"

/**
 * @brief 矩阵键盘读取按键键码
 * @param 无
 * @retval KeyNumber: 按下按键的键码值
 * @description: 如果按键保持按下状态，程序在循环中，松手后，返回按键键码值， \
 *               没有按键按下时返回0，按列扫描
 */
unsigned char MatrixKey() {
	unsigned char KeyNumber = 0;

	P1 = 0xFF;
	P1_3 = 0;
	if (P1_7 == 0) {Delay(20); while(P1_7 == 0); Delay(20); KeyNumber = 1;}
	if (P1_6 == 0) {Delay(20); while(P1_6 == 0); Delay(20); KeyNumber = 5;}
	if (P1_5 == 0) {Delay(20); while(P1_5 == 0); Delay(20); KeyNumber = 9;}
	if (P1_4 == 0) {Delay(20); while(P1_4 == 0); Delay(20); KeyNumber = 13;}

	P1 = 0xFF;
	P1_2 = 0;
	if (P1_7 == 0) {Delay(20); while(P1_7 == 0); Delay(20); KeyNumber = 2;}
	if (P1_6 == 0) {Delay(20); while(P1_6 == 0); Delay(20); KeyNumber = 6;}
	if (P1_5 == 0) {Delay(20); while(P1_5 == 0); Delay(20); KeyNumber = 10;}
	if (P1_4 == 0) {Delay(20); while(P1_4 == 0); Delay(20); KeyNumber = 14;}

	P1 = 0xFF;
	P1_1 = 0;
	if (P1_7 == 0) {Delay(20); while(P1_7 == 0); Delay(20); KeyNumber = 3;}
	if (P1_6 == 0) {Delay(20); while(P1_6 == 0); Delay(20); KeyNumber = 7;}
	if (P1_5 == 0) {Delay(20); while(P1_5 == 0); Delay(20); KeyNumber = 11;}
	if (P1_4 == 0) {Delay(20); while(P1_4 == 0); Delay(20); KeyNumber = 15;}

	P1 = 0xFF;
	P1_0 = 0;
	if (P1_7 == 0) {Delay(20); while(P1_7 == 0); Delay(20); KeyNumber = 4;}
	if (P1_6 == 0) {Delay(20); while(P1_6 == 0); Delay(20); KeyNumber = 8;}
	if (P1_5 == 0) {Delay(20); while(P1_5 == 0); Delay(20); KeyNumber = 12;}
	if (P1_4 == 0) {Delay(20); while(P1_4 == 0); Delay(20); KeyNumber = 16;}

	return KeyNumber;
}
```

### main.c

```c
#include <REGX52.H>
#include "Delay.h"
#include "LCD1602.h"
#include "MatrixKey.h"

unsigned char KeyNum;

void main() {
	LCD_Init();
	LCD_ShowString(1, 1, "MatrixKey:");
	while (1) {
		KeyNum = MatrixKey();
		if (KeyNum) { // 如果不加判断，LCD显示对应按键数值后，立即被置零
			LCD_ShowNum(2, 1, KeyNum, 2);
		}
	}
}
```

程序执行过程：

如果有检测到按下键盘，键码值不为0，LCD显示该键码值

如果没有检测到按下键盘，键码值为0，LCD显示不被触发

## 矩阵键盘密码锁

### main.c

```c
#include <REGX52.H>
#include "Delay.h"
#include "LCD1602.h"
#include "MatrixKey.h"

unsigned char KeyNum;

/**
 * 按键S1-S10：数字1-9、0
 * 按键S11：确认密码键，并对密码进行正确性判断
 * 按键S12：清空输入键
 */
#define PASSWORD 2345
unsigned char KeyNum;	// 按键输入
unsigned int password;	// 输入密码
unsigned int count;		// 输入密码位数
void main() {
	LCD_Init();
	LCD_ShowString(1, 1, "Password:");
	while (1) {
		KeyNum = MatrixKey();
		// 存在按键按下
		if (KeyNum) {
			// S1-S10
			if (KeyNum <= 10) {
				if (count < 4) {
					password *= 10;
					password += KeyNum % 10;
					++count;
				}
				LCD_ShowNum(2, 1, password, 4);
			}
			// S11
			if (KeyNum == 11) {
				if (password == PASSWORD) {
					LCD_ShowString(1, 14, "OK ");
				} else {
					LCD_ShowString(1, 14, "ERR");
				}
				password = 0;
				count = 0;
				LCD_ShowNum(2, 1, password, 4);
			}
			// S12
			if (KeyNum == 12) {
				password = 0;
				count = 0;
				LCD_ShowNum(2, 1, password, 4);
			}
		}
	}
}
```