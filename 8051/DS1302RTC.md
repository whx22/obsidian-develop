---
aliases:
  - DS1302实时时钟
tags:
  - DS1302
  - 51单片机
  - c
summary: 
create time: 2023-12-17T13:05:00
modify time:
---
## 原理

![[Pasted image 20231217131505.png]]

### 单片机定时器时钟对比DS1302实时时钟

1. DS1302精度更高
2. 定义器时钟占用单片机CPU时间，CPU不断进入定时器中断处理函数执行
3. 定时器时钟掉电无法运行，DS1302实时时钟掉电仍然可以运行（使用备用电池）

### RTC(Real Time Clock)

实时时钟，是一种集成电路，通常称为时钟芯片。

### 引脚定义

![[Pasted image 20231217132958.png]]

### 内部框图

![[Pasted image 20231217134952.png]]

### 内部寄存器定义

#### RTC相关寄存器

![[Pasted image 20231217135236.png]]

#### 时钟脉冲串寄存器，通用RAM寄存器，RAM脉冲串寄存器

![[Pasted image 20231217135954.png]]

![[Pasted image 20231217140004.png]]

![[Pasted image 20231217140034.png]]
![[Pasted image 20231217140047.png]]

### 命令字

![[Pasted image 20231217140214.png]]

### 单片机与DS1302芯片通信时序定义

![[Pasted image 20231217140734.png]]

1. SCLK时钟上升沿：单片机向DS1302寄存器写入数据，DS1302读出单片机寄存器数据
2. SCLK时钟下降沿：单片机从DS1302寄存器读出数据，DS1302写入单片机寄存器数据

## BCD

BCD(Binary Coded Decimal)，用4位二进制数表示1位十进制制数。

DS1302中RTC相关寄存器使用BCD编码

### BCD码转十进制（两位BCD码）

`DEC = BCD / 16 * 10 + BCD % 16;`

### 十进制转BCD码（两位BCD码）

`BCD = DEC / 10 * 16 + DEC % 10;`

## 代码

### DS1302.h

```c
#ifndef __DS1302_H__
#define __DS1302_H__

extern char DS1302_Time[];

void DS1302_Init(void);
void DS1302_WriteByte(unsigned char Command, Data);
unsigned char DS1302_ReadByte(unsigned char Command);
void DS1302_SetTime(void);
void DS1302_ReadTime(void);

#endif // __DS1302_H__
```

### DS1302.c

