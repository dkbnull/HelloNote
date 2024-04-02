# 0 开发环境

- JDK：1.8
- Spring Boot：2.7.18

# 1 检查配置

确保**File -> Settings... -> Editor -> File Encodings** 配置中编码为**UTF-8**，否则后续会出现乱码

![image-20240327161749978](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327161749978.png)

# 2 安装插件

安装**Resource Bundle Editor**插件后，Resource Bundle就可以可视化编程了

![image-20240327164847205](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327164847205.png)

# 3 新建配置文件

## 3.1 resources下新建i18n目录

其来源是英文单词 internationalization的首末字符i和n，18为中间的字符数，是“国际化”的简称

## 3.2 i18n下新建配置文件

新建**login.properties**国际化默认配置文件，再新建**login_zh_CN.properties**中文配置文件，此时，系统会自动识别到国际化配置，将文件合并目录并切换到国际化视图。

![image-20240327165221039](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327165221039.png)

我们就可以直接在**Resource Bundle ‘login’**文件夹上右键添加其他语言配置文件

![image-20240327165423318](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327165423318.png)

比如添加英语配置文件

![image-20240327165716358](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327165716358.png)

## 3.3 编写配置文件

打开其中一个配置文件，切换到**Resource Bundle**视图，点击**+**，即可添加配置

![image-20240327170956141](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240327170956141.png)

比如添加**login.username**配置，添加完成后，页面上会出现3个录入框，可配置不同语言的值

![image-20240328093332429](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328093332429.png)

配置完成后，所有配置文件都会自动配置上对应值，比如

![image-20240328193350176](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328193350176.png)

完整配置如下

~~~properties
login.btn=登录
login.error=用户名或密码错误
login.password=密码
login.username=用户名
~~~

# 4 配置yml

~~~yml
spring:
  messages:
    basename: i18n.login
~~~

# 5 页面国际化

## 5.1 引入依赖

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
~~~

## 5.2 新建登录页

**resources**目录下新建**templates**目录，再新建**login.html**

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title></title>

    <style>
        .container {
            width: 300px;
            margin: auto;
            padding: 40px;
            border: 1px solid #ccc;
            background-color: white;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        label {
            display: block;
            margin-bottom: 10px;
        }

        input {
            width: 100%;
            padding: 6px;
            border: 1px solid #ccc;
            outline: none;
        }

        button {
            width: 100%;
            padding: 10px;
            color: white;
            background-color: #4CAF50;
            cursor: pointer;
            border: none;
            outline: none;
        }

        .language {
            width: 100%;
            text-align: center;
            margin-top: 10px;
        }

        .language a {
            color: blue;
            cursor: pointer;
            padding: 10px;
        }

        button:hover,
        a:hover {
            opacity: 0.9;
        }
    </style>
</head>
<body>
<div class="container">
    <form action="" method="post">
        <label for="username" th:text="#{login.username}"></label>
        <input type="text" id="username" name="username"><br><br>

        <label for="password" th:text="#{login.password}"></label>
        <input type="password" id="password" name="password"><br><br>

        <button type="submit" th:text="#{login.btn}"></button>
    </form>
</div>
<div class="language">
    <a th:href="@{/login.html(l=zh_CN)}">中文</a>
    <a th:href="@{/login.html(l=en_US)}">English</a>
</div>
</body>
</html>
~~~

## 5.3 新建Controller

~~~java
@Controller
public class LoginController {

    @GetMapping(value = "login.html")
    public String login() {
        return "login";
    }
}
~~~

## 5.4 新建配置类

页面能够实现国际化，是通过**Locale**区域信息对象来实现的。

**WebMvcAutoConfiguration**中配置了默认的**localeResolver**，我们自定义一个**LocaleResolver**并注入容器，来替换默认的localeResolver。

注意：注入的bean名称必须为**localeResolver**，因为**@ConditionalOnMissingBean**会判断是否已经注入了bean

![image-20240328230733436](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328230733436.png)

~~~java
@Configuration
public class GlobalLocaleResolver implements LocaleResolver {

    /**
     * 解析请求
     *
     * @param request
     * @return
     */
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        //默认地区
        Locale locale = request.getLocale();

        //获取请求中的语言参数
        String language = request.getParameter("l");
        if (!StringUtils.isEmpty(language)) {
            //zh_CN 国家_地区
            String[] values = language.split("_");
            locale = new Locale(values[0], values[1]);
        }

        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }

    @Bean
    public LocaleResolver localeResolver() {
        return new GlobalLocaleResolver();
    }
}
~~~

