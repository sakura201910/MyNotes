# TCP状态转换

![image-20220513200637522](https://cdn.konyue.site/image-20220513200637522.png)

## CLOSED

起始点，在超时或者连接关闭时候进入此状态，这并不是一个真正的状态，而是这个状态图的假想起点和终点

## LISTEN

起始点，在超时或者连接关闭时候进入此状态，这并不是一个真正的状态，而是这个状态图的假想起点和终点

## **SYN_SENT**

第一次握手发生阶段，客户端发起连接。客户端调用 connect，发送 SYN 给服务器端，然后进入 SYN_SENT 状态，等待服务器端确认（三次握手中的第二个报文）。如果服务器端不能连接，则直接进入CLOSED状态**SYN_RCVD**

第二次握手发生阶段，服务器端接收客户端的 SYN，此时服务器由 LISTEN 进入 SYN_RCVD状态，同时服务器端回应一个 ACK+SYN 给客户端。**状态图中还描绘了这样一种情况，当客户端在发送 SYN 的同时也收到服务器端的 SYN请求，即两个同时发起连接请求，那么客户端就会从 SYN_SENT 转换到 SYN_REVD 状态**

## **ESTABLISHED**

第三次握手发生阶段，客户端接收到服务器端的 ACK+SYN 包后，**回复一个 ACK 确认包，客户端立刻进入 ESTABLISHED 状态**，表明客户端这边已经准备好，但TCP 需要两端都准备好才可以进行数据传输。**服务器端收到客户端的 ACK 之后会从 SYN_RCVD 状态转移到 ESTABLISHED 状态**，表明服务器端也准备好进行数据传输了。这样客户端和服务器端都是 ESTABLISHED 状态，就可以进行后面的数据传输了。所以 ESTABLISHED 也可以说是一个数据传送状态

## **FIN_WAIT_1**

主动关闭的一方（执行主动关闭的一方既可以是客户端，也可以是服务器端，这里以客户端执行主动关闭为例），终止连接时，发送 FIN 给对方，然后等待对方返回 ACK 。调用 close() 第一次挥手就进入此状态

## **CLOSE_WAIT**

接收到FIN 之后，同时发送ACK，被动关闭的一方进入此状态。之所以叫 CLOSE_WAIT 可以理解为被动关闭的一方此时正在等待上层应用程序发出关闭连接指令。前面已经说过，TCP关闭是全双工过程，这里客户端执行了主动关闭，被动方服务器端接收到FIN 后也需要调用 close 关闭，这个 CLOSE_WAIT 就是处于这个状态，等待发送 FIN，发送了FIN 则进入 LAST_ACK 状态

## **FIN_WAIT_2**

主动端（这里是客户端）接收到被动方返回的 ACK 后进入此状态

## **LAST_ACK**

被动方（服务器端）发起关闭请求，发送 FIN给对方后，进入此状态

## **CLOSING**

主动方发送完FIN后进入FIN_WAIT_1，在等待被动方的ACK时，没有等到ACK，反而等来了被动方的FIN，意味着被动方也发起了主动关闭请求，主动方回复ACK后，进入CLOSING

# 半连接、半打开、半关闭

## 半连接

发生在TCP3次握手中。

如果A向B发起TCP请求，B也按照正常情况进行响应了，但是A不进行第3次握手，这就是半连接。

### 半连接攻击

 半连接，会造成B分配的内存资源就一直这么耗着，直到资源耗尽。

## 半打开

如果一方已经关闭或异常终止连接，而另一方却不知道。 我们将这样的TCP连接称为半打开（Half-Open）。

## 半关闭

TCP提供了连接的一端在结束它的发送后还能接收来自另一端数据的能力，这就是TCP的半关闭。

当一方关闭发送通道后，仍可接受另一方发送过来的数据，这样的情况叫“半关闭”。

1.  客户端发送FIN，另一端发送对这个FIN的ACK报文段。 此时客户端就处于半关闭。

2.  调用shutdown，shutdown的第二个参数为SHUT_WR时，为半关闭。

```cpp
int shutdown(int sockfd, int how);
	- sockfd:需要关闭的socket的描述符
	- how:允许为shutdown操作选择以下几种方式:
		- SHUT_RD(0):关闭sockfd上的读功能，此选项将不允许sockfd进行读操作。
			该套接字不再接收数据，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。
		- SHUT_wR(1):关闭sockfd的写功能，此选项将不允许sockfd进行写操作。进程不能在对此套接字发出写操作。
        - SHUT_RDMR(2):关闭sockfd的读写功能。相当于调用shutdown两次:首先是以SHUT_RD,然后以SHUT_WR。

```

使用close中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为0时才关闭连接。**shutdown不考虑描述符的引用计数，直接关闭描述符**。也可选择中止一个方向的连接，只中止读或只中止写。

**注意：**

1.如果有多个进程共享一个套接字，close每被调用一次，计数减1，直到计数为0时，也就是所用进程都调用了close，套接字将被释放。
⒉.在多进程中如果一个进程调用了shutdown(sfd,SHUT_RDWR)后，其它的进程将无法进行通信。但如果一个进程close(sfd)将不会影响到其它进程。

# 端口复用

用途：

- 防止服务器重启时之前绑定的端口未释放
- 程序突然退出而系统没有释放端口

```cpp
这个函数可以设置socket的属性，而不仅仅只用于端口复用
int getsockopt(int sockfd, int level, int optname,
                      void *optval, socklen_t *optlen);
int setsockopt(int sockfd, int level, int optname,
                      const void *optval, socklen_t optlen);
	- 参数：
        - socket：要操作的文件描述符
        - level :级别  -SOL_SOCKET(端口复用级别)
        - optname :选项名称
            - SO_REUSEADDR
            - SO_REUSEPORT
        - optval : 端口复用的值
            - 1：可复用
            - 0： 不可复用
        - optlen :optval的大小
           
   样例：
int optval = 1;
setsockopt(lfd,SOL_SOCKET,SO_REUSEPORT,&optval,sizeof(optval));
```

> 设置端口复用需要在服务器绑定之前

















