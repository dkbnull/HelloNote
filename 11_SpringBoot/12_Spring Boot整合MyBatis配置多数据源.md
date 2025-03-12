多数据源即动态数据源，随着项目开发逐渐扩大，单个数据源、单一数据源已经无法满足需求项目的支撑需求。

或是单一数据库无法承载大数据量的访问，需使用多个数据库进行数据的读写分离；

或是某些特殊业务需求，需操作不同的数据库。

在 [Spring Boot整合MyBatis连接数据库](https://blog.csdn.net/dkbnull/article/details/87278817) 文章中，展示了Spring Boot整合MyBatis连接数据库的方法，基于此，Spring Boot 整合MyBatis 配置多数据源。



# 0 开发环境

* JDK：1.8
* Spring Boot：2.1.1.RELEASE
* MySQL：5.7.13

# 1 引入依赖

~~~xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
    <scope>runtime</scope>
</dependency>

<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
~~~

# 2 引入数据源

~~~yml
server:
  port: 8090
spring:
  datasource:
    master:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/test_master?characterEncoding=utf8&serverTimezone=GMT%2B8
      username: root
      password: root
      driver-class-name: com.mysql.cj.jdbc.Driver
    slave:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/test_slave?characterEncoding=utf8&serverTimezone=GMT%2B8
      username: root
      password: root
      driver-class-name: com.mysql.cj.jdbc.Driver
#
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: cn.wbnull.springbootdemo.entity
~~~

该配置方式下，需要操作的两个数据库的Mapper需放置在不同文件夹下，如下图所示：

![1709455284298](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709455284298.png)

# 3 配置master库的源连接

~~~java
@Configuration
@MapperScan(basePackages = "cn.wbnull.springbootdemo.mapper.master", sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterDataSourceConfig {

    @Primary
    @Bean("masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean("masterDataSourceTransactionManager")
    public DataSourceTransactionManager masterDataSourceTransactionManager(@Qualifier("masterDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean("masterSqlSessionFactory")
    public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        Resource[] resources = new PathMatchingResourcePatternResolver().getResources("classpath:mapper/master/*.xml");
        sqlSessionFactory.setMapperLocations(resources);

        return sqlSessionFactory.getObject();
    }
}
~~~

# 4 配置slave库的源连接

~~~java
@Configuration
@MapperScan(basePackages = "cn.wbnull.springbootdemo.mapper.slave", sqlSessionFactoryRef = "slaveSqlSessionFactory")
public class SlaveDataSourceConfig {

    @Bean("slaveDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean("slaveDataSourceTransactionManager")
    public DataSourceTransactionManager slaveDataSourceTransactionManager(@Qualifier("slaveDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean("slaveSqlSessionFactory")
    public SqlSessionFactory slaveSqlSessionFactory(@Qualifier("slaveDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(dataSource);
        Resource[] resources = new PathMatchingResourcePatternResolver().getResources("classpath:mapper/slave/*.xml");
        sqlSessionFactory.setMapperLocations(resources);

        return sqlSessionFactory.getObject();
    }
}
~~~

# 5 测试

## 5.1 新建数据库表

~~~sql
CREATE SCHEMA `test_master` DEFAULT CHARACTER SET utf8mb4 ;

CREATE TABLE `test_master`.`user` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`));

INSERT INTO `test_master`.`user` (`name`) VALUES ('张三');
INSERT INTO `test_master`.`user` (`name`) VALUES ('李四');
INSERT INTO `test_master`.`user` (`name`) VALUES ('王五');
INSERT INTO `test_master`.`user` (`name`) VALUES ('周六');


CREATE SCHEMA `test_slave` DEFAULT CHARACTER SET utf8mb4 ;

CREATE TABLE `test_slave`.`user_info` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `userCode` VARCHAR(20) NOT NULL,
  `userName` VARCHAR(45) NULL,
  `password` VARCHAR(40) NOT NULL,
  PRIMARY KEY (`id`));
  
INSERT INTO `test_slave`.`user_info` (`id`, `userCode`, `userName`, `password`) VALUES ('1', 'zhangsan', '张三三', 'zhangsan');
INSERT INTO `test_slave`.`user_info` (`id`, `userCode`, `userName`, `password`) VALUES ('2', 'lisi', '李四四', 'lisi');
INSERT INTO `test_slave`.`user_info` (`id`, `userCode`, `userName`, `password`) VALUES ('3', 'wangwu', '王五五', 'wangwu');
INSERT INTO `test_slave`.`user_info` (`id`, `userCode`, `userName`, `password`) VALUES ('4', 'zhouliu', '周六六', 'zhouliu');
~~~

## 5.2 新建实体类

~~~java
@Data
public class User {

    private int id;
    private String name;
}
~~~

~~~java
@Data
public class UserInfo {

    private Integer id;
    private String userCode;
    private String userName;
    private String password;
}
~~~

## 5.3 新建Mapper

~~~java
@Repository
public interface UserMapper {

    void add(@Param("user") User user);

    List<User> query();

    void update(@Param("id") int id, @Param("name") String name);

    void delete(@Param("id") int id);
}
~~~

~~~java
@Repository
public interface UserInfoMapper {

    List<User> query();
}
~~~

## 5.4 新建映射文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.wbnull.springbootdemo.mapper.master.UserMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="cn.wbnull.springbootdemo.entity.User">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, name
    </sql>

    <insert id="add">
        INSERT INTO user(<include refid="Base_Column_List"/>)
        VALUES
        (
        #{user.id},
        #{user.name}
        )
    </insert>

    <select id="query" resultMap="BaseResultMap">
        SELECT * FROM user
    </select>

    <update id="update">
        UPDATE user SET name = '${name}' WHERE id = '${id}'
    </update>

    <update id="delete">
        DELETE FROM user where id = '${id}'
    </update>
</mapper>
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.wbnull.springbootdemo.mapper.slave.UserInfoMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="cn.wbnull.springbootdemo.entity.UserInfo">
        <id column="id" property="id" />
        <result column="userCode" property="userCode" />
        <result column="userName" property="userName" />
        <result column="password" property="password" />
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, userCode, userName, password
    </sql>

    <select id="query" resultMap="BaseResultMap">
        SELECT * FROM user_info
    </select>
</mapper>
~~~

## 5.5 新建Service

~~~java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public String add(String name) {
        User user = new User();
        user.setName(name);

        userMapper.add(user);

        return "操作成功";
    }

    public List<User> query() {
        return userMapper.query();
    }

    public String update(int id, String name) {
        userMapper.update(id, name);

        return "操作成功";
    }

    public String delete(int id) {
        userMapper.delete(id);

        return "操作成功";
    }
}
~~~

~~~java
@Service
public class UserInfoService {

    @Autowired
    private UserInfoMapper userInfoMapper;

    public List<User> query() {
        return userInfoMapper.query();
    }
}
~~~

## 5.6 新建Controller

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    public UserService userService;

    @PostMapping(value = "add")
    public String add(@RequestParam(value = "name") String name) {
        return userService.add(name);
    }

    @PostMapping(value = "query")
    public List<User> query() {
        return userService.query();
    }

    @PostMapping(value = "update")
    public String update(@RequestParam(value = "id") int id, @RequestParam(value = "name") String name) {
        return userService.update(id, name);
    }

    @PostMapping(value = "delete")
    public String delete(@RequestParam(value = "id") int id) {
        return userService.delete(id);
    }
}
~~~

~~~java
@Controller
@RequestMapping("userInfo")
public class UserInfoController {

    @Autowired
    public UserInfoService userInfoService;

    @PostMapping(value = "query")
    public List<User> query() {
        return userInfoService.query();
    }
}
~~~

## 5.7 测试

使用Postman测试，输出结果如下

### 5.8.1 master select

![1709454296366](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454296366.png)

### 5.8.2 master insert

![1709454652439](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454652439.png)

数据库中插入成功

![1709454673891](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454673891.png)

### 5.8.3 master update

![1709454839225](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454839225.png)

数据库中更新成功

![1709454855988](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454855988.png)

### 5.8.4 master delete

![1709454900245](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454900245.png)

数据库中删除成功

![1709454914507](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709454914507.png)

### 5.8.5 slave select

![1709455099275](./assets/12_Spring%20Boot整合MyBatis配置多数据源.assets/1709455099275.png)

截至这里，Spring Boot已经成功整合MyBatis多数据源，并连接上了数据库，且测试正常。



---

GitHub：[https://github.com/dkbnull/spring-boot-demo](https://github.com/dkbnull/spring-boot-demo)

Gitee：[https://gitee.com/dkbnull/spring-boot-demo](https://gitee.com/dkbnull/spring-boot-demo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/136433910](https://blog.csdn.net/dkbnull/article/details/136433910)

微信：[https://mp.weixin.qq.com/s/6q3xLMg-QFo5_6RsD1h4YA](https://mp.weixin.qq.com/s/6q3xLMg-QFo5_6RsD1h4YA)

知乎：[https://zhuanlan.zhihu.com/p/685038746](https://zhuanlan.zhihu.com/p/685038746)

---

