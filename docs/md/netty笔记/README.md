
 ## 整个文档主要是一个 [黑马程序员Netty教程](https://www.bilibili.com/video/BV1py4y1E7oA) 的一个记录

----

 * 主要内容分为了两大块：  
 * 第一块：NIO基础，主要介绍NIO的三大组件(Buffer、Channel、Selector)  
 * 第二块：对Netty中的组件进行了介绍，并且通过`聊天室`、`RPC`的实战来加深理解

-----

### NIO基础
- [NIO三大组件](/md/netty笔记/nio基础/01-NIO三大组件.md)
- ByteBuffer
    - [演示工具类](/md/netty笔记/nio基础/ByteBuffer/01.ByteBuffer演示工具类.md)
    - [基础](/md/netty笔记/nio基础/ByteBuffer/02.ByteBuffer基础.md)
    - [常见方法](/md/netty笔记/nio基础/ByteBuffer/03.ByteBuffer常见方法.md)
    - [分散读集中写](/md/netty笔记/nio基础/ByteBuffer/04.ByteBuffer分散读集中写.md)
    - [黏包半包](/md/netty笔记/nio基础/ByteBuffer/05.ByteBuffer黏包半包.md)
- 文件编程
    - [FileChannel](/md/netty笔记/nio基础/文件编程/01.文件编程-FileChannel.md)
    - [数据传输](/md/netty笔记/nio基础/文件编程/02.文件编程-数据传输.md)
    - [Path&Files](/md/netty笔记/nio基础/文件编程/03.文件编程-Path&Files.md)
- 网络编程
    - [阻塞](/md/netty笔记/nio基础/网络编程/01.网络编程-阻塞.md)
    - [非阻塞](/md/netty笔记/nio基础/网络编程/02.网络编程-非阻塞.md)
    - [selector基本使用](/md/netty笔记/nio基础/网络编程/03.网络编程-selector基本使用.md)
    - [消息边界问题](/md/netty笔记/nio基础/网络编程/04.网络编程-消息边界.md)
    - [写入内容过多问题](/md/netty笔记/nio基础/网络编程/04.网络编程-写入内容过多.md)
    - [多线程优化](/md/netty笔记/nio基础/网络编程/06.网络编程-多线程优化.md)
- NIO vs BIO
    - [NIO vs BIO](/md/netty笔记/nio基础/NIOvsBIO/01.NIOvsBIO.md)
    - [AIO](/md/netty笔记/nio基础/NIOvsBIO/02.AIO.md)

----

> 基础部分`NIO`已经学习结束，要开始进入`netty`的学习啦...😄😄😄

### Netty
- 入门
    - [概述](/md/netty笔记/netty/入门/01-概述.md)
    - [组件-EventLoop](/md/netty笔记/netty/入门/02-组件-EventLoop.md)
    - [组件-Channel](/md/netty笔记/netty/入门/03-组件-Channel.md)
    - [组件-Future&Promise](/md/netty笔记/netty/入门/04-组件-Future&Promise.md)
    - [组件-Handler&Pipeline](/md/netty笔记/netty/入门/05-组件-Handler&Pipeline.md)
    - [组件-ByteBuf](/md/netty笔记/netty/入门/06-组件-ByteBuf.md)
    - [课后作业](/md/netty笔记/netty/入门/07-课后作业.md)
- 进阶
    - [粘包半包](/md/netty笔记/netty/进阶/01-粘包半包.md)
    - [协议与解析](/md/netty笔记/netty/进阶/02-协议与解析.md)
    - [聊天室](/md/netty笔记/netty/进阶/03-聊天室.md) 😁 我的实现在[这里](https://gitee.com/tzc_xyh/hm-netty-chat)
- 优化 
    - [优化-扩展序列化算法](/md/netty笔记/netty/优化/01-扩展序列化算法.md) ✌️优化的是聊天室的代码✌️
    - [优化-参数调优](/md/netty笔记/netty/优化/02-参数调优.md)
    - [优化-实现RPC](/md/netty笔记/netty/优化/03-实现RPC.md) 😁 我的实现在[这里](https://gitee.com/tzc_xyh/hm-netty-rpc)
- 源码
    - [源码-源码阅读记录](/md/netty笔记/netty/源码/01-源码阅读记录.md)

----