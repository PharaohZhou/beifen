---
title: 初识Spring(二)
date: 2019-08-07 22:25:00
author: 周李
top: true
cover: true
toc: true
mathjax: false
categories: Spring
tags:
  - java
  - Spring
---

# Spring Beans 自动装配

Spring容器可以在不使用 <constructor-arg>和<property> 元素的情况下自动装配相互协作的bean之间的关系， 这有助于减少编写一个大的基于Spring的应用程序的xml配置的数量。



#### 自动装配模式：

	使用<bean>元素的autowire属性为一个bean定义指定自动装配模式。

| 模式        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| no          | 默认，它意味着没有自动装配                                   |
| byName      | 由属性名自动装配                                             |
| byType      | 由属性数据类型自动装配                                       |
| constructor | 类似于byType，但该类型适用于构造函数参数类型                 |
| autodetect  | Spring先通过constructor自动装配来连接，如果不执行，Spring尝试通过byType来自动装配 |

# Spring基于注解的配置

	从Spring2.5开始就可以使用注解来配置依赖注入。可以使用相关类，方法或字段声明的注解，将bean配置移动到组件类本身。

```java
<context:annotation-config/>
```

#### Spring @Reqired 注释（已过时）

	应用于bean属性的setter方法，它表明受影响的bean属性在配置时必须放在xml配置文件中，否则抛出BeanInitializationException异常。

#### Spring @Autowired 注释

**@Autowired**注释对在哪里和如何完成自动连接提供了更多的细微的控制。

**@Autowried**注解可以用在属性、构造函数、set方法中。在需要注入的类中添加@Autowried注解，在使用的时候，spring会自动加载类实例。

```java
 @Autowired(required = false)
    public void setAge(Integer age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }
    @Autowired
    public void setName(String name) {
        this.name = name;
    }
}
```

使用required = false 就算没值也不会报错

#### Spring @Resource注释

	可以在字段中或者setter方法中使用@Resource注释，它和在JavaEE5中的运作是一样的。@Resource注释使用一个‘name’属性， 该属性以一个bean名称形式被注入，它遵循byName自动连接语义。

```java
@Resource(name="...")
public void setXxx(Xxx xxx) {
}
```

如果没有明确指定一个‘name’，默认名称源于字段名或者setter方法。

# Spring基于Java的配置

#### **@Configuration 和 @Bean 注解**

带有@Configuration的注解表示这个类可以使用Spring IoC容器作为bean定义的来源。@Bean注解告诉Spring，一个带有@Bean的注解方法将返回一个对象，该对象应该被注册在Spring应用程序上下文的bean。

```java
@Configuration
public class HelloWorldConfig {
   @Bean 
   public HelloWorld helloWorld(){
      return new HelloWorld();
   }
}
```

带有@Bean注解的方法名称作为baen的ID，他创建并返回实际的bean。你的配置类可以申明多个@Bean。一旦定义了配置类，你就可以使用AnnotationConfigApplicationContext来加载并提供给Spring容器：

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
```

这样就不用配置文件啦。

 **import注解**

	允许从另一个配置类中加载@Bean定义。

```java
@Configuration
public class ConfigA{
    @Bean
    public A a() {
        return new A();
    }
}
```

```java
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B a() {
      return new A(); 
   }
}
```

这样实例化上下文时，就不要同时指定ConfigA和ConfigB了，只需提供ConfigB即可。

```java
public static void main(String[] args) {
   ApplicationContext ctx = 
   new AnnotationConfigApplicationContext(ConfigB.class);
	// ......
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
```



# Spring 中的事件处理

Spring 提供了以下的标准事件：

| 序号 |                    Spring 内置事件 & 描述                    |
| ---- | :----------------------------------------------------------: |
| 1    | **ContextRefreshedEvent**             ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生。 |
| 2    | **ContextStartedEvent**                 当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。 |
| 3    | **ContextStoppedEvent **             当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。 |
| 4    | **ContextClosedEvent**                当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。 |
| 5    | **RequestHandledEvent**            这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。 |



# Spring 框架的 AOP

Spring 框架的一个关键组件是**面向方向的编程**（AOP）框架。面向方向的编程需要吧程序逻辑分解成不同的部分称为所谓的关注点。跨一个应用程序的多个点的功能被称为**横切关注点**，这些横切关注点在概念上独立于应用程序的业务逻辑。有各种各样的常见的很好的方面的例子，如日志记录、审计、声明式事务、安全性和缓存等。

在OOP中，关键单元模块是类，而在AOP中单元模块是方面。依赖注入帮助你对应用程序对象相互解耦和AOP可以帮助你从它们所影响的对象中对横切关注点解耦。AOP是像编程语言的触发物。

Spring AOP 模块提供拦截器来拦截一个应用程序，例如，当执行一个方法时，你可以在方法执行前和执行后添加额外的功能。

**AOP术语**

| 项            |                             描述                             |
| ------------- | :----------------------------------------------------------: |
| Aspect        | 一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。 |
| Join point    | 在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。 |
| Advice        | 这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。 |
| Pointcut      | 这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。 |
| Introduction  |           引用允许你添加新方法或属性到现有的类中。           |
| Target object | 被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。 |
| Weaving       | Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时，类加载时和运行时完成。 |



#### 通过切点来选择连接点

切点用于准确定位应该再什么地方应用切面的通知。通知和切面是切面的最基本元素。

在Spring AOP中，要使用AspectJ的切点表达式语言来定义切点。

关于Spring AOP的AspectJ切点，最重要的一点就是Spring仅支持ASpectJ切点指示器（pointcut designator）的一个子集。

	表： Spring借助AspectJ的切点表达式语言来定义Spring切面

| AspectJ指示器 | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| arg()         | 限制连接点匹配参数为指定类型的执行方法                       |
| @arg()        | 限制连接点匹配参数由指定注解标注的执行方法                   |
| execution()   | 用于匹配是连接点的执行方法                                   |
| this()        | 限制连接点匹配AOP代理的bean引用为指定类型的类                |
| target        | 限制连接点匹配目标对象为指定类型的类                         |
| @target()     | 限制连接点匹配特定的执行对象，这些对象对应的类要具有指定类型的注解 |
| within()      | 限制连接点匹配指定的类型                                     |
| @winthin()    | 限制链接点匹配指定注解所标注的类型                           |
| @annotation   | 限定匹配带有指定注解的连接点                                 |

#### 编写切点

为了阐述Spring中的切面，我们需要有个主题来定义切面的切点。为此先定义一个Performance接口：

```java
package concert;
public interface Performance {
    public void perform();
}
```

#### 定义切面

```java
package concert;

import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class Audience {
    @Before("execution(**concert.Performance.perform(..))")
    public void silenceCellPhones() {
        System.out.println("Sliencing cell phones");
    }

    @Before("execution(**concert.Performance.perform(..))")
    public void takeSeats() {
        System.out.println("Taking seats");
    }

    @AfterReturning("execution(**concert.Performance.perform(..))")
    public void applause() {
        System.out.println("CLAP CLAP CLAP");
    }

    @AfterThrowing("execution(**concert.Performance.perform(..))")
    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}

```