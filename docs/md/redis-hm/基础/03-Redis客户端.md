
## 一、客户端比较

 Redis官网中提供了各种语言的客户端，地址： https://redis.io/clients

| **名称** | **特性** |  
| :---: | :---: |  
| Jedis | 以Redis命令作为方法名称，学习成本低，简单使用。但是Jedis实例是线程不安全的，多线程环境下需要基于连接池来使用 |  
| lettuce | Lettuce是基于Netty实现的，支持同步、异步和响应式编程方式，并且是线程安全的。支持Redis的哨兵模式、集群模式和管道模式。 |  
| Redisson | Redisson是一个基于Redis实现的分布式、可伸缩的Java数据结构集合。包含了诸如Map、Queue、Lock、Semaphore、AtomicLong等强大功能 |  

## 二、Jedis的使用

Jedis的官网地址： https://github.com/redis/jedis

基本步骤：  
* 1.引入依赖
* 2.创建Jedis对象，建立连接
* 3.使用Jedis，方法名和Redis命令一致
* 4.释放资源

### 1、引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>jedis-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!--jedis-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>4.2.0</version>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.7.0</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

</project>
```

### 2、建立连接

```java
public class TestJedis {
    private Jedis jedis;

    @BeforeEach
    void setUp() {
        jedis = new Jedis("127.0.0.1", 6379);
        //密码
        jedis.auth("password");
        jedis.select(0);
    }
}
```

### 3、使用redis

```java
public class TestJedis {
    private Jedis jedis;

    @Test
    void testString() {
        //插入callInfo
        String result = jedis.set("callInfo", "你好,redis!");
        System.out.println(result);
        //获取callInfo
        String callInfo = jedis.get("callInfo");
        System.out.println(callInfo);
    }
}
```

控制台输出

```
OK
你好,redis!
```

### 4、释放资源

```java
public class TestJedis {
    private Jedis jedis;

    @AfterEach
    void tearDown() {
        if(jedis != null){
            jedis.close();
        }
    }
}
```

### 5、jedis连接池

```java
public class JedisConnectionFactory {

    private static final JedisPool jedisPool;

    static {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //最大连接数
        jedisPoolConfig.setMaxTotal(8);
        //最大空闲连接
        jedisPoolConfig.setMaxIdle(8);
        //最小空闲连接
        jedisPoolConfig.setMinIdle(0);
        //设置最长等待时间 ms
        jedisPoolConfig.setMaxWaitMillis(200);
        jedisPool = new JedisPool(jedisPoolConfig,
                "127.0.0.1",6379,1000,"password");
    }

    /**
     * 获取jedis对象
     * @return
     */
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

修改jedis获取方式

```java
public class TestJedis {
    private Jedis jedis;

    @BeforeEach
    void setUp() {
        //从连接池中获取jedis
        jedis = JedisConnectionFactory.getJedis();
        jedis.select(0);
    }
}
```

### 6、jedis.close()方法说明
```java
    public void close() {
        // 判断是否是连接池的方式
        if (this.dataSource != null) {
            Pool<Jedis> pool = this.dataSource;
            this.dataSource = null;
            //归还
            if (this.isBroken()) {
                pool.returnBrokenResource(this);
            } else {
                pool.returnResource(this);
            }
        } else {
            //不是连接池就是关闭连接
            this.connection.close();
        }
    }
```
