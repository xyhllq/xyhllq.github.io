
----

# netty RPC

项目来源：[黑马程序员-netty教程](https://www.bilibili.com/video/BV1py4y1E7oA)
我的实现代码： [gitee实现](https://gitee.com/tzc_xyh/hm-netty-rpc)
----

这里的代码，复用了 `netty-聊天室` 的大部分代码，只是修改了下 message 和 handler 中的内容  
这个文章感觉写的奇奇怪怪，没啥好说的。可能是自己没抓住重点吧。。。还是看代码吧。。。。

## 服务端实现

```java
@Slf4j
public class RpcServer {

	public static void main(String[] args) {
		NioEventLoopGroup boss = new NioEventLoopGroup();
		NioEventLoopGroup worker = new NioEventLoopGroup();
		LoggingHandler LOGGINF_HANDLER = new LoggingHandler(LogLevel.DEBUG);
		MessageCodecSharable MESSAGE_CODEC = new MessageCodecSharable();

		//rpc 请求消息处理
		RpcRequstMessageHandler RPC_HANDLER = new RpcRequstMessageHandler();
		try {
		    ServerBootstrap serverBootstrap = new ServerBootstrap();
		    serverBootstrap.channel(NioServerSocketChannel.class);
		    serverBootstrap.group(boss, worker);
		    serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

				@Override
				protected void initChannel(SocketChannel ch) throws Exception {
					ch.pipeline().addLast(new ProtocolFrameDecoder());
					ch.pipeline().addLast(LOGGINF_HANDLER);
					ch.pipeline().addLast(MESSAGE_CODEC);
					ch.pipeline().addLast(RPC_HANDLER);
				}
			});
			Channel channel = serverBootstrap.bind(8080).sync().channel();
			channel.closeFuture().sync();
		} catch (InterruptedException e) {
			log.error("server error", e);
		}finally {
			boss.shutdownGracefully();
			worker.shutdownGracefully();
		}
	}
}
```

其实没啥改变，只是添加了一个 `RpcRequstMessageHandler`。这个handler主要是负责根据客户端发过来的消息内容，
进行方法调用

```java
@Slf4j
@ChannelHandler.Sharable
public class RpcRequstMessageHandler extends SimpleChannelInboundHandler<RpcRequestMessage> {

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, RpcRequestMessage msg){
		RpcResponseMessage response = new RpcResponseMessage();
		response.setSequenceId(msg.getSequenceId());
		try{
			HelloService service = (HelloService)ServicesFactory.getService(Class.forName(msg.getInterfaceName()));
			Method method = service.getClass().getMethod(msg.getMethodName(), msg.getParamterTypes());
			Object invoke = method.invoke(service, msg.getParamterValue());
			response.setReturnValue(invoke);
		}catch (Exception e){
			String message = e.getCause().getMessage();
			response.setExceptionValue(new Exception("远程调用出错：" + message));
		}
		ctx.writeAndFlush(response);
	}
}
```

## 客户端实现

定义一个初始化Channel的方法，之后所有的调用，都使用同一个Channel

```java
@Slf4j
public class RpcClientManager {
    //....
	private static Channel channel = null;
	private static final Object LOCK = new Object();
    //单例模式
	public static Channel getChannel(){
		if(channel != null){
			return channel;
		}
		synchronized (LOCK){
			if(channel != null){
				return channel;
			}
			initChannel();
			return channel;
		}
	}


	//初始化channel方法
	private static void initChannel() {
		NioEventLoopGroup group = new NioEventLoopGroup();
		LoggingHandler LOGGING_HANDLER = new LoggingHandler(LogLevel.DEBUG);
		MessageCodecSharable MESSAGE_CODEC = new MessageCodecSharable();
		RpcResponseMessageHandler RPC_HANDLER = new RpcResponseMessageHandler();
		Bootstrap bootstrap = new Bootstrap();
		bootstrap.channel(NioSocketChannel.class);
		bootstrap.group(group);
		bootstrap.handler(new ChannelInitializer<SocketChannel>() {

			@Override
			protected void initChannel(SocketChannel ch) throws Exception {
				ch.pipeline().addLast(new ProtocolFrameDecoder());
				ch.pipeline().addLast(LOGGING_HANDLER);
				ch.pipeline().addLast(MESSAGE_CODEC);
				ch.pipeline().addLast(RPC_HANDLER);
			}
		});
		try {
			channel = bootstrap.connect("localhost", 8080).sync().channel();
			channel.closeFuture().addListener(future -> {
				group.shutdownGracefully();
			});
		} catch (InterruptedException e) {
			log.error("client error", e);
		}
	}
}
```

在使用的时候，使用JDK动态代理来获取发送所需要的信息

```java
@Slf4j
public class RpcClientManager {

	public static void main(String[] args) {
		HelloService service = getProxyService(HelloService.class);
		service.sayHello("zhangsan");
		service.sayHello("lisi");
		service.sayHello("wangwu");
	}

	public static <T> T getProxyService(Class<T> serviceClass){
		ClassLoader loader = serviceClass.getClassLoader();
		Class[] interfaces = {serviceClass};
		Object o = Proxy.newProxyInstance(loader,interfaces,(proxy, method, args) -> {
			int sequenceId = SequenceIdGenerator.nextId();
			RpcRequestMessage msg = new RpcRequestMessage(
					sequenceId,
					serviceClass.getName(),
					method.getName(),
					method.getReturnType(),
					method.getParameterTypes(),
					args
			);
			//将消息对象发送出去
			getChannel().writeAndFlush(msg);

			//准备一个空Promise对象，来接受结果
			DefaultPromise<Object> promise = new DefaultPromise<>(getChannel().eventLoop());
			RpcResponseMessageHandler.PROMISE.put(sequenceId, promise);
			//等待promise结果
			promise.await();
			if(promise.isSuccess()){
				return promise.getNow();
			}else{
				throw new RuntimeException(promise.cause());
			}
		});
		return (T) o;
	}
}

```

客户端处理服务器消息返回的方式，`RpcResponseMessageHandler`处理代码如下：
```java
@Slf4j
@ChannelHandler.Sharable
public class RpcResponseMessageHandler extends SimpleChannelInboundHandler<RpcResponseMessage> {

	public static final Map<Integer, Promise<Object>> PROMISE = new ConcurrentHashMap<>();;

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, RpcResponseMessage msg) throws Exception {
		log.info("{}", msg);
		Promise<Object> promise = PROMISE.remove(msg.getSequenceId());
		if(promise != null){
			Object returnValue = msg.getReturnValue();
			Exception exceptionValue = msg.getExceptionValue();
            //setFailure 和 setSuccess 会放行promise.await();
			if(exceptionValue != null){
				promise.setFailure(exceptionValue);
			}else{
				promise.setSuccess(returnValue);
			}
		}
	}
}
```


