---
aliases:
  - 模块化编程
tags:
  - c
  - 51单片机
summary: 模块化编程（英语：modular programming），是强调将计算机程序的功能分离成独立的、可相互改变的“模块”（module）的软件设计技术，它使得每个模块都包含着执行预期功能的一个唯一方面（aspect）所必需的所有东西。
create time: 2023-12-02T21:00:00
modify time: 2023-12-02T21:11:00
---
## 延迟功能模块化

### header file

```c
#ifndef __DELAY_H__
#define __DELAY_H__

void Delay(unsigned int xms);

#endif // __DELAY_H__
```

### source file

```c
// 延时函数单位ms(STC-ISP生成) @12.000MHz
void Delay(unsigned int xms) {
	unsigned char i, j;
	while(xms--) {
		i = 2;
		j = 239;
		do {
			while (--j);
		} while (--i);
	}
}
```

## 数码管显示模块化

### header file

```c
#ifndef __NIXIE_H__
#define __NIXIE_H__

void Nixie(unsigned char Location, unsigned char Number);

#endif // __NIXIE_H__
```

### source file

```c
#include <REGX52.H>
#include "Delay.h"

// 数码管对应数字对应的段码表
unsigned char NixieTable[] = \
{0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D, 0x07, 0x7F, 0x6F};
// 0     1     2     3     4     5     6     7     8     9

/*
 * 描述：数码管显示
 * 参数1：Location: 指定显示的数码管
 * 参数2：Number: 指定数码管显示的数字
 */
void Nixie(unsigned char Location, unsigned char Number) {
	// 根据具体Location参数，选中对应数码管
	switch(Location) {
		case 1: P2_4 = 1, P2_3 = 1, P2_2 = 1; break; // 第一个数码管：LED8，Y7 
		case 2: P2_4 = 1, P2_3 = 1, P2_2 = 0; break; 
		case 3: P2_4 = 1, P2_3 = 0, P2_2 = 1; break;
		case 4: P2_4 = 1, P2_3 = 0, P2_2 = 0; break;
		case 5: P2_4 = 0, P2_3 = 1, P2_2 = 1; break;
		case 6: P2_4 = 0, P2_3 = 1, P2_2 = 0; break;
		case 7: P2_4 = 0, P2_3 = 0, P2_2 = 1; break;
		case 8: P2_4 = 0, P2_3 = 0, P2_2 = 0; break;
	}
	// 根据Number参数显示对应数字
	P0 = NixieTable[Number];

	// 解决数码管动态显示错位问题——消影 
	Delay(1); // 维持选中并显示，产生视觉残留
	P0 = 0x00; // 段码清零，防止显示数据窜位
}
```