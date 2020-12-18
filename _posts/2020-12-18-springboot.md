---
layout:     post
title:      "Spring Boot"
subtitle:   "Hello World, Hello Blog"
date:       2020-12-10
author:     "Caiiiiii"
catalog:    true
header-img: "img/steam12-10-1.jpg"
tags:
    - Spring
    - Java  
---

# Spring Boot

Spring Boot 是Java的轻量级开发框架， 本质上就是Spring框架的扩展 ，它消除了 Spirng 程序所需的复杂配置。 

## 配置文件的读取

application.yml      //在yml中表示数组用 - [name]来进行分割。
```
name: Caiiiiii

library:
    location: 深圳
    books：
        - name: 书本1
          description: 书本描述1

        - name: 书本2
          description： 书本描述2


my-profile:
    name: Caiiiiii
    email: 995017591@qq.com
``` 

### 1.通过@Value读取简单的配置信息
```
@Value("${name}")
String name;              //Caiiiiii
```

### 2.通过@ConfigurationProperties读取并与Bean绑定

```
@Component              //Bean 绑定
@ConfigurationProperties(prefix = "library")
@Setter
@Getter
class LibraryProperties {
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
}
```

### 3.通过@ConfigurationProperties读取并校验
```
@Getter
@Setter
@ToString
@ConfigurationProperties("my-profile")
@Validated          //校验
public class ProfileProperties {
   @NotEmpty
   private String name;

   @Email              //校验邮箱格式
   @NotEmpty
   private String email;
}
```

### 4.通过@PropertySource 读取指定properties文件
```
@Component
@PropertySource("classpath:website.properties")
@Getter
@Setter
class WebSite {
    @Value("${url}")
    private String url;
}
```

## 异常处理 

### 1.使用 @ControllerAdvice 和 @ExceptionHandler 处理全局异常

1.异常信息实体类，包装异常信息（非必要）
```
public class ErrorResponse {

    private String message;
    private String errorTypeName;
  
    public ErrorResponse(Exception e) {
        this(e.getClass().getName(), e.getMessage());
    }

    public ErrorResponse(String errorTypeName, String message) {
        this.errorTypeName = errorTypeName;
        this.message = message;
    }
    ......省略getter/setter方法
}
```

2.自定义异常类型
一般我们处理的都是 RuntimeException ，所以如果你需要自定义异常类型的话直接继承这个类就可以了。
```
public class ResourceNotFoundException extends RuntimeException {
    private String message;

    public ResourceNotFoundException() {
        super();
    }

    public ResourceNotFoundException(String message) {
        super(message);
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

3. 新建异常处理类  (@ControllerAdvice + @ExceptionHandler)
我们只需要在类上加上@ControllerAdvice注解这个类就成为了全局异常处理类，当然你也可以通过 assignableTypes 指定特定的 Controller 类，让异常处理类只处理特定类抛出的异常。
```
@ControllerAdvice(assignableTypes = {ExceptionController.class})
@ResponseBody
public class GlobalExceptionHandler {

    ErrorResponse illegalArgumentResponse = new ErrorResponse(new IllegalArgumentException("参数错误!"));
    ErrorResponse resourseNotFoundResponse = new ErrorResponse(new ResourceNotFoundException("Sorry, the resourse not found!"));

    @ExceptionHandler(value = Exception.class)// 拦截所有异常, 这里只是为了演示，一般情况下一个方法特定处理一种异常
    public ResponseEntity<ErrorResponse> exceptionHandler(Exception e) {

        if (e instanceof IllegalArgumentException) {
            return ResponseEntity.status(400).body(illegalArgumentResponse);
        } else if (e instanceof ResourceNotFoundException) {
            return ResponseEntity.status(404).body(resourseNotFoundResponse);
        }
        return null;
    }
}
```