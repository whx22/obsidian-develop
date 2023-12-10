---
aliases:
  - 中断系统
tags:
  - 51单片机
  - 中断系统
  - c
summary: 
create time: 2023-12-05T21:21:00
modify time:
---
## Introduction

![[Pasted image 20231208163538.png]]

## STC89C52中断资源

![[Pasted image 20231208163816.png]]

![[Pasted image 20231208164129.png]]

### 中断程序接口函数API

![[Pasted image 20231208164206.png]]

## 中断结构

![[Pasted image 20231209204445.png]]

## 中断寄存器


![[Pasted image 20231208164831.png]]^interrupt-register

### IE：中断允许寄存器

![[Pasted image 20231208171110.png]]

### IPH:IP：中断优先级控制寄存器（可位寻址）

#### 可位寻址VS不可位寻址

1. 可位寻址：可以单独为寄存器每一位赋值
2. 不可位寻址：仅能对寄存器整体赋值