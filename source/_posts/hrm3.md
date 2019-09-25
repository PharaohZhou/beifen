---
title: ssm小项目：人事管理系统(三)
date: 2019-08-29 12:25:00
author: 周李
top: false
cover: false
toc: true
mathjax: false
categories: ssm
tags:
  - java
  - ssm
---

# 实现Web层

> 当控制器接受发到用户请求后，控制器并不会处理用户请求，只是对用户的请求参数进行解析处理，然后调用业务逻辑方法来处理用户请求； 当请求被处理完成后，控制器负责将处理结果通过JSP页面呈现给用户

web.xml

``` XML
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext*.xml</param-value>
  </context-param>
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <jsp-config>
    <jsp-property-group>
      <url-pattern>*.jsp</url-pattern>
      <el-ignored>false</el-ignored>
      <scripting-invalid>true</scripting-invalid>
      <include-prelude>/WEB-INF/jsp/taglib.jsp</include-prelude>
    </jsp-property-group>
  </jsp-config>
<!--  <error-page>-->
<!--    <error-code>404</error-code>-->
<!--    <location>/404.html</location>-->
<!--  </error-page>-->
<!--  <welcome-file-list>-->
<!--    <welcome-file>index.jsp</welcome-file>-->
<!--  </welcome-file-list>-->
</web-app>

```

对于使用Spring mvc的应用而言，控制器实际上由两个部分组成：系统的核心控制器DispatcherServlet和业务控制器Controller。

springmvc-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="top.zhoulis.www.controller"/>
    <mvc:annotation-driven/>

    <mvc:default-servlet-handler/>
<!--    <mvc:interceptors>-->
<!--        <mvc:interceptor>-->
<!--            <mvc:mapping path="/*"/>-->
<!--            <bean class="top.zhoulis.www.interceptor.AuthorizedInterceptor"/>-->
<!--        </mvc:interceptor>-->
<!--    </mvc:interceptors>-->
    <!-- 视图解析器   -->
    <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          p:prefix="/WEB-INF/jsp/" p:suffix=".jsp"/>

<!--    <bean id="multipartResolver"-->
<!--          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">-->
<!--        <property name="maxUploadSize">-->
<!--            <value>10485760</value>-->
<!--        </property>-->
<!--        <property name="defaultEncoding">-->
<!--            <value>UTF-8</value>-->
<!--        </property>-->
<!--    </bean>-->
</beans>
```

# 用户管理

处理用户的UserController

```java
package top.zhoulis.www.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;
import top.zhoulis.www.domain.User;
import top.zhoulis.www.service.HrmService;
import top.zhoulis.www.util.common.HrmConstants;
import top.zhoulis.www.util.tag.PageModel;

import javax.servlet.http.HttpSession;
import java.util.List;

/**
 * 处理用户请求控制器
 */
@Controller
public class UserController {

    /**
     * 自动注入UserService
     * */
    @Autowired
    @Qualifier("hrmService")
    private HrmService hrmService;

    /**
     * 处理登录请求
     * @param  loginname  登录名
     * @param  password 密码
     * @return 跳转的视图
     * */
    @RequestMapping(value="/login")
    public ModelAndView login(@RequestParam("loginname") String loginname,
                              @RequestParam("password") String password,
                              HttpSession session,
                              ModelAndView mv){
        System.out.println("zzzzzzzzzzzzzzzzzzzzzzzzzz");
        // 调用业务逻辑组件判断用户是否可以登录
        User user = hrmService.login(loginname, password);
        System.out.println("执行了login");
        if(user != null){
            // 将用户保存到HttpSession当中
            session.setAttribute(HrmConstants.USER_SESSION, user);
            // 客户端跳转到main页面
            mv.setViewName("redirect:/main");
        }else{
            // 设置登录失败提示信息
            mv.addObject("message", "登录名或密码错误!请重新输入");
            // 服务器内部跳转到登录页面
            mv.setViewName("forward:/loginForm");
        }
        return mv;

    }

    /**
     * 处理查询请求
     * */
    @RequestMapping(value="/user/selectUser")
    public String selectUser(Integer pageIndex,
                             @ModelAttribute User user,
                             Model model){
        System.out.println("user = " + user);
        PageModel pageModel = new PageModel();
        if(pageIndex != null){
            pageModel.setPageIndex(pageIndex);
        }
        /** 查询用户信息     */
        List<User> users = hrmService.findUser(user, pageModel);
        model.addAttribute("users", users);
        model.addAttribute("pageModel", pageModel);
        return "user/user";

    }

    /**
     * 处理删除用户请求
     * */
    @RequestMapping(value="/user/removeUser")
    public ModelAndView removeUser(String ids,ModelAndView mv){
        // 分解id字符串
        String[] idArray = ids.split(",");
        for(String id : idArray){
            // 根据id删除员工
            hrmService.removeUserById(Integer.parseInt(id));
        }
        // 设置客户端跳转到查询请求
        mv.setViewName("redirect:/user/selectUser");
        // 返回ModelAndView
        return mv;
    }


    /**
     * 处理修改用户请求
     * */
    @RequestMapping(value="/user/updateUser")
    public ModelAndView updateUser(
            String flag,
            @ModelAttribute User user,
            ModelAndView mv){
        if(flag.equals("1")){
            // 根据id查询用户
            User target = hrmService.findUserById(user.getId());
            // 设置Model数据
            mv.addObject("user", target);
            // 返回修改员工页面
            mv.setViewName("user/showUpdateUser");
        }else{
            // 执行修改操作
            hrmService.modifyUser(user);
            // 设置客户端跳转到查询请求
            mv.setViewName("redirect:/user/selectUser");
        }
        // 返回
        return mv;
    }


    /**
     * 处理添加请求
     * */
    @RequestMapping(value="/user/addUser")
    public ModelAndView addUser(
            String flag,
            @ModelAttribute User user,
            ModelAndView mv){
        if(flag.equals("1")){
            // 设置跳转到添加页面
            mv.setViewName("user/showAddUser");
        }else{
            // 执行添加操作
            hrmService.addUser(user);
            // 设置客户端跳转到查询请求
            mv.setViewName("redirect:/user/selectUser");
        }
        // 返回
        return mv;
    }

    /**
     * 处理注销退出请求
     * */
    @RequestMapping(value="/logout")
    public ModelAndView logout(
            ModelAndView mv,
            HttpSession session) {
        // 注销session
        session.invalidate();
        // 跳转到登录页面
        mv.setViewName("redirect:/loginForm");
        return mv;
    }

}

```

部署这个Web应用，在浏览器中输入如下URL来测试应用：

http://localhost:8080/hrmanage