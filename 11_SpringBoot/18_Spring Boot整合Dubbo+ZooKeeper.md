# 0 开发环境

- JDK：1.8
- Spring Boot：2.7.18
  - ZooKeeper：3.9.2
- Dubbo-Admin：0.6.0

# 1 安装ZooKeeper

## 1.1 下载压缩包

下载地址：[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)

选择一个版本下载即可

![image-20240322170228461](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322170228461.png)

然后

![image-20240322170239124](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322170239124.png)

## 1.2 下载完成后解压

## 1.3 新建data、log目录

在安装根目录下新建**data**文件夹和**log**文件夹

![image-20240322192343778](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322192343778.png)

## 1.4 修改zoo.cfg

进入**conf**目录，将**zoo_sample.cfg**文件，复制一份，重命名为**zoo.cfg**

![image-20240322172140627](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322172140627.png)

修改**zoo.cfg**配置文件，将 **dataDir=/tmp/zookeeper** 修改成**zookeeper安装目录所在的data**文件夹，再**添加一条数据日志的配置**（根据自己的安装路径修改）

![image-20240322192827866](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322192827866.png)

## 1.5 启动服务端

进入**bin**目录，双击**zkServer.cmd**，启动服务端程序

![image-20240322193059211](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322193059211.png)

控制台显示**binding to port 0.0.0.0/0.0.0.0:2181**，表示服务端启动成功

![image-20240322193438940](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322193438940.png)

## 1.6 启动客户端

双击**zkCli.cmd**，启动客户端程序

控制台出现**Welcome to ZooKeeper!**，表示客户端启动成功

![image-20240322193704085](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322193704085.png)

## 1.7 测试

在客户端页面，可以使用linux命令进行操作

~~~sh
#列出zookeeper根下保存的所有节点
ls /

#创建一个test节点，值为123
create -e /test 123

#获取test节点的值
get /test
~~~

![image-20240322175811172](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240322175811172.png)

# 2 安装Dubbo-Admin

Dubbo其实是一个jar，能够帮助Java程序连接到ZooKeeper，并利用ZooKeeper消费、提供服务。

为了让用户更好的管理监控众多的Dubbo服务，官方提供了一个可视化的监控程序Dubbo-Admin。**该监控程序不安装并不影响Dubbo的使用。**

## 2.1 下载压缩包

