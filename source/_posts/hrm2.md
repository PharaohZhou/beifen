---
title: ssm小项目：人事管理系统(二)
date: 2019-08-22 12:25:00
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

## ssm小项目：人事管理系统(二)

### 部署DAO层

先为其提供对应的数据源 db.properties

```properties
dataSource.driverClass=com.mysql.cj.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://localhost:3306/hrm_db?serverTimezone=GMT
dataSource.user=root
dataSource.password=123456
dataSource.maxPoolSize=20
dataSource.maxIdleTime = 1000
dataSource.minPoolSize=6
dataSource.initialPoolSize=5
```

配置完数据源后就可以在此基础上配置SqlSessionFactory对象：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://mybatis.org/schema/mybatis-spring
       http://mybatis.org/schema/mybatis-spring.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <mybatis:scan base-package="top.zhoulis.www.dao"/>
    <context:component-scan base-package="top.zhoulis.www"/>

    <context:property-override location="classpath:db.properties"/>

<!--
配置c3p0数据源
添加依赖：   <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
    </dependency>
-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"/>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"
          p:dataSource-ref="dataSource"/>
</beans>
```

简化分页功能，设计一个分页的JSP标签

```java
package top.zhoulis.www.util.tag;

import top.zhoulis.www.util.common.HrmConstants;

public class PageModel {
    /*分页总数据条数*/
    private int recordCount;
    /*当前页面*/
    private int pageIndex;
    /*每页分多少数据*/
    private int pageSize = HrmConstants.PAGE_DEFAULT_SIZE = 4;
    /*总页数*/
    private int totalSize;

    public int getRecordCount() {
        this.recordCount = this.recordCount <=0 ? 0:this.recordCount;
        return recordCount;
    }

    public void setRecordCount(int recordCount) {
        this.recordCount = recordCount;
    }

    public int getPageIndex() {
        this.pageIndex = this.pageIndex <=0 ? 0:this.pageIndex;
        //判断当前页面是否超过了总页数 ： 如果超过了默认将最后一页作为当前页
        this.pageIndex = this.pageIndex>=this.getTotalSize()?this.getTotalSize():this.pageIndex;
        return pageIndex;
    }

    public void setPageIndex(int pageIndex) {
        this.pageIndex = pageIndex;
    }

    public int getPageSize() {
        this.pageSize = this.pageSize <=HrmConstants.PAGE_DEFAULT_SIZE ? HrmConstants.PAGE_DEFAULT_SIZE:this.pageSize;
        return pageSize;
    }

    public void setPageSize(int pageSize) {
        this.pageSize = pageSize;
    }

    public int getTotalSize() {
        if (this.getRecordCount() <= 0) {
            totalSize = 0;
        } else {
            totalSize = (this.getRecordCount() - 1) / this.getPageSize() + 1;
        }
        return totalSize;
    }

    public int getFirstLimitParam() {
        return (this.getPageIndex()-1) * this.getPageSize();
    }
}
```

```java
package top.zhoulis.www.util.tag;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.SimpleTagSupport;
import java.io.IOException;

/***
 * 分页标签
 */
public class PagerTag extends SimpleTagSupport {

    /** 定义请求URL中的占位符常量 */
    private static final String TAG = "{0}";

    /** 当前页码 */
    private int pageIndex;
    /** 每页显示的数量 */
    private int pageSize;
    /** 总记录条数 */
    private int recordCount;
    /** 请求URL page.action?pageIndex={0}*/
    private String submitUrl;
    /** 样式 */
    private String style = "sabrosus";

    /** 定义总页数 */
    private int totalPage = 0;