```c
#include <REGX52.H>

// 位声明：单片机与DS1302芯片连接的端口定义
sbit DS1302_SCLK = P3^6;
sbit DS1302_IO = P3^4;
sbit DS1302_CE = P3^5;

// 存放时间数据的数组（单片机数据区，EEPROM）
char DS1302_Time[] = {
	23, // year
	12, // month
	17,	// date
	16,	// hour
	59,	// minute
	55,	// second
	6	// day
};

// DS1302写入命令字地址
#define DS1302_SECOND 	0x80
#define DS1302_MINUTE 	0x82
#define DS1302_HOUR		0x84 	
#define DS1302_DATE		0x86
#define DS1302_MONTH	0x88
#define DS1302_DAY		0x8A
#define DS1302_YEAR		0x8C
#define DS1302_WP 		0x8E

/**
 * @brief	DS1302芯片初始化
 * @param 	无
 * @retval	无
 */
void DS1302_Init(void) {
	DS1302_CE = 0;
	DS1302_SCLK = 0;  
}

/**
 * @brief 	单片机向DS1302芯片写入数据 
 * @param1 	写入命令字，指定向DS1302芯片写入的寄存器号
 * @param2	写入数据
 * @retval	无
 * @note 	DS1302_SCLK 产生16个脉冲信号，
 			16个上升沿信号
 */
void DS1302_WriteByte(unsigned char Command, Data) {
	unsigned char i = 0;
// DS1302芯片开始数据接受，共计16个脉冲
	DS1302_CE = 1;

// DS1302接受命令字
	for (i = 0; i < 8; ++i) {
		DS1302_IO = Command & (0x01 << i);
		// 触发一次SCLK上升沿，
		DS1302_SCLK = 1;
		DS1302_SCLK = 0;
	}
// DS1302接受数据
	for (i = 0; i < 8; ++i) {
		DS1302_IO = Data & (0x01 << i);
		// 触发一次SCLK上升沿，
		DS1302_SCLK = 1;
		DS1302_SCLK = 0;
	}
// DS1302芯片结束数据接受
	DS1302_CE = 0;
}

/**
 * @brief	单片机从DS1302芯片读出数据
 * @param	读出命令字，指定向DS1302芯片读出的寄存器号
 * @retval	读出的数据
 * @note 	DS1302_SCLK 产生15个脉冲信号，
 			8个上升沿信号，8个下降沿信号，
			其中存在一个上升沿信号和下降沿信号共用一个脉冲信号
 */
unsigned char DS1302_ReadByte(unsigned char Command) {
	unsigned char i = 0, Data = 0x00;

	Command |= 0x01; // DS1302写入命令字地址转化为读出命令字地址
// DS1302芯片开始数据输出，共计15个脉冲
	DS1302_CE = 1;

// DS1302接受命令字
	for (i = 0; i < 8; ++i) {
		DS1302_IO = Command & (0x01 << i);
		// 触发一次SCLK上升沿，
		DS1302_SCLK = 0;
		DS1302_SCLK = 1;
	}
// DS1302输出数据
	for (i = 0; i < 8; ++i) {
		DS1302_SCLK = 1;
		DS1302_SCLK = 0;
		if (DS1302_IO) {
			Data |= (0x01 << i);
		}
	}
// DS1302芯片结束数据输出
	DS1302_CE = 0;
	DS1302_IO = 0;
	return Data;
}

/**
 * @brief	使用单片机中的时间数组设置DS1302芯片存储的时间
 * @param	无
 * @retval	无
 * @note	DS1302中时间使用BCD编码，
 			十进制(DEC)编码转为BCD编码
			BCD = DEC / 10 * 16 + DEC % 10;

 */
void DS1302_SetTime(void) {
	// 关闭写保护
	DS1302_WriteByte(DS1302_WP, 0x00);
	DS1302_WriteByte(DS1302_YEAR, DS1302_Time[0] / 10 * 16 + DS1302_Time[0] % 10);
	DS1302_WriteByte(DS1302_MONTH, DS1302_Time[1] / 10 * 16 + DS1302_Time[1] % 10);
	DS1302_WriteByte(DS1302_DATE, DS1302_Time[2] / 10 * 16 + DS1302_Time[2] % 10);
	DS1302_WriteByte(DS1302_HOUR, DS1302_Time[3] / 10 * 16 + DS1302_Time[3] % 10);
	DS1302_WriteByte(DS1302_MINUTE, DS1302_Time[4] / 10 * 16 + DS1302_Time[4] % 10);
	DS1302_WriteByte(DS1302_SECOND, DS1302_Time[5] / 10 * 16 + DS1302_Time[5] % 10);
	DS1302_WriteByte(DS1302_DAY, DS1302_Time[6] / 10 * 16 + DS1302_Time[6] % 10);
	// 打开写保护
	DS1302_WriteByte(DS1302_WP, 0x80);
}

/**
 * @brief  	读取DS1302中存储的时间并设置到单片机中时间数组中	
 * @param  	无
 * @retval	无
 * @note	DS1302中时间使用BCD编码，
 			BCD编码转为十进制(DEC)编码
			DEC = BCD / 16 * 10 + BCD % 16;
 */
void DS1302_ReadTime(void) {
	unsigned char Temp = 0;
	Temp = DS1302_ReadByte(DS1302_YEAR);
	DS1302_Time[0] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_MONTH);
	DS1302_Time[1] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_DATE);
	DS1302_Time[2] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_HOUR);
	DS1302_Time[3] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_MINUTE);
	DS1302_Time[4] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_SECOND);
	DS1302_Time[5] = Temp / 16 * 10 + Temp % 16;
	Temp = DS1302_ReadByte(DS1302_DAY);
	DS1302_Time[6] = Temp / 16 * 10 + Temp % 16;
}
```

