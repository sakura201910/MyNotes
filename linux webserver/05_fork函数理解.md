# fork的实现

fork直接拷贝父进程，但进程id不同，一开始父子进程使用同样的用户区，但写操作时，会拷贝用户区

```cpp
/*
    #include <sys/types.h>
    #include <unistd.h>

    pid_t fork(void); 
        函数作用：用于创建子进程
        返回值：
            fork()的返回值会返回2次。
                一次在父进程中，一次在子进程中
                    父进程中返回创建子进程的id
                    子进程中返回0
            如何区分父进程和子进程：通过fork()的返回值
            在父进程中返回-1，表示创建子进程失败，置errno
        
        父子进程对于变量是不是共享的？
            - 刚开始的时候是共享 ，如果修改数据后不共享
            - 读时共享(子进程刚创建，2进程没有进行任何写操作)，写时拷贝
*/
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char const *argv[])
{
    //创建子进程
    pid_t pid =fork();

    //判断是父进程还是子进程
    if(pid>0){
        //大于0，返回的是创建的子进程的进程号
        printf("father id = %d ,pid = %d\n",getpid(),getppid());
    }
    else if(pid==0){
        //当前是子进程
        printf("child id = %d ,pid = %d\n",getpid(),getppid());
    }

    for(int i=0;i<3;i++){
        printf("i=%d pid = %d\n",i,getpid());
        sleep(1);
    }
    return 0;
}

```

程序运行结果：

![image-20220221143615857](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220221143615857.png)

父子进程轮转使用时间片

# 写时拷贝

fork()是写时拷贝，在写之前父子进程使用同样的虚拟地址空间，写之后拷贝虚拟地址空间

**注意：**

- fork之后父子进程共享文件
- fork产生子进程与父进程相同的文件描述符指向相同文件，引用计数增加，共享文件偏移指针

# 父子进程的关系：

- 区别：
    - 1.fork()函数返回值不同
                        父进程中 >0:返回子进程id
                        子进程中 =0
    - 2.pcb中的一些数据
                        当前进程的id pid 
                        当前进程的pid ppid 
                        信号集
    
- 共同点：
       某些状态下，子进程刚刚被创建出来，还没有执行任何的写数据操作
       - 用户区的数据
       - 文件的描述列表