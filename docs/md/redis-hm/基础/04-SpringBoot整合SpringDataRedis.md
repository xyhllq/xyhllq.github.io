
## 一、SpringDataRedis介绍

SpringData是Spirng中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，
官网地址：https://spring.io/projects/spring-data-redis

* 提供了对不同Redis客户端的整合(Lettuce[默认]和Jedis)
* 提供了RedisTemplate统一API来操作Redis
* 支持Redis的发布订阅模型
* 支持Redis哨兵和Redis集群
* 支持基于Lettuce的响应是编程
* 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
* 支持基于Redis的JDKCollection实现

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

| **API** | **返回值类型** | **说明** |  
| :---: | :---: | :---: |   
| redisTemplate.opsForValue() | ValueOperations | 操作`String`类型数据 |  
| redisTemplate.opsForHash() | HashOperations | 操作`Hash`类型数据 |  
| redisTemplate.opsForList() | ListOperations | 操作`List`类型数据 |  
| redisTemplate.opsForSet() | SetOperations | 操作`Set`类型数据 |  
| redisTemplate.opsForZSet() | ZSetOperations | 操作`SortedSet`类型数据 |  
| redisTemplate |  | 通用命令 |  

## 二、SpringBoot整合SpringDataRedis

### 1、引入依赖

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

### 2、编写配置

```yaml
spring:
  redis:
    host: 127.0.0.1
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

### 3、注入RedisTemplate并使用

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class SpringDataRedisDemoApplicationTests {
    //注入RedisTemplate
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void testString() {
        // 插入一条String类型数据
        redisTemplate.opsForValue().set("name","李四");
        //读取一条String类型数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name= " + name);
    }
}
```

## 三、序列化方案

> RedisTemplate的默认序列化和反序列化是`JdkSerializationRedisSerializer`器，写入前会把Object序列化为字节形式。
> 存在`可读性差`、`内存占用较大`的缺点

### 1、自定义RedisTemplate

> 实现步骤： 
> * 1.自定义RedisTemplate
> * 2.修改RedisTemplate的序列化器为`GenericJackson2JsonRedisSerializer`

#### a、引入jackson依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

#### b、自定义序列化方式

```java
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        //创建Template
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //设置序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //key和hashkey采用String序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        //value和hashvalue采用JSON序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}
```

#### c、注入自定义的序列化方式

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class SpringDataRedisDemoApplicationTests {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Test
    void testString() {
        // 插入一条String类型数据
        redisTemplate.opsForValue().set("name","李四");
        //读取一条String类型数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name= " + name);
    }
}
```

> JSON序列化的方式可以满足我们的需求，但是依然存在一些问题：  
> 为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销


### 2、StringRedisTemplate

为了节省内存空间，我们并不会使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的ket和value。
当需要存储java对象时，手动完成对象的序列化和反序列化  

> 实现步骤：
> * 1.使用StringRedisTemplate
> * 2.写入Redis时，手动把对象序列化为JSON
> * 3.读取Redis时，手动把读取到的JSON反序列化为对象

使用示例：
```java
@SpringBootTest
class StringRedisTemplateTests {
    //注入
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSaveUser() throws JsonProcessingException {
        //准备对象
        User user = new User("李四", 23);
        //手动序列化
        String json = mapper.writeValueAsString(user);
        //写入一条数据到redis
        stringRedisTemplate.opsForValue().set("user:200", json);

        //读取数据
        String val = stringRedisTemplate.opsForValue().get("user:200");
        //反序列化
        User user1 = mapper.readValue(val, User.class);
        System.out.println("user1 = " + user1);
    }
}
```

