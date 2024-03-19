在上一篇文章 [Spring Cloud 服务注册与发现 Eureka](https://blog.csdn.net/dkbnull/article/details/89268194) 中，我们使用Eureka的负载均衡策略解决了服务消费者在调用服务提供者接口时把提供者的地址硬编码在消费者代码里的问题，同时实现了最简单的负载均衡，接口会返回**hello world,this is spring-boot-provider**和**hello world,this is spring-boot-provider-v2**。但是我们使用的是Eureka默认的负载均衡策略。

在微服务架构中，业务会被拆分成一个个彼此独立的服务，服务间的通讯基于http restful。

Spring Cloud有两种服务间调用的方式，**Ribbon**和**Feign**。下面我们简单介绍下Ribbon。

# 0. 开发环境

- IDE：IntelliJ IDEA 2017.1 x64

- jdk：1.8.0_91

- Spring Boot：2.0.9.RELEASE

- Spring Cloud：Finchley.RELEASE


# 1. Ribbon简介

Spring Cloud Ribbon 是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，但它不像服务注册中心、配置中心、API网关那样需要独立部署。它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。微服务间的调用、API网关的请求转发等内容，实际上都是通过Ribbon来实现的。包括Spring Cloud的另一种服务调用方式Feign，也是基于Ribbon实现的。

# 2. Ribbon负载均衡策略

Ribbon的核心组件是IRule，IRule是所有负载均衡策略的父接口，其子类有： 

![1556118245979](03_Spring%20Cloud%20自定义Eureka%20Ribbon负载均衡策略.assets/1556118245979.png)

每一个子类就是一种负载均衡策略

* **RandomRule**：随机选取负载均衡策略，随机Random对象，在所有服务实例中随机找一个服务的索引号，然后从上线的服务中获取对应的服务。
* **RoundRobinRule**：线性轮询负载均衡策略。
* **WeightedResponseTimeRule**：响应时间作为选取权重的负载均衡策略，根据平均响应时间计算所有服务的权重，响应时间越短的服务权重越大，被选中的概率越高。刚启动时，如果统计信息不足，则使用线性轮询策略，等信息足够时，再切换到WeightedResponseTimeRule。
* **RetryRule**：使用线性轮询策略获取服务，如果获取失败则在指定时间内重试，重新获取可用服务。
* **ClientConfigEnabledRoundRobinRule**：默认通过线性轮询策略选取服务。通过继承该类，并且对choose方法进行重写，可以实现更多的策略，继承后保底使用RoundRobinRule策略。
* **BestAvailableRule**：继承自ClientConfigEnabledRoundRobinRule。从所有没有断开的服务中，选取到目前为止请求数量最小的服务。
* **PredicateBasedRule**：抽象类，提供一个choose方法的模板，通过调用AbstractServerPredicate实现类的过滤方法来过滤出目标的服务，再通过轮询方法选出一个服务。
* **AvailabilityFilteringRule**：按可用性进行过滤服务的负载均衡策略，会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数超过阈值的服务，然后对剩余的服务列表进行线性轮询。
* **ZoneAvoidanceRule**：本身没有重写choose方法，用的还是抽象父类PredicateBasedRule的choose。

# 2. 自定义负载均衡策略

上篇文章中我们使用Ribbon实现负载均衡，使用的是Ribbon默认的负载均衡策略，接下来我们自定义负载均衡策略。

## 2.1 代码自定义策略

代码自定义负载均衡策略需要注意，要避免Spring Boot的包扫描，自定义的规则必须在Eureka的规则实例化以后再实例化才会生效。

### 2.1.1 新建负载均衡策略配置类

我们直接在上篇文章的**spring-boot-consumer**服务上进行修改

在启动类**SpringBootConsumerApplication**的上层新建**config**包，然后新建**LoadBalanced**类。使用LoadBalanced类注册一个新的IRule来替换Eureka。

```java
package cn.wbnull.config;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.WeightedResponseTimeRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class LoadBalanced {

    @Bean
    public IRule iRule() {
        return new WeightedResponseTimeRule();
        //return new RandomRule();
    }
}
```

这里我们想使用哪种策略就可以new哪种策略。

### 2.1.2 新建自定义负载均衡策略类

**cn.wbnull.config**包下新建**loadbalancer**包，再新建**GlobalRule**类，

~~~java
package cn.wbnull.config.loadbalancer;

import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.Server;

import java.util.List;

public class GlobalRule implements IRule {

    private ILoadBalancer iLoadBalancer;

    @Override
    public Server choose(Object o) {
        List<Server> servers = iLoadBalancer.getAllServers();
        return servers.get(0);
    }

    @Override
    public void setLoadBalancer(ILoadBalancer iLoadBalancer) {
        this.iLoadBalancer = iLoadBalancer;
    }

    @Override
    public ILoadBalancer getLoadBalancer() {
        return this.iLoadBalancer;
    }
}
~~~

我们这里自定义的负载均衡策略是始终取服务列表的第一个，其他自定义策略参考这种格式修改即可。

然后我们**LoadBalanced**类可以new一个**GlobalRule**

~~~java
@Configuration
public class LoadBalanced {

    @Bean
    public IRule iRule() {
        return new GlobalRule();
    }
}
~~~

### 2.1.3 指定策略规则

**SpringBootConsumerApplication**类上新增注解**@RibbonClient(name = "spring-boot-provider", configuration = cn.wbnull.config.LoadBalanced.class)**，表示spring-boot-provider服务使用cn.wbnull.config.LoadBalanced提供的负载均衡策略。

~~~java
package cn.wbnull.springbootconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "spring-boot-provider", configuration = cn.wbnull.config.LoadBalanced.class)
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

