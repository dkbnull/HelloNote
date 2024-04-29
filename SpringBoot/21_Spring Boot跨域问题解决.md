# 0 开发环境

- JDK：1.8
- Spring Boot：2.7.18

# 1 跨域请求

在请求时，如果出现了以下情况中的任意一种，那么它就是跨域请求：

**协议不同，如 http 和 https；**

**域名不同；**

**端口不同；**

# 2 跨域问题演示

## 2.1 配置端口

~~~yml
server:
  port: 8090
~~~

## 2.2 新建访问接口

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @PostMapping(value = "login")
    public String login() {
        return "姓名：张三；性别：男";
    }
}
~~~

## 2.3 新建前端页面

新建前端页面**login.html**

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>

    <style>
        .container {
            width: 300px;
            margin: auto;
            padding: 40px;
            border: 1px solid #ccc;
            background-color: white;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        button {
            width: 100%;
            padding: 10px;
            color: white;
            background-color: #4CAF50;
            cursor: pointer;
            border: none;
            outline: none;
        }

        button:hover {
            opacity: 0.9;
        }
    </style>
</head>
<body>
<div class="container">
    <button onclick="submit()">登 录</button>
</div>
</body>
<script src="jquery-3.7.1.min.js"></script>
<script>
    function submit() {
        $.ajax({
            url: "http:127.0.0.1:8090/user/login",
            type: "POST",
            success: function (result) {
                alert(result);
            }
        });
    }
</script>
</html>
~~~

## 2.4 测试

启动服务，然后直接本地双击打开**login.html**，点击登录按钮，如下所示，出现跨域问题

![image-20240428171918963](21_Spring%20Boot%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3.assets/image-20240428171918963.png)

# 3 解决办法

通常跨域问题使用以下6个方法解决

* **使用 @CrossOrigin 注解实现跨域；**
* **通过配置文件实现跨域；**
* **通过 CorsFilter 对象实现跨域；**
* **通过 Response 对象实现跨域；**
* **通过实现 ResponseBodyAdvice 实现跨域；**
* **使用过滤器来实现跨域；**

## 3.1 通过@CrossOrigin注解跨域

**@CrossOrigin注解可以修饰类和方法。当修饰类时，表示此类中的所有接口都可以跨域；当修饰方法时，表示此方法可以跨域**，

### 3.1.1 调整接口类

~~~java
@RestController
@RequestMapping("user")
@CrossOrigin(origins = "*")
public class UserController {

    @PostMapping(value = "login")
    public String login() {
        return "姓名：张三；性别：男";
    }
}
~~~

### 3.1.2 测试

启动服务，访问页面，点击登录按钮，接口访问成功

![image-20240428171959021](21_Spring%20Boot%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3.assets/image-20240428171959021.png)

### 3.1.3 优缺点

优点：实现跨域非常简单

缺点：只能实现局部跨域，当一个项目中存在多个类时，需要给所有类上都添加此注解，比较麻烦

## 3.2 通过配置文件跨域

**通过配置文件的方式可实现全局跨域**

~~~java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        //支持所有接口
        registry.addMapping("/**")
                //是否发送Cookie
                .allowCredentials(true)
                //支持域
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowedHeaders("*")
                .exposedHeaders("*");
    }
}
~~~

## 3.3 通过CorsFilter跨域

**可实现全局跨域**

~~~java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class GlobalCorsFilter {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        //支持域
        config.addAllowedOriginPattern("*");
        //是否发送Cookie
        config.setAllowCredentials(true);
        //支持请求方式
        config.addAllowedMethod("*");
        //允许的原始请求头部信息
        config.addAllowedHeader("*");
        //暴露的头部信息
        config.addExposedHeader("*");

        //添加地址映射
        UrlBasedCorsConfigurationSource corsConfigurationSource = new UrlBasedCorsConfigurationSource();
        corsConfigurationSource.registerCorsConfiguration("/**", config);

        return new CorsFilter(corsConfigurationSource);
    }
}
~~~

## 3.4 通过Response跨域

**解决跨域问题最原始的方式，支持任意版本的Spring Boot。但也是局部跨域，仅支持方法级别的跨域**

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @PostMapping(value = "login")
    public String login(HttpServletResponse response) {
        response.setHeader("Access-Control-Allow-Origin", "*");

        return "姓名：张三；性别：男";
    }
}
~~~

## 3.5 通过ResponseBodyAdvice跨域

**通过重写ResponseBodyAdvice接口中的beforeBodyWrite（返回之前重写）方法，可以对所有的接口设置跨域**

**为全局跨域，对整个项目中的所有接口有效**

~~~java
@ControllerAdvice
public class CorsResponseBodyAdvice implements ResponseBodyAdvice {

    /**
     * 内容是否需要重写
     * 可以选择部分控制器和方法进行重写
     * <p>
     * true 重写
     */
    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        //设置跨域
        response.getHeaders().set("Access-Control-Allow-Origin", "*");

        return body;
    }
}
~~~

## 3.6 使用过滤器跨域

实现方式同3.5，不再赘述

# 4 原理

跨域问题本质上是浏览器的行为，它的初衷是为了保证用户的访问安全，防止被恶意网站窃取数据。因此解决跨域问题就只是需要告诉浏览器这是一个安全的请求就可以了，告诉浏览器“我是自己人”。

那怎么告诉浏览器这是一个安全的请求呢？只需要**在返回头中设置“Access-Control-Allow-Origin”参数即可。此参数就是用来表示允许跨域访问的原始域名的，当设置为“\*”时，表示允许所有站点跨域访问**

未设置跨域时：

![image-20240428223807309](21_Spring%20Boot%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3.assets/image-20240428223807309.png)

设置跨域后：

![image-20240428223958782](21_Spring%20Boot%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3.assets/image-20240428223958782.png)

---

CSDN：[https://blog.csdn.net/dkbnull/article/details/138294214](https://blog.csdn.net/dkbnull/article/details/138294214)

微信：[https://mp.weixin.qq.com/s/sUdk5E6cptl0QaWK2qiXUw](https://mp.weixin.qq.com/s/sUdk5E6cptl0QaWK2qiXUw)

知乎：[https://zhuanlan.zhihu.com/p/695342039](https://zhuanlan.zhihu.com/p/695342039)

---