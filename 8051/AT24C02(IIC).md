---
aliases: 
tags:
  - 51单片机
  - c
  - IIC
summary: EEPROM and Inter IC BUS
create time: 2023-12-19T19:55:00
modify time:
---
## AT24C02

存储介质：EEPROM
通信接口：IIC(Inter IC BUS)
存储容量：256Bytes

### 外部接口

![[Pasted image 20231219202706.png]]

### 内部结构框图

![[Pasted image 20231219203325.png]]

### 存储器

#### RAM(Random Access Memory)

1. SRAM(Static Random Access Memory)
2. DRAM(Dynamic Random Access Memory)

#### ROM(Read Only Memory)

1. Mask ROM
2. PROM
3. EPROM
4. EEPROM
5. Flash
6. Disk, CD, DVD

### 通信

#### 串口通信

1. UART
2. RS232
3. RS485

#### 总线通信

1. USB
2. SPI
3. CAN
4. IIC

## IIC

I2C（Inter－Integrated Circuit）总线是由 PHILIPS 公司开发的两线串行总线，用于连接微控制器及其外围设备。

是微电子通信控制领域广泛采用的一种==总线标准==。

它是==同步、半双工、带数据应答通信==的一种特殊形式，具有接口线少，控制方式简单，器件封装形式小，通信速率较高等优点。

I2C 总线只有==两根双向信号线==。一根是==数据线 SDA(Serial Data)==，另一根是==时钟线 SCL(Serial Clock)==。

由于其管脚少，硬件实现简单，可扩展性强等特点，因此被广泛的使用在各大集成芯片内

### 物理层

![[Pasted image 20231219210033.png]]

#### 开漏模式

输入0：连接GND，输出0
输入1：总线处于高阻态，输出不稳定，无驱动能力

##### 弱上拉模式

高电平驱动能力弱，低电平驱动能力强

### 协议层

#### IIC信号定义（信号时许结构）

起始和终止信号都是由主机发出的

##### 起始信号

SCL 线为高电平期间，SDA 线由高电平向低电平的变化表示起始信号

在起始信号产生后，总线就处于被占用的状态。

##### 终止信号

SCL线为高电平期间，SDA 线由低电平向高电平的变化表示终止信号

在终止信号产生后，总线就处于空闲状态。

![[Pasted image 20231219211745.png]]

##### 数据传输（SCL时钟线控制权：主机）

###### 主机向从机发送数据（SDA数据线控制权：主机）

###### 主机接受从机发送的数据（SDA数据线控制权：从机）

##### 应答信号

每当发送器件传输完一个字节的数据后，后面必须紧跟一个校验位，这个校验位是接收端通过控制 SDA（数据线）来实现的，以提醒发送端数据我这边已经接收完成，数据传送可以继续进行。

这个校验位其实就是数据或地址传输过程中的响应。

响应包括==“应答(ACK)”==和==“非应答(NACK)”==两种信号。

作为数据接收端时，当设备(无论主从机)接收到 I2C 传输的一个字节数据或地址后，若希望对方继续发送数据，则需要向对方发送“应答(ACK)”信号即特定的低电平脉冲，发送方会继续发送下一个数据；若接收端希望结束数据传输，则向对方发送“非应答(NACK)”信号即特定的高电平脉冲，发送方接收到该信号后会产生一个停止信号，结束信号传输。应答响应时序图如下：

![[Pasted image 20231219213351.png]]

- ! 每一个字节必须保证是 8 位长度。数据传送时，先传送最高位（MSB），每一个被传送的字节后面都必须跟随一位应答位（即一帧共有 9 位）。

###### 发送应答（从机向主机返回接受数据是否成功标识）

###### 接受应答（主机向从机返回接受数据是否成功标识）

#### IIC 数据帧

##### 主机向从机发送数据

![[Pasted image 20231219213742.png]]

##### 主机向从机读取数据

![[Pasted image 20231219213826.png]]

##### 复合格式

![[Pasted image 20231219213905.png]]

### AT24C02 data frame

AT24C02固定地址为1010，可配置地址STC89C52为000

==SLAVE ADDRESS + W = 0xA0==

==SLAVE ADDRESS + R = 0xA1==

[24C02](./attachments/24C02)

## 代码

#### I2C.h

```c
#ifndef __I2C_H__
#define __I2C_H__

void I2C_Start(void);
void I2C_Stop(void);
void I2C_SendByte(unsigned char Byte);
unsigned char I2C_ReceiveByte(void);
void I2C_SendAck(unsigned char AckBit);
unsigned char I2C_ReceiveAck(void);

#endif // __I2C_H__
```

#### I2C.c

