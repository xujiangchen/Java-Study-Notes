## 基础结构

- **src/main/java:** 存放代码
- **src/main/resources:**
  - static: 存放静态文件，比如 css、js、image,（访问方式http://localhost:8080/js/main.js）
  - templates:存放静态页面jsp,html,tpl
  - config:存放配置文件,application.properties
  - resources:

## 同个文件的加载顺序,静态资源文件

**Spring Boot** 默认会挨个从**META/resources > resources > static > public** 里面找是否存在相应的资源，如果有则直接返回。

## package组织原则

先分模块，再分层。先分层，再分模块的组织方式，**被称为“贫血模型”**，不利于面向对象思维，需要尽量避免。

