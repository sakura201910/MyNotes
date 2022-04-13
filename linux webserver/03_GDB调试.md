# 环境准备相关

- 将文件编译为适合进行调试的可执行文件`g++ -g -Wall xxx.c -o xxx`

- -g     在编译中加入源代码信息，但并不是把整个文件嵌入到可执行文件中，调试时需要确保gdb能够找到源文件

- -Wall   打开所有的警告

# 查看使用相关

启动:`gdb 可执行文件名`

退出:`quit`

给程序设置参数：`set args 10 20`

获取参数设置：`show args`

GDB使用帮助：`help`

- 查看文件代码
    - `l/list `
    - `l/list 行号`
    - `l/list 函数名`
- 查看非当前文件代码
    - `l/list 文件名: xx`

- 设置显示行数
    - `show list/listsize`
    - `set list/listsize 行数` 

# 断点操作

- 设置断点

    - `b/break 行号`	

    - `b/break 函数名`

    - `b/break 文件名:行号`

    - `b/break 文件名:函数`

- 查看断点
    - `i b`或`info break`

- 删除断点
    - `d/del/delete 断点编号`
- 设置断点无效
    - `dis/disable 断点编号`
- 设置断点生效
    - `ena/enable 断点编号`
- 设置条件断点
    - `b/break 10 if i==5`

# 调试常用命令

![image-20220219164904425](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220219164904425.png)

