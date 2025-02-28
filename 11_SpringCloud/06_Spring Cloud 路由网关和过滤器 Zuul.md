# 0. 开发环境

- IDE：IntelliJ IDEA 2017.1 x64

- JDK：1.8.0_91

- Spring Boot：2.0.9.RELEASE

- Spring Cloud：Finchley.RELEASE

# 1. Zuul简介

Zuul是Netflix开源的一个基于JVM路由和服务端的API Gateway服务器，是一个负载均衡器。

Zuul的主要功能是路由转发和过滤器。路由转发功能是微服务中很重要的一部分。比如 api/sale/* 接口转发到sale服务， api/pay/* 接口转发到pay服务。

Zuul默认整合了Ribbon，实现了负载均衡。

## 1.1 Zuul功能

* Authentication：认证
* Insights：洞察
* Stress Testing：压力测试
* Canary Testing：金丝雀测试
* Dynamic Routing：动态路由
* Service Migration：服务迁移
* Load Shedding：负载均衡
* Security：安全
* Static Response handling：静态响应处理
* Active/Active traffic management：主动/主动交通管理

# 2.路由转发

## 2.1 新建路由网关服务

新建路由网关服务**spring-cloud-zuul**

## 2.2 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
~~~

## 2.3 新建启动类

**@EnableZuulProxy** 注解表示开启Zuul功能。

~~~java
package cn.wbnull.springcloudzuul;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
public class SpringCloudZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudZuulApplication.class, args);
    }
}
~~~

## 2.4 新建application.yml

~~~yml
server:
  port: 8091
  servlet:
    context-path: /springcloudzuul

spring:
  application:
    name: spring-cloud-zuul

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/

zuul:
  routes:
    zuul-a:
      path: /zuul-a/**
      serviceId: spring-boot-provider
    zuul-b:
      path: /zuul-b/**
      serviceId: spring-boot-provider
~~~

## 2.5 测试

依次启动spring-cloud-eureka、spring-boot-provider、spring-boot-provider-v2、spring-cloud-zuul

浏览器访问 <http://127.0.0.1:8091/springcloudzuul/zuul-a/springbootprovider/gateway> 和 <http://127.0.0.1:8091/springcloudzuul/zuul-b/springbootprovider/gateway> ， *hello world,this is spring-boot-provider* 和 *hello world,this is spring-boot-provider-v2* 交替返回，路由成功。

* 127.0.0.1:8091/springcloudzuul 是 spring-cloud-zuul 的访问地址（ip:port/server.servlet.context-path）

* zuul-a 和 zuul-b 是我们要路由的地址，根据application.yml配置，这俩地址被路由到 服务id名称为 spring-boot-provider 的服务。

* springbootprovider/gateway 是 spring-boot-provider 服务的接口访问地址（server.servlet.context-path/接口名），与之前相同，serviceId替换的只是ip:port部分。

## 2.6 路由配置方式2

也可以使用如下方式配置路由地址

~~~yml
zuul:
  routes:
    zuul-a:
      path: /zuul-a/**
      url: http://localhost:8081/springbootprovider
    zuul-b:
      path: /zuul-b/**
      url: http://localhost:8083/springbootprovider
~~~

依次启动spring-cloud-eureka、spring-boot-provider、spring-boot-provider-v2、spring-cloud-zuul

浏览器访问 <http://127.0.0.1:8091/springcloudzuul/zuul-a/gateway> ，返回 *hello world,this is spring-boot-provider* ;浏览器访问 <http://127.0.0.1:8091/springcloudzuul/zuul-b/gateway> ，返回 *hello world,this is spring-boot-provider-v2* ，路由成功。

# 3. 服务过滤

刚才我们已经说过，Zuul的主要功能是路由转发和过滤器，现在我们来介绍下过滤器。

## 3.1 新建过滤器类

*cn.wbnull.springcloudzuul* 包下新建 **filter** 包，再新建 **GlobalFilter** 类，并 **注入到IoC容器中**。

~~~java
package cn.wbnull.springcloudzuul.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Component
public class GlobalFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest servletRequest = context.getRequest();
        Object token = servletRequest.getHeader("token");
        if (token == null) {
            context.setSendZuulResponse(false);
            context.setResponseStatusCode(401);
            try {
                context.getResponse().getWriter().write("error: token is null");
            } catch (IOException e) {
            }
        }

        return null;
    }
}
~~~

* filterType()：过滤器类型，执行过滤器的时机
  * pre：路由前
  * routing：路由时
  * post：路由后
  * error：发送错误时
* filterOrder()：过滤顺序
* shouldFilter()：是否需要过滤，可以丰富代码进行逻辑判断，true：过滤；false：不过滤。就像上面示例代码中不进行逻辑判断直接 return true 表示所有路由都过滤。
* run()：过滤器具体逻辑实现。

## 3.2 测试

依次启动spring-cloud-eureka、spring-boot-provider、spring-boot-provider-v2、spring-cloud-zuul，浏览器访问 <http://127.0.0.1:8091/springcloudzuul/zuul-a/springbootprovider/gateway> ，返回 **error: token is null** 。

我们换Postman，Header加上token，再次请求，返回正常，且 *hello world,this is spring-boot-provider* 和 *hello world,this is spring-boot-provider-v2* 交替返回，过滤成功。

# 4. Zuul整合Hystrix

上面我们测试，路由配置的是服务提供者的地址，这样我们之前配置的负载均衡策略、断路器就都无法生效了。

下面我们修改路由服务Id名称，使路由到服务消费者。

## 4.1 修改spring-boot-consumer-feign-hystrix 

修改 **spring-boot-consumer-feign-hystrix** 服务 **application.yml** 配置文件

~~~yml
eureka:
  client:
    register-with-eureka: true
~~~

其他配置不变。

## 4.1 修改spring-cloud-zuul

修改 **spring-cloud-zuul** 服务 **application.yml** 配置文件

这里我们不仅要配置zuul路由，还要配置ribbon超时时间，否则可能启动服务失败或断路器不起作用。

~~~yml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000

zuul:
  routes:
    zuul-a:
      path: /zuul-a/**
      serviceId: spring-boot-consumer-feign-hystrix
~~~

## 4.2 测试

1、依次启动 spring-cloud-eureka、spring-boot-provider、spring-boot-provider-v2、spring-boot-consumer-feign-hystrix、spring-cloud-zuul，使用Postman进行测试，测试地址：http://127.0.0.1:8091/springcloudzuul/zuul-a/springbootconsumer/users ，Header加上token，可以看到返回正常，且两组返回信息交替返回。

~~~json
{
  "name": "Zuul 测试 name",
  "hello world": "spring-boot-provider"
}
~~~

~~~json
{
  "name": "Zuul 测试 name",
  "hello world": "spring-boot-provider-v2"
}
~~~

2、将 spring-boot-provider、spring-boot-provider-v2 服务停止，再次请求，可以看到返回熔断方法内参数。

~~~json
{
  "name": "Zuul 测试 name",
  "fallback": "spring-boot-consumer-feign-hystrix by GatewayFallbackFactory",
  "throwable": "feign.RetryableException: Connection refused: connect executing POST http://spring-boot-provider/springbootprovider/users"
}
~~~



---

GitHub：[https://github.com/dkbnull/SpringCloudDemo](https://github.com/dkbnull/SpringCloudDemo)

Gitee：[https://gitee.com/dkbnull/SpringCloudDemo](https://gitee.com/dkbnull/SpringCloudDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/89736893](https://blog.csdn.net/dkbnull/article/details/89736893)

微信：[https://mp.weixin.qq.com/s/lWMudofECNpskWTioI2cIg](https://mp.weixin.qq.com/s/lWMudofECNpskWTioI2cIg)

微博：[https://weibo.com/ttarticle/p/show?id=2309404370511336471294](https://weibo.com/ttarticle/p/show?id=2309404370511336471294)

知乎：[https://zhuanlan.zhihu.com/p/65360773](https://zhuanlan.zhihu.com/p/65360773)

---