    /**  在页面上引用自定义标签就会触发一个标签处理类   */
    @Override
    public void doTag() throws JspException, IOException {
        /** 定义它拼接是终的结果 */
        StringBuilder res = new StringBuilder();
        /** 定义它拼接中间的页码 */
        StringBuilder str = new StringBuilder();
        /** 判断总记录条数 */
        if (recordCount > 0){   //1499 / 15  = 100
            /** 需要显示分页标签，计算出总页数 需要分多少页 */
            totalPage = (this.recordCount - 1) / this.pageSize + 1;

            /** 判断上一页或下一页需不需要加a标签 */
            if (this.pageIndex == 1){ // 首页
                str.append("<span class='disabled'>上一页</span>");

                /** 计算中间的页码 */
                this.calcPage(str);

                /** 下一页需不需要a标签 */
                if (this.pageIndex == totalPage){
                    /** 只有一页 */
                    str.append("<span class='disabled'>下一页</span>");
                }else{
                    String tempUrl = this.submitUrl.replace(TAG, String.valueOf(pageIndex + 1));
                    str.append("<a href='"+ tempUrl +"'>下一页</a>");
                }
            }else if (this.pageIndex == totalPage){ // 尾页
                String tempUrl = this.submitUrl.replace(TAG, String.valueOf(pageIndex - 1));
                str.append("<a href='"+ tempUrl +"'>上一页</a>");

                /** 计算中间的页码 */
                this.calcPage(str);

                str.append("<span class='disabled'>下一页</span>");
            }else{ // 中间
                String tempUrl = this.submitUrl.replace(TAG, String.valueOf(pageIndex - 1));
                str.append("<a href='"+ tempUrl +"'>上一页</a>");

                /** 计算中间的页码 */
                this.calcPage(str);

                tempUrl = this.submitUrl.replace(TAG, String.valueOf(pageIndex + 1));
                str.append("<a href='"+ tempUrl +"'>下一页</a>");
            }

            /** 拼接其它的信息 */
            res.append("<table width='100%' align='center' style='font-size:13px;' class='"+ style +"'>");
            res.append("<tr><td style='COLOR: #0061de; MARGIN-RIGHT: 3px; PADDING-TOP: 2px; TEXT-DECORATION: none'>" + str.toString());
            res.append("&nbsp;跳转到&nbsp;&nbsp;<input style='text-align: center;BORDER-RIGHT: #aaaadd 1px solid; PADDING-RIGHT: 5px; BORDER-TOP: #aaaadd 1px solid; PADDING-LEFT: 5px; PADDING-BOTTOM: 2px; MARGIN: 2px; BORDER-LEFT: #aaaadd 1px solid; COLOR: #000099; PADDING-TOP: 2px; BORDER-BOTTOM: #aaaadd 1px solid; TEXT-DECORATION: none' type='text' size='2' id='pager_jump_page_size'/>");
            res.append("&nbsp;<input type='button' style='text-align: center;BORDER-RIGHT: #dedfde 1px solid; PADDING-RIGHT: 6px; BACKGROUND-POSITION: 50% bottom; BORDER-TOP: #dedfde 1px solid; PADDING-LEFT: 6px; PADDING-BOTTOM: 2px; BORDER-LEFT: #dedfde 1px solid; COLOR: #0061de; MARGIN-RIGHT: 3px; PADDING-TOP: 2px; BORDER-BOTTOM: #dedfde 1px solid; TEXT-DECORATION: none' value='确定' id='pager_jump_btn'/>");
            res.append("</td></tr>");
            res.append("<tr align='center'><td style='font-size:13px;'><tr><td style='COLOR: #0061de; MARGIN-RIGHT: 3px; PADDING-TOP: 2px; TEXT-DECORATION: none'>");
            /** 开始条数 */
            int startNum = (this.pageIndex - 1) * this.pageSize + 1;
            /** 结束条数 */
            int endNum = (this.pageIndex == this.totalPage) ? this.recordCount : this.pageIndex * this.pageSize;

            res.append("总共<font color='red'>"+ this.recordCount +"</font>条记录，当前显示"+ startNum +"-"+ endNum +"条记录。");
            res.append("</td></tr>");
            res.append("</table>");
            res.append("<script type='text/javascript'>");
            res.append("   document.getElementById('pager_jump_btn').onclick = function(){");
            res.append("      var page_size = document.getElementById('pager_jump_page_size').value;");
            res.append("      if (!/^[1-9]\\d*$/.test(page_size) || page_size < 1 || page_size > "+ this.totalPage +"){");
            res.append("          alert('请输入[1-"+ this.totalPage +"]之间的页码！');");
            res.append("      }else{");
            res.append("         var submit_url = '" + this.submitUrl + "';");
            res.append("         window.location = submit_url.replace('"+ TAG +"', page_size);");
            res.append("      }");
            res.append("}");
            res.append("</script>");


        }else{
            res.append("<table align='center' style='font-size:13px;'><tr><td style='COLOR: #0061de; MARGIN-RIGHT: 3px; PADDING-TOP: 2px; TEXT-DECORATION: none'>总共<font color='red'>0</font>条记录，当前显示0-0条记录。</td></tr></table>");
        }
        this.getJspContext().getOut().print(res.toString());
    }