### DS1302时钟

```c
#include <REGX52.H>
#include "LCD1602.h"
#include "DS1302.h"

void main() {
	LCD_Init();
	DS1302_Init();
	LCD_ShowString(1, 1, "  -  -  ");
	LCD_ShowString(2, 1, "  :  :  ");	

//	DS1302_WriteByte(0x8E, 0x00); // 解除DS1302写保护
	
	DS1302_SetTime();
	while (1) {
		DS1302_ReadTime();
		LCD_ShowNum(1, 1, DS1302_Time[0], 2);
		LCD_ShowNum(1, 4, DS1302_Time[1], 2);
		LCD_ShowNum(1, 7, DS1302_Time[2], 2);
		LCD_ShowNum(2, 1, DS1302_Time[3], 2);
		LCD_ShowNum(2, 4, DS1302_Time[4], 2);
		LCD_ShowNum(2, 7, DS1302_Time[5], 2);
	}
}
```

### DS1302可调时钟

```c
#include <REGX52.H>
#include "LCD1602.h"
#include "DS1302.h"
#include "Key.h"
#include "Timer0.h"

unsigned char KeyNum, MODE, TimeSetSelect, TimeSetFlashFlag;

/**
 * @brief 	对于给定的年、月，返回其对应的天数
 * @param1 	年
 * @param2	月
 * @retval	其对应的天数
 */
char days_of_month(char year, month) {
	char days_arrays[12] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
	if (year % 400 == 0 || (year % 4 == 0 && year % 100 != 0)) {
		days_arrays[1] = 29;
	}
	return days_arrays[month - 1];
}

/**
 * @breif 	显示模式工作函数
 			读取DS1302中的日期数据到单片机数据区（EEPROM）
 			并使用LCD1602显示日期数据
 * @param	无
 * @retval 	无
 */
void TimeShow(void) {
	DS1302_ReadTime();
	LCD_ShowNum(1, 1, DS1302_Time[0], 2);
	LCD_ShowNum(1, 4, DS1302_Time[1], 2);
	LCD_ShowNum(1, 7, DS1302_Time[2], 2);
	LCD_ShowNum(2, 1, DS1302_Time[3], 2);
	LCD_ShowNum(2, 4, DS1302_Time[4], 2);
	LCD_ShowNum(2, 7, DS1302_Time[5], 2);
}

/**
 * @breif 	设置模式工作函数
 			设置单片机数据区（EEPROM）存储的日期
 * @param	无
 * @retval 	无
 * @note 	设置结束后，将单片机数据区（EEPROM）存储的日期存储到DS1302
 */
void TimeSet(void) {
	if (KeyNum == 2) { // 选择设置位
		++TimeSetSelect;
		TimeSetSelect %= 6;
	}
	if (KeyNum == 3) { // 设置对应数字增加
		++DS1302_Time[TimeSetSelect];
		if (DS1302_Time[0] > 99) {DS1302_Time[0] = 0;}
		if (DS1302_Time[1] > 12) {DS1302_Time[1] = 1;}
		if (DS1302_Time[2] > days_of_month(DS1302_Time[0], DS1302_Time[1])) {DS1302_Time[2] = 1;}
		if (DS1302_Time[3] > 23) {DS1302_Time[3] = 0;}
		if (DS1302_Time[4] > 59) {DS1302_Time[4] = 0;}
		if (DS1302_Time[5] > 59) {DS1302_Time[5] = 0;}
	}
	if (KeyNum == 4) { // 设置对应数字减小
		--DS1302_Time[TimeSetSelect];
		if (DS1302_Time[0] < 0) {DS1302_Time[0] = 99;}
		if (DS1302_Time[1] < 1) {DS1302_Time[1] = 12;}
		if (DS1302_Time[2] < 1) {DS1302_Time[2] = days_of_month(DS1302_Time[0], DS1302_Time[1]);}
		if (DS1302_Time[3] < 0) {DS1302_Time[3] = 23;}
		if (DS1302_Time[4] < 0) {DS1302_Time[4] = 59;}
		if (DS1302_Time[5] < 0) {DS1302_Time[5] = 59;}
	}
	// 实现设置模式选中闪烁
	if (TimeSetSelect == 0 && TimeSetFlashFlag == 1) {
		LCD_ShowString(1, 1, "  ");
	} else {
		LCD_ShowNum(1, 1, DS1302_Time[0], 2);
	}
	if (TimeSetSelect == 1 && TimeSetFlashFlag == 1) {
		LCD_ShowString(1, 4, "  ");
	} else {
		LCD_ShowNum(1, 4, DS1302_Time[1], 2);
	}
	if (TimeSetSelect == 2 && TimeSetFlashFlag == 1) {
		LCD_ShowString(1, 7, "  ");
	} else {
		LCD_ShowNum(1, 7, DS1302_Time[2], 2);
	}
	if (TimeSetSelect == 3 && TimeSetFlashFlag == 1) {
		LCD_ShowString(2, 1, "  ");
	} else {
		LCD_ShowNum(2, 1, DS1302_Time[3], 2);
	}
	if (TimeSetSelect == 4 && TimeSetFlashFlag == 1) {
		LCD_ShowString(2, 4, "  ");
	} else {
		LCD_ShowNum(2, 4, DS1302_Time[4], 2);
	}
	if (TimeSetSelect == 5 && TimeSetFlashFlag == 1) {
		LCD_ShowString(2, 7, "  ");
	} else {
		LCD_ShowNum(2, 7, DS1302_Time[5], 2);
	}
	//LCD_ShowNum(2, 10, TimeSetSelect, 2); // (debug)显示当前设置位
	//LCD_ShowNum(2, 13, TimeSetFlashFlag, 2); // (debug)显示当前闪烁标志 
}

void main() {
	LCD_Init();
	DS1302_Init();
	Timer0Init();
	LCD_ShowString(1, 1, "  -  -  ");
	LCD_ShowString(2, 1, "  :  :  ");	

//	DS1302_WriteByte(0x8E, 0x00); // 解除DS1302写保护
														  
	DS1302_SetTime(); // 使用单片机数据区（EEPROM）存储的时间数据设置DS1302初始值
	while (1) {
		KeyNum = Key();
		if (KeyNum == 1) { // 模式切换
			if (MODE == 0) { // 显示模式 -> 设置模式
				MODE = 1;
				TimeSetSelect = 0; // 进入设置模式时，设置位数清零
			} else if (MODE == 1) { // 设置模式 -> 显示模式 
				MODE = 0;
				DS1302_SetTime(); // 退出设置模式时，保存设置 
			}
		}
		switch (MODE) {
			case 0: TimeShow(); break; // 单片机工作在显示模式
			case 1: TimeSet(); break;  // 单片机工作在设置模式
		}
	}
}

/**
 * @brief	显示设置模式下，对应正在设置位闪烁
 			每1ms触发一次该定时器中断函数并计数1次
			计数达到500次（500ms）对应正在设置位闪烁标志位（TimeSetFlashFlag）翻转一次
 * @note  	该定时器中断函数不断被触发，即闪烁标志位（TimeSetFlashFlag）不断翻转
 			但只在设置模式下使用闪烁标志位（TimeSetFlashFlag），
			控制对应位闪烁闪烁标志位（TimeSetFlashFlag）
 */
void Timer0_Routine() interrupt 1 {
	static unsigned int T0Count;
	// 设施T0Count每个1ms加1
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	++T0Count;
	if (T0Count >= 500) {
		T0Count = 0;
		TimeSetFlashFlag = !TimeSetFlashFlag;
	}	
}
```