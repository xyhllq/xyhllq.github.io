
> 没有什么特别大的意义，只是记录一下，方便之后自己创建一个测试项目

## 一、集成logback

### 1、引入maven依赖

```xml
<dependencies>
    <!--logback日志-->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.11</version>
    </dependency>
</dependencies>
```

### 2、在resources文件夹下添加logback配置

新建`logback.xml`文件 然后复制下面的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd'T'HH:mm:ss.SSS zXXX} [%thread] %-5level %logger{50}:%L - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 日志输出级别 默认DEBUG 不区分大小写-->
    <root level="DEBUG">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

### 3、使用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogbackTest {

    private static final Logger LOGGER = LoggerFactory.getLogger(LogbackTest.class);

    public static void main(String[] args) {
        LOGGER.debug("日志打印...注意import的包，不要引入错误...");
    }
}

```


## 二、使用lombok简化一下

> 主要是为了使用`@Slf4j`简化`private static final Logger LOGGER = LoggerFactory.getLogger(LogbackTest.class);`

### 1、引入maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.24</version>
    </dependency>
</dependencies>
```

### 2、使用

```java
import lombok.extern.slf4j.Slf4j;

/**
 *使用线程的两种方式
 */
@Slf4j
public class LombokTest {
    
    public static void main(String[] args) {
        log.debug("日志打印...使用lombok的@Slf4j注解...");
    }
}
```