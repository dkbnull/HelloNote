众所周知，Spring拥有两大特性：**IoC和AOP**。

**IoC，英文全称Inversion of Control，意为控制反转。AOP，英文全称Aspect-Oriented Programming，意为面向切面编程**。

Spring核心容器的主要组件是Bean工厂（BeanFactory），Bean工厂使用控制反转（IoC）模式来降低程序代码之间的耦合度，并提供了面向切面编程（AOP）的实现。

简单来说，<font color='red'>Spring是一个轻量级的控制反转（IoC）和面向切面编程（AOP）的容器框架。</font>>

下面，我们简要说明下这两大特性。

# Spring常用注解

在具体介绍IoC和AOP之前，我们先简要说明下Spring常用注解

1、**@Controller**：用于标注控制器层组件

2、**@Service**：用于标注业务层组件

3、**@Component** : 用于标注这是一个受 Spring 管理的组件，组件引用名称是类名，第一个字母小写。可以使用@Component("beanID") 指定组件的名称

4、**@Repository**：用于标注数据访问组件，即DAO组件

5、**@Bean**：方法级别的注解，主要用在@Configuration和@Component注解的类里，@Bean注解的方法会产生一个Bean对象，该对象由Spring管理并放到IoC容器中。引用名称是方法名，也可以用@Bean(name = "beanID")指定组件名

6、**@Scope("prototype")**：将组件的范围设置为原型的（即多例）。保证每一个请求有一个单独的action来处理，避免action的线程问题。

由于Spring默认是单例的，只会创建一个action对象，每次访问都是同一个对象，容易产生并发问题，数据不安全。

7、**@Autowired**：默认按类型进行自动装配。在容器查找匹配的Bean，当有且仅有一个匹配的Bean时，Spring将其注入@Autowired标注的变量中。

8、**@Resource**：默认按名称进行自动装配，当找不到与名称匹配的Bean时会按类型装配。



简单点说，就是，<font color='red'>能够明确该类是一个控制器类组件的，就用@Controller；能够明确是一个服务类组件的，就用@Service；能够明确该类是一个数据访问组件的，就用@Repository；不知道他是啥或者不好区分他是啥，但是就是想让他动态装配的就用@Component。</font>

@Controller、@Service、@Component、@Repository都是类级别的注解，<font color='red'>如果一个方法也想动态装配，就用@Bean。</font>

<font color='red'>当我们想按类型进行自动装配时，就用@Autowired；当我们想按名称（beanID）进行自动装配时，就用@Resource；当我们需要根据比如配置信息等来动态装配不同的组件时，可以用getBean("beanID")。</font>

到这里，如果对这些注解，或是自动装配不太理解，可以继续往下，看完 控制反转(IoC) 内容后再回来理解这里的内容。
# 控制反转(IoC)

<font color='red'>控制反转，简单点说，就是创建对象的控制权，被反转到了Spring框架上。</font>

通常，我们实例化一个对象时，都是使用类的构造方法来new一个对象，这个过程是由我们自己来控制的，而控制反转就把new对象的工交给了Spring容器。

《expert ONE-ON-ONE J2EE Development without EJB》第6章中指出

~~~
P128
IoC Implementation Strategies
IoC is a broad concept that can be implemented in different ways. There are two main types:
Dependency Lookup: The container provides callbacks to components, and a lookup context.This is the EJB and Apache Avalon approach. It leaves the onus on each component to use container APIs to look up resources and collaborators. The Inversion of Control is limited to the container invoking callback methods that application code can use to obtain resources.
Dependency Injection: Components do no look up; they provide plain Java methods enabling the container to resolve dependencies. The container is wholly responsible for wiring up components, passing resolved objects in to JavaBean properties or constructors. Use of JavaBean properties is called Setter Injection; use of constructor arguments is called Constructor Injection.

P130
The second IoC strategy-Dependency Injection-is usually preferable.
~~~

主要意思为：

~~~
IoC的主要实现方式有两种：依赖查找、依赖注入。

依赖注入是一种更可取的方式。
~~~

那么依赖查找和依赖注入有什么区别呢？

依赖查找，主要是容器为组件提供一个回调接口和上下文环境。这样一来，组件就必须自己使用容器提供的API来查找资源和协作对象，控制反转仅体现在那些回调方法上，容器调用这些回调方法，从而应用代码获取到资源。

依赖注入，组件不做定位查询，只提供标准的Java方法让容器去决定依赖关系。容器全权负责组件的装配，把符合依赖关系的对象通过Java Bean属性或构造方法传递给需要的对象。

## IoC容器

IoC容器：具有依赖注入功能的容器，可以创建对象的容器。IoC容器负责实例化、定位、配置应用程序中的对象并建立这些对象之间的依赖。

