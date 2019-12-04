---
layout: post
title: 套接字和地址：像电话和电话号码一样理解它们
tags: network tcp/ip C 网络编程实战
categories: network
---

* TOC
{:toc}

#socket到底是什么？

客户端和服务器工作的核心逻辑：
![客户端服务器逻辑][客户端服务器逻辑]
客户端发起连接请求前，服务器端必须初始化好。服务器端初始化的过程是：首先初始化socket，之后服务器端需要执行bind函数，将自己的服务能力绑定在一个众所周知的地址和端口上，紧接着，服务器端执行listen操作，将原先的socket转化为服务端的socket，服务端最后阻塞在accept上等待客户端请求的到来。

此时，客户端已经准备就绪。客户端需要先初始化socket，再执行connect向服务器端的地址和端口发起连接请求，这里的地址和端口必须是客户端预先知晓的。这个过程就是著名的三次握手。

一旦三次握手完成，客户端和服务器端建立连接，就进入了数据传输过程。
具体来说，客户端进程向操作系统内核发起write字节流写操作，内核协议栈将字节流通过网络设备传输到服务器端，服务器端从内核得到信息，将字节流从内核读入到进程中，并开始业务逻辑的处理，完成之后，服务器端再将得到的结果以同样的方式写给客户端。可以看到，一旦连接建立，数据的传输就不再是单向的，而是双向的，这也是TCP的一个显著特性。

当客户端完成和服务器端的交互后，比如执行一次Telnet操作，或者一次HTTP请求，需要和服务器端断开连接时，就会执行close函数，操作系统内核此时会通过原先的连接链路向服务器端发送一个FIN包，服务器收到之后执行被动关闭，这时候整个链路处于半关闭状态，此后，服务器端也会执行close函数，整个链路才会真正关闭。半关闭的状态下，发起close请求的一方在没有收到对方FIN包之前都认为连接是正常的；而在全关闭的状态下，双方都感知连接已经关闭。

socket是用来建立连接，传输数据的唯一途径。



# 通用套接字地址格式

```c
/* POSIX.1g 规范规定了地址族为 2 字节的值.  */
typedef unsigned short int sa_family_t;
/* 描述通用套接字地址  */
struct sockaddr {
    sa_family_t sa_family;  /* 地址族.  16-bit*/
    char sa_data[14];   /* 具体的地址值 112-bit */
}; 
```

sa_family参数：

- AF_LOCAL: 表示本地地址，对应的是Unix套接字。一般用于本地socket通信，也可以写成AF_UNIX、AF_FILE
- AF_INET: 因特网使用的IPv4地址
- AF_INET6: 因特网使用的IPv6地址

AF\_的含义是Address Family,  但在很多情况下，我们也会看到以PF\_表示的宏，比如PF\_INET、PF\_INET6等，实际上PF\_的意思是Protocol Family，也就是协议族的意思。我们用AF\_xxx这样的值来初始化socket地址，用PF\_xxx这样的值初始化socket。在<sys/socket.h>头文件中，可以清楚的看到，这两个值是一一对应的：

```c
/* 各种地址族的宏定义  */
#define AF_UNSPEC 		PF_UNSPEC
#define AF_LOCAL  		PF_LOCAL
#define AF_UNIX   		PF_UNIX
#define AF_FILE   		PF_FILE
#define AF_INET   		PF_INET
#define AF_AX25   		PF_AX25
#define AF_IPX    		PF_IPX
#define AF_APPLETALK  PF_APPLETALK
#define AF_NETROM 		PF_NETROM
#define AF_BRIDGE 		PF_BRIDGE
#define AF_ATMPVC 		PF_ATMPVC
#define AF_X25    		PF_X25
#define AF_INET6  		PF_INET6
```

sockaddr是一个通用的地址结构，通用的意思是适用于多种地址族。

# IPv4套接字格式地址

常用的IPv4地址族的结构：

