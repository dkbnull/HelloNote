在微服务架构中，业务会被拆分成一个个服务，服务间可以彼此调用。为了保证服务的高可用性，单个服务通常会被集群部署，但是由于网络等原因，服务并不能保证100%可用。如果某个服务出现了问题，那么调用这个服务就会出现线程阻塞，如果此时又有大量的请求涌入，Servlet容器的线程资源就会被迅速消耗殆尽，最终导致服务瘫痪。服务与服务之间的依赖，导致了故障会传播，以致于会对整个微服务系统造成影响，导致整个服务瘫痪，这种现象叫做雪崩现象。

如何保证在一个服务出现问题的情况下，不会导致整个服务瘫痪，这就是Hystrix需要做的事情。

# 0. 开发环境

- IDE：IntelliJ IDEA 2017.1 x64

- jdk：1.8.0_91

- Spring Boot：2.0.9.RELEASE

- Spring Cloud：Finchley.RELEASE


# 1. Hystrix简介

Hystrix是一个实现了断路器模式的库，提供了熔断、隔离、Fallback、cache、监控等功能，能够在一个或多个依赖出现问题时保证系统依然可用。

我们可以把Hystrix想象成一个保险丝。在我们家庭的电路系统中，外部电路入户时通常都会加上一个保险丝，当家庭电路系统中某一处发生意外，外部电压过高，达到保险丝熔点的时候，保险丝就会被熔断，切断家庭与外部电路的联通，进而保障家庭用电系统不会受到损坏。

Hystrix提供的断路器就有类似功能，当在一定时间段内，服务消费者调用服务提供者的服务，次数达到设定的阈值，并且出错的次数也达到设置的出错阈值时，就会进行服务降级，让服务消费者直接执行本地设置的降级策略，不再调用服务消费者的服务。

Hystrix提供的断路器具有自我反馈和自我恢复的功能，Hystrix会根据接口调用的情况，让断路器在closed,open,half-open三种状态之间自动切换。

* open状态表示打开熔断，即服务消费者执行本地设置的降级策略，不再调用服务消费者。
* closed状态表示关闭熔断，此时服务消费者直接调用服务提供者。
* half-open状态，这是一个中间状态，当断路器处于该状态时，服务消费者直接调用服务提供者。 

# 2. 新建服务消费者

## 2.1 新建服务消费者

![1556263855029](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556263855029.png)

## 2.2 引入依赖

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-boot-consumer-hystrix</artifactId>
    <packaging>jar</packaging>

    <parent>
        <groupId>cn.wbnull</groupId>
        <artifactId>spring-cloud-demo</artifactId>
        <version>1.0.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>
</project>
~~~

## 2.3 新建application.yml

~~~yml
server:
  port: 8085
  servlet:
    context-path: /springbootconsumer

spring:
  application:
    name: spring-boot-consumer-hystrix

eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/
~~~

## 2.4 新建启动类

新建Spring Boot启动类SpringBootConsumerHystrixApplication，增加如下注解，其中**@EnableHystrix**注解表示开启Hystrix

~~~java
package cn.wbnull.springbootconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
@EnableHystrix
public class SpringBootConsumerHystrixApplication {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootConsumerHystrixApplication.class, args);
    }
}
~~~

## 2.5新建控制器类

这里我们在接口方法users()上增加了@HystrixCommand(fallbackMethod = "usersFallback")，**@HystrixCommand**注解表示对该方法开启断路器功能，指定了熔断方法是usersFallback。

~~~java
package cn.wbnull.springbootconsumer.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Scope;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@RestController
@Scope("prototype")
public class GatewayController {

    @Autowired
    private RestTemplate restTemplate;

    @PostMapping(value = "/users")
    @HystrixCommand(fallbackMethod = "usersFallback")
    public Map<String, String> users(@RequestBody Map<String, String> request) throws Exception {
        ResponseEntity<Map> responseEntity = restTemplate.postForEntity("http://spring-boot-provider/springbootprovider/users", request, Map.class);

        return responseEntity.getBody();
    }

    public Map<String, String> usersFallback(Map<String, String> request) throws Exception {
        request.put("fallback", "spring-boot-consumer-hystrix");

        return request;
    }
}
~~~

## 2.6 测试

依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer-hystrix。然后打开Postman，配置如下，不断点击Send按钮，可以看到返回信息正常，且两组返回信息交替出现。

