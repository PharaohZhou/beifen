---
title: 【Spring Boot实战】论坛项目（一）
date: 2019-09-24 20:45:00
author: 周李
top: true
cover: true
toc: true
mathjax: false
categories: SpringBoot
tags:
  - java
  - SpringBoot
---

## 前言

此项目的主要意图时为了通过项目来学习Spring Boot。把书作为参考书，这样会很容易看到自己哪些地方薄弱再对症下药。

### 开发环境

项目采用Spring Boot、JDK1.8开发。

会用到什么? 

**Spring Boot , IDEA , Refactor , Git/Github , Bootstrap , Github Authorization , MySQL , MyBatis,.........**

### 快速搭建Spring Boot项目

使用IDEA很容易搭建一个Spring Boot项目 

![新建SpringBoot](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569331167/blog/W9_24_VPPP___BHG4JNMA4_z3agbg.png)

在新建完成后就可以着手使用Github

### 使用Github托管代码

先创建一个仓库community

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569331721/blog/QQ%E6%88%AA%E5%9B%BE20190924212722_urslfg.png)

在IDEA中打开Terminal：执行

git init （当前文件目录作为仓库）

git add . (全部文件丢到暂存里)

git commit -m "init repo" (提供commit信息)

git remote add origin 仓库的地址 (添加远程地址)

git push -u origin master  (提交到远程地址)



### Bootstrap 编写导航栏样式

​	为了降低学习成本直接去官网下载Bootstrap来快速实现前端功能

[Bootstrap下载](<https://v3.bootcss.com/getting-started/>)

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569332768/blog/bootstrap_cpsjjn.png)

下载Bootstrap将内文件复制到项目src\main\resources\static文件夹下新建index.html文件

将Bootstrap引入到index.html中

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569333082/blog/index1_u0l5su.png)

[Bootstrap](<https://v3.bootcss.com/components/>)组件标签里选择导航条：

复制导航条中第一个实例代码到index.html<body>元素中修改其中内容：

```
<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse"
                    data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                <span class="sr-only">Code社区</span>

            </button>
            <a class="navbar-brand" href="#">Code社区</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <form class="navbar-form navbar-left">
                <div class="form-group">
                    <input type="text" class="form-control" placeholder="搜索话题">
                </div>
                <button type="submit" class="btn btn-default">搜索</button>
            </form>
            <ul class="nav navbar-nav navbar-right">
                <li><a href="https://github.com/login/oauth/authorize?client_id=75b0ed1277b782ab053d&redirect_uri=http://localhost:8887/callback&scope=user&state=1">登录</a></li>
                <li class="dropdown">
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true"
                       aria-expanded="false">我 <span class="caret"></span></a>
                    <ul class="dropdown-menu">
                        <li><a href="#">消息中心</a></li>
                        <li><a href="#">个人资料</a></li>
                        <li><a href="#">退出登录</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

### 实现登录功能

##### 注册Github App

Setting-> Developer settings->OAuthApps

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569333834/blog/OAuth_dfrwwc.png)

这里的URL对应上文index.html中的链接地址

##### 图解Github登录流程

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569334740/blog/%E6%97%B6%E5%BA%8F%E5%9B%BE_dul90y.png)

：用户访问code社区进行登录操作 code社区会调用Github的authorize接口去访问Github，这时候Github会直接跳转回给他携带的redirect-uri并且携带一个code 当code社区接受到这个code时会再次调用Github的access-token并且携带刚刚传回的code。这步是为了去获取token。如果验证成功Github会直接返回access-token。code社区再次使用access-token调用Github的User API 这样就可以获取到Github的user信息。这个时候就可以更新登录状态。返回用户登陆成功！

### 测试登录功能

创建个简单的控制器

```
@Controller
public class IndexController {
    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```

运行项目：在浏览器中输入：http://localhost:8080

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569335771/blog/%E5%AF%BC%E8%88%AA%E6%A0%8F_wsojsp.png)

点击登录拉取Github登录此网站

![](https://res.cloudinary.com/ds5gmzvbq/image/upload/v1569335977/blog/QQ%E6%88%AA%E5%9B%BE20190924223849_qbtmbj.png)

