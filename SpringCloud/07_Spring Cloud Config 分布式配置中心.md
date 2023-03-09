在Spring Boot服务中，我们可以通过设置多个profile来配置不同的`application-{profile}.yml`，然后在启动服务时指定要加载的配置文件`spring.profiles.active={profile}`。

但是在Spring Cloud微服务架构中，这种配置方式不再适用。Spring Cloud对于配置有着更高、更灵活的要求，需要做到统一管理，实时更新，这时我们使用分布式配置中心组件来管理配置，也就是Spring Cloud Config。

# 0. 开发环境

- IDE：IntelliJ IDEA 2017.1 x64

- JDK：1.8.0_91

- Spring Boot：2.0.9.RELEASE

- Spring Cloud：Finchley.RELEASE

# 1. Spring Cloud Config简介

Spring Cloud Config是一个解决分布式系统的配置管理方案，为分布式系统外部化配置提供了支持，包含Config Server和Config Client两部分，Server提供配置文件存储，对外提供接口以获取配置文件的内容，Client通过接口获取数据，并初始化自己。

# 2. 创建Config Server

## 2.1 新建Git仓库

新建Git仓库，用于存放配置文件。

这里我们直接在[SpringCloudDemo](https://github.com/dkbnull/SpringCloudDemo)仓库中新建**spring-cloud-config-repo**文件夹，然后新建三个配置文件

spring-cloud-config-client-pro.yml

~~~yml
version: 1.0
profile: pro
~~~

spring-cloud-config-client-dev.yml

~~~yml
version: 1.0
profile: dev
~~~

spring-cloud-config-client-test.yml

~~~yml
version: 1.0
profile: test
~~~

## 2.2 新建Config Server

新建Config Server服务**spring-cloud-config-server**

## 2.3 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
~~~

## 2.4 新建启动类

**@EnableConfigServer** 注解表示开启配置服务器

~~~java
package cn.wbnull.springcloudconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigServerApplication.class, args);
    }
}
~~~

## 2.5 新建application.yml

~~~yml
server:
  port: 8092
  servlet:
    context-path: /springcloudconfig

spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/dkbnull/SpringCloudDemo
          search-paths: spring-cloud-config-repo
          username: ""
          password: ""
      label: master
~~~

* spring.cloud.config.server.git.uri	配置Git仓库的地址
* spring.cloud.config.server.git.searchPaths    配置文件的路径
* spring.cloud.config.label    配置仓库的分支
* spring.cloud.config.server.git.username    访问Git仓库的用户名，公开仓库可不填写
* spring.cloud.config.server.git.password    访问Git仓库的密码，公开仓库可不填写

## 2.6 测试

启动spring-cloud-config-server服务，浏览器访问 http://127.0.0.1:8092/springcloudconfig/spring-cloud-config-client/pro ，返回如下

~~~json
{
	"name": "spring-cloud-config-client",
	"profiles": [
		"pro"
	],
	"label": null,
	"version": "8bddb99fecf031d284f1d7503b70c212f665e86f",
	"state": null,
	"propertySources": [
		{
			"name": "https://github.com/dkbnull/SpringCloudDemo/spring-cloud-config-repo/spring-cloud-config-client-pro.yml",
			"source": {
				"version": 1.0,
				"profile": "pro"
			}
		}
	]
}
~~~

浏览器访问 http://127.0.0.1:8092/springcloudconfig/spring-cloud-config-client/dev ，返回如下

~~~json
{
	"name": "spring-cloud-config-client",
	"profiles": [
		"dev"
	],
	"label": null,
	"version": "2fa4983d777889ef29e453a7b4818645269124b1",
	"state": null,
	"propertySources": [
		{
			"name": "https://github.com/dkbnull/SpringCloudDemo/spring-cloud-config-repo/spring-cloud-config-client-dev.yml",
			"source": {
				"version": 1.0,
				"profile": "dev"
			}
		}
	]
}
~~~

从Git仓库获取配置成功。

## 2.7 配置文件映射关系

Http请求地址与配置文件映射关系如下：

```
映射{application}-{profile}.yml或{application}-{profile}.properties配置文件

/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.yml
/{label}/{application}-{profile}.properties
```

* {application}通常使用微服务的名称，对应于Git仓库中配置文件文件名的前缀
* {profile}对应于{application}-{profile}的{profile}
* {label}对应于Git仓库的分支名，默认为master

# 3. 创建Config Client

## 3.1 新建Config Client

新建Config Client服务**spring-cloud-config-client**

## 3.2 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
~~~

## 3.3 新建启动类

~~~java
package cn.wbnull.springcloudconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringCloudConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigClientApplication.class, args);
    }
}
~~~

## 3.4 新建application.yml

~~~yml
server:
  port: 8093
  servlet:
    context-path: /springcloudconfig
~~~

## 3.5 新建bootstrap.yml

**`Spring Boot服务启动时会加载application.yml/application.properties配置文件，Spring Cloud中有”引导上下文“的概念，引导上下文会加载bootstrap.yml/bootstrap.properties配置文件，即bootstrap.yml/bootstrap.properties会在Spring Boot服务启动之前加载，具有更高的优先级。默认情况下bootstrap.yml/bootstrap.properties中的属性不能被覆盖。`**

~~~yml
spring:
  application:
    name: spring-cloud-config-client
  cloud:
    config:
      uri: http://127.0.0.1:8092/springcloudconfig
      profile: pro
      label: master
~~~

* spring.application.name	配置Config Server获取的配置文件的{application}
* spring.cloud.config.uri    配置Config Server的地址
* spring.cloud.config.profile   配置Config Server获取的配置文件的{profile}

* spring.cloud.config.label    配置Config Server获取的配置文件的{label}

## 3.6 新建控制器类

~~~java
package cn.wbnull.springcloudconfig.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Scope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Scope("prototype")
public class GatewayController {

    @Value("${version}")
    private String version;

    @Value("${profile}")
    private String profile;

    @GetMapping(value = "/gateway")
    public String gateway() throws Exception {
        return "version:" + version + ",profile:" + profile;
    }
}
~~~

## 3.7 测试

依次启动spring-cloud-config-server、spring-cloud-config-client，浏览器访问 http://127.0.0.1:8093/springcloudconfig/gateway ，返回如下

~~~
version:1.0,profile:pro
~~~

spring-cloud-config-client成功通过spring-cloud-config-server从Git仓库中获取到了指定配置文件的配置信息。



---

GitHub：[https://github.com/dkbnull/SpringCloudDemo](https://github.com/dkbnull/SpringCloudDemo)

Gitee：[https://gitee.com/dkbnull/SpringCloudDemo](https://gitee.com/dkbnull/SpringCloudDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/89934484](https://blog.csdn.net/dkbnull/article/details/89934484)

微信：[https://mp.weixin.qq.com/s/jcFZUliJ5Fq5MSQWQNombw](https://mp.weixin.qq.com/s/jcFZUliJ5Fq5MSQWQNombw)

微博：[https://weibo.com/ttarticle/p/show?id=2309404371771666085647](https://weibo.com/ttarticle/p/show?id=2309404371771666085647)

知乎：[https://zhuanlan.zhihu.com/p/65654453](https://zhuanlan.zhihu.com/p/65654453)

---