![1556267786005](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556267786005.png)

然后我们关闭spring-boot-provider，spring-boot-provider-v2服务，再在Postman中测试，可以看到返回正常，返回信息如下，正是我们刚才usersFallback()方法中的处理。

![1556267913399](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556267913399.png)

这里如果我们没有使用Hystrix的话，正常应该是会等待超时，然后返回超时信息。现在启用Hystrix，当spring-boot-provider和spring-boot-provider-v2服务不可用时，spring-boot-consumer-hystrix调用spring-boot-provider的接口不通，会执行快速失败，直接返回usersFallback()方法的处理结果，而不是等待超时，防止了线程阻塞。

## 2.7 修改Hystrix默认超时时间

### 2.7.1 修改控制器类

修改GatewayController类中users()方法，增加上休眠2秒。

~~~java
    @PostMapping(value = "/users")
    @HystrixCommand(fallbackMethod = "usersFallback")
    public Map<String, String> users(@RequestBody Map<String, String> request) throws Exception {
        Thread.sleep(2000);

        ResponseEntity<Map> responseEntity = restTemplate.postForEntity("http://spring-boot-provider/springbootprovider/users", request, Map.class);

        return responseEntity.getBody();
    }
~~~

然后我们再依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer-hystrix。然后继续使用Postman测试，发现依旧返回熔断方法中参数。

![1556283792704](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556283792704.png)

这是因为Hystrix默认的超时时间是1秒，也就是服务消费者在等待服务提供者返回信息时，如果超过1秒没有得到返回信息的话，就认为服务提供者故障，直接执行本地设置的降级策略，也就是usersFallback()方法。

我们可以通过配置文件修改Hystrix超时时间

### 2.7.2 修改Hystrix超时时间

application.yml文件增加如下配置

~~~yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000
~~~

然后我们重启spring-boot-consumer-hystrix，再次使用Postman测试，返回信息正常。

![1556283960427](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556283960427.png)

# 3. Feign中使用断路器

Feign中已经默认集成了Hystrix，但是Spring Cloud在D版本之后没有默认打开。需要在配置文件中配置打开它。

~~~properties
feign.hystrix.enabled=true
~~~

## 3.1 新建Feign服务消费者

![1556285821616](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556285821616.png)

## 3.2 引入依赖

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-boot-consumer-feign-hystrix</artifactId>
    <packaging>jar</packaging>

    <parent>
        <groupId>cn.wbnull</groupId>
        <artifactId>spring-cloud-demo</artifactId>
        <version>1.0.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
</project>
~~~

## 3.3 新建application.yml

~~~yml
server:
  port: 8086
  servlet:
    context-path: /springbootconsumer

spring:
  application:
    name: spring-boot-consumer-feign-hystrix

eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8090/springcloudeureka/eureka/

feign:
  hystrix:
    enabled: true
~~~

## 3.4 新建启动类

~~~java
package cn.wbnull.springbootconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class SpringBootConsumerFeignHystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootConsumerFeignHystrixApplication.class, args);
    }
}
~~~

## 3.5 新建FeignClient类

这里熔断之后的降级处理策略不是指定单个熔断方法，而是指定一个类。@FeignClient(value = "spring-boot-provider", fallback = GatewayFallback.class)

~~~java
package cn.wbnull.springbootconsumer.feign;

import cn.wbnull.springbootconsumer.fallback.GatewayFallback;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.Map;

@Component
@FeignClient(value = "spring-boot-provider", fallback = GatewayFallback.class)
public interface GatewayFeignClient {

    @PostMapping(value = "/springbootprovider/users")
    Map<String, String> users(Map<String, String> request) throws Exception;
}
~~~

## 3.6 新建Fallback类

新建熔断处理类GatewayFallback，GatewayFallback类需要实现上面的GatewayFeignClient接口，并注入到IoC容器中。

~~~java
package cn.wbnull.springbootconsumer.fallback;

import cn.wbnull.springbootconsumer.feign.GatewayFeignClient;
import org.springframework.stereotype.Component;

import java.util.Map;

@Component
public class GatewayFallback implements GatewayFeignClient {

    @Override
    public Map<String, String> users(Map<String, String> request) throws Exception {
        request.put("fallback", "spring-boot-consumer-feign-hystrix");

        return request;
    }
}
~~~