    /** 计算中间页码的方法 */
    private void calcPage(StringBuilder str) {
        /** 判断总页数 */
        if (this.totalPage <= 11){
            /** 一次性显示全部的页码 */
            for (int i = 1; i <= this.totalPage; i++){
                if (this.pageIndex == i){
                    /** 当前页码 */
                    str.append("<span class='current'>"+ i +"</span>");
                }else{
                    String tempUrl = this.submitUrl.replace(TAG, String.valueOf(i));
                    str.append("<a href='"+ tempUrl +"'>"+ i +"</a>");
                }
            }
        }else{
            /** 靠近首页 */
            if (this.pageIndex <= 8){
                for (int i = 1; i <= 10; i++){
                    if (this.pageIndex == i){
                        /** 当前页码 */
                        str.append("<span class='current'>"+ i +"</span>");
                    }else{
                        String tempUrl = this.submitUrl.replace(TAG, String.valueOf(i));
                        str.append("<a href='"+ tempUrl +"'>"+ i +"</a>");
                    }
                }
                str.append("...");
                String tempUrl = this.submitUrl.replace(TAG, String.valueOf(this.totalPage));
                str.append("<a href='"+ tempUrl +"'>"+ this.totalPage +"</a>");
            }
            /** 靠近尾页 */
            else if (this.pageIndex + 8 >= this.totalPage){
                String tempUrl = this.submitUrl.replace(TAG, String.valueOf(1));
                str.append("<a href='"+ tempUrl +"'>1</a>");
                str.append("...");

                for (int i = this.totalPage - 10; i <= this.totalPage; i++){
                    if (this.pageIndex == i){
                        /** 当前页码 */
                        str.append("<span class='current'>"+ i +"</span>");
                    }else{
                        tempUrl = this.submitUrl.replace(TAG, String.valueOf(i));
                        str.append("<a href='"+ tempUrl +"'>"+ i +"</a>");
                    }
                }
            }
            /** 在中间 */
            else{
                String tempUrl = this.submitUrl.replace(TAG, String.valueOf(1));
                str.append("<a href='"+ tempUrl +"'>1</a>");
                str.append("...");

                for (int i = this.pageIndex - 4; i <= this.pageIndex + 4; i++){
                    if (this.pageIndex == i){
                        /** 当前页码 */
                        str.append("<span class='current'>"+ i +"</span>");
                    }else{
                        tempUrl = this.submitUrl.replace(TAG, String.valueOf(i));
                        str.append("<a href='"+ tempUrl +"'>"+ i +"</a>");
                    }
                }

                str.append("...");
                tempUrl = this.submitUrl.replace(TAG, String.valueOf(this.totalPage));
                str.append("<a href='"+ tempUrl +"'>"+ this.totalPage +"</a>");
            }
        }
    }

    /** setter 方法 */
    public void setPageIndex(int pageIndex) {
        this.pageIndex = pageIndex;
    }
    public void setPageSize(int pageSize) {
        this.pageSize = pageSize;
    }
    public void setRecordCount(int recordCount) {
        this.recordCount = recordCount;
    }
    public void setSubmitUrl(String submitUrl) {
        this.submitUrl = submitUrl;
    }
    public void setStyle(String style) {
        this.style = style;
    }
}
```

要使用jsp标签库还需要增加一个tld标签文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
                   http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
                      version="2.1">
   
  <!-- 描述 自定义标签版本的一种描述 --> 
  <description>Pager 1.0 core library</description>
  <!-- 显示的名称 导包进行的一个展示 -->
  <display-name>Pager core</display-name>
  <!-- 版本号 -->
  <tlib-version>1.0</tlib-version>
  <!-- 短名 -->
  <short-name>fkjava</short-name>
  <!-- uri ：导包 -->
  <uri>/pager-tags</uri>
  
  <!-- 定义一个标签 -->
  <tag>
      <!-- 标签名 -->
      <name>pager</name>
      <!-- 标签处理类 -->
      <tag-class>top.zhoulis.www.util.tag.PagerTag</tag-class>
      <!-- 设置标签为空 -->
      <body-content>empty</body-content>
      
      <!-- 定义标签的属性 -->
      <attribute>
         <!-- 属性名 表示分页的第几页 -->
         <name>pageIndex</name>
         <!-- 必须的 -->
         <required>true</required>
         <!-- run time expression value 为true支持EL表达式 -->
         <rtexprvalue>true</rtexprvalue>
      </attribute>
      
      <!-- 定义标签的属性 -->
      <attribute>
         <!-- 属性名 表示分页标签 ，每页显示多少条数据 -->
         <name>pageSize</name>
         <!-- 必须的 -->
         <required>true</required>
         <!-- run time expression value 为true支持EL表达式 -->
         <rtexprvalue>true</rtexprvalue>
      </attribute>
      <!-- 定义标签的属性 -->
      <attribute>
         <!-- 属性名  记录分页的总数 -->
         <name>recordCount</name>
         <!-- 必须的 -->
         <required>true</required>
         <!-- run time expression value 为true支持EL表达式 -->
         <rtexprvalue>true</rtexprvalue>
      </attribute>
      <!-- 定义标签的属性 -->
      <attribute>
         <!-- 属性名 -->
         <name>submitUrl</name>
         <!-- 必须的 -->
         <required>true</required>
         <!-- run time expression value 为true支持EL表达式 -->
         <rtexprvalue>true</rtexprvalue>
      </attribute>
      <!-- 定义标签的属性 -->
      <attribute>
         <!-- 属性名 -->
         <name>style</name>
         <!-- 必须的 -->
         <required>false</required>
         <!-- run time expression value 为true支持EL表达式 -->
         <rtexprvalue>true</rtexprvalue>
      </attribute>
  </tag>
</taglib>
```