## 5.5 测试

启动服务，浏览器访问127.0.0.1:8090/login.html

![image-20240328233943370](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328233943370.png)

切换英语：

![image-20240328234028927](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234028927.png)

切换中文：

![image-20240328234047223](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234047223.png)

页面可正常切换中英文，测试通过。

# 7 返回信息国际化

## 7.1 新建工具类

~~~java
public class I18nUtils {

    public static String getMessage(String key) {
        Locale locale = LocaleContextHolder.getLocale();
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("i18n/login");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource.getMessage(key, null, locale);
    }
}
~~~

## 7.2 新建Service

~~~java
@Service
public class UserService {

    public String login() {
        return I18nUtils.getMessage("login.error");
    }
}
~~~

## 7.3 新建Controller

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping(value = "login")
    public String login() {
        return userService.login();
    }
}
~~~

## 7.4 测试

启动服务，Postman请求，默认中文如下：

![image-20240329000551838](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234231750.png)

> 附：HTTP请求头Accept-Language含义
>
> **Accept-Language:zh-CN,zh;q=0.9**
>
> Accept-Language：浏览器支持的语言类型
>
> zh-CN：简体中文
>
> zh：中国地区
>
> q：用户对该范围指定的语言的偏好，为空则默认为1，范围[0, 1]，值越大，权重越大

英语如下：

![image-20240328234431750](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234431750.png)

根据传入的地区信息，返回对应语言的提示信息，测试通过。

# 8 返回信息拼接参数

上面我们正常返回了中英文的用户名或密码错误，现在升级一下，返回类似 xxx用户名或密码错误

## 8.1 调整配置文件

![image-20240328234758259](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234758259.png)

## 8.2 调整工具类

~~~java
public class I18nUtils {

    public static String getMessage(String key) {
        Locale locale = LocaleContextHolder.getLocale();
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("i18n/login");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource.getMessage(key, null, locale);
    }

    public static String getMessage(String key, Object... args) {
        Locale locale = LocaleContextHolder.getLocale();
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("i18n/login");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource.getMessage(key, args, locale);
    }
}
~~~

## 8.3 调整Service

~~~java
@Service
public class UserService {

    public String login() {
        return I18nUtils.getMessage("login.error");
    }

    public String loginSuccess(String username) {
        return I18nUtils.getMessage("login.success", username, new Date());
    }
}
~~~

## 8.4 调整Controller

~~~java
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping(value = "login")
    public String login() {
        return userService.login();
    }

    @PostMapping(value = "loginSuccess")
    public String loginSuccess(@RequestParam("username") String username) {
        return userService.loginSuccess(username);
    }
}
~~~

## 8.5 测试

启动服务，Postman请求，中文如下：

![image-20240328234655019](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328234855019.png)

英语如下：

![image-20240328235138540](16_Spring%20Boot%E9%A1%B5%E9%9D%A2%E5%9B%BD%E9%99%85%E5%8C%96.assets/image-20240328235138540.png)

根据传入的地区信息，返回对应语言的提示信息，且正确组装传入的参数，测试通过。

至此，国际化完成，且测试通过。



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/137202903](https://blog.csdn.net/dkbnull/article/details/137202903)

微信：[https://mp.weixin.qq.com/s/UA5nVO8BQxeWUjGfW1wocg](https://mp.weixin.qq.com/s/UA5nVO8BQxeWUjGfW1wocg)

知乎：[https://zhuanlan.zhihu.com/p/690021887](https://zhuanlan.zhihu.com/p/690021887)

---