---
title: Thymeleaf异常Circular view path [list]: would dispatch back to the current handler URL
date: 2019-09-09 13:38:00
author: 周李
top: false
cover: false
toc: true
mathjax: false
categories: Exception
tags:
  - java
  - EXception
---



记录一次bug

SpringBoot 用Thymeleaf时 出现错误

javax.servlet.ServletException: Circular view path [list]: would dispatch back to the current handler URL [/list] again. Check your ViewResolver setup! (Hint: This may be the result of an unspecified view, due to default view name generation.

网上有两种解决办法：

1. 缺省转发
2. view和path同名

所以，解决方案如下
1.消除缺省转发
2.修改view和path，让他们不同名

##############

没有解决！

经过google的排查发现 把Maven中的

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<groupId>org
```

version	改为2.1.7即可 具体为什么会出错 俺也不知道 