
## 一.简介
> REST (Representational State Transfer) 表现形式状态转换  
>&nbsp;&nbsp;&nbsp;&nbsp;REST风格 不是一个规范，可以打破  
>&nbsp;&nbsp;&nbsp;&nbsp;使用行为动作(GET-查、POST-增、PUT-改、DELETE-删)区分  
>&nbsp;&nbsp;&nbsp;&nbsp;描述模块名，最好使用复数 (users、books、orders)  
>&nbsp;&nbsp;&nbsp;&nbsp;用 REST 描述资源，称为 RESTFul  

优点
> 隐藏资源的访问行为，无法通过地址得知对资源是何种操作  
> 书写简化  


## 二.入门案例

### 1.案例代码
```java
@Controller
public class SysUserController {
    /**
     * 增
     * @param sysUser
     * @return
     */
    @RequestMapping(value = "/sysUsers",method = RequestMethod.POST)
    @ResponseBody
    public String insert(@RequestBody SysUser sysUser){
        System.out.println("insert user..." + sysUser.toString());
        return "{'model':'user insert'}";
    }

    /**
     * 删
     * @param id
     * @return
     */
    @RequestMapping(value = "/sysUsers/{id}",method = RequestMethod.DELETE)
    @ResponseBody
    public String delete(@PathVariable Integer id){
        System.out.println("delete user..." + id);
        return "{'model':'user delete'}";
    }

    /**
     * 改
     * @param sysUser
     * @return
     */
    @RequestMapping(value = "/sysUsers",method = RequestMethod.PUT)
    @ResponseBody
    public String update(@RequestBody SysUser sysUser){
        System.out.println("update...." + sysUser.toString());
        return "{'model':'user update'}";
    }

    /**
     * 查一个
     * @param id
     * @return
     */
    @RequestMapping(value = "/sysUsers/{id}",method = RequestMethod.GET)
    @ResponseBody
    public String getById(@PathVariable Integer id){
        System.out.println("getById...." + id);
        return "{'model':'user getById'}";
    }

    /**
     * 查全部
     * @return
     */
    @RequestMapping(value = "/sysUsers",method = RequestMethod.GET)
    @ResponseBody
    public String getAll(){
        System.out.println("getAll...");
        return "{'model':'user getAll'}";
    }
}

```

### 2.@RequestMapping
类型：方法注解  
位置：SpringMVC控制器方法定义上方  
作用：设置当前控制器方法请求访问路径  
属性： value(默认)，请求访问路径  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;method:http请求动作，标准动作(GET/POST/PUT/DELETE)  
代码案例：
```java
    @RequestMapping(value = "/sysUsers",method = RequestMethod.GET)
@ResponseBody
public String getAll(){
    System.out.println("getAll...");
    return "{'model':'user getAll'}";
}
```

### 3.PathVariable
类型：形参注解  
位置：SpringMVC控制器方法形参定义前面  
作用：绑定路径参数与处理器方法形参间的关系，要求路径参数名与形参名一一对应   
代码案例：
```java
@RequestMapping(value = "/books/{id}", method = RequestMethod.GET)
public String getById(@PathVariable Integer id){
    System.out.println("book getById..."+id);
    return "{'model':'book getById'}";
}
```

### 4.@RequestBody、@RequestcParam、@PathVariable
> 区别  
> &nbsp;&nbsp;&nbsp;&nbsp;@RequestcParam用于接收url地址传参或表单传参  
> &nbsp;&nbsp;&nbsp;&nbsp;@RequestBody用于接收json数据  
> &nbsp;&nbsp;&nbsp;&nbsp;@PathVariable用于接收路径参数，使用{参数名称}描述路径参数  
> 应用  
> &nbsp;&nbsp;&nbsp;&nbsp;发送请求参数超过1小时，以json格式位置，@RequestBody应用较广  
> &nbsp;&nbsp;&nbsp;&nbsp;发送非json格式数据，选用@RequestcParam接收请求参数  
> &nbsp;&nbsp;&nbsp;&nbsp;采用RESTful进行开发，当参数数量较少时，例如1个，可以采用@PathVariable接收请求路径  

## 三.简化开发

### 1.@RestController
类型：类注解  
位置：基于SpringMVC的RESTful开发控制器类定义上方  
作用：设置当前控制器类为RESTful风格，等同于@Controller与@ResponseBody两个注解组合功能  
代码案例：
```java
@RestController
public class BookController{
    
}
```

### 2.@GetMapping、@PostMapping、@PutMapping、@DeleteMapping
类型：方法注解  
位置：基于SpringMVC的RESTful开发控制器方法定义上方  
作用：设置当前控制器方法请求访问路径与请求动作，每种对应一个请求动作  
属性：value(默认)，请求访问路径  
代码案例：
```java
@GetMapping("/{id}")
public String getById(@PathVariable Integer id){
    System.out.println("book getById..."+id);
    return "{'model':'book getById'}";
}
```