## 依赖注入

**DI，英文全称，Dependency Injection，意为依赖注入。**

依赖注入：由IoC容器动态地将某个对象所需要的外部资源（包括对象、资源、常量数据）注入到组件(Controller, Service等）之中。<font color='red'>简单点说，就是IoC容器会把当前对象所需要的外部资源动态的注入给我们。</font>

Spring依赖注入的方式主要有四个，基于注解注入方式、set注入方式、构造器注入方式、静态工厂注入方式。推荐使用基于注解注入方式，配置较少，比较方便。

<font color='red'>**基于注解注入方式**</font>

服务层代码

~~~java
@Service
public class AdminService {
    //code
}
~~~

控制层代码

~~~java
@Controller
@Scope("prototype")
public class AdminController {
 
    @Autowired
    private AdminService adminService;
 
    //code
}
~~~

@Autowired与@Resource都可以用来装配Bean，都可以写在字段、setter方法上。他们的区别是：

<font color='red'>@Autowired默认按类型进行自动装配</font>（该注解属于Spring），默认情况下要求依赖对象必须存在，如果要允许为null，需设置required属性为false，例：@Autowired(required=false)。如果要使用名称进行装配，可以与@Qualifier注解一起使用。

~~~java
    @Autowired
    @Qualifier("adminService")
    private AdminService adminService;
~~~

<font color='red'>@Resource默认按照名称进行装配</font>（该注解属于J2EE），名称可以通过name属性来指定。如果没有指定name属性，当注解写在字段上时，默认取字段名进行装配；如果注解写在setter方法上，默认取属性名进行装配。当找不到与名称相匹配的Bean时，会按照类型进行装配。但是，name属性一旦指定，就只会按照名称进行装配。

~~~java
    @Resource(name = "adminService")
    private AdminService adminService;
~~~

除此之外，对于一些复杂的装载Bean的时机，比如我们需要根据配置装载不同的Bean，以完成不同的操作，可以使用getBean("beanID")的方式来加载Bean。

通过BeanID加载Bean方法如下：

~~~java
@Component
public class BeanUtils implements ApplicationContextAware {
 
    private static ApplicationContext applicationContext;
 
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        if (BeanUtils.applicationContext == null) {
            BeanUtils.applicationContext = applicationContext;
        }
    }
 
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
 
    public static Object getBean(String id) throws Exception {
        try {
            return applicationContext.containsBean(id) ? applicationContext.getBean(id) : null;
        } catch (BeansException e) {
            e.printStackTrace();
            throw new Exception("not found bean id: " + id);
        }
    }
}
~~~

我们在需要加载Bean的地方调用该方法即可。

~~~java
public class BaseController {
 
    protected IService loadService(String id) throws Exception {
        IService iService = (IService) BeanUtils.getBean(id);
        if (iService != null) {
            return iService;
        } else {
            throw new Exception("加载Bean错误");
        }
    }
}
~~~

# 面向切面编程(AOP)

面向切面编程（AOP）就是纵向的编程。比如业务A和业务B现在需要一个相同的操作，传统方法我们可能需要在A、B中都加入相关操作代码，而应用AOP就可以只写一遍代码，A、B共用这段代码。并且，当A、B需要增加新的操作时，可以在不改动原代码的情况下，灵活添加新的业务逻辑实现。

在实际开发中，比如商品查询、促销查询等业务，都需要记录日志、异常处理等操作，AOP把所有共用代码都剥离出来，单独放置到某个类中进行集中管理，在具体运行时，由容器进行动态织入这些公共代码。

AOP主要一般应用于<font color='red'>签名验签、参数校验、日志记录、事务控制、权限控制、性能统计、异常处理等地。</font>

## AOP涉及名词

**切面（Aspect）**：共有功能的实现。如日志切面、权限切面、验签切面等。<font color='red'>在实际开发中通常是一个存放共有功能实现的标准Java类。当Java类使用了\@Aspect注解修饰时，就能被AOP容器识别为切面。</font>

**通知（Advice）**：切面的具体实现。就是要给目标对象织入的事情。以目标方法为参照点，根据放置的地方不同，可分为前置通知（Before）、后置通知（AfterReturning）、异常通知（AfterThrowing）、最终通知（After）与环绕通知（Around）5种。<font color='red'>在实际开发中通常是切面类中的一个方法，具体属于哪类通知，通过方法上的注解区分。</font>

**连接点（JoinPoint）**：程序在运行过程中能够插入切面的地点。例如，方法调用、异常抛出等。Spring只支持方法级的连接点。<font color='red'>一个类的所有方法前、后、抛出异常时等都是连接点。</font>

