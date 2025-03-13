# 0. 开发环境

- IDE：IntelliJ IDEA 2019.1.2
- JDK：1.8.0_211
- Spring Boot：2.0.9.RELEASE
- Spring Cloud：Finchley.RELEASE

# 1. Zipkin简介

Zipkin是一个开放源代码的分布式的跟踪系统，每个服务向Zipkin报告计时数据，Zipkin会根据调用关系通过Zipkin UI生成依赖关系图。

# 2. 创建Zipkin服务

Spring Boot2.0以后，官方不推荐我们自定义Zipkin服务端，而是使用官方提供的jar包。下载地址：

`https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/`

这里我们不使用官方提供的jar包，方便我们以后扩展。

## 2.1 新建Zipkin服务

新建Zipkin服务**spring-cloud-zipkin**

## 2.2 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.11.7</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <version>2.11.7</version>
        </dependency>
~~~

## 2.3 新建启动类

**@EnableZipkinServer**注解表示注册为Zipkin服务

~~~java
package cn.wbnull.springcloudzipkin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import zipkin2.server.internal.EnableZipkinServer;

@SpringBootApplication
@EnableEurekaClient
@EnableZipkinServer
public class SpringCloudZipkinApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudZipkinApplication.class, args);
    }
}
~~~

## 2.4 新建application.yml

~~~yml
server:
  port: 8092

spring:
  application:
    name: spring-cloud-zipkin
  sleuth:
    sampler:
      probability: 1.0

management:
  metrics:
    web:
      server:
        auto-time-requests: false

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/
  instance:
    prefer-ip-address: true
~~~

# 3. 创建Zipkin客户端

这里我们直接改造之前的服务**spring-boot-provider**、**spring-boot-provider-v2**、**spring-boot-consumer-feign-hystrix**、**spring-cloud-zuul**，使其成为Zipkin客户端。所有服务均做如下操作。

## 3.1 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
~~~

## 3.2 修改application.yml

指定Zipkin服务的地址，日志发送方式为web。

~~~yml
spring:
  zipkin:
    base-url: http://localhost:8092
    sender:
      type: web
~~~

# 4.测试

依次启动**spring-cloud-eureka**、**spring-boot-provider**、**spring-boot-provider-v2**、**spring-boot-consumer-feign-hystrix**、**spring-cloud-zuul**、**spring-cloud-zipkin**

PostMan多请求几次

![1561648809931](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561648809931.png)

浏览器访问<http://localhost:8092/zipkin/>，界面如下

![1561649118176](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561649118176.png)

点击**Find Traces**按钮

![1561649394977](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561649394977.png)

点击某个调用链路之后可以看到该链路的调用详情

![1561649808773](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561649808773.png)

点击某个服务可以查看该服务的调用详情

![1561649982832](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561649982832.png)

菜单栏选择Dependencies还可以查看各服务之间的关系

![1561649884300](./assets/08_Spring%20Cloud%20全链路跟踪%20Zipkin.assets/1561649884300.png)



---

GitHub：[https://github.com/dkbnull/spring-cloud-demo](https://github.com/dkbnull/spring-cloud-demo)

Gitee：[https://gitee.com/dkbnull/spring-cloud-demo](https://gitee.com/dkbnull/spring-cloud-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/93928005](https://blog.csdn.net/dkbnull/article/details/93928005)

微信：[https://mp.weixin.qq.com/s/03ffLZfep8ZU6hgy8g_tQg](https://mp.weixin.qq.com/s/03ffLZfep8ZU6hgy8g_tQg)

微博：[https://weibo.com/ttarticle/p/show?id=2309404387944860409936](https://weibo.com/ttarticle/p/show?id=2309404387944860409936)

知乎：[https://zhuanlan.zhihu.com/p/71162391](https://zhuanlan.zhihu.com/p/71162391)

----

