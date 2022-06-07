
## 一、Redis数据结构介绍
Redis是一个key-value的数据库，key一般是String类型，不过value的类型多种多样
基本类型：String、Hash、List、Set、SortedSet
特殊类型：GEO、BitMap、HyperLog

> 💡 在Redis中查看命令的两种方式：  
> 1、[官网](https://redis.io/commands) ： (可以根据group分组查询)  
> 2、命令行界面 ：`help命令/help @分组  `
> 💡 获取具体用法也是两种  
> 1、[官网](https://redis.io/commands) ：直接点击具体命令查看--比较详细  
> 2、命令行界面：`help [command]  `

----

## 二、Redis通用命令(常见)

> 💡 查看方式：  
>  1、[官网](https://redis.io/commands)：group选择`generic`
>  2、命令行：`help @generic`

### 1、KEYS
 
 * 查看符合模版的所有key
 * **不建议在生产环境中使用**，数据量大的情况下，执行效率低，会阻塞redis其他命令的执行


![keys用法查看](../../../assets/img/redis-hm/keys用法查看.png)

用例：查询a开头的所有key  

![keys用例](../../../assets/img/redis-hm/keys用例.png)

### 2、DEL

 * 删除一个指定的key
 * 可以同时指定**多个key**
 * 即使指定的key值不存在，也**不会报错**
 * 返回值代表**实际**删除key的**个数**

![DEL用法查看](../../../assets/img/redis-hm/DEL用法查看.png)

用例：删除多个  

![DEL用例.png](../../../assets/img/redis-hm/DEL用例.png)

### 3、EXISTS

 * 判断key是否存在
 * 可以同时指定**多个key**
 * 返回值代表**实际**存在的**个数**

用例：判断多个key是否存在

![EXISTS用例.png](../../../assets/img/redis-hm/EXISTS用例.png)

### 4、EXPIRE & TTL

 * EXPIRE:给一个key设置有效期，有效期到期时该key会被自动删除
 * TTL:查看一个key的剩余有效期
    * 返回`-1`表示永久有效
    * 返回`-2`表示已经被移除/不存在

用例：添加一个指定事件的key-value，用TTL查看这个key的剩余有效期

![EXPIRE&TTL用例.png](../../../assets/img/redis-hm/EXPIRE&TTL用例.png)

----

## 三、String类型

> String类型，也就是字符串类型，是Redis中最简单的存储类型。  
> 其value值是字符串，不过根据字符串的格式不同，又可以分为3类：  
>    * String: 普通字符串  
>    * int: 整数类型，可以做自增、自减操作  
>    * float: 浮点类型，可以做自增、自减操作  
> 不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能操作512m

### 1、SET & GET & SET & MGET
 * SET：**添加**或者**修改已经存在**的一个String类型键值对
 * GET：根据key获取String类型的value
 * MSET：批量添加多个String类型的键值对
 * MGET：根据多个key获取多个String类型的value

![String-SET&GET&MSET&MGET用例.png](../../../assets/img/redis-hm/String-SET&GET&MSET&MGET用例.png)

### 2、INCR & INCRBY

 * INCR：让一个整型的key自增1
 * INCRBY：让一个整型的key自增指定步长，例如` INCRBY num 2 `让num自增2

![String-INCR&INCRBY用例.png](../../../assets/img/redis-hm/String-INCR&INCRBY用例.png)

### 3、INCRBYFLOAT

 * INCRBYFLOAT:让一个浮点类型的数字自增并指定步长

![INCRBYFLOAT用例.png](../../../assets/img/redis-hm/INCRBYFLOAT用例.png)

### 4、SETNX

SETNX:添加一个String类型的键值对，前提是这个key不存在，否则不执行

> 使用效果与`SET key value NX`一致

![SETNX用例.png](../../../assets/img/redis-hm/SETNX用例.png)

### 5、SETEX

SETEX:添加一个String类型的键值对，并且指定有效期

> 使用效果与`SET key value EX second`一致

![SETEX用法查看.png](../../../assets/img/redis-hm/SETEX用法查看.png)

----
> 💡 Redis中没有类似MySQL中的Table概念，应该如何区分不同类型的key？  
> 例如：需要存储用户、商品信息到redis中，有一个用户id是1，有一个商品id恰好也是1  
> 因此： Redis的key允许有多个单词形成层级结构，多个单词之间用"`:`"隔开，格式如下：  
>    `项目名:业务名:类型:id`

----

## 四、Hash类型

> Hash类型，也较散列，其value是一个无序字典，类似Java中的HashMap结构。
> Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD：

![Hash类型.png](../../../assets/img/redis-hm/Hash类型.png)

### 1、HSET
 
 * 添加或者修改hash类型key的field值

```shell
#HSET KEY FIELD VALUE
HSET Test:Student:1 name lilei
```

### 2、HGET

 * 获取一个hash类型key的field的值

```shell
# HGET KEY FIELD
HGET Test:Student:1 name
```

### 3、HMSET

 * 批量添加多个hash类型key的field的值
 * 使用`help HMSET`查看，命令格式和HSET一致，只是描述不一样

### 4、HMGET

 * 批量获取多个hash类型key中的field

```shell
#HMGET key field...
HMGET Test:Student:1 name age
```

### 5、HGETALL

 * 获取一个hash类型的key中的所有field和value

```shell
#HGETALL key
HGETALL Test:Student:1
```

### 6、HKEYS

 * 获取一个hash类型的key中的所有field

```shell
#HKEYS key
HKEYS Test:Student:1
```

### 7、HVALS

 * 获取一个hash类型的key中的所有value

```shell
#HVALS key
HVALS Test:Student:1
```

### 8、HINCRBY

 * 让一个hash类型key的字段值自增并指定步长

```shell
#HINCRBY key field increment
HINCRBY Test:Student:1 age 12
```

### 9、HSETNX

 * 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行

```shell
#HSETNX key field value
HSETNX Test:Student:1 sex man
```

----

## 五、List类型

----


## 六、Set类型

----

## 七、SortedSet类型

----