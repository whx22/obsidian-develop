---
author: whx
title: switch is “jump table”
time: 2023-11-16-Thursday
tags:
  - c
  - switch
  - assembly
---
![[Pasted image 20231116174620.png]]

跳转表存储在目标文件的==只读节(.section .rodata)==，并按4字节边界对齐==(.align 4)==。