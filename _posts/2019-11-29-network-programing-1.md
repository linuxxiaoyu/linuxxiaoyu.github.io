---
layout: post
title: 网络编程模型：认识客户端-服务器网络模型的基本概念
tags: network tcp/ip 网络编程实战
categories: network
---

* TOC
{:toc}
# OSI & TCP/IP

![OSI][OSI]



# IP和端口

- 一个网络连接可以通过客户端-服务器端的IP和端口唯一确定，这叫做套接字对，按照下面的四元组表示：

  ```
  （clientaddr:clientport, serveraddr: serverport)
  ```

# 子网掩码

- 网络分类：

  ![网络分类][网络分类]



- 网络(network)表示一组IP共同的部分; 主机(host)表示的是一组IP中不同的部分。



- 子网掩码的二进制格式是前面一连串的1，后面跟着一连串的0。后来为了直观，出现了新的方法：将一个斜线放在IP地址后面，接着用一个十进制来表示网络的位数。类似192.0.2.12/30表示30个1，2个0，所以主机个数是4。

# 数据报和字节流

- TCP: 字节流套接字(Stream Socket)。TCP通过连接管理，拥塞控制，数据流与窗口管理，超时和重传等一系列精巧而详细的设计，提供了高质量的端对端的通信方式。

  

- UDP: 数据报套接字(Datagram Socket)。UDP比TCP的传输速度快。UDP也可以做到更高的可靠性，只不过这种可靠性，需要应用程序进行设计处理，比如对报文进行编号，设计Request-Ack机制，再加上重传等，再一定程度上可以达到更为高可靠的UDP程序。当然，这种可靠性和TCP相比还是有一定的差距，不过也可以弥补实战中UDP的一些不足。

  

# 总结

1. 网络编程需要牢牢树立起“客户端 ”和“服务器”模型，两者编程的方法和框架是明显不同的。
2. TCP连接是客户端 - 服务器的IP和端口四元组唯一确定的，IP是一台机器在网络世界的唯一标识。
3. 有两种截然不同的传输层协议，面向连接的“数据流”协议TCP，以及无连接的“数据报”协议UDP。



[下一篇][下一篇]



[OSI]: {{"/network/OSI.png" | prepend: site.imgrepo }}
[网络分类]: {{"/network/networkABC.png" | prepend: site.imgrepo }}

[下一篇]:https://linuxxiaoyu.github.io/network/network-programing-2/