下载地址：[https://github.com/apache/dubbo-admin/releases](https://github.com/apache/dubbo-admin/releases)

![image-20240323002450144](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240323002450144.png)

## 2.2 下载完成后解压

## 2.3 修改application.properties

进入**dubbo-admin-server\src\main\resources**目录，修改**application.properties**，设置zookeeper地址**dubbo.registry.address**

这里默认就是**127.0.0.1:2181**，无需修改

~~~properties
#
# ...其他配置...
#

server.port=38080
dubbo.protocol.port=30880
dubbo.application.qos-port=32222

# centers in dubbo, if you want to add parameters, please add them to the url
admin.registry.address=zookeeper://127.0.0.1:2181
admin.config-center=zookeeper://127.0.0.1:2181
admin.metadata-report.address=zookeeper://127.0.0.1:2181

#
# ...其他配置...
#

admin.root.user.name=root
admin.root.user.password=root

#
# ...其他配置...
#

#dubbo config
dubbo.application.name=dubbo-admin
dubbo.registry.address=${admin.registry.address}

#
# ...其他配置...
#
~~~

## 2.4 打包dubbo-admin

根目录下执行Maven打包命令

~~~bat
mvn clean package -Dmaven.test.skip=true
~~~

第一次打包时间会有点久，等到出现**BUILD SUCCESS**，表示打包成功

![image-20240323011020688](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240323011020688.png)

## 2.5 启动dubbo-admin

进入**dubbo-admin-server\target**，启动**dubbo-admin-server-0.6.0.jar**服务

**注：启动dubbo-admin服务时，一定要确保ZooKeeper服务是启动成功的**

~~~bat
java -jar dubbo-admin-server-0.6.0.jar
~~~

![image-20240323011745572](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240323011745572.png)

## 2.6 测试

浏览器访问**127.0.0.1:38080**

![image-20240323012135572](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240323012135572.png)

录入用户名和密码，**application.properties**中默认配置的都是 **root**，登录成功后，进入主页

![image-20240323012338984](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240323012338984.png)

# 3 新建服务提供者

新建module **spring-boot-dubbo-provider**

## 3.1 导入依赖

~~~xml
        <!--    dubbo    -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>3.2.11</version>
        </dependency>

        <!--    zookeeper 客户端    -->
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>

        <!--    zookeeper 服务端    -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.9.2</version>
        </dependency>

        <!--    zookeeper 服务端相关依赖    -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>5.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>5.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-x-discovery</artifactId>
            <version>5.6.0</version>
        </dependency>
~~~

## 3.2 配置dubbo

~~~yml
#访问端口
server:
  port: 8090
#
dubbo:
  #服务应用名字
  application:
    name: dubbo-provider
  #注册中心地址
  registry:
    address: zookeeper://127.0.0.1:2181
  #需注册的服务
  scan:
    base-packages: cn.wbnull.springbootdemo.service
~~~

## 3.3 新建服务

~~~java
package cn.wbnull.springbootdemo.service;

public interface UserInfoService {

    public String getUserInfo();
}
~~~

服务实现类

~~~java
package cn.wbnull.springbootdemo.service;

import org.apache.dubbo.config.annotation.DubboService;
import org.springframework.stereotype.Service;

@DubboService
@Service
public class UserInfoServiceImpl implements UserInfoService {

    @Override
    public String getUserInfo() {
        return "name:zhangsan;age:18";
    }
}
~~~

使用**@DubboService**后，可以被扫描到，在项目已启动就自动注册到注册中心

**注意：@Service在dubbo包下也有一个，不要用错了**

## 3.4 测试

启动服务，服务启动成功后，可以在Dubbo-Admin中看到注册的服务

![image-20240326092427279](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240326092427279.png)

查看详情，可以看到注册的服务的接口等信息

![image-20240326092606441](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240326092606441.png)

# 4 新建服务消费者

新建module **spring-boot-dubbo-consumer**

## 4.1 导入依赖

**与3.1中服务提供者spring-boot-dubbo-provider导入依赖包相同**

## 4.2 配置dubbo

~~~yml
#访问端口
server:
  port: 8091
#
dubbo:
  #消费者名字
  application:
    name: dubbo-consumer
  #注册中心地址
  registry:
    address: zookeeper://127.0.0.1:2181
~~~

## 4.3 新建服务

~~~java
package cn.wbnull.springbootdemo.service;

import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    /**
     * 使用服务提供者提供的服务，需要先从注册中心中获取服务
     */
    @DubboReference
    public UserInfoService userInfoService;

    public String getUserInfo() {
        return userInfoService.getUserInfo();
    }
}
~~~

要使用服务提供者提供的服务，需要先从注册中心中获取服务，获取服务使用**@DubboReference**注解。

这里**@DubboReference**注解跟**@Autowired**注解用法是类似的，只是@Autowired注解是自动装配本地的服务，@DubboReference是远程引用服务提供者的服务。

在使用@DubboReference注解远程引用时，会发现本模块中并没有UserInfoService服务，无法使用，有两种方法解决

* 1、使用pom坐标
* 2、定义路径相同的接口名

以方法2为例，新建UserInfoService

~~~java
package cn.wbnull.springbootdemo.service;

public interface UserInfoService {

    public String getUserInfo();
}
~~~

## 4.4 测试

新建测试类

~~~java
@SpringBootTest(classes = DubboConsumerApplication.class, webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class DubboConsumerApplicationTest {

    @Autowired
    public UserService userService;

    @Test
    public void contextLoads() {
        String userInfo = userService.getUserInfo();
        System.out.println("用户信息：" + userInfo);
    }
}
~~~

启动测试类，控制台上成功打印

![image-20240326100409989](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240326100409989.png)

至此，服务消费者成功从注册中心中获取到了服务提供者提供的服务，并成功调用其接口。

## 4.5 服务关系

我们再直接启动**spring-boot-dubbo-consumer**服务，服务启动成功后，可以在Dubbo-Admin中看到消费者和提供者的服务调用关系

![image-20240326101502825](./assets/18_Spring%20Boot%E6%95%B4%E5%90%88Dubbo+ZooKeeper.assets/image-20240326101502825.png)

Spring Boot整合Dubbo+Zookeeper成功，且测试通过



---

GitHub：[https://github.com/dkbnull/spring-boot-demo](https://github.com/dkbnull/spring-boot-demo)

Gitee：[https://gitee.com/dkbnull/spring-boot-demo](https://gitee.com/dkbnull/spring-boot-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/137616028](https://blog.csdn.net/dkbnull/article/details/137616028)

微信：[https://mp.weixin.qq.com/s/yB5BZT8MZNGek589JL5jiw](https://mp.weixin.qq.com/s/yB5BZT8MZNGek589JL5jiw)

知乎：[https://zhuanlan.zhihu.com/p/691814446](https://zhuanlan.zhihu.com/p/691814446)

---

