---
title: org.springframework.beans.factory.UnsatisfiedDependencyException异常
date: 2019-08-20 20:47:00
author: 周李
top: false
cover: false
toc: false
mathjax: false
categories: Exception
tags:
  - java
  - Exceprion
---

### org.springframework.beans.factory.UnsatisfiedDependencyException异常：



``` 
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dictServiceImpl': Unsatisfied dependency expressed through field 'dictMapper'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dictMapper' defined in file [G:\javaproject\mybtisspring\target\mybtis-spring\WEB-INF\classes\tk\mybatis\web\mapper\DictMapper.class]: Unsatisfied dependency expressed through bean property 'sqlSessionFactory'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sqlSessionFactory' defined in class path resource [applicationContext.xml]: Invocation of init method failed; nested exception is org.springframework.core.NestedIOException: Failed to parse config resource: class path resource [mybatis-config.xml]; nested exception is org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: org.apache.ibatis.logging.LogException: Error setting Log implementation.  Cause: java.lang.NoClassDefFoundError: org/apache/log4j/Priority
		at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:588)
		at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
		at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:366)
		at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1264)
		at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:553)
		at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
		at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
		at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
		at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
		at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
		at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:761)
		at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:867)
		at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:543)
		at org.springframework.web.context.ContextLoader.configureAndRefreshWebApplicationContext(ContextLoader.java:443)
		at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:325)
		at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:107)
		at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4770)
		at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5236)
		at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
		at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:754)
		at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:730)
		at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:744)
		at org.apache.catalina.startup.HostConfig.manageApp(HostConfig.java:1730)
		at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
		at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.lang.reflect.Method.invoke(Method.java:498)
		at org.apache.tomcat.util.modeler.BaseModelMBean.invoke(BaseModelMBean.java:287)
		at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819)
		at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801)
		at org.apache.catalina.mbeans.MBeanFactory.createStandardContext(MBeanFactory.java:483)
		at org.apache.catalina.mbeans.MBeanFactory.createStandardContext(MBeanFactory.java:432)
		at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
		at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.lang.reflect.Method.invoke(Method.java:498)
		at org.apache.tomcat.util.modeler.BaseModelMBean.invoke(BaseModelMBean.java:287)
		at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819)
		at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801)
		at com.sun.jmx.remote.security.MBeanServerAccessController.invoke(MBeanServerAccessController.java:468)
		at javax.management.remote.rmi.RMIConnectionImpl.doOperation(RMIConnectionImpl.java:1468)
		at javax.management.remote.rmi.RMIConnectionImpl.access$300(RMIConnectionImpl.java:76)
		at javax.management.remote.rmi.RMIConnectionImpl$PrivilegedOperation.run(RMIConnectionImpl.java:1309)
		at java.security.AccessController.doPrivileged(Native Method)
		at javax.management.remote.rmi.RMIConnectionImpl.doPrivilegedOperation(RMIConnectionImpl.java:1408)
		at javax.management.remote.rmi.RMIConnectionImpl.invoke(RMIConnectionImpl.java:829)
		at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
		at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
		at java.lang.reflect.Method.invoke(Method.java:498)
		at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:357)
		at sun.rmi.transport.Transport$1.run(Transport.java:200)
		at sun.rmi.transport.Transport$1.run(Transport.java:197)
		at java.security.AccessController.doPrivileged(Native Method)
		at sun.rmi.transport.Transport.serviceCall(Transport.java:196)
		at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:573)
		at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(TCPTransport.java:834)
		at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(TCPTransport.java:688)
		at java.security.AccessController.doPrivileged(Native Method)
		at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:687)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
		at java.lang.Thread.run(Thread.java:748)
```

解决办法：

​	在mybatis-config.xml中的settings 中如果有<setting name="logImpl"value="LOG4J"/> 那jar有没有导入 文件有没有配置好

再就是service接口实现类上有没有加上@Service注解.

再就是扫包的路径有没有拼错

整合 要把Spring，mybatis配置导入到web.xml中：

``` 
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  
  
  
  
  
---------------------
<init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:mybatis-servlet.xml</param-value>
    </init-param>
```

