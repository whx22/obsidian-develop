---
aliases: 
tags:
  - 51单片机
  - LED点阵屏
summary: 
create time: 2023-12-10T20:32:00
modify time:
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

// code keywork : 将动画数组存放到存放代码的flash中，默认数据存放置EEPROM中
// EEPROM : 数据段存储器，存放全局变量，静态变量，存储空间小，初始化后不可以修改，只读
// flash : 代码段存储器，存放指令代码，存储空间大，初始化后可以修改，可读可写