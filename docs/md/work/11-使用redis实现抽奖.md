
> 使用redis实现抽奖[同一用户不能重复参与，同一用户不允许二次开奖]  
> 在掘金上看了下面这篇文章后 [千万用户3毫秒内抽奖100名如何实现？](https://juejin.cn/post/7134137292880838692) 也想自己实现一下  
> 不过因为自己机器原因，没有实际测试到千万，只是测试到百万。

一、新建一个项目

1、添加maven配置

```xml
   <!--Redis依赖-->
   <!--默认是lettuce客户端，如果想使用jedis还需要引入jedis依赖-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>

   <!--连接池依赖-->
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
```

2、添加yml配置

```yaml
spring:
  redis:
    host: 192.168.56.80
    port: 6379
    password: 123456
    #默认是 lettuce 客户端
    lettuce:
      pool:
        max-active: 8 # 最大连接数
        max-idle: 8 # 最大空闲连接
        min-idle: 0 # 最小空闲连接
        max-wait: 100ms #连接等待时间
```

二、代码实现

```java
package com.xyh.luckDrawRedis;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.util.StopWatch;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@SpringBootTest
class LuckDrawRedisApplicationTests {

	//注入
	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	@Test
	void luckDraw(){
		StopWatch sw = new StopWatch();
		sw.start("添加100w的uuid数据");
		for(int i = 1; i <= 10000; i++){
			List<String> userIdList = new ArrayList<>();
			for(int j = 1; j <= 100; j++){
				userIdList.add(getUUUID());
			}
			stringRedisTemplate.opsForSet().add("userIdList", userIdList.toArray(new String[100]));
      		System.out.println("初始化第" + i + "组数据");
		}
		sw.stop();
		sw.start("抽取100个uuid");
		List<String> userIdList = stringRedisTemplate.opsForSet().pop("userIdList", 100);
		for (String userId : userIdList) {
			System.out.println(userId);
		}
		sw.stop();
		System.out.println("执行结果详情：--------------------");
		System.out.println(sw.prettyPrint());
	}
    /**
    * 生成uuid
    * @return
    */
	private String getUUUID() {
		return UUID.randomUUID().toString().replace("-", "");
	}
}
```

输出

```
执行结果详情：--------------------
StopWatch '': running time = 6805437900 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
6800653900  100%  添加100w的uuid数据
004784000  000%  抽取100个uuid
```