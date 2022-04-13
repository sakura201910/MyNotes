# Makefile

功能：自动化编译，完成编译的先后顺序等

命名：makefile或者Makefile

## Makefile规则：

目标...：依赖...

​		命令（Shell）

​		...

- 目标：生成的文件（伪目标）
- 依赖：生成目标需要的文件或者目标
- 命令：通过执行命令对于依赖执行的操作（命令前必须有缩进）

示例：

```makefile
app:sub.o add.o
	gcc sub.o add.o -o app
	
sub.o:sub.c
	gcc -c sub.c -o sub.o
add.o:add.c
	gcc -c sub.c -o add.o
```

示例中 **sub.o,add.o** 2个目标都为app所服务

## 特点

- 检查更新，使用文件时间比较进行检查，已存在的依赖不会再执行
- 所有规则为第一个规则服务

## 变量

![image-20220218204840578](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220218204840578.png)

## 模式匹配

![image-20220218210041544](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220218210041544.png)

**变量与模式匹配示例**

```makefile
#定义变量
src = sub.o add.o
target = app

$(target):$(src)
	$(CC) $(src) -o $(target)
%.o:%.c		#模式匹配
	$(CC) -c $< -o $@
```

## 函数

在makefile中可以使用函数，再次优化makefile文件的写法，函数中可以使用通配符等

### wildcard函数

![image-20220218210624156](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220218210624156.png)

### patsubst函数

![image-20220218210817990](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220218210817990.png)

**2个函数使用示例**

```makefile
#定义变量
tem = $(wildcard ./*.c)
src = $(patsubst %.c, %.o, $(tem))
target = app

$(target):$(src)
	$(CC) $(src) -o $(target)
%.o:%.c		#模式匹配
	$(CC) -c $< -o $@
```

## 在makefile中使用命令执行一些操作

例如需要删除.o文件

```cpp
#定义变量
tem = $(wildcard ./*.c)
src = $(patsubst %.c, %.o, $(tem))
target = app

$(target):$(src)
	$(CC) $(src) -o $(target)
%.o:%.c		#模式匹配
	$(CC) -c $< -o $@
	
.PHONY:clean		#将clean定义为伪目标，防止检查文件是否是新的，无法执行clean
clean:
	rm $(src) -f
```

执行make时clean不会被执行，因为第一条命令并不依赖clean

执行方法：`make clean`



