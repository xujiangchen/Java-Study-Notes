- [SpringBoot 注解](#springboot---)
  * [一、启动类注解](#一启动类注解)
    + [1.1@SpringBootApplication](#11springbootapplication)
  * [二、HTTP请求注解](#二HTTP请求注解)
    + [2.1 @RestController](#21-restcontroller)
    + [2.2 从请求路径中获取参数](#22-从请求路径中获取参数)
    + [2.2 @GetMapping](#22-getmapping)
    + [2.3 @RequestParam](#23-requestparam)
    + [2.4 @RequestBody](#24-requestbody)
    + [2.5 @RequestHeader](#25-requestheader)
    + [2.6 @PostMapping](#26-postmapping)
# SpringBoot 注解

## 一、启动类注解

### 1.1@SpringBootApplication

其实这个注解是多个注解的集合

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

其中 这三个注解是核心，将这三个替换(SpringBootApplication) springboot依旧可以正常启动

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

- `SpringBootConfiguration`: 继承自@Configuration，二者功能也一致，标注当前类是配置类
- `EnableAutoConfiguration`: 帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot，并创建对应配置类的Bean，并把该Bean实体交给IoC容器进行管理。
- `ComponentScan`: 根据指定的配置自动扫描package，将符合条件的组件加入到IOC容器中；

## 二、HTTP请求注解

### 2.1 @RestController

- `@RestController = @Controller+@ResponseBody`
- `@ResponseBody` 作用是将java对象转为json格式的数据

```java
@RestController
public class SampleController {

    @RequestMapping("/")
    @ResponseBody
    String home(){
        return "hello world";
    }

    @RequestMapping("/test")
    public Map<String, String> testMap(){
        Map<String,String> map = new HashMap<>();
        map.put("name","mapTest");
        return map;
    }
}
```

### 2.2 从请求路径中获取参数

```java
1）@RequestMapping(path = "/{id}", method = RequestMethod.GET)
public String getUser(@PathVariable String id ) {}

2）@RequestMapping(path = "/{depid}/{userid}", method = RequestMethod.GET)
getUser(@PathVariable("depid") String departmentID,@PathVariable("userid") String userid)



/**
 * 功能描述：测试restful协议，从路径中获取字段
 * @param depId
 * @param userId
 * @return
 */
@RequestMapping(path = "/{dep_id}/{user_id}", method = RequestMethod.GET)
//@GetMapping(path = "/{dep_id}/{user_id}")
public Object findUser(@PathVariable("dep_id") String depId,@PathVariable("user_id") String userId){
    param.clear();
    param.put("depId",depId);
    param.put("userId",userId);
    return  param;
}
```

### 2.2 @GetMapping

- `@GetMapping = @RequestMapping(method = RequestMethod.GET)`

```java
/**
 * 功能描述:测试GetMapping
 * @param num
 * @param s
 * @return
 */
@GetMapping("/dome/getmapping")
public Object getMapping(int num, String s){
    param.clear();
    param.put("num", num);
    param.put("s", s);
    return param;
}
```

### 2.3 @RequestParam

```java
/**
 * 功能描述:默认值，是否必须的参数
 * @param page
 * @param size
 * @return
 */
@GetMapping("/demo/demo_page")
public Object page(@RequestParam(defaultValue = "0", name = "page")String page, int size){
    param.clear();
    param.put("page", page);
    param.put("size", size);
    return param;
}
```

### 2.4 @RequestBody

```java
/**
 * 功能描述：bean对象的传值
 * 注意：1、http头要指定content—type为application/json
 *      2、要使用body传值
 * @param user
 * @return
 */
 
@RequestMapping("save_user")
// 也可以使用@GetMapper("save_user")
public Object saveUser(@RequestBody User user){
    param.clear();
    param.put("user",user);
    return param;
}
```

### 2.5 @RequestHeader

```java
/**
 * 功能描述:测试获取Http的头部信息
 *
 * @param taken
 * @param id
 * @return
 */
@GetMapping("get_header")
public Object getHeader(@RequestHeader("taken")String taken,String id){
    param.clear();
    param.put("taken",taken);
    param.put("id",id);
    return param;
}
```

### 2.6 @PostMapping

- `@PostMapping = @RequestMapping(method = RequestMethod.POST)`

```java
@PostMapping(value = "/create")
public Map<String, Object> createBanner(@RequestBody Map<String, Object> banner){
    banner.put("id", 1243);
    return banner;
}
```