```c
/* IPV4 套接字地址，32bit 值.  */
typedef uint32_t in_addr_t;
struct in_addr {
    in_addr_t s_addr;
};
  
/* 描述 IPV4 的套接字地址格式  */
struct sockaddr_in {
    sa_family_t sin_family; /* 16-bit */
    in_port_t sin_port;     /* 端口口  16-bit*/
    struct in_addr sin_addr;    /* Internet address. 32-bit */

    /* 这里仅仅用作占位符，不做实际用处  */
    unsigned char sin_zero[8];
};
```

sin_port参数是short类型，最大支持2的16次方，所以支持寻址的端口号最多就是65535。保留端口就是大家约定俗成的，已经被对应服务广为使用的端口，比如ftp的21端口，ssh的22端口，http的80端口等。一般而言，大于5000的端口可以作为我们自己应用程序的端口使用。

保留端口：

```c
/* Standard well-known ports.  */
enum {
    IPPORT_ECHO = 7,    /* Echo service.  */
    IPPORT_DISCARD = 9,   /* Discard transmissions service.  */
    IPPORT_SYSTAT = 11,   /* System status service.  */
    IPPORT_DAYTIME = 13,  /* Time of day service.  */
    IPPORT_NETSTAT = 15,  /* Network status service.  */
    IPPORT_FTP = 21,    /* File Transfer Protocol.  */
    IPPORT_TELNET = 23,   /* Telnet protocol.  */
    IPPORT_SMTP = 25,   /* Simple Mail Transfer Protocol.  */
    IPPORT_TIMESERVER = 37, /* Timeserver service.  */
    IPPORT_NAMESERVER = 42, /* Domain Name Service.  */
    IPPORT_WHOIS = 43,    /* Internet Whois service.  */
    IPPORT_MTP = 57,




    IPPORT_TFTP = 69,   /* Trivial File Transfer Protocol.  */
    IPPORT_RJE = 77,
    IPPORT_FINGER = 79,   /* Finger service.  */
    IPPORT_TTYLINK = 87,
    IPPORT_SUPDUP = 95,   /* SUPDUP protocol.  */


    IPPORT_EXECSERVER = 512,  /* execd service.  */
    IPPORT_LOGINSERVER = 513, /* rlogind service.  */
    IPPORT_CMDSERVER = 514,
    IPPORT_EFSSERVER = 520,


    /* UDP ports.  */
    IPPORT_BIFFUDP = 512,
    IPPORT_WHOSERVER = 513,
    IPPORT_ROUTESERVER = 520,


    /* Ports less than this value are reserved for privileged processes.  */
    IPPORT_RESERVED = 1024,


    /* Ports greater this value are reserved for (non-privileged) servers.  */
    IPPORT_USERRESERVED = 5000
```

# IPv6套接字地址格式

IPv6的地址结构：

```c
struct sockaddr_in6 {
    sa_family_t sin6_family; /* 16-bit */
    in_port_t sin6_port;  /* 传输端口号 # 16-bit */
    uint32_t sin6_flowinfo; /* IPv6 流控信息 32-bit*/
    struct in6_addr sin6_addr;  /* IPv6 地址 128-bit */
    uint32_t sin6_scope_id; /* IPv6 域 ID 32-bit */
};
```

本地套接字格式：

```c
struct sockaddr_un {
    unsigned short sun_family; /* 固定为 AF_LOCAL */
    char sun_path[108];   /* 路径名 */
};
```

# 几种套接字地址格式比较

![套接字格式比较][套接字格式比较]





[上一篇][上一篇]
[下一篇][下一篇]





[客户端服务器逻辑]: {{"/network/tcp_server_client.jpg" | prepend: site.imgrepo }}

[套接字格式比较]: {{"/network/sockaddr.png" | prepend: site.imgrepo }}

[上一篇]:https://linuxxiaoyu.github.io/network/network-programing-1/
[下一篇]:https://linuxxiaoyu.github.io/network/network-programing-3/