## 3.7 新建控制器类

~~~java
package cn.wbnull.springbootconsumer.controller;

import cn.wbnull.springbootconsumer.feign.GatewayFeignClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Scope;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@Scope("prototype")
public class GatewayController {

    @Autowired
    private GatewayFeignClient gatewayFeignClient;

    @PostMapping(value = "/users")
    public Map<String, String> users(@RequestBody Map<String, String> request) throws Exception {
        return gatewayFeignClient.users(request);
    }
}
~~~

## 3.8 测试

依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer-feign-hystrix。然后使用Postman测试，可以看到返回信息正常，且两组返回信息交替出现。

然后我们关闭spring-boot-provider，spring-boot-provider-v2服务，再在Postman中测试，可以看到返回正常，返回信息如下，正是我们在GatewayFallback类users()方法中的处理。

![1556287343104](./assets/05_Spring%20Cloud%20熔断器断路器%20Hystrix.assets/1556287343104.png)

## 3.9 修改Hystrix默认超时时间

### 3.9.1 修改控制器类

修改GatewayController类中users()方法，增加上休眠2秒。

~~~java
    @PostMapping(value = "/users")
    public Map<String, String> users(@RequestBody Map<String, String> request) throws Exception {
        Thread.sleep(2000);

        return gatewayFeignClient.users(request);
    }
~~~

然后我们再依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer-feign-hystrix。然后使用Postman测试，返回熔断方法中参数。

### 3.9.2 修改Hystrix超时时间

Feign默认集成了Ribbon，Ribbon也有一个超时时间，超时时间为1秒，所以这里不仅要设置Hystrix的超时时间，还要设置Ribbon的超时时间。

~~~yml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000
~~~

然后我们重启spring-boot-consumer-feign-hystrix，再次使用Postman测试，返回信息正常。

# 4. 获取服务异常信息

到现在，我们已经使用Hystrix解决了服务提供者故障的问题，但是我们还是无法确定故障原因，所以我们需要捕捉提供者抛出的异常。恰巧，Hystrix支持该操作。

## 4.1 新建FallbackFactory

新建GatewayFallbackFactory类，实现FallbackFactory\<T>接口，并注入到IoC容器中。

~~~java
package cn.wbnull.springbootconsumer.fallback;

import cn.wbnull.springbootconsumer.feign.GatewayFeignClient;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;

@Component
public class GatewayFallbackFactory implements FallbackFactory<GatewayFeignClient> {

    @Override
    public GatewayFeignClient create(Throwable throwable) {
        return (request) -> {
            request.put("fallback", "spring-boot-consumer-feign-hystrix by GatewayFallbackFactory");
            request.put("throwable", throwable.toString());

            return request;
        };
    }
}
~~~

## 4.2 修改GatewayFeignClient

修改GatewayFeignClient类注解 **@FeignClient**

~~~java
@FeignClient(value = "spring-boot-provider", fallbackFactory = GatewayFallbackFactory.class)
~~~

## 4.3 测试

依次启动spring-cloud-eureka，spring-boot-consumer-feign-hystrix。然后使用Postman测试，可以看到返回信息如下。

~~~json
{
  "name": "熔断测试name",
  "fallback": "spring-boot-consumer-feign-hystrix by GatewayFallbackFactory",
  "throwable": "java.lang.RuntimeException: com.netflix.client.ClientException: Load balancer does not have available server for client: spring-boot-provider"
}
~~~

异常信息也捕捉到了，完美。



---

GitHub：[https://github.com/dkbnull/spring-cloud-demo](https://github.com/dkbnull/spring-cloud-demo)

Gitee：[https://gitee.com/dkbnull/spring-cloud-demo](https://gitee.com/dkbnull/spring-cloud-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/89578323](https://blog.csdn.net/dkbnull/article/details/89578323)

微信：[https://mp.weixin.qq.com/s/VvuyF5JtYmA0dDyGD4uNIw](https://mp.weixin.qq.com/s/VvuyF5JtYmA0dDyGD4uNIw)

微博：[https://weibo.com/ttarticle/p/show?id=2309404369957537327375](https://weibo.com/ttarticle/p/show?id=2309404369957537327375)

知乎：[https://zhuanlan.zhihu.com/p/65133651](https://zhuanlan.zhihu.com/p/65133651)

---

