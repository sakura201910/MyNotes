# UDP实现

server.c

```cpp
/* 
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                    const struct sockaddr *dest_addr, socklen_t addrlen); 
    - 参数
        - socket    通信的fd
        - buf       要发送的数据
        - len       发送数据的长度
        - flags     0
        - dest_addr 通信另外一端的地址信息
        - addrlen   地址的内存大小

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                    struct sockaddr *src_addr, socklen_t *addrlen);
    和上一个类似

- 返回值
    -1 错误
    >0  发送/接收到的数据个数
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main(int argc, char const *argv[])
{
    int fd = socket(PF_INET,SOCK_DGRAM,0);
    if(fd == -1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in addr;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons(9999);
    addr.sin_family = AF_INET;

    int ret = bind(fd,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1){
        perror("bind");
        exit(-1);
    }
    // printf("ok\n");
    while(1){
        char buf[1024];
        struct sockaddr_in cliaddr;
        int len = sizeof(cliaddr);

        //接收数据
        // printf("recv wait\n");
        int num = recvfrom(fd,buf,sizeof buf,0,(struct sockaddr*)&cliaddr,&len);
        // printf("recv ok\n");
        if(num == -1){
            perror("recvfrom");
            exit(-1);
        }

        char ip[16];
        printf("client ip: %s:%d\n",inet_ntop(AF_INET,&cliaddr.sin_addr.s_addr,ip,sizeof ip),cliaddr.sin_port);

        printf("client say : %s\n",buf);
        //发送数据
        sendto(fd,buf,(sizeof buf)+1,0,(struct sockaddr*)&cliaddr,sizeof(cliaddr));

    }
    close(fd);

    return 0;
}

```

client.c

```cpp
/* 
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                    const struct sockaddr *dest_addr, socklen_t addrlen); 
    - 参数
        - socket    通信的fd
        - buf       要发送的数据
        - len       发送数据的长度
        - flags     0
        - dest_addr 通信另外一端的地址信息
        - addrlen   地址的内存大小

ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                    struct sockaddr *src_addr, socklen_t *addrlen);
    和上一个类似

- 返回值
    -1 错误
    >0  发送/接收到的数据个数
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

int main(int argc, char const *argv[])
{
    int fd = socket(PF_INET,SOCK_DGRAM,0);
    if(fd == -1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in saddr;
    saddr.sin_port = htons(9999);
    saddr.sin_family = AF_INET;
    inet_pton(AF_INET,"127.0.0.1",&saddr.sin_addr.s_addr);

    
    int i=0;
   
    while(1){
        //发送数据
        char sendbuf[1024];
        sprintf(sendbuf,"hello i an client %d",i++);
        
        sendto(fd,sendbuf,(sizeof sendbuf)+1,0,(struct sockaddr*)&saddr,sizeof(saddr));
        // printf("send ok\n");
        //接收数据
        int num  = recvfrom(fd,sendbuf,sizeof(sendbuf),0,NULL,NULL);
        printf("server say : %s\n",sendbuf);

        sleep(1);

    }
    close(fd);

    return 0;
}
```

# 广播

> 向子网中多台计算机发送消息，并且子网中所有的计算机都可以接收到发送方发送的消息，每个广播消息都包含一个特殊的IP地址，这个IP中子网内主机标志部分的二进制全部为1。
> a.只能在局域网中使用
> b.客户端需要绑定服务器广播使用的端口，才可以接收到广播消息。

使用\*.\*.\*.255    ip即可

`setsockopt`

# 组播（多播）

> 单播地址标识单个IP接口，广播地址标识某个子网的所有IP接口，多播地址标识一组IP接口。单播和广播是寻址方案的两个极端(要么单个要么全部)，多播则意在两者之间提供一种折中方案。多播数据报只应该由对它感兴趣的接口接收，也就是说由运行相应多播会话应用系统的主机上的接口接收。另外，广播一般局限于局域网内使用，而多播则既可以用于局域网，也可以跨广域网使用。
> a.组播既可以用于局域网，也可以用于广域网
> b.客户端需要加入多播组，才能接收到多播的数据

# 本地套接字通信

