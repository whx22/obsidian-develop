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