---
author: whx
title: Makefile Template
time: 2023-11-14-Tuesday
tags:
  - makefile
  - c
  - build
  - tool
  - template
---
## Template 1 : 生成若干个可执行文件

使用伪目标并为其指定依赖文件的技巧

```makefile
all ： prog1 prog2 prog3
.PYONY : all

prog1 : prog1.o utils.o
	cc -o prog1 prog1.o utils.o

prog2 : prog2.o
	cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
		cc -o prog3 prog3.o sort.o utils.o
```

## Template 2 : 生成可执行文件

1. 源文件目录：`./src` 
2. 头文件目录：`./include` 
3. 库文件（静态库、动态库）目录：`./lib`
4. 可执行目标文件目录：`./output` 
### Instance

```makefile
VERSION   =1.00
CC        =gcc
DEBUG     =-DUSE-DEBUG
CFLAGS    =-Wall
SOURCES   =$(wildcard ./src/*.c)
INCLUDES  =-I./include
LIB_NAMES =-lfun_a -lfun_so
LIB_PATH  =-L./lib
OBJ       =$(patsubst %.c, %.o, $(SOURCES))
TARGET    =app

# links
$(TARGET) : $(OBJ)
	@mkdir -p output
	$(CC) $(OBJ) $(LIB_PATH) $(LIB_NAMES) -o output/$(TARGET)$(VERSION)
	@rm -rf $(OBJ)
	
# compile
%.o : %.c
	$(CC) $(INCLIDES) $(DEBUG) -c $(CFLAGS) $< -o $@

.PHONY : clean
clean:
	@echo "Remove linked and compiled files......"
	rm -rf $(OBJ) $(TARGET) output
```
## Template 3 : 生成静态库文件

头文件、源文件均在当前目录下
### Instance

```makefile
VERSION   =
CC        =gcc
DEBUG     =
CFLAGS    =-Wall
AR        =ar
ARFLAGS   =rv
SOURCES   =$(wildcard ./src/*.c)
INCLUDES  =-I.
LIB_NAMES =
LIB_PATH  =
OBJ       =$(patsubst %.c, %.o, $(SOURCES))
TARGET    =libfun_a

# links
$(TARGET) : $(OBJ)
	@mkdir -p output
	$(AR) $(ARFLAGS) output/$(TARGET)$(VERSION).a $(OBJ)
	@rm -rf $(OBJ)
		
# compile
%.o : %.c
	$(CC) $(INCLIDES) $(DEBUG) -c $(CFLAGS) $< -o $@

.PHONY : clean
clean:
	@echo "Remove linked and compiled files......"
	rm -rf $(OBJ) $(TARGET) output
```

## Template 4 : 生成动态库文件

头文件、源文件均在当前目录下

### Instance

```makefile
VERSION   =
CC        =gcc
DEBUG     =
CFLAGS    =-fPIC -shared
LFLAGS    =-fPIC -shared
SOURCES   =$(wildcard *.c)
INCLUDES  =-I.
LIB_NAMES =
LIB_PATH  =
OBJ       =$(patsubst %.c, %.o, $(SOURCES))
TARGET    =libfun_so

# links
$(TARGET) : $(OBJ)
	@mkdir -p output
	$(CC) $(OBJ) $(LIB_PATH) $(LIB_NAMES) $(LFLAGS) \
	-o output/$(TARGET)$(VERSION).so
	@rm -rf $(OBJ)
		
# compile
%.o : %.c
	$(CC) $(INCLIDES) $(DEBUG) -c $(CFLAGS) $< -o $@

.PHONY : clean
clean:
	@echo "Remove linked and compiled files......"
	rm -rf $(OBJ) $(TARGET) output
```
## conclusion

### reference

[Makefile Template](https://mp.weixin.qq.com/s/1nXoEcdURd5EUWo4fb_Umg) 

[How to Write Makefile]([https://seisman.github.io/how-to-write-makefile/](https://seisman.github.io/how-to-write-makefile/))  