### 2.1.4 测试

我们依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer。然后浏览器访问http://127.0.0.1:8082/springbootconsumer/gateway，我们发现，无论怎么刷新，返回信息都是同一个。

![1556121903438](03_Spring%20Cloud%20自定义Eureka%20Ribbon负载均衡策略.assets/1556121903438.png)

我们在GlobalRule中打上断点，然后再刷新浏览器http://127.0.0.1:8082/springbootconsumer/gateway访问页面，可以看到每次刷新都能进到断点这里。

![1556122979054](03_Spring%20Cloud%20自定义Eureka%20Ribbon负载均衡策略.assets/1556122979054.png)

查看debug数据，servers.get(0)返回的是8081服务，我们浏览器接收到的返回参数也是8081服务的返回参数。

![1556123114678](03_Spring%20Cloud%20自定义Eureka%20Ribbon负载均衡策略.assets/1556123114678.png)

### 2.1.5 ComponentScan注解自定义扫描类

上面我们为了避免Spring Boot的包扫描，将新建的负载均衡策略相关类都放在了启动类SpringBootConsumerApplication的上级。我们也可以仍旧将负载均衡策略相关类放到启动类SpringBootConsumerApplication同级或下级包内，然后使用**@ComponentScan**注解排除不需扫描的类。

* 1、在cn.wbnull.springbootconsumer包下新建config包和config.loadbalancer；

* 2、把2.1.1和2.1.2中新建的LoadBalanced类和GlobalRule类复制到cn.wbnull.springbootconsumer对应包下

* 3、注释掉2.1.1和2.1.2中新建的LoadBalanced类和GlobalRule类

* 4、启动类SpringBootConsumerApplication增加如下注解

```java
@RibbonClient(name = "spring-boot-provider", configuration = cn.wbnull.springbootconsumer.config.LoadBalanced.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = cn.wbnull.springbootconsumer.config.LoadBalanced.class)})
```

增加注解后SpringBootConsumerApplication：

~~~java
package cn.wbnull.springbootconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "spring-boot-provider", configuration = cn.wbnull.springbootconsumer.config.LoadBalanced.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = cn.wbnull.springbootconsumer.config.LoadBalanced.class)})
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

* 5、依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer。然后浏览器访问http://127.0.0.1:8082/springbootconsumer/gateway，效果跟刚才相同。

## 2.2 配置文件自定义策略

上面我们使用代码自定义策略，使用LoadBalanced类配置负载均衡，这种方式虽然能够实现自定义负载均衡策略，但当我们有大量配置类时，对每个Ribbon客户端的配置信息将会分散在这些配置类中，这样管理起来极其麻烦。在Camden版本之后，Spring Cloud Ribbon对Ribbon客户端的个性化配置进行了优化，可以通过<font color="#FF0000">**服务id名称.ribbon.参数=值**</font>的形式进行配置，如：

~~~yml
spring-boot-provider:
  ribbon:
    NFLoadBalancerRuleClassName: cn.wbnull.springbootconsumer.config.loadbalancer.GlobalRule
~~~

Ribbon配置的优先级：属性配置 > Java配置 > Netflix Ribbon默认配置

### 2.2.1 修改启动类

SpringBootConsumerApplication启动类注释掉如下代码

```
@RibbonClient(name = "spring-boot-provider", configuration = cn.wbnull.springbootconsumer.config.LoadBalanced.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = cn.wbnull.springbootconsumer.config.LoadBalanced.class)})
```

注释后SpringBootConsumerApplication：

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

### 2.2.2 修改application.yml

application.yml增加如下配置

~~~yml
spring-boot-provider:
  ribbon:
    NFLoadBalancerRuleClassName: cn.wbnull.springbootconsumer.config.loadbalancer.GlobalRule
~~~

### 2.2.3 测试

依次启动spring-cloud-eureka，spring-boot-provider，spring-boot-provider-v2，spring-boot-consumer。然后浏览器访问http://127.0.0.1:8082/springbootconsumer/gateway，效果跟刚才相同。

### 2.2.4 注意

当我们使用配置文件自定义负载均衡策略时，就不再需要2.1.1中新建的负载均衡策略配置类LoadBalanced，只需要2.1.2中新建的负载均衡策略类GlobalRule。



---

GitHub：[https://github.com/dkbnull/SpringCloudDemo](https://github.com/dkbnull/SpringCloudDemo)

Gitee：[https://gitee.com/dkbnull/SpringCloudDemo](https://gitee.com/dkbnull/SpringCloudDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/89506462](https://blog.csdn.net/dkbnull/article/details/89506462)

微信：[https://mp.weixin.qq.com/s/D3wHuZiuhqbqXI5MudwxMg](https://mp.weixin.qq.com/s/D3wHuZiuhqbqXI5MudwxMg)

微博：[https://weibo.com/ttarticle/p/show?id=2309404368874400313637](https://weibo.com/ttarticle/p/show?id=2309404368874400313637)

知乎：[https://zhuanlan.zhihu.com/p/64760029](https://zhuanlan.zhihu.com/p/64760029)

---