```c
#include <REGX52.H>

sbit I2C_SCL = P2^1;
sbit I2C_SDA = P2^0;

/**
 * @brief 	主机发送数据传输开始信号
 * @param	无
 * @retval	无
 */
void I2C_Start(void) {
	// 保证开始信号之前，SDA、SCL处于高电平
	I2C_SDA = 1;
	I2C_SCL = 1;
	// 开始信号：SCL高电平时SDA产生一个下降沿信号
	// SDA先拉低，SCL后拉低
	I2C_SDA = 0;
	I2C_SCL = 0;
}

/**
 * @brief 	主机发送数据传输结束信号
 * @param	无
 * @retval	无
 */
void I2C_Stop(void) {
	// 保证停止之前，SDA处于低电平
	I2C_SDA = 0;
	// 停止信号：SCL高电平时SDA产生一个上升沿信号
	// SCL先拉高，SDA后拉高
	I2C_SCL = 1;
	I2C_SDA = 1;
}

/**
 * @brief 	主机向从机发送1Byte的数据
 * @param   需要发送的数据
 * @retval	无
 */
void I2C_SendByte(unsigned char Byte) {
	unsigned char i;
	for (i = 0; i < 8; ++i) {
		// 从高位到低位，依次发送
		I2C_SDA = Byte & (0x80 >> i);
		// 根据AT24C02交流电气特性，
		// STC89C52单片机执行语句周期小于AT24C02读取周期
		// 程序中直接翻转SCL，不适用Delay函数，
		// AT24C02可以直接实现读取数据
		// note1: 高速单片机，需要查阅手册，实现传输和接受数据速率匹配
		// note2: AT24C02写周期，接受周期需要5ms，需要对应的Delay函数
		I2C_SCL = 1;
		I2C_SCL = 0;	
	}
}

/**
 * @brief 	主机接受从机发送的1Byte数据
 * @param	接受到的1Byte数据
 * @retval	无
 */
unsigned char I2C_ReceiveByte(void) {
	unsigned char i = 0, Byte = 0x00;

  	// 主机释放SDA总线控制权，交给从机发送数据
	I2C_SDA = 1;

	for (i = 0; i < 8; ++i) {
		// 主机接受从机数据，从高位到低位，依次发送
		I2C_SCL = 1;
		if (I2C_SDA) { Byte |= (0x80 >> i);}
		I2C_SCL = 0;
	}
	return Byte;
}

/**
 * @brief 	主机接受从机发送的1Byte数据后，主机向从机发送数据接受完成的应答信号
 * @param	主机应答信号的具体值，应答：0，不应答：1
 * @retval	无
 */
void I2C_SendAck(unsigned char AckBit) {
	I2C_SDA = AckBit; // 应答：0，不应答：1
	I2C_SCL = 1;
	I2C_SCL = 0;
}

/**
 * @brief 	主机向从机发送1Byte的数据，从机向主机发送数据接受完成的应答信号
 * @param	无
 * @retval	从机发送的应答信号的具体值，应答：0，不应答：1
 */
unsigned char I2C_ReceiveAck(void) {
	unsigned char AckBit = 0;	
	// 主机释放SDA总线控制权，交给从机发送应答信号
	I2C_SDA = 1;
	// 主机接受从机应答并返回
	I2C_SCL = 1;
	AckBit = I2C_SDA;
	I2C_SCL = 0;
	return AckBit;
}
```

#### AT24C02.h

```c
#ifndef __AT24C02_H__
#define __AT24C02_H__

void AT24C02_WriteByte(unsigned char WordAddress, Data);
unsigned char AT24C02_ReadByte(unsigned char WordAddress);

#endif // __AT24C02_H__
```

#### AT24C02.c

```c
#include <REGX52.H>
#include "I2C.h"

#define AT24C02_ADDRESS 0xA0

/**
 * @brief 	单片机向AT24C02写入1Byte的数据
 * @param1	写入到AT24C02的内部地址
 * @param2  写入的数据
 * @retval	无
 */
void AT24C02_WriteByte(unsigned char WordAddress, Data) {
	I2C_Start();
	I2C_SendByte(AT24C02_ADDRESS);
	I2C_ReceiveAck(); // 需要考虑处理从机发送非应答的情况：重新发送数据或其他
   	I2C_SendByte(WordAddress);
	I2C_ReceiveAck();
	I2C_SendByte(Data);
	I2C_ReceiveAck();
	I2C_Stop();
}

/**
 * @brief 	单片机向AT24C02读取1Byte的数据
 * @param1	读取的AT24C02的内部地址
 * @retval	返回读取到的数据
 */
unsigned char AT24C02_ReadByte(unsigned char WordAddress) {
	unsigned char Data;
	I2C_Start();
	I2C_SendByte(AT24C02_ADDRESS);
	I2C_ReceiveAck(); // 需要考虑处理从机发送非应答的情况：重新发送数据或其他
   	I2C_SendByte(WordAddress);
	I2C_ReceiveAck();
	I2C_Start();
	I2C_SendByte(AT24C02_ADDRESS | 0x01); // 写地址转换为读地址
	I2C_ReceiveAck();
	Data = I2C_ReceiveByte();
	I2C_SendAck(1); // 最后一次读，发送非应答
	I2C_Stop();
	return Data;
}
```

