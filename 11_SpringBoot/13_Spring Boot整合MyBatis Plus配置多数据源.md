在 [Spring Boot整合MyBatis配置多数据源](https://blog.csdn.net/dkbnull/article/details/136433910) 文章中，展示了Spring Boot整合Mybatis配置多数据源的方法。那么，如果使用MyBatis Plus，如何配置多数据源呢？

官方文档：[https://baomidou.com/pages/a61e1b/](https://baomidou.com/pages/a61e1b/)

MyBatis Plus连接数据库参考：[Spring Boot整合MyBatis Plus连接数据库](https://blog.csdn.net/dkbnull/article/details/136331111)



# 0 开发环境

* JDK：1.8
* Spring Boot：2.1.1.RELEASE
* MySQL：5.7.13

# 1 引入依赖

~~~xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.4</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.5.2</version>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
~~~

# 2 配置数据源

~~~yml
server:
  port: 8090
spring:
  datasource:
    dynamic:
      primary: master       #设置默认的数据源或者数据源组，默认值即为master
      strict: false         #严格匹配数据源，默认false。true未匹配到指定数据源时抛异常，false使用默认数据源
      datasource:
        master:
          url: jdbc:mysql://127.0.0.1:3306/test_master?characterEncoding=utf8&serverTimezone=GMT%2B8
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver     #3.2.0开始支持SPI可省略此配置
        slave:
          url: jdbc:mysql://127.0.0.1:3306/test_slave?characterEncoding=utf8&serverTimezone=GMT%2B8
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver
#
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.wbnull.springbootdemo.entity
~~~

# 3 使用@DS切换数据源

**@DS** 可以注解在方法上或类上，**同时存在就近原则 方法上注解 优先于 类上注解**。

| 注解          | 结果                                     |
| ------------- | ---------------------------------------- |
| 没有@DS       | 默认数据源                               |
| @DS("dsName") | dsName可以为组名也可以为具体某个库的名称 |

~~~java
@DS("slave")
@Service
public class UserInfoService {

    @Autowired
    private UserInfoMapper userInfoMapper;

    public List<UserInfo> query() {
        LambdaQueryWrapper<UserInfo> queryWrapper = new LambdaQueryWrapper<>();

        return userInfoMapper.selectList(queryWrapper);
    }
}
~~~

# 4 测试

直接使用 [Spring Boot整合MyBatis配置多数据源](https://blog.csdn.net/dkbnull/article/details/136433910) 中创建的数据库表

## 4.1 新建实体类

~~~java
@Data
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private int id;
    private String name;
}
~~~

~~~java
@Data
public class UserInfo implements Serializable {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    @TableField("userCode")
    private String userCode;

    @TableField("userName")
    private String userName;

    private String password;
}
~~~

注：这里因为数据库里字段是驼峰式命名，所以使用 **@TableField** 指定对应的数据库字段名，也可以通过yml配置关闭属性映射

~~~yml
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.wbnull.springbootdemo.entity
  configuration:
    map-underscore-to-camel-case: false
~~~

## 4.2 新建Mapper

~~~java
@Repository
public interface UserMapper extends BaseMapper<User> {

}
~~~

~~~java
@Repository
public interface UserInfoMapper extends BaseMapper<UserInfo> {

}
~~~

## 4.3 新建映射文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.wbnull.springbootdemo.mapper.UserMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="cn.wbnull.springbootdemo.entity.User">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, name
    </sql>
</mapper>
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.wbnull.springbootdemo.mapper.UserInfoMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="cn.wbnull.springbootdemo.entity.UserInfo">
        <id column="id" property="id"/>
        <result column="userCode" property="userCode"/>
        <result column="userName" property="userName"/>
        <result column="password" property="password"/>
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, userCode, userName, password
    </sql>

</mapper>
~~~

## 4.4 新建Service

~~~java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public List<User> query() {
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();

        return userMapper.selectList(queryWrapper);
    }
}
~~~

~~~java
/**
 * 3使用@DS切换数据源 中已展示过该类
 */
@DS("slave")
@Service
public class UserInfoService {

    @Autowired
    private UserInfoMapper userInfoMapper;

    public List<UserInfo> query() {
        LambdaQueryWrapper<UserInfo> queryWrapper = new LambdaQueryWrapper<>();

        return userInfoMapper.selectList(queryWrapper);
    }
}
~~~

## 4.5 新建Controller

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    public UserService userService;

    @PostMapping(value = "query")
    public List<User> query() {
        return userService.query();
    }
}
~~~

~~~java
@RestController
@RequestMapping("userInfo")
public class UserInfoController {

    @Autowired
    public UserInfoService userInfoService;

    @PostMapping(value = "query")
    public List<UserInfo> query() {
        return userInfoService.query();
    }
}
~~~

## 4.6 测试

使用Postman测试，输出结果如下

### 4.6.1 master select

![image-20240304151714947](./assets/13_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E9%85%8D%E7%BD%AE%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90.assets/image-20240304151714947.png)

### 4.6.2 slave select

![image-20240304155206664](./assets/13_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E9%85%8D%E7%BD%AE%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90.assets/image-20240304155206664.png)

---

GitHub：[https://github.com/dkbnull/spring-boot-demo](https://github.com/dkbnull/spring-boot-demo)

Gitee：[https://gitee.com/dkbnull/spring-boot-demo](https://gitee.com/dkbnull/spring-boot-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/136611367](https://blog.csdn.net/dkbnull/article/details/136611367)

微信：[https://mp.weixin.qq.com/s/e6g0K6n9Uu8GdSOqu6iNwg](https://mp.weixin.qq.com/s/e6g0K6n9Uu8GdSOqu6iNwg)

知乎：[https://zhuanlan.zhihu.com/p/686275755](https://zhuanlan.zhihu.com/p/686275755)

---

