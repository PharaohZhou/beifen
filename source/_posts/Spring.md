---
title: 初识Spring(一)
date: 2019-08-07 09:25:00
author: 周李
top: false
cover: false
toc: true
mathjax: false
categories: Spring
tags:
  - java
  - Spring
---



# Spring Bean 作用域

* **singleton   单例模式：（默认值）在Spring IoC容器仅存在一个Bean实例**
* **prototype  原型模式： 每次从容器中调用Bean都会返回一个新实例。**
* request 请求模式：每次Http请求都会创建一个新的Bean，仅使用于WebApplicationContext环境
* Session 会话模式：同一个Http Session 共享一个Bean， 不用的Session使用不同的Bean，仅使用于WebApplicationContext环境
* global-session模式：一般用于Portlet应用环境，该运用仅使用于WebApplicationContext环境

#### singleton 作用域：

​	singleton是单例类型， 就是在创建起容器就同时自动创建了一个bean对象， 不管你是否使用， 他都存在了， 每次获取到的对象都是同一个对象。

``` xml
<bean id="..." class="..." scope="singleton">
	<!-- collaborators and configuration for this bean go here-->
</bean>
```



#### prototype 作用域：

​	它再我们创建容器的时候并没有实例化，而是我们获取bean的时候才去创建一个对象，而且每次获取到的对象都不是同一个对象。

```xml
<bean id="..." class="..." scope="prototype">
	<!-- collaborators and configuration for this bean go here-->
</bean>
```



# Spring Bean 生命周期

> 定义安装和拆卸一个bean，声明 init-method 和destroy-method参数的。
>
> 实例化bean时立即调用 init-methood 从容器中移除bean时调用 destroy-method
>
> Bean的生命周期可以表达为：Bean的定义-----Bean的初始化---Bean的使用-----Bean的销毁

```xml
<bean id="..." class="..." scope="singleton"
      init-method="..." destory-method="...">
	<!-- collaborators and configuration for this bean go here-->
</bean>
```

#### 默认的初始化和销毁方法：

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>

```

# Spring Bean 后置处理器

Bean后置处理器允许再调用初始化方法前后对bean进行额外的处理。

#### BeanPostProcessor接口定义回调方法：

​	可以实现该接口提供自己的实例化逻辑，依赖解析逻辑等。

可以通过BeanPostProcessor实现的Ordered接口提供的order属性来控制这些BeanPostProcessor接口的执行顺序。

ApplicationContext会自动检测由BeanPostProcessor接口定义的bean，注册这些bean为后置处理器，然后通过再容器中创建bean，在适当的时候调用它。

```java
package com.aop.chapter;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class InitHelloWorld implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeforeInitialization:" + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("AfterInitialization:" + beanName);
        return bean;
    }
}

```



# Spring 依赖注入

> 依赖注入就是把类粘合在一起同时保持他们独立。

通常使用类的构造函数或setter方法参数引入另一个类来保证互相的独立性。

``` java
public class TextEditor {
   private SpellChecker spellChecker;
   public TextEditor(SpellChecker spellChecker) {
      this.spellChecker = spellChecker;
   }
}
```

这样只需将SpellChecker用作TextEditor的参数。然后实例化整个过程是由Spring框架的控制。

#### Spring 基于构造函数的依赖注入

> 当容器调用带有一组参数的类构造函数时，基于构造函数的DI就完成了，其中每个参数代表一个对其他类的依赖。

```java
package com.aop.chapter.editor;

public class SpellChecker {
    public SpellChecker() {
        System.out.println("Inside SpellChecker Constructor");
    }
    public void CheckSpelling() {
        System.out.println("Inside CheckSpell");
    }
}

```



```java
package com.aop.chapter.editor;

public class TextEditor {
    private SpellChecker spellChecker;

    public TextEditor(SpellChecker spellChecker) {
        System.out.println("Inside TextEditor Constructor");
        this.spellChecker = spellChecker;
    }
    public void SpellCheck() {
        spellChecker.CheckSpelling();
    }
}

```



```java
package com.aop.chapter;

import com.aop.chapter.editor.TextEditor;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
        TextEditor textEditor = (TextEditor)context.getBean("textEditor");
        textEditor.SpellCheck();
    }
}
```

```xml
<bean class="com.aop.chapter.editor.TextEditor" id="textEditor">
    <constructor-arg ref="spellChecker"/>
</bean>

<bean class="com.aop.chapter.editor.SpellChecker" id="spellChecker">

</bean>
```

#### Spring 基于setter函数的依赖注入

将上文的构造函数改为setter函数 && 将<bean>标签中的<constructor-arg>元素改为 <property>元素 注意:引用的是对象的话要用**ref**，若直接传值的话用**value**属性。

#### Spring 注入内部Beans

在java中内部类是在其它类中定义的一个类，同理，inner beans是在其他bean中定义的bean -->称为内部bean，例如：

```java
<bean id="..." class="...">
	<property name="...">
		<bean id="..." class="..."/>
	</property>
</bean>
```