### 实现Service持久层

封装了6个DAO组件 采用了门面设计模式 使用业务逻辑组件将DAO对象封装在一起

业务逻辑组件面向DAO接口编程，可以让业务从DAO组件实现中分离。

下面是HRmService接口的源代码。

```java
package top.zhoulis.www.service;

import top.zhoulis.www.domain.*;
import top.zhoulis.www.util.tag.PageModel;

import java.util.List;


public interface HrmService {


   /**
    * 用户登录
    * @param  loginname
    * @param  password
    * @return User对象
    * */
   User login(String loginname, String password);
   
   /**
    * 根据id查询用户
    * @param id
    * @return 用户对象
    * */
   User findUserById(Integer id);
   
   /**
    * 获得所有用户
    * @return User对象的List集合
    * */
   List<User> findUser(User user, PageModel pageModel);
   
   /**
    * 根据id删除用户
    * @param id
    * */
   void removeUserById(Integer id);
   
   /**
    * 修改用户
    * @param user 用户对象
    * */
   void modifyUser(User user);
   
   /**
    * 添加用户
    * @param user 用户对象
    * */
   void addUser(User user);
   
   /**
    * 获得所有员工
    * @param employee 查询条件
    * @param pageModel 分页对象
    * @return Dept对象的List集合
    * */
   List<Employee> findEmployee(Employee employee, PageModel pageModel);
   
   /**
    * 根据id删除员工
    * @param id
    * */
   void removeEmployeeById(Integer id);
   
   /**
    * 根据id查询员工
    * @param id
    * @return 员工对象
    * */
   Employee findEmployeeById(Integer id);
   
   /**
    * 添加员工
    * @param employee 员工对象
    * */
   void addEmployee(Employee employee);
   
   /**
    * 修改员工
    * @param employee 员工对象
    * */
   void modifyEmployee(Employee employee);
   
   /**
    * 获得所有部门，分页查询
    * @return Dept对象的List集合
    * */
   List<Dept> findDept(Dept dept, PageModel pageModel);
   
   /**
    * 获得所有部门
    * @return Dept对象的List集合
    * */
   List<Dept> findAllDept();
   
   /**
    * 根据id删除部门
    * @param id
    * */
   public void removeDeptById(Integer id);
   
   /**
    * 添加部门
    * @param dept 部门对象
    * */
   void addDept(Dept dept);
   
   /**
    * 根据id查询部门
    * @param id
    * @return 部门对象
    * */
   Dept findDeptById(Integer id);
   
   /**
    * 修改部门
    * @param dept 部门对象
    * */
   void modifyDept(Dept dept);
   
   /**
    * 获得所有职位
    * @return Job对象的List集合
    * */
   List<Job> findAllJob();
   
   List<Job> findJob(Job job,PageModel pageModel);
   
   /**
    * 根据id删除职位
    * @param id
    * */
   public void removeJobById(Integer id);
   
   /**
    * 添加职位
    * @param job
    * */
   void addJob(Job job);
   
   /**
    * 根据id查询职位
    * @param id
    * @return 职位对象
    * */
   Job findJobById(Integer id);
   
   /**
    * 修改职位
    * @param job
    * */
   void modifyJob(Job job);
   
   
   /**
    * 获得所有公告
    * @return Notice对象的List集合
    * */
   List<Notice> findNotice(Notice notice, PageModel pageModel);
   
   /**
    * 根据id查询公告
    * @param id
    * @return 公告对象
    * */
   Notice findNoticeById(Integer id);
   
   /**
    * 根据id删除公告
    * @param id
    * */
   public void removeNoticeById(Integer id);
   
   /**
    * 添加公告
    * @param notice 公告对象
    * */
   void addNotice(Notice notice);
   
   /**
    * 修改公告
    * @param notice 公告对象
    * */
   void modifyNotice(Notice notice);
   
   /**
    * 获得所有文档
    * @return Document对象的List集合
    * */
   List<Document> findDocument(Document document,PageModel pageModel);
   
   /**
    * 添加文档
    * @param document 文件对象
    * */
   void addDocument(Document document);
   
   /**
    * 根据id查询文档
    * @param id
    * @return 文档对象
    * */
   Document findDocumentById(Integer id);
   
   /**
    * 根据id删除文档
    * @param id
    * */
   public void removeDocumentById(Integer id);
   
   /**
    * 修改文档
    * @param document 公告对象
    * */
   void modifyDocument(Document document);
   
   
}
```

