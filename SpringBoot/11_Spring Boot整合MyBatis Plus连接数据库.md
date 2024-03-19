在 [Spring Boot整合MyBatis连接数据库](https://blog.csdn.net/dkbnull/article/details/87278817) 这篇文章中，我们已经可以使用Spring Boot整合MyBatis来连接数据库，但随着使用，我们发现，MyBatis还是稍微有点复杂，那有没有更加简单的方式来操作数据库呢，我们惊奇的发现了MyBatis Plus。



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

<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
~~~

# 2 引入数据源

application.yml 增加如下配置信息

~~~yml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
#
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml                #对应mapper映射xml文件所在路径
  type-aliases-package: cn.wbnull.springbootdemo.entity   #对应实体类路径
~~~

# 3 测试

## 3.1 新建数据库表

~~~sql
CREATE SCHEMA `test` DEFAULT CHARACTER SET utf8mb4 ;

CREATE TABLE `test`.`user` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`));

INSERT INTO `test`.`user` (`name`) VALUES ('张三');
INSERT INTO `test`.`user` (`name`) VALUES ('李四');
INSERT INTO `test`.`user` (`name`) VALUES ('王五');
INSERT INTO `test`.`user` (`name`) VALUES ('周六');
~~~

## 3.2 创建实体类

~~~java
package cn.wbnull.springbootdemo.entity;

import lombok.Data;

@Data
public class User {

    private int id;
    private String name;
}
~~~

## 3.3 创建Mapper

~~~java
package cn.wbnull.springbootdemo.mapper;

import cn.wbnull.springbootdemo.entity.User;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.springframework.stereotype.Repository;

@Repository
public interface UserMapper extends BaseMapper<User> {

}
~~~

## 3.4 创建映射文件

在 **resources** 目录下新建 **mapper** 文件夹，用于存放MyBatis Plus映射文件

```xml
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
```

## 3.5 创建Service

~~~java
package cn.wbnull.springbootdemo.service;

import cn.wbnull.springbootdemo.entity.User;
import cn.wbnull.springbootdemo.mapper.UserMapper;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public String add(String name) {
        User user = new User();
        user.setName(name);

        userMapper.insert(user);

        return "操作成功";
    }

    public List<User> query() {
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();

        return userMapper.selectList(queryWrapper);
    }

    public String update(int id, String name) {
        LambdaUpdateWrapper<User> updateWrapper = new LambdaUpdateWrapper<>();
        updateWrapper.set(User::getName, name);
        updateWrapper.eq(User::getId, id);

        userMapper.update(updateWrapper);

        return "操作成功";
    }

    public String delete(int id) {
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(User::getId, id);

        userMapper.delete(queryWrapper);

        return "操作成功";
    }
}
~~~

## 3.6 创建Controller

~~~java
package cn.wbnull.springbootdemo.controller;

import cn.wbnull.springbootdemo.entity.User;
import cn.wbnull.springbootdemo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

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

## 3.7 项目启动类

增加@MapperScan，扫描mapper

~~~java
package cn.wbnull.springbootdemo;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("cn.wbnull.springbootdemo.mapper")
public class MybatisPlusApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusApplication.class, args);
    }
}
~~~

## 3.8 测试

使用Postman进行测试，输出结果如下

### 3.8.1 select

![image-20240219172312713](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172312713.png)

### 3.8.2 insert

![image-20240219172351958](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172351958.png)

我们查下数据库，并再用postman请求

![image-20240219172555534](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172555534.png)



![image-20240219172621594](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172621594.png)

### 3.8.3 update

![image-20240219172742641](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172742641.png)

数据库中成功更新

![image-20240219172759750](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172759750.png)

### 3.8.4 delete

![image-20240219172841970](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172841970.png)

数据库中成功删除

![image-20240219172849909](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240219172849909.png)

截至这里，Spring Boot已经成功整合MyBatis Plus并连接上了数据库，且测试正常。

对比发现，我们使用LambdaQueryWrapper来操作数据库会特别方便。



并且，在我们实际开发中，如果存在大量数据库表，我们依旧可以使用Generator来自动生成代码

# 4 条件构造器 QueryWrapper

上面 3.5 中，我们创建的Service类中，使用了QueryWrapper简化SQL，其基本用法可参考官方文档：[https://baomidou.com/pages/10c804/](https://baomidou.com/pages/10c804/)

以下简单整理可供参考

| 函数名              | 说明                                                         | 例子                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **allEq**           | 全部eq(或个别isNull)                                         | allEq({id:1,name:"老王",age:null})`--->`id = 1 and name = '老王' and age is null<br />allEq({id:1,name:"老王",age:null}, false)`--->`id = 1 and name = '老王' |
| **eq**              | 等于 =                                                       | eq("name", "老王")`--->`name = '老王'                        |
| **ne**              | 不等于 <>                                                    | ne("name", "老王")`--->`name <> '老王'                       |
| **gt**              | 大于 >                                                       | gt("age", 18)`--->`age > 18                                  |
| **ge**              | 大于等于 >=                                                  | ge("age", 18)`--->`age >= 18                                 |
| **lt**              | 小于 <                                                       | lt("age", 18)`--->`age < 18                                  |
| **le**              | 小于等于 <=                                                  | le("age", 18)`--->`age <= 18                                 |
| **between**         | BETWEEN 值1 AND 值2                                          | between("age", 18, 30)`--->`age between 18 and 30            |
| **notBetween**      | NOT BETWEEN 值1 AND 值2                                      | notBetween("age", 18, 30)`--->`age not between 18 and 30     |
| **like**            | LIKE '%值%'                                                  | like("name", "王")`--->`name like '%王%'                     |
| **notLike**         | NOT LIKE '%值%'                                              | notLike("name", "王")`--->`name not like '%王%'              |
| **likeLeft**        | LIKE '%值'                                                   | likeLeft("name", "王")`--->`name like '%王'                  |
| **likeRight**       | LIKE '值%'                                                   | likeRight("name", "王")`--->`name like '王%'                 |
| **notLikeLeft**     | NOT LIKE '%值'                                               | notLikeLeft("name", "王")`--->`name not like '%王'           |
| **notLikeRight**    | NOT LIKE '值%'                                               | notLikeRight("name", "王")`--->`name not like '王%'          |
| **isNull**          | 字段 IS NULL                                                 | isNull("name")`--->`name is null                             |
| **isNotNull**       | 字段 IS NOT NULL                                             | isNotNull("name")`--->`name is not null                      |
| **in**              | 字段 IN (value.get(0), value.get(1), ...)<br />字段 IN (v0, v1, ...) | in("age",{1,2,3})`--->`age in (1,2,3)<br />in("age", 1, 2, 3)`--->`age in (1,2,3) |
| **notIn**           | 字段 NOT IN (value.get(0), value.get(1), ...)<br />NOT IN (v0, v1, ...) | notIn("age",{1,2,3})`--->`age not in (1,2,3)<br />notIn("age", 1, 2, 3)`--->`age not in (1,2,3) |
| **inSql**           | 字段 IN ( sql语句 )                                          | inSql("age", "1,2,3,4,5,6")`--->`age in (1,2,3,4,5,6)<br />inSql("id", "select id from table where id < 3")`--->`id in (select id from table where id < 3) |
| **notInSql**        | 字段 NOT IN ( sql语句 )                                      | notInSql("age", "1,2,3,4,5,6")`--->`age not in (1,2,3,4,5,6)<br />notInSql("id", "select id from table where id < 3")`--->`id not in (select id from table where id < 3) |
| **groupBy**         | 分组：GROUP BY 字段, ...                                     | groupBy("id", "name")`--->`group by id,name                  |
| **orderByAsc**      | 排序：ORDER BY 字段, ... ASC                                 | orderByAsc("id", "name")`--->`order by id ASC,name ASC       |
| **orderByDesc**     | 排序：ORDER BY 字段, ... DESC                                | orderByDesc("id", "name")`--->`order by id DESC,name DESC    |
| **orderBy**         | 排序：ORDER BY 字段, ...                                     | orderBy(true, true, "id", "name")`--->`order by id ASC,name ASC |
| **having**          | HAVING ( sql语句 )                                           | having("sum(age) > 10")`--->`having sum(age) > 10<br />having("sum(age) > {0}", 11)`--->`having sum(age) > 11 |
| **func**            | func 方法(主要方便在出现if...else下调用不同方法能不断链)     | func(i -> if(true) {i.eq("id", 1)} else {i.ne("id", 1)})     |
| **or**              | 拼接 OR                                                      | eq("id",1).or().eq("name","老王")`--->`id = 1 or name = '老王'<br />or(i -> i.eq("name", "李白").ne("status", "活着"))`--->`or (name = '李白' and status <> '活着') |
| **and**             | AND 嵌套                                                     | and(i -> i.eq("name", "李白").ne("status", "活着"))`--->`and (name = '李白' and status <> '活着') |
| **nested**          | 正常嵌套 不带 AND 或者 OR                                    | nested(i -> i.eq("name", "李白").ne("status", "活着"))`--->`(name = '李白' and status <> '活着') |
| **apply**           | 拼接 sql                                                     | apply("id = 1")`--->`id = 1<br />apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")<br /> apply("date_format(dateColumn,'%Y-%m-%d') = {0}", "2008-08-08")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'") |
| **last**            | 无视优化规则直接拼接到 sql 的最后                            | last("limit 1")                                              |
| **exists**          | 拼接 EXISTS ( sql语句 )                                      | exists("select id from table where age = 1")`--->`exists (select id from table where age = 1) |
| **notExists**       | 拼接 NOT EXISTS ( sql语句 )                                  | notExists("select id from table where age = 1")`--->`not exists (select id from table where age = 1) |
|                     |                                                              |                                                              |
| ***QueryWrapper***  |                                                              |                                                              |
| **select**          | 设置查询字段                                                 | select("id", "name", "age")<br />select(i -> i.getProperty().startsWith("test")) |
|                     |                                                              |                                                              |
| ***UpdateWrapper*** |                                                              |                                                              |
| **set**             | SQL SET 字段                                                 | set("name", "老李头")<br />set("name", "")`--->`数据库字段值变为空字符串<br />set("name", null)`--->`数据库字段值变为null |
| **setSql**          | 设置 SET 部分 SQL                                            | setSql("name = '老李头'")                                    |

# 5 代码生成器

## 5.1 引入依赖

~~~xml
<!-- 代码生成器 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.4</version>
</dependency>

<!-- 模板引擎 -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
~~~

## 5.2 新建代码生成器类

代码生成器所有配置可参考官方文档：[https://baomidou.com/pages/981406/](https://baomidou.com/pages/981406/)

~~~java
package cn.wbnull.springbootdemo.mybatis;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.rules.DateType;

import java.util.Collections;
import java.util.Scanner;

public class MybatisPlusGenerator {

    private static final String URL = "jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&serverTimezone=GMT%2B8";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "root";

    private static final String PACKAGE_PATH = System.getProperty("user.dir") + "/spring-boot-mybatis-plus/src/main/java";
    private static final String RESOURCES_MAPPER_PATH = System.getProperty("user.dir") + "/spring-boot-mybatis-plus/src/main/resources/mapper/";

    public static void main(String[] args) {
        DataSourceConfig dataSourceConfig = new DataSourceConfig.Builder(URL, USERNAME, PASSWORD)
                .build();

        AutoGenerator autoGenerator = new AutoGenerator(dataSourceConfig);
        autoGenerator.global(globalConfig());
        autoGenerator.packageInfo(packageConfig());
        autoGenerator.strategy(strategyConfig());

        autoGenerator.execute();
    }

    private static GlobalConfig globalConfig() {
        return new GlobalConfig.Builder()
                .outputDir(PACKAGE_PATH)
                .author("null")
                .dateType(DateType.TIME_PACK)
                .commentDate("yyyy-MM-dd")
                .disableOpenDir()
                .build();
    }

    private static PackageConfig packageConfig() {
        return new PackageConfig.Builder()
                .parent("cn.wbnull.springbootdemo")
                .pathInfo(Collections.singletonMap(OutputFile.xml, RESOURCES_MAPPER_PATH))
                .build();
    }

    private static StrategyConfig strategyConfig() {
        return new StrategyConfig.Builder()
                .addInclude(scanner().split(","))
                .mapperBuilder()
                .enableBaseResultMap()
                .enableBaseColumnList()
                .entityBuilder()
                .enableLombok()
                .enableTableFieldAnnotation()
                .build();
    }

    public static String scanner() {
        Scanner scanner = new Scanner(System.in);
        String hint = "请输入数据库表名，多个表名使用英文逗号分隔：";
        System.out.println(hint);
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (ipt != null && ipt.length() > 0) {
                return ipt;
            }
        }

        throw new MybatisPlusException("请输入正确的数据库表名");
    }
}
~~~

## 5.3 测试

### 5.3.1 新建数据库表

我们先新建一个数据库表，便于一会测试自动生成代码

~~~~sql
CREATE TABLE `test`.`user_info` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `userCode` VARCHAR(20) NOT NULL,
  `userName` VARCHAR(45) NULL,
  `password` VARCHAR(40) NOT NULL,
  PRIMARY KEY (`id`));
~~~~

### 5.3.2 测试

运行MybatisPlusGenerator，输入需要生成的表名

![image-20240220100754038](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240220100754038.png)

文件生成完成

![image-20240220100811704](11_Spring%20Boot%E6%95%B4%E5%90%88MyBatis%20Plus%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93.assets/image-20240220100811704.png)

生成文件如下

![1708964390170](11_Spring%20Boot整合MyBatis Plus连接数据库.assets/1708964390170.png)

这样，对于大量的数据库表，我们就可以使用Generator来生成基本的代码，然后自己再添加其他所需要的代码即可。



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/136331111](https://blog.csdn.net/dkbnull/article/details/136331111)

微信：[https://mp.weixin.qq.com/s/ZJTKX_gmn6ffsY7hNrspHQ](https://mp.weixin.qq.com/s/ZJTKX_gmn6ffsY7hNrspHQ)

知乎：[https://zhuanlan.zhihu.com/p/684251625](https://zhuanlan.zhihu.com/p/684251625)

---