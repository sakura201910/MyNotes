# wait()

```cpp
/*
    #include <sys/types.h>
    #include <sys/wait.h>
    pid_t wait(int *wstatus);
        功能：等待任意一个子进程结束，如果子进程结束了，此函数会回收子进程的资源
        参数： int *wstatus
            进程退出时的状态信息，传入的是一个int类型的地址，传出参数
        返回值：
            - 成功：返回被回收的子进程的id
            - 失败：-1(所有的子进程都结束，调用函数失败)

    调用wait函数的进程会被挂起（阻塞）直到他的一个子进程退出或者收到一个不能被忽视的信号
    如果没有子进程了，函数会立即返回(-1);如果子进程都已经结束了，也会立即返回(-1)
*/
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    //有一个父进程想要创建5个子进程

    pid_t pid;

    //创建5个子进程
    for (int i = 0; i < 5; ++i)
    {
        pid = fork();
        if (pid == 0)
        {
            break;
        }
    }
    if (pid > 0)
    {
        while (1)
        {
            printf("parent pid =%d\n", getpid());
            int ret = wait(NULL);
            if (ret == -1)
                break; //所有子进程释放后，返回值为-1，此时可以break

            // 宏的使用方法：
            // int st;
            // int ret=wait(&st);    
            // if(WIFEXITED(s)){
            //     //是不是正常退出
            //     printf("退出的状态码 %d\n",WEXITSTATUS(st));
            // }
            
            printf("child die , pid = %d\n", ret);
        }
    }
    else if (pid == 0)
    {
        printf("child ,pid = %d\n", getpid());
    }

    return 0;
}

```

# wait相关宏

![image-20220221215052643](https://gitee.com/sakuryu/img-bed/raw/master/img/image-20220221215052643.png)

# waitpid()

```cpp
/* 
    #include <sys/types.h>
    #include <sys/wait.h>
    pid_t waitpid(pid_t pid, int *wstatus, int options); 
        功能：回收指定进程号的子进程，可以设置是否阻塞
        参数：
            - pid 
                pid > 0 :   某个子进程的 pid
                pid = 0 :   回收当前进程组的所有子进程
                pid = -1 :  回收所有子进程，相当与 wait()
                pid < -1 :  某个进程组的id的绝对值，回收指定进程组中的子进程
            - options: 设置阻塞或者非阻塞
                0 ：        阻塞
                WNOHANG :   非阻塞
            - 返回值：
                > 0 :       返回子进程的id
                = 0 :       options=WNOHANG,表示还有子进程活着
                = -1 ：     错误，或者没有子进程了
*/
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    //有一个父进程想要创建5个子进程

    pid_t pid;

    //创建5个子进程
    for (int i = 0; i < 5; ++i)
    {
        pid = fork();
        if (pid == 0)
        {
            break;
        }
    }
    if (pid > 0)
    {
        while (1)
        {
            printf("parent pid =%d\n", getpid());
            int st=0;
            int ret = waitpid(-1,&st,WNOHANG);//设置非阻塞

            if (ret == -1){
                break; //所有子进程释放后，返回值为-1，此时可以break
            } else if(ret == 0){
                //还有子进程存在
                continue;
            } else if(ret > 0){
                printf("child die , pid = %d\n", ret);
            }
        }
    }
    else if (pid == 0)
    {
        printf("child ,pid = %d\n", getpid());
    }

    return 0;
}

```

