# tips

## htons （主机到短整型转换）
### uint16_t htons(uint16_t hostshort);
function converts the unsigned short integer hostshort from host byte order to network byte order.
## inet_pton (呈现形式到数值)
### int inet_pton(int af, const char *src, void *dst);
convert IPv4 and IPv6 addresses from text to binary form
## getaddrinfo (实现协议无关)
### int getaddrinfo(const char *node, const char *service, const struct addrinfo *hints, struct addrinfo **res);
### void freeaddrinfo(struct addrinfo *res);
### int listen(int sockfd, int backlog);
The backlog argument defines the maximum length to which the queue of pending connections for sockfd may grow.  If a connection request arrives when the queue is full, the client may receive an error with an indication of ECONNREFUSED or, if the underlying protocol supports retransmission, the request may be ignored so that a later reattempt at connection succeeds.
## raw socket
绕过传输层直接使用IPV4或IPV6协议收发数据，可以自己组帧。
## 本地ip地址一般用*通配本地所有ip，通配地址通过在调用bind之前将套接字中的地址结构中的ip地址字段设置成INADDR_ANY
## 套接字发送缓冲区(SO_SNDBU)
如果该套接字的发送缓冲区容不下改应用进程的所有数据，该应用程序将进入睡眠（默认套接字阻塞），内核将不从write系统调用中返回，直到进程缓冲区中的所有数据都复制到套接字发送缓冲区，因此write调用成功返回仅仅表示可以重新使用原来的应用缓冲区。
![新建位图图像 (17).bmp](attachments\41cf9bc7.bmp)

## ipv4套接字地址结构
```
<netinet/in.h>

struct in_addr {
in_addr s_addr; //32bits

};

struct sockaddr_in {
  uint8_t sin_len; //16 bytes
  sa_famliy_t sin_family; //AF_INET
  in_port_t sin_port; //port number
  struct in_addr sin_addr; //32bit addr
  char sin_zero[8]; //unused, compliance with other protocal
};
```
## 通用套接字地址结构
```
<sys/socket.h>

struct sockaddr {
  uint8_t sa_len; //16 bytes
  sa_family_t sa_family; //address family AF_xxx value
  char sa_data[14]; //protocol specific address
}
```
## ipv6套接字地址结构
```
struct in6_addr {
  uint8_t s6_addr[16]; //128bit ipv6 address, network byte ordered
};

struct sockaddr_in6 {
  uint8_t sin6_len; //28 bytes
  sa_family_t sin6_family; //AF_INET6
  in_port_t sin6_port; //transport layer port
  
  uint32_t sin6_flowinfo; //flow information undefined
  struct in6_addr sin6_addr; // ipv6 address
  uint32_t sin6_scope_id; //set of interfaces for a scope
  
};
```
## connect()
该函数需要指定size，该size是为了告诉内核需要复制多少套接字地址结构数据到内核。

