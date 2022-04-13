# Linux系统IO



## open()

```cpp
//打开一个已经存在的文件
    int open(const char *pathname, int flags);
        参数：
            - pathname: 要打开的文件路径
            - flags:  对文件操作权限设置还有其他的设置
                O_RDONLY, O_WRONLY, or O_RDWR 三个操作互斥
        返回值：
            返回一个文件描述符，如果调用失败返回-1并且设置errno
    errno: 属于linux系统函数库，库里面的一个全局变量，记录的是最近的错误号
```

```cpp
  //创建一个新文件
int open(const char *pathname, int flags, mode_t mode);
    参数：
        - pathname: 要创建的文件路径
        - flags: 对文件的操作权限及其他设置
            - 必选项：O_RDONLY, O_WRONLY, or O_RDWR 三者互斥
            - 可选项：O_CREAT（文件不存在创建新文件）       等，可以参考文档
        -mote:八进制数，表示创建出的新的文件的操作权限，最高权限为 0777   前面的0表示八进制
        最终权限是 mode & ~umask
        umask用于防止用户创建不合理的权限，umask的值可修改
```

## perror()

```cpp
   //打印error对应的错误描述
    void perror(const char *s);
        s参数： 用户描述，例如ssss,最终输出的内容是ssss:xxx(实际的错误描述)
```

## read() write()

```cpp
	ssize_t read(int fd, void *buf, size_t count);
        参数：
            - fd:文件描述符，open得到的，通过这个文件描述符操作某文件
            - buf: 需要读取数据存放的地方，数组的地址（传出参数）
            - count: 指定数组的大小
        返回值：
            - 成功：
                > 0: 返回实际读取到的字节数
                = 0: 文件已经读取完
            - 失败：
                -1，并设置errno
```

```cpp
    ssize_t write(int fd, const void *buf, size_t count);
        参数：
            - fd:文件描述符，open得到的，通过这个文件描述符操作某文件
            - buf: 要往磁盘写入的数据
            - count: 写入数据的实际大小
        返回值：
            - 成功：
                > 0: 返回实际写入的字节数
                = 0: 没有写入任何数据
            - 失败：
                -1，并设置errno
```

## offset()

```cpp
off_t lseek(int fd, off_t offset, int whence);
       参数：
            - fd:文件描述符，通过open得到
            - offset: 偏移量
            - whence:
                SEEK_SET
                    设置文件的偏移量
                SEEK_CUR
                    设置偏移量：当前位置+第二个参数offset的值
                SEEK_END
                    设置偏移量：文件大小+第二个参数offset的值
        返回值：返回文件指针的位置
    作用：
        1.移动文件指针到文件头
            lseek(fd,0,SEEK_SET)
        2.获取当前文件指针的位置
            lseek(fd,0,SEEK_CUR)
        3.获取文件长度
            lseek(fd,0,SEEK_END)
        4.拓展文件长度，当前文件10b,增加100个字节
            lseek(fd,100,SEEK_END)
            需要在末尾插入任意符号

            用于传输文件时先占用硬盘空间，再传输中写入文件
```

## stat() lstat()

```cpp
 int stat(const char *pathname, struct stat *statbuf);
        作用：获取文件的相关信息
        参数：
            - pathname: 操作文件的路径
            - statbuf: 结构体变量，传出参数 
        返回值：
            成功：返回0
            失败：返回-1，设置errno

    int lstat(const char *pathname, struct stat *statbuf);
        作用：获取软连接文件的相关信息
        参数：
            - pathname: 操作文件的路径
            - statbuf: 结构体变量，传出参数 
        返回值：
            成功：返回0
            失败：返回-1，设置errno
```

# 文件属性操作



## access()

判断文件权限或者是否存在

`int access(const char *pathname, int mode);`

## chmod()

修改文件权限

`int chmod(const char *pathname, mode_t mode);`

## chown()

修改文件所有者，所在组

`int chown(const char *pathname, uid_t owner, gid_t group);`

## truncate()

缩减或者扩展文件大小

`int truncate(const char *path, off_t length);`

# 目录操作函数

![image-20220220224250728](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220220224250728.png)

# 目录遍历函数

## opendir()

打开目录

`DIR *opendir(const char *name);`

## readdir()

读取目录

`struct dirent *readdir(DIR *dirp);`

## closedir()

关闭目录

`int closedir(DIR *dirp);`

# dup() dup2()

![image-20220220224625423](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220220224625423.png)

# fcntl()

- 复制文件描述符
- 设置/获取文件的状态标志

`int fcntl(int fd, int cmd, ... /* arg */ );`
