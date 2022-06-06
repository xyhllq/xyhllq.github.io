
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

## 二、Redis通用命令(常见)

> 💡 查看方式：  
>  1、[官网](https://redis.io/commands)：group选择`generic`
>  2、命令行：`help @generic`

### 1、KEYS(查看符合模版的所有key)

![keys用法查看](../../../assets/img/redis-hm/keys用法查看.png)

用例：查询a开头的所有key  

![keys用例](../../../assets/img/redis-hm/keys用例.png)


## 三、String类型



## 四、Hash类型




## 五、List类型




## 六、Set类型



## 七、SortedSet类型