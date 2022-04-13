# execl()

```cpp
/*
    #include <unistd.h>
    int execl(const char *pathname, const char *arg, ...); 
        - 作用：
            调用可执行文件（一般在子进程中进行调用）
        - 参数：
            - path 需要指定的执行文件的路径或名称
                a.out /home/a.out
            - arg 是执行可执行文件所需要的参数列表
                第一个参数一般没有什么作用，为方便，一般填写执行程序的名称
                第二个参数往后，就是程序执行所需要的参数列表
                参数最后需要以null结束（哨兵）

        - 返回值
            只有调用失败才返回-1,设置errno
            如果调用成功，没有返回值
        
        - 原理
            使用需要调用的可执行文件的用户区替换掉原本程序的用户区

*/

#include <unistd.h>
#include <stdio.h>

int main(int argc, char const *argv[])
{
    //创建子进程，在子进程中执行exec函数族
    pid_t pid = fork();

    if(pid>0){
        //父进程
        printf("father pid =%d\n",getpid());
        sleep(1);
    }
    else if(pid==0){
        execl("/usr/bin/echo","echo","echo hello!",NULL);
        printf("child id =%d\n",getpid());
    }
    for(int i=0;i<3;++i){
        printf("i=%d pid=%d\n ",i,getpid());
    }
    return 0;
}

```

运行结果：

![image-20220221153221883](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220221153221883.png)

可以看到子进程只输出了echo的内容，没有输出其余内容，因为用户区已经被替换掉



# 其余函数

```cpp
 		#include <unistd.h>

       extern char **environ;

       int execl(const char *pathname, const char *arg, ...
                       /* (char  *) NULL */);
       int execlp(const char *file, const char *arg, ...
                       /* (char  *) NULL */);
       int execle(const char *pathname, const char *arg, ...
                       /*, (char *) NULL, char *const envp[] */);
       int execv(const char *pathname, char *const argv[]);
       int execvp(const char *file, char *const argv[]);
       int execvpe(const char *file, char *const argv[],
                       char *const envp[]);
```

execlp用例：

`execlp("ps","ps","aux",NULL);`

## 后缀意义：

l(list) 			参数地址列表，以空指针结尾

v(vector)		存有各参数地址的指针数组的地址

p(path)			按PATH环境变量指定的目录搜索可执行文件

e(environment)		存有环境变量字符串地址的指针数组的地址