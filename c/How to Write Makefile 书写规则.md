---
author: whx
title: Makefile
time: 2023-11-13-Monday
tags:
  - makefile
  - c
  - tool
  - build
---
```makefile
target ... ： prerequisites ...
	recipe
	...
```

OR

```makefile
target ... : prerequistites ... ; recipe ...
```

> make只管文件的依赖性

> target : 可执行文件，目标文件，标签（伪目标）

> make 使用标准shell : `/bin/sh` 
## GNU make 工作时的执行步骤：

1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐式规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。

> 一般来说，定义在Makefile中的目标可能会有很多，但是第一条规则中的目标将被确立为最终的目标。如果第一条规则中的目标有很多个，那么，第一个目标会成为最终的目标。make所完成的也就是这个目标。

> makefile 中变量类似与C/C++中的宏。

## 通配符和变量使用

```makefile
# objects is *.o
objects = *.o

# objects is 所有.o的文件名的集合
objects := $(wildcard *.o)
```

## 文件搜索

### 特殊变量 VPATH

```makefile
VPATH = directories-1:directories-2 # 使用 ":" 分隔，POSIX规范
```

### 关键字 vpath

```makefile
vpath <pattern> <directories>

vpath %.h ..headers # % : 匹配零个或多个字符
```

## 伪目标

伪目标不是一个文件，只是一个标签。“伪目标”的取名不能和文件名重名。

### .PHONY 显示指明一个目标为“伪目标”。

```makefile
.PHONY : clean
clean :
	rm *.o temp
```

### 为伪目标指定依赖文件，作为默认目标

如果你的Makefile需要一口气生成若干个可执行文件，并且所有的目标文件都写在一个Makefile中。

```makefile
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o

prog2 : prog2.o
    cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```

Makefile中的第一个目标会被作为其默认目标。声明了一个“all”的伪目标，其依赖于其它三个目标。由于默认目标的特性是，总是被执行的，但由于“all”又是一个伪目标，伪目标只是一个标签不会生成文件，所以不会有“all”文件产生，其它三个目标的规则总是会被决议。

### 伪目标作为依赖

```makefile
.PHONY : cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
    rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```

“make cleanall”将清除所有要被清除的文件。

我们可以输入“make cleanall”和“make cleanobj”和“make cleandiff”命令来达到清除不同种类文件的目的。

## 多目标

自动化变量 `$@` : 目前规则中所有目标的集合

```makefile
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@
```

===

```makefile
bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

其中， `-$(subst output,,$@)` 中的 `$` 表示执行一个Makefile的函数

函数名为subst，替换字符串，后面的为参数。

## 静态模式
```makefile
<target ...> : <target-pattern> : <prereq-patterns ...>
	<commands>
	...
```

如果我们的`<target-pattern>` 定义为`%.o` ，意思是我们的`<target>` ==目标集合== 中都是以`.o` 结尾的，而如果我们的`<prereq-patterns>` 定义成`%.c` ，取`<target-pattern>` 模式中的`%` (也就是去掉了`.o` 这个结尾)，并为其加上`.c` 这个结尾，，形成的新集合作为是==依赖集合== 。
### Instance

```makefile
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

自动化变量

1. `$<` : 第一个依赖文件
2. `$@` : 目标集合

===

```makefile
foo.o : foo.c
    $(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
    $(CC) -c $(CFLAGS) bar.c -o bar.o
```

## 自动生成依赖项

### C/C++编译器编译选项：‘-M’

自动寻找源文件中包含的头文件，并生成一个依赖关系。

`cc -M main.c` output `main.o : main.c headerfile.h`

`gcc -M main.c` output
```
main.o: main.c defs.h /usr/include/stdio.h /usr/include/features.h \\
    /usr/include/sys/cdefs.h /usr/include/gnu/stubs.h \\
    /usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stddef.h \\
    /usr/include/bits/types.h /usr/include/bits/pthreadtypes.h \\
    /usr/include/bits/sched.h /usr/include/libio.h \\
    /usr/include/_G_config.h /usr/include/wchar.h \\
    /usr/include/bits/wchar.h /usr/include/gconv.h \\
    /usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stdarg.h \\
    /usr/include/bits/stdio_lim.h
```

`gcc -MM main.c` output
```
main.o: main.c defs.h
```

GNU的C/C++编译器使用 `-MM` 参数

`-M`参数会把一些标准库的头文件也包含进来。

### `name.d` : `name.c` 的依赖关系文件

由于无法直接在主Makefile中使用这个编译器自动生成依赖关系的功能，以实现主Makefile中的依赖关系自动更新。

GNU组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个文件中，为每一个 `name.c` 的文件都生成一个 `name.d` 的Makefile文件， `.d` 文件中就存放对应`.c` 文件的依赖关系。

```makefile
%.d: %.c
    @set -e; rm -f $@; \\
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \\
    sed 's,\\($*\\)\\.o[ :]*,\\1.o $@ : ,g' < $@.$$$$ > $@; \\
    rm -f $@.$$$$
```

这个规则的意思是，所有的 `.d` 文件依赖于 `.c` 文件， `rm -f $@` 的意思是删除所有的目标，也就是 `.d` 文件，第二行的意思是，为每个依赖文件 `$<` ，也就是 `.c` 文件生成依赖文件， `$@` 表示模式 `%.d` 文件，如果有一个C文件是name.c，那么 `%` 就是`name` ， `$$$$` 意为一个随机编号，第二行生成的文件有可能是“name.d.12345”，第三行使用sed命令做了一个替换，关于sed命令的用法请参看相关的使用文档。第四行就是删除临时文件。

于是，我们的 `.d` 文件也会自动更新了，并会自动生成了，当然，你还可以在这个 `.d` 文件中加入的不只是依赖关系，包括生成的命令也可一并加入，让每个 `.d` 文件都包含一个完整的规则。一旦我们完成这个工作，接下来，我们就要把这些自动生成的规则放进我们的主Makefile中。

```makefile
sources = foo.c bar.c

include $(sources:.c=.d)
```

`include` : 引入其他Makefile文件到主Makefile文件命令。

`$(sources:.c=.d)` 中的 `.c=.d` 的意思是做一个替换，把变量`$(sources)` 所有 `.c` 的字串都替换成 `.d` 。