> 实现本地套接字通信
>
> - 有关进程间的通信
> - 没有关系的进程间通信
>
> 本地套接字的实现流程和网络套接字类似，一般采用TCP通信流程

```c
/*
// 本地套接字通信流程 - tcp

//服务器端
1.创建监听的套接字
    int lfd = socket(AF_UNIX/AF_LOCAL,SOCK_STREAM,0);
2.监听的套接字绑定本地的套接字文件
    struct sockaddr_un addr;
    //绑定成功之后，指定的sun_path中的套接字文件会自动生成
    bind(lfd,addr,len);
3.监听
    listen(lfd,100);
4.等待并接受客户端连接请求
    struct sockaddr_un cliadrr;
    int cfd = accept(ldf,&cliaddr,len);
5. 通信
    接收数据：read/recv
    发送数据: write/send
6.关闭连接
    close();

//客户端流程
1.创建通信的套接字
    inf fd = socket(AF_UNIX/AF_LOCAL,SOCK_STREAM,0);
2.绑定本地的套接字文件
    struct sockaddr_un addr;
    //绑定成功之后，指定的sun_path中的套接字文件会自动生成
    bind(lfd,addr,len);
3.连接服务器
    struct sockaddr_un serveraddr;
    connect(&serveraddr);
4.通信
    接收数据：read/recv
    发送数据：write/send
5.关闭连接
    close();
*/
```

工作中本地套接字会创建server.sock以及client.sock的空文件，利用缓冲区进行通信

服务器

```cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/un.h>

int main(int argc, char const *argv[])
{
    unlink("server.sock");
    //创建socket
    int lfd = socket(AF_LOCAL,SOCK_STREAM,0);
    if(lfd == -1){
        perror("socket");
        exit(-1);
    }

    //绑定
    struct sockaddr_un addr;
    addr.sun_family = AF_LOCAL;
    strcpy(addr.sun_path,"server.sock");
    int ret = bind(lfd,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1){
        perror("bind");
        exit(-1);
    }

    //监听
    ret = listen(lfd,100);
    
    //同意
    struct sockaddr_un cliaddr;
    int len = sizeof(cliaddr);
    int cfd =  accept(lfd,(struct sockaddr*)&cliaddr,&len);
    if(cfd == -1){
        perror("accept");
        exit(-1);
    }

    printf("client sockfilename:  %s\n",cliaddr.sun_path);

    //通信
    while(1){
        char buf[128];
        int len = recv(cfd,buf,sizeof(buf),0);

        if(len ==-1){
            perror("recv");
            exit(-1);
        }else if( len == 0){
            printf("client close...\n");
            break;
        }else if(len >0){
            printf("cli say : %s\n",buf);
            send(cfd,buf,len,0);
        }

    }
    close(lfd);
    close(cfd);
    return 0;
}
```

客户端

```cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/un.h>

int main(int argc, char const *argv[])
{
    unlink("client.sock");
    int cfd = socket(AF_LOCAL,SOCK_STREAM,0);
    if(cfd == -1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_un addr;
    addr.sun_family = AF_LOCAL;
    strcpy(addr.sun_path,"client.sock");
    int ret = bind(cfd,(struct sockaddr *)&addr,sizeof(addr));
    if(ret == -1){
        perror("bind");
        exit(-1);
    }

    struct sockaddr_un seraddr;
    seraddr.sun_family =AF_LOCAL;
    strcpy(seraddr.sun_path,"server.sock");
    ret = connect(cfd,(struct sockaddr*)&seraddr,sizeof seraddr);
    if(ret == -1){
        perror("connect");
        exit(-1);
    }
    
    int i=0;
    while(1){
        //发送数据
        char buf[128];
        sprintf(buf,"hello i am client %d\n",i++);
        send(cfd,buf,strlen(buf)+1,0);
        printf("%s\n",buf);

        int len = recv(cfd,buf,sizeof(buf),0);
        if(len ==-1){
            perror("recv");
            exit(-1);
        }else if( len == 0){
            printf("server close...\n");
            break;
        }else if(len >0){
            printf("server say : %s\n",buf);
           // send(cfd,buf,len,0);
        }
        sleep(1);
    }
    close(cfd);
    return 0;
}
```



