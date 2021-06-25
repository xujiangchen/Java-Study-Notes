- [Springboot的异常处理](#springboot-----)
  * [一、自定义错误页面](#一自定义错误页面)
  * [二、@ExceptionHandle 注解处理异常](#二ExceptionHandle-注解处理异常)
  * [三、@ControllerAdvice+@ExceptionHandler 注解处理异常](#三ControllerAdviceExceptionHandler-注解处理异常)
  * [四、SimpleMappingExceptionResolver 处理异常](#SimpleMappingExceptionResolver-处理异常)
  * [五、自定义 HandlerExceptionResolver 类处理异常](#五自定义-HandlerExceptionResolver-类处理异常)



# Springboot的异常处理

## 一、自定义错误页面

Springboot自己提供了一套处理异常的机制，一旦程序中出现了异常SpringBoot会像error的url发送请求。在SpringBoot中提供了一个叫BasicExceptionController来处理/error 请求，然后跳转到默认显示异常的页面来展示异常信息。

如果我们需要将所有的异常都跳转到自定义的错误页面，那么就需要在src/main/resources/templates下创建一个 error.html 页面，**必须叫error**

## 二、@ExceptionHandle 注解处理异常

如果需要针对特定的异常做出不同的处理，我们可以通过@ExceptionHandle来处理实现，具体如下

**Controller**

```java
/**
 * @program: springboot-exception
 * @description: 异常处理
 * @author: xujiangchen
 */
@Controller
public class DemoController {

	 /**
     * 模拟 NullPointerException
     * @return
     */
    @RequestMapping("/show1")
    public String showInfo(){
        String str = null;
        str.length();
        return "index";
    }

    /**
     * 模拟 ArithmeticException
     * @return
     */
    @RequestMapping("/show2")
    public String showInfo2(){
        int a = 10/0;
        return "index";
    }

    /**
     * java.lang.ArithmeticException
     * 该方法需要返回一个 ModelAndView：目的是可以让我们封装异常信息以及视图的指定
     * 参数 Exception e:会将产生异常对象注入到方法中
     */
    @ExceptionHandler(value={java.lang.ArithmeticException.class})
    public ModelAndView arithmeticExceptionHandler(Exception e){
        ModelAndView mv = new ModelAndView();
        mv.addObject("error", e.toString());
        mv.setViewName("error1");
        return mv;
    }
    /**
     * java.lang.NullPointerException
     * 该方法需要返回一个 ModelAndView：目的是可以让我们封装异常信息以及视
     图的指定
     * 参数 Exception e:会将产生异常对象注入到方法中
     */
    @ExceptionHandler(value={java.lang.NullPointerException.class})
    public ModelAndView nullPointerExceptionHandler(Exception e){
        ModelAndView mv = new ModelAndView();
        mv.addObject("error", e.toString());
        mv.setViewName("error2");
        return mv;
    }

}
```

## 三、@ControllerAdvice+@ExceptionHandler 注解处理异常

第二中方式中的处理有一个非常明显的弊端，异常处理的代码和业务代码放在一个类中了，这种方式**耦合性太强了**，最好的方式是将两者拆开管理，这时候我们就可以使用 `ControllerAdvice` 来定义一个专门处理异常的类。这样逻辑代码中，只需要关注自己所写的业务逻辑即可，异常信息同一交由这个特殊的类来处理。

**异常处理类**

```java
**
 * @program: springboot-exception
 * @description: 全局异常处理
 * @author: xujianchen 
 */
@ControllerAdvice
public class GlobalException {

    /**
     * java.lang.ArithmeticException
     * 该方法需要返回一个 ModelAndView：目的是可以让我们封装异常信息以及视图的指定
     * 参数 Exception e:会将产生异常对象注入到方法中
     */
    @ExceptionHandler(value={java.lang.ArithmeticException.class})
    public ModelAndView arithmeticExceptionHandler(Exception e){
        ModelAndView mv = new ModelAndView();
        mv.addObject("error", e.toString()+" -- advice");
        mv.setViewName("error1");
        return mv;
    }

    /**
     * java.lang.NullPointerException
     * 该方法需要返回一个 ModelAndView：目的是可以让我们封装异常信息以及视
     图的指定
     * 参数 Exception e:会将产生异常对象注入到方法中
     */
    @ExceptionHandler(value={java.lang.NullPointerException.class})
    public ModelAndView nullPointerExceptionHandler(Exception e){
        ModelAndView mv = new ModelAndView();
        mv.addObject("error", e.toString()+" -- advice");
        mv.setViewName("error2");
        return mv;
    }
}

```

## 四、SimpleMappingExceptionResolver 处理异常

第三种方法中也有一个小的弊端，就是每种类型的异常，都要写一个单独的方法，并指定对应的页面，这时候就可以使用 `SimpleMappingExceptionResolver` 来简化这个步骤

```java
/**
 * @program: springboot-exception
 * @description: 全局异常处理器
 * @author: xujiangchen
 */
@Configuration
public class GlobalException {
    /**
     * 该方法必须要有返回值。返回值类型必须是：
     SimpleMappingExceptionResolver
     */
    @Bean
    public SimpleMappingExceptionResolver
    getSimpleMappingExceptionResolver(){
        SimpleMappingExceptionResolver resolver = new
                SimpleMappingExceptionResolver();
        Properties mappings = new Properties();
        /**
         * 参数一：异常的类型，注意必须是异常类型的全名
         * 参数二：视图名称
         */
        mappings.put("java.lang.ArithmeticException", "error1");
        mappings.put("java.lang.NullPointerException","error2");
        //设置异常与视图映射信息的
        resolver.setExceptionMappings(mappings);
        return resolver;
    }
}

```

## 五、自定义 HandlerExceptionResolver 类处理异常

```java
/**
 * @program: springboot-exception
 * @description: 全局异常处理器
 * @author: xujiangchen
 * @create: 2019-05-17 10:07
 */
@Configuration
public class GlobalException implements HandlerExceptionResolver {

    /**
     * @param httpServletRequest
     * @param httpServletResponse
     * @param o
     * @param e
     * @return
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest
            , HttpServletResponse httpServletResponse
            , Object o, Exception ex) {
        ModelAndView mv = new ModelAndView();
        //判断不同异常类型，做不同视图跳转
        if (ex instanceof ArithmeticException) {
            mv.setViewName("error1");
        }
        if (ex instanceof NullPointerException) {
            mv.setViewName("error2");
        }
        mv.addObject("error", ex.toString());
        return mv;
    }
}
```
