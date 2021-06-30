- [一、SpringBoot启动默认加载的Filter](#一SpringBoot启动默认加载的Filter)
- [二、Filter优先级](#二Filter优先级)
- [三、Servlet3.0自定义Filter](#三Servlet3.0自定义Filter)
  * [自定义过滤器](#自定义过滤器)
  * [启动类](#启动类)
- [四、Servlet3.0的注解自定义原生Listener](#四Servlet30的注解自定义原生Listener)


# 一、SpringBoot启动默认加载的Filter

```
- characterEncodingFilter
  	是spring内置过滤器的一种，用来指定请求或者响应的编码格式。在web开发中经常被从来使用
  
- hiddenHttpMethodFilter
	浏览器form表单只支持Get和POST请求，而Delete、Put等method并不支持。而HiddenHttpMethodFilter，可以将这些请求转换为标准的http方法，使得支持GET、POST、PUT和DELETE请求
	
- httpPutFormContentFilter
	用于自动的封装前台传递过来的PUT请求的参数
	
- requestContextFilter
	基于LocalThread将HTTP request对象绑定到为该请求提供服务的线程上。
```

# 二、Filter优先级

低位值意味着更高的优先级，自定义Filter，避免和默认的Filter优先级一样，不然会冲突。

```
- 设置最高优先级	
	Ordered.HIGHEST_PRECEDENCE
	@Order(Ordered.HIGHEST_PRECEDENCE)
	
- 设置最低优先级
	Ordered.LOWEST_PRECEDENCE
	@Order(Ordered.LOWEST_PRECEDENCE)
```

# 三、Servlet3.0自定义Filter

- 1、使用Servlet3.0的注解进行配置
- 2、启动类里面增加 @ServletComponentScan，进行扫描
- 3、新建一个Filter类，implements Filter，并实现对应的接口
- 4、@WebFilter 标记一个类为filter，被spring进行扫描

## 自定义过滤器

```java
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

// urlPatterns 拦截请求路径，filterName 名称
// @WebFilter（serlvet3.0提供）注解以注册到servlet容器中
@WebFilter(urlPatterns = "/api/*", filterName = "loginFilter")
public class LoginFilter implements Filter {


    @Override
    // 在启动容器的时候进行加载
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init loginFilter");
    }

    @Override
    // 处理业务的方法
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("doFilter loginFilter");

        // 进行强转，拿到前台给我传来的参数
        HttpServletRequest req = (HttpServletRequest)servletRequest;
        HttpServletResponse resp = (HttpServletResponse)servletResponse;
        // 业务操作，进行判断

        // 符合的进行放行
        filterChain.doFilter(servletRequest,servletResponse);

        // 不符合的自行接解决
    }

    @Override
    // 容器销毁的时候调用的方法
    public void destroy() {
        System.out.println("destroy loginFilter");
    }
}
```

## 启动类

```java
// 添加 ServletComponentScan 注解对Filter进行扫描
@SpringBootApplication
@ServletComponentScan
public class Application{
   public static void main(String[] args){
      SpringApplication.run(Application.class, args);
   }
}
```

# 四、Servlet3.0的注解自定义原生Listener

常用的监听器接口有 ServletContextListener、ServletRequestListener、ServletSessionListener，分别监听容器、请求、会话的创建和销毁。

ServletContextAttributeListener、ServletRequestAttributeListener、ServletSessionAttributeListener对应监听三者的 addAttribute和 removeAttribute方法。

```java
package top.zhenganwen.springbootweb.servlet;
 
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
 
@WebServlet(name = "userServlet",urlPatterns = "/user/*")
public class UserServlet extends HttpServlet{
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
 
        resp.getWriter().write("hello");
        resp.getWriter().flush();
        resp.getWriter().close();
    }
 
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req, resp);
    }
}
```
