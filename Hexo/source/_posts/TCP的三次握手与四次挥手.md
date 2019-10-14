---
title: TCP的三次握手与四次挥手
date: 2019-09-02 14:08:36
categories:
- [net]
typora-root-url: ../../source
---

# 引言

前段时间一直在准备面试，本以为准备的挺好，然而被腾讯面试官问道网络问题的时候，发现自己对TCP协议的理解真的是停留在表面，不够深入。于是本着提高自己的想法，去查了些资料，这里主要是总结我对TCP建立与断开连接过程的理解。

# 常见题目

在面试中网络问题是一定会考察的，而TCP协议则是考察网络知识的重点。经常会被问道的问题如下：

1. 请讲一下TCP协议建立连接的过程
2. 请介绍TCP协议中的三次握手和四次挥手是怎么样的
3. 为什么TCP协议要三次握手来确立连接，而不是两次，也不是4次
4. TCP连接发起是的syn序号为什么不能从同一个序号开始，比如说1

接下来，我将介绍我理解的TCP三次握手和4次挥手的过程，如果错误还请指正，谢谢。

# TCP协议

## 三次握手过程

首先需要服务器监听特定的端口，等待客户端来请求连接。当客户端需要建立连接时，客户端会先向服务器发送syn报文，将报文中syn置为随机生成的序号n（这里假设序号为1000）。服务器收到同步报文后，会回复一个ack报文，把ACK位置位n+1（这里的序号应该为1001），同时设置syn为y（这里假设为2000）。客户端收到服务器发送的ack报文后，会回复一个ACK报文给服务器，其中ACK位置为y+1（这里即为2001）。当服务器收到ACK消息后，即认为连接进入稳定状态。状态机与流程图如下：

![](/images/state_machine.png)


![](/images/tcp3waysynch.png)

## 四次挥手过程

当client从app接收到关闭指令后，client会给server发送FIN消息（表明client不会再给server发送数据），client进入finish-wait-1状态。server收到finish消息后，回复确认消息ack给client，自身进入close-wait状态。client接收到ack消息后，进入到FIN-WAIT-2状态。并且在此状态等待服务器发送finish消息。当server接收到app的关闭指令后，server给client发送FIN消息。服务器进入到LAST-ACK状态。客户端收到FIN消息后，会回复ACK消息，同时进入到TIME-WAIT状态，来等待server收到ack消息，客户端会在接下来的2MSL（maximum segment lifetime）的时间内保持TIME-WAIT状态。为什么是2MSL时间呢，一是为了server有足够的时间收到ACK消息，并在消息丢失时重发。二是为了在此连接结束后的后续连接提供缓冲期。如果不是2倍MSL的话，就可能混合来自不同连接的数据包，造成消息混乱。状态机与流程图如下：

![](/images/tcp_close_state_machine.png)

![](/images/tcpclose1.png)

## 完整过程

以下为TCP从建立连接到断开的完整流程图

![](/images/tcpfsm1.png)

# 开始答题

有了上面的介绍，基本能够回答前两个问题。

### 请讲一下TCP协议建立连接的过程

看上面

### 请介绍TCP协议中的三次握手和四次挥手是怎么样的

看上面

### 为什么TCP协议要三次握手来确立连接，而不是两次，也不是4次

首先呢，根本不存在可靠的连接，tcp只是提供相对可靠的连接。三次握手的主要目的是交换通信需要的参数，主要是server与client的syn序号，这个序号是用于收发数据的。如果只有两次握手的话，当服务器发送ack+syn消息后，就会认为建立了稳定连接，这个时候如果ack+syn丢失了，client并没有收到这个消息，那么客户端就会认为连接建立不成功，而直接进入close状态。这样就会造成，server一直在哪傻等，永远不会有client来发送数据，这就会造成服务器资源的浪费。至于为什么不是四次握手，是因为握手三次成功以后，就可以认定当前连接是可靠的了，不然的话还需要client与server互相之间发送ack消息，这样就无休无止了。

### TCP连接发起是的syn序号为什么不能从同一个序号开始，比如说1

因为现实中的网络状况不可预知，比如说客户端在第一次连接时，使用序号为1为初始序号进行数据发送，发送了1到30的数据片段，这个时候因为网络问题断开了连接。然后客户端是syn为1重新建立了新的连接，这个时候服务器收到了之前发送的30个字节的数据，服务器就会以为这30个字节的数据是新发的，这就会导致数据混乱。

# 参考资料

说明，本文所有图片均来自[The TCP/IP Guide](http://www.tcpipguide.com/free/t_TCPConnectionTermination-2.htm)。

参考资料如下：

[The TCP/IP Guide](http://www.tcpipguide.com/free/t_TCPConnectionTermination-2.htm)

[TCP为什么需要3次握手与4次挥手](https://blog.csdn.net/xifeijian/article/details/12777187)

[TCP三次握手四次挥手详解](https://zhuanlan.zhihu.com/p/40013850)