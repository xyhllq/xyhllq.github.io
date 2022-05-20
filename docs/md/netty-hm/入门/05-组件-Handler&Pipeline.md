
## 一、简介

`ChannelHandler`用来处理`Channel`上的各种事件，分为入站、出站两种。所有`ChannelHandler`被连成一串，
就是`Pipeline`
 * 入站处理器通常是`ChannelInboundHandlerAdapter`的子类，主要用来读取客户端数据，写回结果
 * 出站处理器通常是`ChannelOutBoundHandlerAdapter`的子类，主要对写回结果进行加工

> 打个比喻：每个`Channel`是一个产品的加工车间，`Pipeline`是车间中的流水线，`ChannelHandler`就是流水线上
> 的各道工序，而后面要讲的`ByteBuf`是原材料，经过很多工序的加工；先经过一道道入站工序，再经过一道道出
> 站工序最终变成产品

## 二、演示Pipeline.addLast()的出站、入站

> 结论：  
> * addLast的顺序： head(默认) -> h1(入站) -> h2(入站) -> h3(出站) -> h4(出站) -> tail(默认)
> * 出站操作需要`writeAndFlush`
> * 出站操作是逆序的，即 h4 —> h3

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

/**
 * 用于理解 入站、出站的顺序
 */
@Slf4j
public class TestPipeline {
    public static void main(String[] args) {
        new ServerBootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        //1.通过channel拿到pipeline
                        ChannelPipeline pipeline = ch.pipeline();
                        //2.添加处理器
                        pipeline.addLast("h1", new ChannelInboundHandlerAdapter(){
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                log.info("1");
                                super.channelRead(ctx, msg);
                            }
                        });
                        pipeline.addLast("h2", new ChannelInboundHandlerAdapter(){
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                log.info("2");
                                super.channelRead(ctx, msg);
                                //触发出站操作
                                ch.writeAndFlush(ctx.alloc().buffer().writeBytes("server...".getBytes()));
                            }
                        });
                        pipeline.addLast("h3", new ChannelOutboundHandlerAdapter(){

                            @Override
                            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                log.info("3");
                                super.write(ctx, msg, promise);
                            }
                        });
                        pipeline.addLast("h4", new ChannelOutboundHandlerAdapter(){

                            @Override
                            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                log.info("4");
                                super.write(ctx, msg, promise);
                            }
                        });
                    }
                })
                .bind(8080);
    }
}

```

控制台输出：
```
20:00:16 [INFO ] [nioEventLoopGroup-2-2] m.x.n.c.TestPipeline - 1
20:00:16 [INFO ] [nioEventLoopGroup-2-2] m.x.n.c.TestPipeline - 2
20:00:16 [INFO ] [nioEventLoopGroup-2-2] m.x.n.c.TestPipeline - 4
20:00:16 [INFO ] [nioEventLoopGroup-2-2] m.x.n.c.TestPipeline - 3
```

> 注意：  
> * 入站处理器需要使用`super.channelRead(ctx, msg)`或者`ctx.fireChannelRead(msg)`调用下一个`入站处理器`
> * 在调用`ch.writeAndFlush()`的时候是`从tail向前`执行出站操作
> * 在调用`ctx.writeAndFlush()`的时候是`从当前处理器向前`执行出站操作


## 三、EmbeddedChannel

> netty提供的测试工具类，不用启动服务端和客户端，就可以测试入站、出站操作

```java
import io.netty.buffer.ByteBufAllocator;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelOutboundHandlerAdapter;
import io.netty.channel.ChannelPromise;
import io.netty.channel.embedded.EmbeddedChannel;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class TestEmbeddedChannel {

    public static void main(String[] args) {
        ChannelInboundHandlerAdapter h1 = new ChannelInboundHandlerAdapter() {
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                log.info("1");
                super.channelRead(ctx, msg);
            }
        };
        ChannelInboundHandlerAdapter h2 = new ChannelInboundHandlerAdapter() {
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                log.info("2");
                super.channelRead(ctx, msg);
            }
        };
        ChannelOutboundHandlerAdapter h3 = new ChannelOutboundHandlerAdapter() {
            @Override
            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                log.info("3");
                super.write(ctx, msg, promise);
            }
        };
        ChannelOutboundHandlerAdapter h4 = new ChannelOutboundHandlerAdapter() {
            @Override
            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                log.info("4");
                super.write(ctx, msg, promise);
            }
        };

        EmbeddedChannel channel = new EmbeddedChannel(h1, h2, h3, h4);
        //模拟入站操作
        //channel.writeInbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("hello".getBytes()));
        //模拟出站操作
        channel.writeOutbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("hello".getBytes()));
    }
}

```













