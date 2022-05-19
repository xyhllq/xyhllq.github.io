
## 一.简介

> 在异步处理时，经常用到这两个接口

`netty`中的`Future`与`jdk`中的`Future`同名，但是是两个接口。`netty`的`Future`继承自`jdk`的
`Future`，而`Promise`又对`netty Future`进行了扩展。
  * `jdk Future`只能同步等待任务结束(**或成功**、**或失败**)才能得到结果
  * `netty Future`**可以同步**等待任务结束得到结果，**也可以异步方式**得到结果，但都是要等任务结束
  * `netty Promise`不仅有netty Future 的功能，而且脱离了任务**独立存在**，只作为两个线程间传递结果的容器

比较：  

| **功能/名称** | **jdk Future** | **netty Future** | **Promise** |  
| :---: | :---: | :---: | :---: |  
| cancel | 取消任务 | + | + |
| isCanceled | 任务是否取消 | + | + |
| isDone | 任务是否完成，不能区分成功失败 | + | + |
| get | 获取任务结果，阻塞等待 | + | + |
| getNow | - | 获取任务结果，非阻塞，还未产生结果时返回null | + |
| await | - | 等待任务结束，如果任务失败，不会抛异常，而是通过isSuccess判断| + |
| sync | - | 等待任务结束，如果任务失败，抛出异常 | + |
| isSuccess| - | 判断任务是否成功 | + |
| cause | - | 获取失败信息，非阻塞，如果没有失败，返回null | + |
| addListener | - | 添加回调，异步接收结果 | + |
| setSuccess | - | - |设置成功结果|
| setFailure | - | - |设置失败结果|


## 二.代码演示

### 1.jdk Future
```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.*;

@Slf4j
public class TestJdkFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //线程池
        ExecutorService service = Executors.newFixedThreadPool(2);
        //提交任务
        Future<Integer> future = service.submit(new Callable<Integer>() {

            @Override
            public Integer call() throws Exception {
                log.info("执行计算...");
                Thread.sleep(1000);
                return 50;
            }
        });
        //主线程等待执行结果
        log.info("等待计算结果...");
        log.info("执行结果是：{}", future.get());

    }
}
```
控制台输出：
```
20:24:38 [INFO ] [pool-1-thread-1] m.x.n.c.TestJdkFuture - 执行计算...
20:24:38 [INFO ] [main] m.x.n.c.TestJdkFuture - 等待计算结果...
20:24:39 [INFO ] [main] m.x.n.c.TestJdkFuture - 执行结果是：50
```

### 2.netty Future

```java
import io.netty.channel.EventLoop;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.util.concurrent.Future;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;

@Slf4j
public class TestNettyFuture {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        EventLoop eventLoop = group.next();
        Future<Integer> future = eventLoop.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                log.info("执行计算...");
                Thread.sleep(1000);
                return 70;
            }
        });
        //同步
        //log.info("等待计算结果...");
        //log.info("执行结果是：{}", future.get());

        //异步
        future.addListener(new GenericFutureListener<Future<? super Integer>>() {
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception {
                log.info("执行结果是：{}", future.getNow());
            }
        });
    }
}
```

控制台输出：
```
//同步执行结果
20:31:05 [INFO ] [nioEventLoopGroup-2-1] m.x.n.c.TestNettyFuture - 执行计算...
20:31:05 [INFO ] [main] m.x.n.c.TestNettyFuture - 等待计算结果...
20:31:06 [INFO ] [main] m.x.n.c.TestNettyFuture - 执行结果是：70

//异步执行结果：
20:35:39 [INFO ] [nioEventLoopGroup-2-1] m.x.n.c.TestNettyFuture - 执行计算...
20:35:40 [INFO ] [nioEventLoopGroup-2-1] m.x.n.c.TestNettyFuture - 执行结果是：50
```

### 3.netty Promise

```
import io.netty.channel.EventLoop;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.util.concurrent.DefaultPromise;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.ExecutionException;

@Slf4j
public class TestNettyPromise {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 1. 准备 EventLoop
        EventLoop eventLoop = new NioEventLoopGroup().next();
        // 2.主动创建promise，结果容器
        DefaultPromise<Integer> promise = new DefaultPromise<>(eventLoop);
        new Thread(() -> {
            //3. 任意一个线程执行计算，计算完毕后，向promise填充结果
            log.info("开始计算...");
            try {
                Thread.sleep(1000);
                //设置成功结果
                promise.setSuccess(80);
            }catch (InterruptedException e){
                //抛出异常
                promise.setFailure(e);
            }
        }).start();
        log.info("等待结果....");
        //同步等待结果
        log.info("结果是：{}",promise.get());
    }
}
```

控制台输出：

```
20:45:12 [INFO ] [Thread-0] m.x.n.c.TestNettyPromise - 开始计算...
20:45:12 [INFO ] [main] m.x.n.c.TestNettyPromise - 等待结果....
20:45:13 [INFO ] [main] m.x.n.c.TestNettyPromise - 结果是：80
```





