
## 一.Channel简介
channel的主要方法：
  * 关闭channel：`close()`
  * channel关闭后做一些处理(例如：停止`EvrntLoopGroup`)：
     * 方案一：使用同步方法`ChannelFuture.sync()`
     * 方案二：使用异步等待channel关闭`ChannelFuture.addListener()`
  * 添加处理器： `pipeline()`
  * 将数据写入： `write()`
  * 将数据刷出： `flush()`
  * 将数据写入并刷出：`writeAndFlush()`

## 二.ChannelFuture简介

> `new Bootstrap().connect(new InetSocketAddress("localhost", 8080));` 获取到ChannelFuture。
> `connect`方法是一个异步非阻塞的方法。
> `ChannelFuture.sync()`会阻塞主线程，等待`connect`方法执行结束
> `ChannelFuture.addListener()` 是异步的，在`connect`方法执行结束，回调这个方法

## 三.客户端发送数据(sync/addListener)代码演示

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;

@Slf4j
public class TestChannelClient {

    public static void main(String[] args) throws InterruptedException {
        //1.启动器
        ChannelFuture channelFuture = new Bootstrap()
                //2.添加 EventLoop
                .group(new NioEventLoopGroup())
                //3.选择客户端 channel 实现
                .channel(NioSocketChannel.class)
                //4.添加处理器
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override//建立连接后被调用
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                //5.连接到服务器
                //异步非阻塞，main发起调用，真正执行 connect是nio线程
                .connect(new InetSocketAddress("localhost", 8080));
        
        //使用sync 方法同步处理结果
        /*channelFuture.sync();//阻塞方法执行，等待连接成功
        Channel channel = channelFuture.channel();
        channel.write("sync发送数据");
        channel.flush();*/

        //使用异步发送数据
        channelFuture.addListener(new ChannelFutureListener() {
            @Override
            //操作(这里指connect操作)完成调用
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                Channel channel = channelFuture.channel();
                channel.writeAndFlush("addListener发送数据");
            }
        });

    }
}

```

## 四.客户端退出(sync/addListener)代码演示

> 客户端接收输入，并将输入内容发送给服务器端。输入`q`退出，并关闭其他资源(channel、EventLoopGroup)  
> 1.使用同步的方式，获取连接后的`channel`  
> 2.获取`channel`的`closeFuture`，使用异步的方式优雅关闭`EventLoopGroup`

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

import java.net.InetSocketAddress;
import java.util.Scanner;

@Slf4j
public class TestCloseFutureClient {

    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override//建立连接后被调用
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        //加入日志输出，需要在日志文件里面配置一下
                        ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress("localhost", 8080));
        //使用sync 方法同步处理结果
        channelFuture.sync();//阻塞方法执行，等待连接成功
        Channel channel = channelFuture.channel();
        //接收控制台输入，并发给服务器
        new Thread(() -> {
            //接收控制台输入
            Scanner sc = new Scanner(System.in);
            while(true){
                String info = sc.nextLine();
                if("q".equals(info)){//如果是q，关闭channel，并退出循环
                    channel.close();
                    break;
                }else{
                    channel.writeAndFlush(info);
                }
            }
        }).start();

        log.info("get closeFuture...");
        ChannelFuture closeFuture = channel.closeFuture();
        closeFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                log.info("closeFuture group.shutdownGracefully....");
                group.shutdownGracefully();
            }
        });
    }
}
```
控制台输出/输入:
```
20:04:47 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b] REGISTERED
20:04:47 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b] CONNECT: localhost/127.0.0.1:8080
20:04:47 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 - R:localhost/127.0.0.1:8080] ACTIVE
20:04:47 [INFO ] [main] m.x.n.c.TestCloseFutureClient - get closeFuture...
//这是输入
hello server
20:04:55 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 - R:localhost/127.0.0.1:8080] WRITE: 12B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 73 65 72 76 65 72             |hello server    |
+--------+-------------------------------------------------+----------------+
20:04:55 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 - R:localhost/127.0.0.1:8080] FLUSH
//这是输入
q
20:04:58 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 - R:localhost/127.0.0.1:8080] CLOSE
20:04:58 [INFO ] [nioEventLoopGroup-2-1] m.x.n.c.TestCloseFutureClient - closeFuture group.shutdownGracefully....
20:04:58 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 ! R:localhost/127.0.0.1:8080] INACTIVE
20:04:58 [DEBUG] [nioEventLoopGroup-2-1] i.n.h.l.LoggingHandler - [id: 0x39b26a3b, L:/127.0.0.1:50006 ! R:localhost/127.0.0.1:8080] UNREGISTERED
```