接下来是这些业务逻辑方法的实现：

```java
package top.zhoulis.www.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import top.zhoulis.www.dao.*;
import top.zhoulis.www.domain.*;
import top.zhoulis.www.service.HrmService;
import top.zhoulis.www.util.tag.PageModel;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Transactional(propagation= Propagation.REQUIRED,isolation= Isolation.DEFAULT)
@Service("hrmService")
public class HrmServiceImpl implements HrmService {

    /**
     * 自动注入持久层Dao对象
     * */
    @Autowired
    private UserDao userDao;

    @Autowired
    private DeptDao deptDao;

    @Autowired
    private EmployeeDao employeeDao;

    @Autowired
    private JobDao jobDao;

    @Autowired
    private NoticeDao noticeDao;

    @Autowired
    private DocumentDao documentDao;

    /*****************用户服务接口实现*************************************/
    /**
     * HrmServiceImpl接口login方法实现
     *  @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public User login(String loginname, String password) {
//    System.out.println("HrmServiceImpl login -- >>");
        return userDao.selectByLoginnameAndPassword(loginname, password);
    }

    /**
     * HrmServiceImpl接口findUser方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<User> findUser(User user, PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("user", user);
        int recordCount = userDao.count(params);
        pageModel.setRecordCount(recordCount);
        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }
        List<User> users = userDao.selectByPage(params);

        return users;
    }

    /**
     * HrmServiceImpl接口findUserById方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public User findUserById(Integer id) {
        return userDao.selectById(id);
    }

    /**
     * HrmServiceImpl接口removeUserById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeUserById(Integer id) {
        userDao.deleteById(id);

    }

    /**
     * HrmServiceImpl接口addUser方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyUser(User user) {
        userDao.update(user);

    }

    /**
     * HrmServiceImpl接口modifyUser方法实现
     * @see { HrmService }
     * */
    @Override
    public void addUser(User user) {
        userDao.save(user);

    }

    /*****************部门服务接口实现*************************************/
    @Transactional(readOnly=true)
    @Override
    public List<Dept> findAllDept() {

        return deptDao.selectAllDept();
    }

    /**
     * HrmServiceImpl接口findDept方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<Dept> findDept(Dept dept,PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("dept", dept);
        int recordCount = deptDao.count(params);
        pageModel.setRecordCount(recordCount);

        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }

        List<Dept> depts = deptDao.selectByPage(params);

        return depts;
    }

    /**
     * HrmServiceImpl接口removeUserById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeDeptById(Integer id) {
        deptDao.deleteById(id);

    }

    /**
     * HrmServiceImpl接口addDept方法实现
     * @see { HrmService }
     * */
    @Override
    public void addDept(Dept dept) {
        deptDao.save(dept);

    }

    /**
     * HrmServiceImpl接口findDeptById方法实现
     * @see { HrmService }
     * */
    @Override
    public Dept findDeptById(Integer id) {

        return deptDao.selectById(id);
    }

    /**
     * HrmServiceImpl接口modifyDept方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyDept(Dept dept) {
        deptDao.update(dept);

    }
    /*****************员工服务接口实现*************************************/
    /**
     * HrmService接口findEmployee方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<Employee> findEmployee(Employee employee, PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("employee", employee);

        int recordCount = employeeDao.count(params);
        pageModel.setRecordCount(recordCount);

        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }
        List<Employee> employees = employeeDao.selectByPage(params);
        return employees;
    }
    /**
     * HrmService接口removeEmployeeById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeEmployeeById(Integer id) {
        employeeDao.deleteById(id);

    }
    /**
     * HrmService接口findEmployeeById方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public Employee findEmployeeById(Integer id) {

        return employeeDao.selectById(id);
    }

    /**
     * HrmService接口addEmployee方法实现
     * @see { HrmService }
     * */
    @Override
    public void addEmployee(Employee employee) {
        employeeDao.save(employee);

    }

    /**
     * HrmService接口modifyEmployee方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyEmployee(Employee employee) {
        employeeDao.update(employee);
    }

    /*****************职位接口实现*************************************/

    /**
     * HrmService接口findAllJob方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<Job> findAllJob() {

        return jobDao.selectAllJob();
    }

    /**
     * HrmService接口findJob方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<Job> findJob(Job job, PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("job", job);
        int recordCount = jobDao.count(params);
        pageModel.setRecordCount(recordCount);

        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }

        List<Job> jobs = jobDao.selectByPage(params);

        return jobs;
    }

    /**
     * HrmService接口removeJobById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeJobById(Integer id) {
        jobDao.deleteById(id);

    }

    /**
     * HrmService接口addJob方法实现
     * @see { HrmService }
     * */
    @Override
    public void addJob(Job job) {
        jobDao.save(job);

    }

    /**
     * HrmService接口findJobById方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public Job findJobById(Integer id) {

        return jobDao.selectById(id);
    }

    /**
     * HrmService接口modifyJob方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyJob(Job job) {
        jobDao.update(job);

    }

    /*****************公告接口实现*************************************/
    @Transactional(readOnly=true)
    @Override
    public List<Notice> findNotice(Notice notice, PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("notice", notice);
        int recordCount = noticeDao.count(params);
        pageModel.setRecordCount(recordCount);

        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }

        List<Notice> notices = noticeDao.selectByPage(params);

        return notices;
    }

    /**
     * HrmService接口findNoticeById方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public Notice findNoticeById(Integer id) {

        return noticeDao.selectById(id);
    }

    /**
     * HrmService接口removeNoticeById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeNoticeById(Integer id) {
        noticeDao.deleteById(id);

    }

    /**
     * HrmService接口addNotice方法实现
     * @see { HrmService }
     * */
    @Override
    public void addNotice(Notice notice) {
        noticeDao.save(notice);

    }

    /**
     * HrmService接口modifyNotice方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyNotice(Notice notice) {
        noticeDao.update(notice);

    }

    /*****************文件接口实现*************************************/

    /**
     * HrmService接口findDocument方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public List<Document> findDocument(Document document, PageModel pageModel) {
        /** 当前需要分页的总数据条数  */
        Map<String,Object> params = new HashMap<>();
        params.put("document", document);
        int recordCount = documentDao.count(params);
        pageModel.setRecordCount(recordCount);

        if(recordCount > 0){
            /** 开始分页查询数据：查询第几页的数据 */
            params.put("pageModel", pageModel);
        }

        List<Document> documents = documentDao.selectByPage(params);

        return documents;
    }

    /**
     * HrmService接口addDocument方法实现
     * @see { HrmService }
     * */
    @Override
    public void addDocument(Document document) {
        documentDao.save(document);

    }
    /**
     * HrmService接口removeDocumentById方法实现
     * @see { HrmService }
     * */
    @Override
    public void removeDocumentById(Integer id) {
        documentDao.deleteById(id);

    }
    /**
     * HrmService接口modifyDocument方法实现
     * @see { HrmService }
     * */
    @Override
    public void modifyDocument(Document document) {
        documentDao.update(document);

    }
    /**
     * HrmService接口findDocumentById方法实现
     * @see { HrmService }
     * */
    @Transactional(readOnly=true)
    @Override
    public Document findDocumentById(Integer id) {

        return documentDao.selectById(id);
    }


}
```

### 事务管理

事务管理配置在applicationContext.xml中：

```
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
      p:dataSource-ref="dataSource"/>

<tx:annotation-driven transaction-manager="transactionManager"/>
```