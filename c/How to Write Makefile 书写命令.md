---
author: whx
title: How to Write Makefile 书写命令
time: 2023-11-14-Tuesday
tags:
  - c
  - makefile
  - build
  - tool
---
## 显示命令

###  `@`：隐藏命令显示
make通常将要执行的命令在命令执行前输出到屏幕上。
`@<command>` ：该命令不会被make显示出来。

###  `-n` or `--just-print` ：显示命令，不执行命令

### `-s` or `--silent` or `--quiet` 全面禁止命令的显示

## 命令执行

上一条命令的结果应用在下一条命令执行。使用`;` 分隔，而不是使用==换行符==。

instance 1 : 
```makefile
exec:
	cd /home/whx
	pwd
# result : /home
```

instance 2 : 
```makefile
exec:
	cd /home/whx; pwd
# result : /home/whx
```

## 命令出错

每条命令运行结束后，make检查每个命令的返回值，命令返回正常，make执行下一条命令。规则中所有的命令成功返回后，这个规则成功。如果规则中某个命令返回值非零，make终止执行当前规则，将可能终止所有规则的执行。

### `-` ：忽略命令的返回值

`-<command>` ：标记命令忽略命令返回值，始终标记为成功执行。

### `-i` or `--ignore-errors` make 参数

忽略makefile中所有命令的返回值。

### `-k` or `--keep-going` make 参数

如果某个规则中的命令出错了，那么就终止该规则的执行，但会继续执行其他规则。

## 嵌套执行make

大型工程中，不同模块或是不同功能的源文件和头文件使用不同目录组织，每个模块目录中都有一个makefile，并使用==总控makefile== 组织，优点：
1. makefile更加简洁，更容易维护。
2. 利于模块编译，分段编译。

### Instance

主目录下有一个子模块目录`subdir` 。

`$(MAKE)` 宏变量，定义为变量便于曾加参数，便于维护。

```makefile
subsystem:
	cd subdir && $(MAKE)
```

===

```makefile
subsystem:
	$(MAKE) -C subdir
```

### `export` ：将总控makefile变量传递到下级makefile中

`export <variable ...>`：传递某个变量到下级makefile。

`unexport <variable ...>` ：不传递某个变量到下级makefile。

`export` ： 传递所有变量到下级makefile。

> 系统级环境变量总是传递。`SHELL` and `MAKEFLAGS` （make参数信息）

> `-w` or `--print-directory` 打印make执行目录（make进入时和离开时）

## 命令包

使用方法类似与变量。

```makefile
define <command-back>
<command>
...
endef
```