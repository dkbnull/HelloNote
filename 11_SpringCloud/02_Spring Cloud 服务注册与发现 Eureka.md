在上一篇文章 [Spring Cloud整合Spring Boot（服务提供者和服务消费者）](https://blog.csdn.net/dkbnull/article/details/89223691) 中，我们已经创建了一个服务提供者和服务消费者，但是在消费者调用提供者接口的时候，我们把提供者的地址硬编码在了消费者代码里，这样我们的代码极其不优雅，也不利于维护。现在，我们使用Eureka来优雅地实现消费者调用提供者。

# 0. 开发环境

- IDE：IntelliJ IDEA 2017.1 x64

- jdk：1.8.0_91

- Spring Boot：2.0.9.RELEASE

- Spring Cloud：Finchley.RELEASE


# 1. 新建Eureka服务注册中心

## 1.1 新建Eureka服务注册中心

![1555080362878](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555080362878.png)

## 1.2 引入依赖

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-eureka</artifactId>
    <packaging>jar</packaging>

    <parent>
        <groupId>cn.wbnull</groupId>
        <artifactId>spring-cloud-demo</artifactId>
        <version>1.0.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
</project>
~~~

## 1.3 新建application.yml

~~~yml
server:
  port: 8090
  servlet:
    context-path: /springcloudeureka

spring:
  application:
    name: spring-cloud-eureka

eureka:
  instance:
    hostname: localhost
  client:
    #此项目不作为客户端注册
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:${server.port}/springcloudeureka/eureka/
~~~

这里我们解释下这两个配置的含义

* register-with-eureka：表示是否注册自身到Eureka服务器
* fetch-registry：表示是否从Eureka服务器上获取注册信息

## 1.4 新建Spring Boot入口类

这里要注意，入口类增加<font color="#FF0000">**@EnableEurekaServer**</font>注解，表示此项目是Eureka服务端

~~~java
package cn.wbnull.springcloudeureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class SpringCloudEurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudEurekaApplication.class, args);
    }
}
~~~

## 1.5 启动服务

启动Eureka服务，浏览器访问 http://127.0.0.1:8090/springcloudeureka/ 可以打开Eureka界面，这里因为我们未注册服务，所以Application是空的。

![1555082087410](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555082087410.png)

# 2. 新建服务提供者

**这里我们直接修改上篇中创建的服务提供者spring-boot-provider，使其成为Eureka客户端**

## 2.1 引入依赖

~~~yml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
~~~

## 2.2 修改application.yml

application.yml 增加如下配置

~~~yml
eureka:
  client:
    #将此项目注册到Eureka服务
    register-with-eureka: true
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/
~~~

## 2.3 修改Spring Boot入口类

SpringBootProviderApplication 增加注解<font color="#FF0000">**@EnableEurekaClient**</font>，表示此项目是Eureka客户端

~~~java
@SpringBootApplication
@EnableEurekaClient
public class SpringBootProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootProviderApplication.class, args);
    }
}
~~~

## 2.4 启动服务

我们先启动Eureka服务，再启动spring-boot-provider，然后再次浏览器访问 http://127.0.0.1:8090/springcloudeureka/ ，服务注册成功。

![1555084777547](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555084777547.png)

# 3. 再新建一个服务提供者

这里，我们再创建一个新的服务提供者spring-boot-provider-v2，可以直接复制spring-boot-provider。

复制创建完成后，我们修改一些配置以区别两个服务提供者

## 3.1 pom.xml

两个服务提供者的<font color="#FF0000">**artifactId**</font>不能相同，这里我们修改第二个服务提供者<font color="#FF0000">**artifactId**</font>为<font color="#FF0000">**spring-boot-provider-v2**</font>

~~~xml
    <artifactId>spring-boot-provider-v2</artifactId>
    <packaging>jar</packaging>
~~~

## 3.2 application.yml