### AT24C02数据存储

- ! AT24C02写周期时间：是指从一个写时序的有效停止开始至内部写周期结束的时间。最长5ms。

```c
#include <REGX52.H>
#include "Delay.h"
#include "LCD1602.h"
#include "Key.h"
#include "AT24C02.h"

unsigned char KeyNum;
unsigned int Num;

void main() {
	LCD_Init();
	LCD_ShowNum(1, 1, Num, 5);
	while (1) {
		KeyNum = Key();
		if (KeyNum == 1) {
			++Num;
			LCD_ShowNum(1, 1, Num, 5);
		}
		if (KeyNum == 2) {
			--Num;
			LCD_ShowNum(1, 1, Num, 5);
		}
		if (KeyNum == 3) { // 写入AT24C02
			// 写入低八位 
			AT24C02_WriteByte(0, Num % 256); 
			Delay(5);
			// 写入高八位
			AT24C02_WriteByte(1, Num / 256);
			Delay(5);
			LCD_ShowString(2, 1, "Write OK");
			Delay(1000);
			LCD_ShowString(2, 1, "        ");
		}
		if (KeyNum == 4) { // 从AT24C02读出
			Num = AT24C02_ReadByte(0);
			Num |= AT24C02_ReadByte(1) << 8;
			LCD_ShowNum(1, 1, Num, 5);
			LCD_ShowString(2, 1, "Read  OK");
			Delay(1000);
			LCD_ShowString(2, 1, "        ");
		} 
	}
}
```

### 秒表（利用定时器扫描按键数码管）

#### 使用中断按键扫描实现按键按下不阻塞主程序

##### Key.h

```c
#ifndef __KEY_H__
#define __KEY_H__

unsigned char Key(void);
void Key_Loop(void);

#endif // __KEY_H__
```

##### Key.c

```c
#include <REGX52.h>
#include "Delay.h"

unsigned char Key_KeyNumber;

/**
 * @brief 	返回按下的键码值，并重置全局变量Key_KeyNumber
 * @param	void
 * @retval 	按下的按键键码
 * @caller	main
 * @callee	null
 */
unsigned char Key(void) {
	// 实现按键只触发一次，后置0，使用Temp保存，并重置全局变量Key_KeyNumber
	unsigned char Temp = 0;
	Temp = Key_KeyNumber;
	Key_KeyNumber = 0;
	return Temp;
}

/**
 * @brief 	获取按键状态函数
 * @param	void
 * @retval	按键状态
 * @caller	Key_Loop
 * @callee	null
 * @note	按键按下：按键对应端口为低电平
 */
unsigned char Key_GetState(void) {
	unsigned char KeyNumber = 0;
	if (P3_1 == 0) {KeyNumber = 1;}
	if (P3_0 == 0) {KeyNumber = 2;}
	if (P3_2 == 0) {KeyNumber = 3;}
	if (P3_3 == 0) {KeyNumber = 4;}
	return KeyNumber;
}

/**
 * @brief 	按键扫描，将全局变量Key_GetState设置为获取到的键码
 * @param	void
 * @retval	void
 * @caller	Timer0_Routine interrupt 1
 * @callee	Key_GetState
 * @note 	通过判断按键状态是否存在从按下到松手的状态转换，实现松手时相应
 			Timer0_Routine中每间隔20ms触发一次该函数，
			实现不断扫描按键而不阻塞，按键按下和松开的防抖
 */
void Key_Loop(void) {
	static unsigned char NowState, LastState;
	LastState = NowState;
	NowState = Key_GetState();
	if (LastState == 1 && NowState == 0) {
		Key_KeyNumber = 1;
	}
	if (LastState == 2 && NowState == 0) {
		Key_KeyNumber = 2;
	}
	if (LastState == 3 && NowState == 0) {
		Key_KeyNumber = 3;
	}
	if (LastState == 4 && NowState == 0) {
		Key_KeyNumber = 4;
	}	
}
```

#### 使用中断扫描设置数码管实现主程序进行其他操作时数码管正常显示

##### Nixie.h

```c
#ifndef __NIXIE_H__
#define __NIXIE_H__

void Nixie_SetBuf(unsigned char Location, Number);
void Nixie_Scan(unsigned char Location, Number);
void Nixie_Loop(void);

#endif // __NIXIE_H__
```

##### Nixie.c

