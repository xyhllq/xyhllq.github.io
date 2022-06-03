
> 不知道写点什么

----

## 视频学习：bilibili牛皮

----

> [黑马程序员的Netty](https://www.bilibili.com/video/BV1py4y1E7oA) 

### NIO基础
- [NIO三大组件](/md/nio-hm/01.NIO三大组件.md)
- ByteBuffer
    - [演示工具类](/md/nio-hm/ByteBuffer/01.ByteBuffer演示工具类.md)
    - [基础](/md/nio-hm/ByteBuffer/02.ByteBuffer基础.md)
    - [常见方法](/md/nio-hm/ByteBuffer/03.ByteBuffer常见方法.md)
    - [分散读集中写](/md/nio-hm/ByteBuffer/04.ByteBuffer分散读集中写.md)
    - [黏包半包](/md/nio-hm/ByteBuffer/05.ByteBuffer黏包半包.md)
- 文件编程
    - [FileChannel](/md/nio-hm/文件编程/01.文件编程-FileChannel.md)
    - [数据传输](/md/nio-hm/文件编程/02.文件编程-数据传输.md)
    - [Path&Files](/md/nio-hm/文件编程/03.文件编程-Path&Files.md)
- 网络编程
    - [阻塞](/md/nio-hm/网络编程/01.网络编程-阻塞.md)
    - [非阻塞](/md/nio-hm/网络编程/02.网络编程-非阻塞.md)
    - [selector基本使用](/md/nio-hm/网络编程/03.网络编程-selector基本使用.md)
    - [消息边界问题](/md/nio-hm/网络编程/04.网络编程-消息边界.md)
    - [写入内容过多问题](/md/nio-hm/网络编程/04.网络编程-写入内容过多.md)
    - [多线程优化](/md/nio-hm/网络编程/06.网络编程-多线程优化.md)
- NIO vs BIO
    - [NIO vs BIO](/md/nio-hm/NIOvsBIO/01.NIOvsBIO.md)
    - [AIO](/md/nio-hm/NIOvsBIO/02.AIO.md)

----

> 基础部分`NIO`已经学习结束，要开始进入`netty`的学习啦...😄😄😄

### Netty
- 入门
    - [概述](/md/netty-hm/入门/01-概述.md)
    - [组件-EventLoop](/md/netty-hm/入门/02-组件-EventLoop.md)
    - [组件-Channel](/md/netty-hm/入门/03-组件-Channel.md)
    - [组件-Future&Promise](/md/netty-hm/入门/04-组件-Future&Promise.md)
    - [组件-Handler&Pipeline](/md/netty-hm/入门/05-组件-Handler&Pipeline.md)
    - [组件-ByteBuf](/md/netty-hm/入门/06-组件-ByteBuf.md)
    - [课后作业](/md/netty-hm/入门/07-课后作业.md)
- 进阶
    - [粘包半包](/md/netty-hm/进阶/01-粘包半包.md)
    - [协议与解析](/md/netty-hm/进阶/02-协议与解析.md)
    - [聊天室](/md/netty-hm/进阶/03-聊天室.md) 😁 我的实现在[这里](https://gitee.com/tzc_xyh/hm-netty-chat)
- 优化 
    - [优化-扩展序列化算法](/md/netty-hm/优化/01-扩展序列化算法.md) ✌️优化的是聊天室的代码✌️
    - [优化-参数调优](/md/netty-hm/优化/02-参数调优.md)
    - [优化-实现RPC](/md/netty-hm/优化/03-实现RPC.md) 😁 我的实现在[这里](https://gitee.com/tzc_xyh/hm-netty-rpc)


----

> [黑马程序员的Spirngboot2](https://www.bilibili.com/video/BV15b4y1a7yG)

- springboot黑马
  - [基础篇-入门程序](/md/springboot-hm/01.搭建SpringBoot项目.md)
  - [基础篇-REST风格](/md/springboot-hm/02.REST风格.md)

----

## 工作中的一些记录

> 会比较杂,等东西多了再分类吧 ！！！～

- java使用记录
    - [Stream](/md/work/01.java8的stream.md) 主要是Stream的一些使用
    - [拼接字符串](md/work/02.拼接字符串.md)
    - [mysql-dblink](md/work/03.mysql-dblink.md)
    - [aliyun验证码智能验证](md/work/04.aliyun验证码智能验证.md)
    - [Mapper.xml热加载](md/work/05.Mapper.xml热加载.md) 💗💗 个人比较喜欢这个东西
    - [误提交文件导致gitignore失效](md/work/06.误提交文件导致gitignore失效.md)
    - [IDEA新建分支后，切换远程分支关联](md/work/07.IDEA新建分支后，切换远程分支关联.md)
    - [jacoco使用记录](md/work/08-jacoco使用记录.md)
   