## getpeername
- int getpeername(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
getpeername() returns the address of the peer connected to the socket sockfd, in the buffer pointed to by addr.  The addrlen argument should be initialized to indicate the amount of space pointed to by addr.  On return it contains the actual size of the name returned (in bytes).  The name is truncated if the buffer provided is too small.

## 字节序转换（主要用于port的转换）
- 主机字节序->网络字节序
uint16_t htons(uint16_t value);
uint32_t htonl(uint32_t value);

- 网络字节序->主机字节序
uint16_t ntohs(uint16_t value);
uint32_t ntohl(uint16_t value);

## 字节操纵函数
void bzero(void *dest, size_t nbytes); //源自Berkeley

## 地址转换函数（在点分十进制可识别的地址和内核可用的数据地址之间转换）
```
#include <arpa/inet.h>

int inet_aton(const char *strptr, struct in_addr *addrptr);
char *inet_ntoa(struct in_addr inaddr);
in_addr_t inet_addr(const char * strptr);

```
## 新的网络地址转换函数（iPv4和iPv6都适用）
```c
int inet_pton(int family, const char *strptr, void *addrptr);
const char *inet_ntop(int family, const void *addrptr, char *strptr, size_t len);
```

## 按照字节数读写网络数据demo
```c
ssize_t readn(int fd, void *vptr, size_t n)
{
  size_t nleft; //剩下的字节数
  ssize_t nread; //每一次读到的字节数
  char *ptr;
  nleft = n;
  
  while(nleft > 0)
  {
    if(nread = read(fd, ptr, nleft) < 0)
    {
      if(errno == EINTR) //被信号中断
      {
        nread = 0;
      }
      else
      {
        return -1;
      }
    
    }
    else if (nread == 0)
    {
      break;
    }
    
    nleft -= nread;
    ptr += nread;
  
  }
  
  return (n - nleft);

}

```
```
size_t writen(int fd, const void *vptr, size_t n)
{
  size_t nleft;
  ssize_t nwritten;
  const char *ptr;
  
  ptr = vptr;
  nleft = n;
  while(nleft > 0)
  {
    if((nwritten = write(fd, ptr, n, nleft)) <= 0)
    {
      if(nwritten < 0 && errno == EINTR) //被信号中断
      {
        nwritten = 0;  
      }
      else
      {
        return (-1)
      }
    
    }
    nleft -= nwritten;
    ptr += nwritten;
  
  }
  reuturn (n);
}
```
## 如果TCP客户端没有收到SYN分节的响应，则返回ETIMEDOUT。
![新建位图图像 (6).bmp](attachments\d48560c0.bmp)
## 如果客户端的syn的响应是RST，表示服务器主机在我们指定的端口上没有进程在等待与之连接，这是一种硬错误，客户一收到RST马上就要返回ECONNREFUSED
## 如果一个TCP客户端或服务器未曾调用bind绑定一个端口，当调用connect或者listen时，内核就会为相应的套接字选择一个临时端口。让内核选择一个临时端口对于TCP客户端来说是正常的。除非需要用一个预留的端口，对于TCP服务器来说却极为罕见。
## listen
内核为审核一个给定的监听套接字维护两个队列
（1）未完成连接队列。
每个syn分节对应其中的一项。服务器正在等待完成相应的tcp三路握手过程。这些套接字处于sync_rcvd状态。不能超过listen的第二个参数。
（2）已完成连接队列。
每个已完成tcp三路握手过程的客户对应其中的一项。这些套接字处于established状态。
## accept
int accept(int sockfd, struct sockadddr *cliaddr, socklen_t *adddrlen);
- 该函数由tcp服务器调用，用于从已完成连接队列头返回下一个已完成连接。
- 第二个参数和第三个参数用来返回已连接的对端进程的协议地址。
- 如果accept成功，返回值是由内核自动生成的全新的描述符。代表与返回客户的tcp连接。
- 第一个参数为监听套接字，（有socket创建，bind和listen来绑定监听）
## fork
- 一次调用返回两次，在子进程中会返回0，父进程中返回子进程pid。
- 父进程中调用fork之前打开的所有描述符在fork返回后由子进程分享。网络服务器利用这个特性。父进程调用accept之后调用fork，所接受的已连接套集资随后在父进程和子进程中共享。通常情况下子进程接着读写这个已连接套接字，父进程直接关闭这个已连接的套接字。
- fork的典型用法：
  1. 一个进程创建一个自身的副本，这样每个副本都可以在另一个副本执行其他任务的同时处理各自的操作。
  2. 先创建一个自身的副本，再通过exec函数中的某一个替换成新的程序。exec程序只有在出错时才返回到调用者。否则是一个新程序的起点。
![新建位图图像 (18).bmp](attachments\8b85bdee.bmp)
  3. 进程在调用exec之前打开的描述符通常在exec之后继续保持打开。默认使用fcntl设置FD_CLOEXEC，描述符是禁止掉的
## close
- close一个tcp套接字的默认行为是把该套接字标记成已关闭，然后立即返回到调用进程。该套接字描述符不能再由调用进程使用。并对其引用计数减一。当引用计数等于0时在tcp连接上会发FIN。
- 如果想直接在tcp上发送一个FIN那么可以调用shutdown函数来替代close。
- 如果父进程对每个由accept返回到已连接的套接字都不调用close，那最终并发服务器会耗尽可用的描述符。并且没有一个客户的连接会终止。因为最终引用计数都不会变为0.
![新建位图图像 (7).bmp](attachments\57d6d0c5.bmp)

## getsockname
当没有调用bind的tcp客户端上，connect成功后，该接口用于获得由内核赋予该链接的本地IP地址，本地端口号。
## getpeername
当服务器端通过accept之后。它能够获得客户身份的唯一途径就是条用getpeername

## I/O 模型
### 阻塞式I/O
![新建位图图像.bmp](attachments\d97639af.bmp)
### 非阻塞式I/O
![新建位图图像 (2).bmp](attachments\67ccc8ea.bmp)
### I/O复用模型
![新建位图图像 (3).bmp](attachments\b56c5921.bmp)
### 信号驱动式I/O模型
![新建位图图像 (4).bmp](attachments\c52b941d.bmp)
### 异步I/O模型
![新建位图图像 (5).bmp](attachments\fa9865dc.bmp)

### 异步I/O和信号驱动I/O的主要区别
信号驱动式I/O是由内核通知我们合适可以启动一个I/O操作，而异步I/O模型是由内核通知我们I/O操作何时完成。


## select
```c
struct timeval {
  long tv_sec; /*second*/
  long tv_usec; /*microseconds*/
};
int select(int maxfdpl, fd_set *readset, fd_set *writesst, fd_set *exceptset, const struct timeval *timeout);
```
- 虽然给了一个微秒级的分辨率但是内核不会准确。maxfdpl参数指定了待测试的描述符个数（最大描述符+1），他的值是待测试的最大描述符加1，描述符0,1,2,..., maxfdpl-1都将被测试。
- 调用该函数时，我们指定所关心的描述符的值，该函数返回时，结果将指示哪些描述已就绪，该函数返回后，我们使用FD_ISSET宏来测试fd_set数据类型中的描述符。描述符集内任何与未就绪对应的位返回时均清0，为此每次重新调用select函数时，我们都需要再次把关心的描述符集相关位置1(FD_SET)。
- 描述符就绪条件
  - 准备好读
    - 该套接字接受缓冲区中的数据字节大于等于套接字接收缓冲区低水位标记
    - 该连接的读半部关闭（接收到FIN的TCP连接）。对这样的套接字读操作将不阻塞并返回0（EOF）
    - 该套接字是一个监听套接字且已完成连接数不为0，对这样的套接字accept不会阻塞。
    - 有一个套接字的错误待处理
  - 准备好写
    - 发送缓冲区中可用空间字节数大于等于套接字发送缓冲区低水位标记
    - 该连接的写半部关闭。对这样的套接字的写操作将产生sigpipe信号
    - 使用非阻塞式connect的套接字已建立连接。或者connect已经宣告失败
    - 有错误待处理
- 如果对端TCP发送了一个FIN（对端进程终止），那么套接字变为可读，并read返回0。
- 如果对端TCP发送了一个RST（对端主机崩溃并重启），那么该套集资变为可读，并read返回-1，而errno中海油确切的错误码。
## FD_ZERO FD_SET FD_CLR FD_ISSET
```c
void FD_ZERO(fd_set *fdset); //clear all bits in fdset
void FD_SET(int fd, fd_set *fdset); //turn on the bit for fd in fdset
void FD_CLR(int fd, fd_set *fdset); //turn off the bit for fd in fdset
int FD_ISSET(int fd, fd_set *fdset); //is the bit for fd on in fdset
```
描述符集初始化很重要
FD_ZERO(&fdset);

## shutdown
- close让描述符引用计数减1，仅在计数变为0时才关闭套接字。shutdown不管套接字就直接激发TCP正常连接终止序列
- close终止读写两个方向的数据传输。
![新建位图图像 (8).bmp](attachments\4013a83e.bmp)
- shutdown可以分别关闭不同方向
  - SHUT_RD 
  关闭读这一半，套接字中不再有数据可读。而且套接字接收区的现有数据都丢弃。由该套接字接收到的来自对端的数据都会被确认，然后悄悄丢弃。
  - SHUT_WR
  关闭连接写这一半，对于TCP套接字，这称为半关闭。当前留在套接字发送缓冲区的数据将被发送掉，后跟TCP的正常连接终止序列。
  - SHUT_RDWR
  连接的读半部和写半部都关闭这与调用shutdown两次等效第一次调用SHUT_RD，第二次调用指定SHUT_WR

## poll
```c
struct pollfd{
  int fd; //descriptor to check
  short events; //events of interest on fd
  short revents; //events that occurred on fd
};
int poll(struct pollfd *fdarray, unsigend long nfds, int timeout);

```

![新建位图图像 (10).bmp](attachments\13f4b135.bmp)

## 套接字选项
```
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t option);
```
![新建位图图像.bmp](attachments\5e673fe9.bmp)
![新建位图图像 (2).bmp](attachments\8d3a1be7.bmp)
![新建位图图像 (3).bmp](attachments\4c2c7123.bmp)

## 给网络接口添加路由

route add -host 192.168.1.11 dev xxxxinterface

当一个机器在一个网段有多个网口时不要给每个网口都分ip，添加路由就可以

-host表示该地址是主机地址，-dev表示该地址是子网地址