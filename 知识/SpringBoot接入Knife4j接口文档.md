# SpringBoot接入Knife4j接口文档

![image-20240518164120795](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518164120795.png)

## 0.介绍

### 1） Knife4j是什么

Knife4j是Java MVC框架集成Swagger生成Api文档的增强解决方案，前身是swagger-bootstrap-ui，有着比Swagger更为美观的UI以及功能。

例如以下效果图：

![image-20240518120553328](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518120553328.png)

![image-20240518120718512](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518120718512.png)

### 2） 官方链接

官网：[Knife4j · 集Swagger2及OpenAPI3为一体的增强解决方案. | Knife4j (xiaominfo.com)](https://doc.xiaominfo.com/)

代码：[xiaoymin/knife4j: Knife4j is a set of Swagger2 and OpenAPI3 All-in-one enhancement solution (github.com)](https://github.com/xiaoymin/knife4j)

### 3）SpringBoot版本兼容性

以下为简略的对应版本：

| Spring Boot版本 | Knife4j Swagger2规范  | Knife4j OpenAPI3规范 |
| --------------- | --------------------- | -------------------- |
| 1.5.x~2.0.0     | <Knife4j 2.0.0        | >=Knife4j 4.0.0      |
| 2.0~2.2         | Knife4j 2.0.0 ~ 2.0.6 | >=Knife4j 4.0.0      |
| 2.2.x~2.4.0     | Knife4j 2.0.6 ~ 2.0.9 | >=Knife4j 4.0.0      |
| 2.4.0~2.7.x     | >=Knife4j 4.0.0       | >=Knife4j 4.0.0      |
| >= 3.0          | >=Knife4j 4.0.0       | >=Knife4j 4.0.0      |

更详细的介绍查看官网即可：

[Knife4j版本参考 | Knife4j (xiaominfo.com)](https://doc.xiaominfo.com/docs/quick-start/start-knife4j-version)



接下来以 SpringBoot 2.7.6 + Knife4j 4.4.0 作为示例，介绍一些常用的功能。

## 1.SpringBoot整合Knife4j

### 1）引入依赖

这里选择的SpringBoot版本是2.7.6

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
```

### 2）运行项目

实际测试发现无需其他配置，引入依赖后运行项目，进入`ip:port/doc.html`页面即可访问接口文档。

**以下仅为示例，注解演示的时候代码和下方的会有区别。**

注意：ReqesutMapping注册的接口，会被识别为多个，建议接口使用@GetMapping等特定方式

![image-20240518154153932](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518154153932.png)

上图效果对应的Controller代码如下：

```java
@Controller
public class BasicController {

    // http://127.0.0.1:8080/hello?name=lisi
    @GetMapping("/hello")
    @ResponseBody
    public String hello(@RequestParam(name = "name", defaultValue = "unknown user") String name) {
        return "Hello " + name;
    }

    // http://127.0.0.1:8080/user
    @RequestMapping("/user")
    @ResponseBody
    public User user() {
        User user = new User();
        user.setName("theonefx");
        user.setAge(666);
        return user;
    }

    // http://127.0.0.1:8080/save_user?name=newName&age=11
    @RequestMapping("/save_user")
    @ResponseBody
    public String saveUser(User u) {
        return "user will save: name=" + u.getName() + ", age=" + u.getAge();
    }

    // http://127.0.0.1:8080/html
    @RequestMapping("/html")
    public String html() {
        return "index.html";
    }

    @ModelAttribute
    public void parseUser(@RequestParam(name = "name", defaultValue = "unknown user") String name
            , @RequestParam(name = "age", defaultValue = "12") Integer age, User user) {
        user.setName("zhangsan");
        user.setAge(18);
    }
}

```

实体类：

```java
public class User {

    private String name;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

## 2.常用注解

此处仅以个人工作和学习的常用场景和属性示例，每个注解可能有多个属性，详细说明请参考文档和代码。

### 1) @Tag

位置：使用在Controller类上

```java
@Controller
@Tag(name = "BasicController - 基本控制器")
public class BasicController {
    ...
```

效果：对应文档中的Tag名字，即红框内容，不加Tag的话此处名字为`basic-controller`

![image-20240518154553338](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518154553338.png)

### 2) @Operation

位置：使用在方法上

```java
  	@GetMapping("/hello")
    @ResponseBody
    @Operation(summary = "say hello")
    public String hello(@RequestParam(name = "name", defaultValue = "unknown user") String name) {
        return "Hello " + name;
    }
```

效果：使用summary修改方法的描述信息

![image-20240518160559457](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518160559457.png)

### 3) @Parameter

位置：使用在参数上或者方法上

```java
    @GetMapping("/hello")
    @ResponseBody
    @Operation(summary = "say hello")
    public String hello(@Parameter(description = "用户名") @RequestParam(name = "name", defaultValue = "unknown user") String name) {
        return "Hello " + name;
    }

    @GetMapping("/user/{id}")
    @ResponseBody
    @Operation(summary = "获取用户信息")
    @Parameters({
            @Parameter(name = "id", description = "用户ID", required = true, example = "1234567890"),
            @Parameter(name = "name", description = "用户名", required = true, example = "theonefx")
    })
    public User getUesr(@PathVariable("id") Long id, @RequestParam String name) {
        User user = new User();
        user.setName(name);
        user.setAge(22);
        user.setId(id);
        return user;
    }
```

效果：

注意这里的示例值，是读取的@RequestParam的

![image-20240518161411620](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518161411620.png)

这里的示例值是@Parameter里的

![image-20240518161446386](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518161446386.png)



### 4) @Schema 

位置：使用在实体类上及其属性上

```java
@Schema(description = "User实体类")
public class User {

    @Schema(description = "用户ID")
    private Long id;

    @Schema(description = "用户名")
    private String name;

    @Schema(description = "用户年龄")
    private Integer age;
...
```

效果：

![image-20240518162206328](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518162206328.png)

这里的注解不仅对请求中的会有效果，也会影响包含此实体的响应的效果。

![image-20240518162352945](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518162352945.png)

### X) 汇总

个人常用汇总：

| 注解       | 属性        | 描述                                                         |
| ---------- | ----------- | ------------------------------------------------------------ |
| @Tag       | name        | 使用在接口类上，用来影响接口文档一级标签的描述信息           |
| @Operation | summary     | 使用在接口上，影响接口文档二级标签的接口描述信息             |
| @Parameter | description | 使用在请求参数或者接口上（需要配合@Parameters），影响接口文档的请求参数-参数说明 |
| @Schema    | description | 使用在实体类上，影响包含实体类的请求和响应的参数说明         |

![image-20240518163550953](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240518163550953.png)

## FAQ

### 1.接口已经定义好，但是在接口文档里找不到

检查下接口有没有@ResponseBody注解



