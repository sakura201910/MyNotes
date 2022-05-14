# TCP与UDP的区别

UDP:用户数据报协议，面向无连接，可以单播，多播，广播，面向数据报，不可靠

TCP:传输控制协议，面向连接的，可靠的,基于字节流，仅支持单播传输



|              |              UDP               |             TCP              |
| :----------: | :----------------------------: | :--------------------------: |
| 是否创建连接 |             无连接             |           面向连接           |
|   是否可靠   |             不可靠             |             可靠             |
| 连接对象个数 | 一对一、多对一、一对多、多对多 |         仅支持一对一         |
|   传输方式   |           面向数据报           |          面向字节流          |
|   首部开销   |             8字节              |          至少20字节          |
|   适用场景   |   实时应用（视频会议、直播）   | 可靠性高的应用（文件传输……） |

# TCP通信流程

## 服务器端

1. 创建一个用于监听的套接字
     - 监听:监听有客户端的连接
     - 套接字:这个套接字其实就是一个文件描述符
2. 将这个监听文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息）
    - 客户端连接服务器的时候使用的就是这个IP和端口
3. 设置监听，监听的fd开始工作
4. 阻塞等待，当有客户端发起连接，解除阻塞，接受客户端的连接，会得到一个和客户端通信的套接字(fd)
5. 通信
    - 接收数据
    - 发送数据
6. 通信结束，断开连接

##  客户端

1. 创建一个用于通信的套接字(fd)
2. 连接服务器，需要指定连接的服务器的IP和端口
3. 连接成功了，客户端可以直接和服务器通信
    - 接收数据
    - 发送数据
4. 通信结束，断开连接

![image-20220513201046116](https://cdn.konyue.site/image-20220513201046116.png)

# 套接字函数

```cpp
/*

int socket(int domain, int type, int protocol);
    - 功能:创建一个套接字
    - 参数:
        - domain:协议族
            AF_INET : ipv4
            AF_INET6 : ipv6
            AF_UNIX，AF_LOCAL :本地套接字通信(进程间通信)
        - type:通信过程中使用的协议类型
            - SOCK_STREAM︰流式协议
            - SOCK_DGRAM:报式协议
        - protoco1 :具体的一个协议。一般写0
            - SOCK_STREAM:流式协议默认使用TCP
            - SOCK_DGRAM:报式协议默认使用UDP
        -返回值:
            -成功:返回文件描述符，操作的就是内核缓冲区。
            -失败:-1

int bind(int sockfd, const struct sockaddr *addr,socklen_t addrlen);
    - 功能:绑定，将fd和本地的IP+端口进行绑定
    - 参数:
        - sockfd :通过socket函数得到的文件描述符
        - addr :需要绑定的socket地址，这个地址封装了ip和端口号的信息
        - addr len :第二个参数结构体占的内存大小

int listen(int sockfd, int backlog);
    -功能: 监听这个socket上的连接
    -参数:
        - sockfd : 通过socket()函数得到的文件描述符
        - backlog: 未连接的和已经连接的和的最大值
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    - 功能:接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接
    - 参数:
        - sockfd: 用于监听的文件描述符
        - addr: 传出参数，记录了连接成功后客户端的地址信息（ip, port)
        - addrlen: 指定第二个参数的对应的内存大小
    - 返回值:
        - 成功:用于通信的文件描述符
        - 失败:-1 

int connect(int sockfd, const struct sockaddr *addr,socklen_t addrlen);
    - 功能:客户端连接服务器
    - 参数:
        - sockfd: 用于通信的文件描述符
        - addr:  客户端要连接的服务器的地址信息
        - addrlen :第二个参数的内存大小
    – 返回值:
        成功0，失败-1

ssize_t write(int fd, const void *buf, size_t count);   //写数据
ssize_t read(int fd, void *buf, size_t count);      //读数据
*/
```

# TCP通信简单实现

```cpp
//服务端.c
/*
1. 创建一个用于监听的套接字
     - 监听:监听有客户端的连接
     - 套接字:这个套接字其实就是一个文件描述符
2. 将这个监听文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息）
    - 客户端连接服务器的时候使用的就是这个IP和端口
3. 设置监听，监听的fd开始工作
4. 阻塞等待，当有客户端发起连接，解除阻塞，接受客户端的连接，会得到一个和客户端通信的套接字(fd)
5. 通信
    - 接收数据
    - 发送数据
6. 通信结束，断开连接
*/

#include <sys/types.h>         
#include <sys/socket.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
int main()
{
    //创建套接字
    int lfd = socket(AF_INET,SOCK_STREAM,0);
    if(lfd==-1){
        perror("socket");
        exit(-1);
    }
    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;     //协议
    //inet_pton(AF_INET,"127.0.0.1",saddr.sin_addr.s_addr);
    saddr.sin_addr.s_addr = INADDR_ANY ;    //0.0.0.0
    saddr.sin_port  = htons(9999);      //端口

    //绑定
    int ret = bind(lfd,(struct sockaddr *)&saddr,sizeof(saddr));
    if(ret == -1){
        perror("bind");
        exit(-1);
    }

    //监听
    ret = listen(lfd,8);
    if(ret == -1){
        perror("listen");
        exit(-1);
    }

    //接收客户端连接
    struct sockaddr_in clientaddr;
    int lenn = sizeof(clientaddr);
    int cfd = accept(lfd,(struct sockaddr *)&clientaddr,&lenn);
    if(cfd == -1){
        perror("accept");
        exit(-1);
    }

    //输出客户端信息
    char cip[16];
    inet_ntop(AF_INET,&clientaddr.sin_addr.s_addr,cip,sizeof(cip));
    unsigned short cport = ntohs(clientaddr.sin_port);
    printf("cip is %s , port is %d\n",cip,cport);

    //获取客户端数据
    char recvBuf[1024] = {0};
    int len = read(cfd,recvBuf,sizeof(recvBuf));
    if(len == -1){
        perror("read");
        exit(-1);
    }else if(len >0){
        printf("recv buf: %s",recvBuf);
    }else if(len == 0){
        //表示客户端断开连接
        printf("client close...");
    }

    //向客户端发送数据
    char * date = "hello i am server\n";
    write(cfd,date,strlen(date));

    close(cfd);
    close(lfd);

    return 0;
}


//客户端.c
/*
1. 创建一个用于通信的套接字(fd)
2. 连接服务器，需要指定连接的服务器的IP和端口
3. 连接成功了，客户端可以直接和服务器通信
    - 接收数据
    - 发送数据
4. 通信结束，断开连接
*/

#include <sys/types.h>         
#include <sys/socket.h>
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
int main()
{
    //创建套接字
    int lfd = socket(AF_INET,SOCK_STREAM,0);
    if(lfd==-1){
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in saddr;
    saddr.sin_family = AF_INET;     //协议
    inet_pton(AF_INET,"127.0.0.1",&saddr.sin_addr.s_addr);
    saddr.sin_port  = htons(9999);      //端口

    int ret = connect(lfd,(struct sockaddr *)&saddr,sizeof(saddr));
    if(ret == -1){
        perror("connect");
        exit(-1);
    }

    //向服务端发送数据
    char * date = "hello i am client\n";
    write(lfd,date,strlen(date));

    //获取客户端数据
    char recvBuf[1024] = {0};
    int len = read(lfd,recvBuf,sizeof(recvBuf));
    if(len == -1){
        perror("read");
        exit(-1);
    }else if(len >0){
        printf("recv server buf: %s",recvBuf);
    }else if(len == 0){
        //表示客户端断开连接
        printf("server close...");
    }

    close(lfd);

    return 0;
}
```





