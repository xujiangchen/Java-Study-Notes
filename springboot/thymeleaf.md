- [模板引擎Thymeleaf](#模板引擎Thymeleaf)
  * [1、什么是模板引擎](#1什么是模板引擎)
  * [2、为什么jsp被淘汰了](#2为什么jsp被淘汰了)
  * [3、导入Thymeleaf](#3导入Thymeleaf)
  * [4、Thymeleaf配置](#4Thymeleaf配置)
  * [5、实战](#5实战)
    + [TestController](#testcontroller)
    + [test.html](#testhtml)
  * [6、thymeleaf语法](#6thymeleaf语法)

# 模板引擎Thymeleaf

## 1、什么是模板引擎

是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的[HTML]文档；SpringBoot推荐使用模板引擎

## 2、为什么jsp被淘汰了

虽然jsp也是算是模拟中的一种吗，但是现在强调前后端分离，前端写前端的代码，后端写后端的代码，没有必要将前端和后端代码融合在一起，所以jsp使用就受限了。

- 动态资源和静态资源全部耦合在一起，无法做到真正的动静分离。服务器压力大，因为服务器会收到各种http请求，例如css的http请求、js的、图片的、动态代码的等等。一旦服务器出现状况，前后台一起玩完，用户体验极差。

- 前端工程师做好html后，需要由Java工程师来将html修改成jsp页面，出错率较高（因为页面中经常会出现大量的js代码），修改问题时需要双方协同开发，效率低下。

- JSP 必须要在支持Java的Web服务器里运行（例如tomcat等），无法使用nginx等（nginx单实例http并发高达5w），性能提不上来。

- 第一次请JSP，必须要在web服务器中编译成servlet，第一次运行会较慢。

- 每次请求JSP都是访问Servlet再用输出流输出的html页面，效率没有直接使用html高。

- JSP 内有较多标签和表达式，前端工程师在修改页面时会捉襟见肘，遇到很多痛点。

- 如果JSP中的内容很多，页面响应会很慢，因为是同步加载。

## 3、导入Thymeleaf
```xml
<!--thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 4、Thymeleaf配置
```
# 开发时关闭缓存,不然没法看到实时页面
spring.thymeleaf.cache=false
spring.thymeleaf.mode=HTML5
# 前缀
spring.thymeleaf.prefix=classpath:/templates/
# 编码
spring.thymeleaf.encoding=UTF-8
# 类型
spring.thymeleaf.content-type=text/html
# 名称的后缀
spring.thymeleaf.suffix=.html
```

## 5、实战

### TestController
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class TestController {

    @RequestMapping("/test")
    public String TestThymeleaf(Model model) {
        model.addAttribute("msg", "Hello,Thymeleaf");
        return "test";
    }

}
```
### test.html

- 文件位于springboot资源目录，resources的templates目录之下

首先引入thymeleaf命名空间约束
```html
xmlns:th="http://www.thymeleaf.org"
```
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>测试Thymeleaf</title>
</head>
<body>
<!--th:text就是将div中的内容设置为它指定的值-->
<div th:text="${msg}"></div>
</body>
</html>
```

## 6、thymeleaf语法

参考，https://www.cnblogs.com/itdragon/archive/2018/04/13/8724291.html