```c
#include <REGX52.H>
#include "Delay.h"

// 数码管显示缓存数组
unsigned char Nixie_Buf[9] = {0, 10, 10, 10, 10, 10, 10, 10, 10};

// 数码管对应数字对应的段码表
unsigned char NixieTable[] = \
{0x3F, 0x06, 0x5B, 0x4F, 0x66, 0x6D, 0x7D, 0x07, 0x7F, 0x6F, 0x00, 0x40};
// 0     1     2     3     4     5     6     7     8     9	  空  	 -


/**
 * @brief 	将指定位设置为需要显示的数据
 * @param	Location: 指定显示的数码管
 * @param	Number: 指定数码管显示的数字
 * @retval	void
 * @caller	main
 * @callee 	null
 */
void Nixie_SetBuf(unsigned char Location, Number) {
	Nixie_Buf[Location] = Number;	
}

/**
 * @brief 	指定的数码管上显示指定数据
 * @param	Location: 指定显示的数码管
 * @param	Number: 指定数码管显示的数字
 * @retval	void
 * @caller  Nixie_Loop
 * @callee	null
 */
void Nixie_Scan(unsigned char Location, Number) {
	P0 = 0x00; // 断码清零
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
}

/**
 * @brief 	数码管的每一位显示对应位的数据
 * @param	void
 * @retval	void
 * @caller 	Timer0_Routine interrupt 1
 * @callee	Nixie_Scan
 * @note	Timer0_Routine中每间隔2ms触发一次该函数，
 			实现数码管动态刷新显示，不被其他高cpu占用指令影响正常显示
 */
void Nixie_Loop(void) {
	static unsigned char i = 1 ;
	Nixie_Scan(i, Nixie_Buf[i]);
	++i;
	if (i > 8) { // 逐位显示缓存数组中的内容
		i = 1;
	}
}
```

##### main.c

```c
#include <REGX52.H>
#include "Timer0.h"
#include "Key.h"
#include "Nixie.h"
#include "Delay.h"
#include "AT24C02.h"

unsigned char KeyNum; // 被按下的按键键值
unsigned char Min, Sec, MiniSec; // 秒表时间数据
unsigned RunFlag; // 秒表运行标志

void main() {
	Timer0_Init();
	while (1) {
		KeyNum = Key();
		if (KeyNum == 1) { // 秒表工作状态转换
			RunFlag = !RunFlag;
		}
		if (KeyNum == 2) { // 秒表清零
			Min = 0;
			Sec = 0;
			MiniSec = 0;
		}
		if (KeyNum == 3) { // 秒表计时写入AT24C02
			AT24C02_WriteByte(0, Min);
			Delay(5);
			AT24C02_WriteByte(1, Sec);
			Delay(5);
			AT24C02_WriteByte(2, MiniSec);
			Delay(5);
		}
		if (KeyNum == 4) { // 从AT24C02读出数据作为秒表计时
		  	Min = AT24C02_ReadByte(0);
			Sec = AT24C02_ReadByte(1);
			MiniSec = AT24C02_ReadByte(2);
		}
		// 数码管显示设置
		Nixie_SetBuf(1, Min / 10);
		Nixie_SetBuf(2, Min % 10);
		Nixie_SetBuf(3, 11);
		Nixie_SetBuf(4, Sec / 10);
		Nixie_SetBuf(5, Sec % 10);
		Nixie_SetBuf(6, 11);
		Nixie_SetBuf(7, MiniSec / 10);
		Nixie_SetBuf(8, MiniSec % 10);
	}
}

/**
 * @brief 	秒表计时
 * @param	void
 * @retval  void
 * @caller	Timer0_Routine() interrupt 1
 * @callee  null
 * @note 	Timer0_Routine中每间隔10ms触发一次该函数，
 			仅工作在秒表运行模式
 */
void Second_Loop(void) {
	if (RunFlag) {
	  	++MiniSec;
		if (MiniSec > 99) {
			MiniSec = 0;
			++Sec;
			if (Sec > 59) {
				Sec = 0;
				++Min;
				if (Min > 59) {
					Min = 0;
				}
			}
		}
	}
} 

/**
 * @brief 	定时器0中断处理函数	
 * @param	void
 * @retval	void
 * @caller 	hardware
 * @callee 	null
 */
void Timer0_Routine(void) interrupt 1 {
	static unsigned int T0Count1, T0Count2, T0Count3;
	// 设施T0Count每个1ms加1
	TL0 = 0x18;		//设置定时初值
	TH0 = 0xFC;		//设置定时初值
	
	// 中断实现按键扫描
	++T0Count1;
	if (T0Count1 >= 20) {
		T0Count1 = 0;
		Key_Loop();
	}

	// 中断实现数码管扫描
	++T0Count2;
	if (T0Count2 >= 2) {
		T0Count2 = 0;
		Nixie_Loop();
	}

	// 中断实现时间计数
	++T0Count3;
	if (T0Count3 >= 10) {
		T0Count3 = 0;
		Second_Loop();
	}
}
```