**切入点（Pointcut）**：用于定义通知应该切入到哪些连接点上。不同的通知通常需要切入到不同的连接点上，这种精准的匹配是由切入点的正则表达式来定义的。

比如，在上面所说的连接点的基础上，来定义切入点。我们有一个类，类里有10个方法，那就产生了几十个连接点。但是我们并不想在所有方法上都织入通知，我们只想让其中的几个方法，在调用之前检验下入参是否合法，那么就用切点来定义这几个方法，让切点来筛选连接点，选中我们想要的方法。<font color='red'>切入点就是来定义哪些类里面的哪些方法会得到通知。</font>

**目标对象（Target）**：<font color='red'>那些即将切入切面的对象</font>，也就是那些被通知的对象。这些对象专注业务本身的逻辑，所有的共有功能等待AOP容器的切入。

**代理对象（Proxy）**：将通知应用到目标对象之后被动态创建的对象。可以简单地理解为，代理对象的功能等于目标对象本身业务逻辑加上共有功能。代理对象对于使用者而言是透明的，是程序运行过程中的产物。<font color='red'>目标对象被织入共有功能后产生的对象。</font>

**织入（Weaving）**：将切面应用到目标对象从而创建一个新的代理对象的过程。这个过程可以发生在编译时、类加载时、运行时。Spring是在运行时完成织入，运行时织入通过Java语言的反射机制与动态代理机制来动态实现。

## Pointcut用法

Pointcut格式为：

~~~java
execution(modifier-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
~~~

**修饰符匹配 modifier-pattern? **例：public private

**返回值匹配 ret-type-pattern** 可以用 \* 表示任意返回值

**类路径匹配 declaring-type-pattern?** 全路径的类名

**方法名匹配 name-pattern** 可以指定方法名或者用 \* 表示所有方法；set\* 表示所有以set开头的方法

**参数匹配 (param-pattern) **可以指定具体的参数类型，多个参数用","分隔；可以用 \* 表示匹配任意类型的参数；可以用 (..) 表示零个或多个任意参数

**异常类型匹配throws-pattern?** 例 throws Exception

<font color='red'>其中后面跟着 ? 表示可选项</font>

例

~~~java
@Pointcut("execution(public * cn.wbnull. springbootdemo.controller.*.*(..))")
private void sign() {
 
}
~~~

## 一个例子

以[Spring Boot使用AOP实现拦截器](https://blog.csdn.net/dkbnull/article/details/82847647)中的AOP为例

~~~java
@Aspect
@Component
public class SignAop {
 
}
~~~

SignAop类使用了\@Aspect注解，则该类可以被AOP容器识别为切面。

~~~java
@Aspect
@Component
public class SignAop {
 
    @Pointcut("execution(public * cn.wbnull.springbootdemo.controller.*.*(..))")
    private void signAop() {
 
    }
}
~~~

@Pointcut声明一个切入点，范围为controller包下所有的类的所有方法

注：作为切入点签名的方法必须返回void类型

~~~java
@Aspect
@Component
public class SignAop {
 
    @Pointcut("execution(public * cn.wbnull.springbootdemo.controller.*.*(..))")
    private void signAop() {
 
    }
 
    @Before("signAop()")
    public void doBefore(JoinPoint joinPoint) throws Exception {
        //code
       }
 
    @AfterReturning(value = "signAop()", returning = "params")
    public JSONObject doAfterReturning(JoinPoint joinPoint, JSONObject params) {
        //code
        }
}
~~~

doBefore()方法使用@Before("signAop()")注解，表示前置通知（在某连接点之前执行的通知），但这个通知不能阻止连接点之前的执行流程，除非它抛出一个异常。

doAfterReturning()方法使用@AfterReturning(value = "signAop()", returning = "params")注解，表示后置通知（在某连接点正常完成后执行的通知），通常在一个匹配的方法返回的时候执行。

实际运行时，在进入controller包下所有方法前，都会进入doBefore()方法，在controller包下方法执行完成后，都会进入doAfterReturning()方法。

AOP具体使用可以参考[Spring Boot使用AOP实现拦截器](https://blog.csdn.net/dkbnull/article/details/82847647)

到这里，Spring两大特性IoC和AOP基本讲述完成，后续若想起其他再进行补充。



---

CSDN：[https://blog.csdn.net/dkbnull/article/details/87219562](https://blog.csdn.net/dkbnull/article/details/87219562)

微信：[https://mp.weixin.qq.com/s/Si7kodmH-GnMh-D2nk_DVQ](https://mp.weixin.qq.com/s/Si7kodmH-GnMh-D2nk_DVQ)

---