两个服务提供者的端口不能冲突，这里我么修改第二个服务提供者的 <font color="#FF0000">**server.port=8083**</font>，但是<font color="#FF0000">**spring.application.name=spring-boot-provider**</font>，两个服务提供者名字相同

~~~yml
server:
  port: 8083
  servlet:
    context-path: /springbootprovider

spring:
  application:
    name: spring-boot-provider
~~~

## 3.3 GatewayController

区别两个服务提供者的返回信息，这里我们修改第二个服务提供者GatewayController接口返回信息

~~~java
@RestController
@Scope("prototype")
public class GatewayController {

    @GetMapping(value = "/gateway")
    public String gateway() throws Exception {
        return "hello world,this is spring-boot-provider-v2";
    }
}
~~~

## 3.4 启动服务

 启动新建的服务提供者，然后再次浏览器访问 http://127.0.0.1:8090/springcloudeureka/ ，如下所示，有两个spring-boot-provider服务提供者，端口分别是8081,8083

![1555085750848](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555085750848.png)

# 4. 新建服务消费者

回到我们开篇说的，我们上篇创建的服务消费者在调用服务提供者接口时把提供者的地址硬编码在了消费者代码里，现在我们来解决这个问题。

**这里我们也直接修改上篇中创建的服务消费者spring-boot-consumer，使其成为Eureka客户端**

## 4.1 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
~~~

## 4.2 修改application.yml

~~~yml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/
~~~

## 4.3 修改Spring Boot入口类

SpringBootConsumerApplication增加两个注解，<font color="#FF0000">**@EnableEurekaClient**表示此项目是Eureka客户端</font>，<font color="#FF0000">**@LoadBalanced**表示开启负载均衡</font>

~~~java
package cn.wbnull.springbootconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
public class SpringBootConsumerApplication {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootConsumerApplication.class, args);
    }
}
~~~

## 4.4 修改GatewayController

将原来硬编码写死的服务提供者的地址 http://localhost:8081/springbootprovider/gateway 改成 http://spring-boot-provider/springbootprovider/gateway 

**spring-boot-provider** 就是我们刚才创建的服务提供者的 **spring.application.name**，也就是注册到Eureka中的服务的id。

<font color="#FF0000">**在Eureka中，我们可以不使用 ip:port 而是直接使用 服务id 去访问服务。但是这里要注意，服务id 替换的只是 ip:port，server.servlet.context-path 还是要保留的。**</font>

~~~java
@RestController
@Scope("prototype")
public class GatewayController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/gateway")
    public String gateway() throws Exception {
        return restTemplate.getForObject("http://spring-boot-provider/springbootprovider/gateway", String.class);
    }
}
~~~

## 4.5 测试

我们把四个服务依次启动起来，然后浏览器访问 http://127.0.0.1:8082/springbootconsumer/gateway 

不断刷新几次，可以看到，接口有时会返回**hello world,this is spring-boot-provider**，有时会返回**hello world,this is spring-boot-provider-v2**。这是因为我们使用**@LoadBalanced**注解开启了负载均衡。

![1555090117969](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555090117969.png)

![1555090136926](02_Spring%20Cloud%20服务注册与发现%20Eureka.assets/1555090136926.png)



---

GitHub：[https://github.com/dkbnull/SpringCloudDemo](https://github.com/dkbnull/SpringCloudDemo)

Gitee：[https://gitee.com/dkbnull/SpringCloudDemo](https://gitee.com/dkbnull/SpringCloudDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/89268194](https://blog.csdn.net/dkbnull/article/details/89268194)

微信：[https://mp.weixin.qq.com/s/L4WVLGh7d6ULl9bP6IdxVg](https://mp.weixin.qq.com/s/L4WVLGh7d6ULl9bP6IdxVg)

微博：[https://weibo.com/ttarticle/p/show?id=2309404366699653683350](https://weibo.com/ttarticle/p/show?id=2309404366699653683350)

知乎：[https://zhuanlan.zhihu.com/p/64284070](https://zhuanlan.zhihu.com/p/64284070)

---

