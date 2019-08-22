---
title: ssm小项目：人事管理系统(一)
date: 2019-08-21 15:25:00
author: 周李
top: true
cover: true
toc: true
mathjax: false
categories: ssm
tags:
  - java
  - ssm
---

## ssm小项目：人事管理系统(一)

该项目包括用户管理、部门管理、职位管理、员工管理、、公告管理下载中心等系统功能。
	采用Spring+SpringMVC+MyBatis架构，Spring的IOC容器负责管理业务逻辑组件、持久层组件及控制层组件。

- 用户管理的功能包括:添加用户，用户可以为管理员或者普通用户;查询用户，可以查询
所有用户或根据用户名和用户状态进行模糊查询;删除用户;修改用户。

- 部门管理的功能包括:添加部门;查询部门，可以查询所有部门或根据部门名称进行模糊
查询;删除部门;修改部门。
-  职位管理的功能包括:添加职位;查询职位，可以查询所有职位或根据职位名称进行模糊
查询;删除职位;修改职位。
-  员工管理的功能包括:添加员工;查询员工，可以查询所有员工或根据员工姓名、身份证
号、手机号、性别、职位、部门进行模糊查询;删除员工;修改员工。
- 公告管理的功能包括:添加公告;查询公告，可以查询所有公告或根据公告名称、公告内
容进行模糊查询;删除公告;修改公告。
- 下载中心的功能包括:上传文件;查询文件，可以查询所有文件或根据文件标题进行模糊查询;
预览文件内容;删除文件;下载文件。所有查询页面统--使用分页处理。

### 设计数据库表

首先，根据模块功能设计数据库表，具体sql语句如下：

``` mysql
#创建数据库hrm_db
CREATE DATABASE hrm_db;
#使用数据库hrm_db
USE hrm_db;
#创建表dept_inf
CREATE TABLE dept_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  NAME VARCHAR(50) NOT NULL,
  REMARK VARCHAR(300) DEFAULT NULL,
  PRIMARY KEY (ID)
) ENGINE=INNODB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;
INSERT  INTO dept_inf(ID,NAME,REMARK) VALUES (1,'技术部','技术部'),(2,'运营部','运营部'),(3,'财务部','财务部'),(5,'总公办','总公办'),(6,'市场部','市场部'),(7,'教学部','教学部');
#创建表job_inf
CREATE TABLE job_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  NAME VARCHAR(50) NOT NULL,
  REMARK VARCHAR(300) DEFAULT NULL,
  PRIMARY KEY (ID)
) ENGINE=INNODB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;
INSERT  INTO job_inf(ID,NAME,REMARK) VALUES (1,'职员','职员'),(2,'Java开发工程师','Java开发工程师'),(3,'Java中级开发工程师','Java中级开发工程师'),(4,'Java高级开发工程师','Java高级开发工程师'),(5,'系统管理员','系统管理员'),(6,'架构师','架构师'),(7,'主管','主管'),(8,'经理','经理'),(9,'总经理','总经理');
#创建表user_inf
CREATE TABLE user_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  loginname VARCHAR(20) NOT NULL,
  PASSWORD VARCHAR(16) NOT NULL,
  STATUS INT(11) NOT NULL DEFAULT '1',
  createdate TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  username VARCHAR(20) DEFAULT NULL,
  PRIMARY KEY (ID)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
INSERT  INTO user_inf(ID,loginname,PASSWORD,USERSTATUS,createdate,username) VALUES (1,'admin','123456',2,'2016-03-12 09:34:28','超级管理员');
#创建表employee_inf
CREATE TABLE employee_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  DEPT_ID INT(11) NOT NULL,
  JOB_ID INT(11) NOT NULL,
  NAME VARCHAR(20) NOT NULL,
  CARD_ID VARCHAR(18) NOT NULL,
  ADDRESS VARCHAR(50) NOT NULL,
  POST_CODE VARCHAR(50) DEFAULT NULL,
  TEL VARCHAR(16) DEFAULT NULL,
  PHONE VARCHAR(11) NOT NULL,
  QQ_NUM VARCHAR(10) DEFAULT NULL,
  EMAIL VARCHAR(50) NOT NULL,
  SEX INT(11) NOT NULL DEFAULT '1',
  PARTY VARCHAR(10) DEFAULT NULL,
  BIRTHDAY DATETIME DEFAULT NULL,
  RACE VARCHAR(100) DEFAULT NULL,
  EDUCATION VARCHAR(10) DEFAULT NULL,
  SPECIALITY VARCHAR(20) DEFAULT NULL,
  HOBBY VARCHAR(100) DEFAULT NULL,
  REMARK VARCHAR(500) DEFAULT NULL,
  CREATE_DATE TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (ID),
  KEY FK_EMP_DEPT (DEPT_ID),
  KEY FK_EMP_JOB (JOB_ID),
  CONSTRAINT FK_EMP_DEPT FOREIGN KEY (DEPT_ID) REFERENCES dept_inf (ID),
  CONSTRAINT FK_EMP_JOB FOREIGN KEY (JOB_ID) REFERENCES job_inf (ID)
) ENGINE=INNODB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8;
INSERT  INTO employee_inf(ID,DEPT_ID,JOB_ID,NAME,CARD_ID,ADDRESS,POST_CODE,TEL, PHONE,QQ_NUM,EMAIL,SEX,PARTY,BIRTHDAY,RACE,EDUCATION,SPECIALITY,HOBBY,REMARK,CREATE_DATE) 
VALUES (1,1,8,'爱丽丝','4328011988','广州天河','510000','020-77777777', '13902001111','36750066','251425887@qq.com',0,'党员','1980-01-01 00:00:00','满','本科','美声','唱歌','四大天王','2016-03-14 11:35:18'),
(2,2,1,'杰克','22623','43234','42427424','42242','4247242','42424', 
'8********@qq.com',2,NULL,NULL,NULL,NULL,NULL,NULL,NULL,'2016-03-14 11:35:18'),
 (3,1,2,'bb','432801197711251038','广州','510000','020-99999999','13907351532', '36750064','36750064@qq.com',1,'党员','1977-11-25 00:00:00','汉','本科','计算机','爬山','无','2016-07-14 09:54:52');
#创建表notice_inf
CREATE TABLE notice_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  TITLE VARCHAR(50) NOT NULL,
  CONTENT TEXT NOT NULL,
  CREATE_DATE TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  USER_ID INT(11) DEFAULT NULL,
  PRIMARY KEY (ID),
  KEY FK_NOTICE_USER (USER_ID),
  CONSTRAINT FK_NOTICE_USER FOREIGN KEY (USER_ID) REFERENCES user_inf (ID)
) ENGINE=INNODB AUTO_INCREMENT=19 DEFAULT CHARSET=utf8;
#创建表document_inf
CREATE TABLE document_inf (
  ID INT(11) NOT NULL AUTO_INCREMENT,
  TITLE VARCHAR(50) NOT NULL,
  filename VARCHAR(300) NOT NULL,
  REMARK VARCHAR(300) DEFAULT NULL,
  CREATE_DATE TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  USER_ID INT(11) DEFAULT NULL,
  PRIMARY KEY (ID),
  KEY FK_DOCUMENT_USER (USER_ID),
  CONSTRAINT FK_DOCUMENT_USER FOREIGN KEY (USER_ID) REFERENCES user_inf (ID)
) ENGINE=INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

```

###  设计实体化实体

将六个数据库表转化为六个PO 分别对应：

user_info   -------- User    dept_info ---------- Dept

job_info -------- Job    employee_info --------Employee

notice_info  --------Notice    document_info ----------> Document



可以借助IDEA中插件MyBatisCodeHelperPro中自动生成pojo：
<img width=200 src= "https://res.cloudinary.com/ds5gmzvbq/image/upload/v1566456430/1_gnbnfo.png"> <img width=200 src="https://res.cloudinary.com/ds5gmzvbq/image/upload/v1566456431/2_htf4ng.png" >

图中1所指的是po的类名 2：所在的包 

User.java

``` java
package top.zhoulis.www.domain;

import java.io.Serializable;
import java.util.Date;
import lombok.Data;

@Data
public class User implements Serializable {
    private Integer id;

    private String loginname;

    private String password;

    private Integer status;

    private Date createDate;

    private String username;

    private static final long serialVersionUID = 1L;
}
```

Dept.java:

```java
package top.zhoulis.www.domain;

import java.io.Serializable;
import lombok.Data;

@Data
public class Dept implements Serializable {
    private Integer id;

    private String name;

    private String remark;

    private static final long serialVersionUID = 1L;
}
```

Job.java

```java
package top.zhoulis.www.domain;

import java.io.Serializable;
import lombok.Data;

@Data
public class Job implements Serializable {
    private Integer id;

    private String name;

    private String remark;

    private static final long serialVersionUID = 1L;
}
```

Employee.java

```java
package top.zhoulis.www.domain;

import java.io.Serializable;
import java.util.Date;
import lombok.Data;
import org.springframework.format.annotation.DateTimeFormat;

@Data
public class Employee implements Serializable {
    private Integer id;

    private Dept dept;

    private Job job;

    private String name;

    private String cardId;

    private String address;

    private String postCode;

    private String tel;

    private String phone;

    private String qqNum;

    private String email;

    private Integer sex;

    private String party;

    /**
     * 使用@ModelAttribute接受参数时
     * form表单中有日期，Spring不知如何转换，
     * 要在实体类的日期上加@DateTimeFormat(pattern="yyyy-MM-dd")注解
     */
    @DateTimeFormat(pattern="yyyy-MM-dd")
    private Date birthday;

    private String race;

    private String education;

    private String speciality;

    private String hobby;

    private String remark;

    private Date createDate;

    private static final long serialVersionUID = 1L;
}
```

Notice.java

```java
package top.zhoulis.www.domain;

import java.io.Serializable;
import java.util.Date;
import lombok.Data;

@Data
public class Notice implements Serializable {
    private Integer id;

    private String title;

    private String content;

    private Date createDate;

    private User user;

    private static final long serialVersionUID = 1L;
}
```

Document.java

```java
package top.zhoulis.www.domain;

import java.io.Serializable;
import java.util.Date;
import lombok.Data;
import org.springframework.web.multipart.MultipartFile;

@Data
public class Document implements Serializable {
    private Integer id;

    private String title;

    private String fileName;

    private MultipartFile file;

    private String remark;

    private Date createDate;

    private User user;

    private static final long serialVersionUID = 1L;
}
```

### 实现DAO持久层

每个DAO组件包含了数据库的访问逻辑；每个DAO组件可对一个数据库表完成基本的CRUD等操作

**公共常量类：**

```java
package top.zhoulis.www.util.common;

/**
 * @author zhou
 */
public class HrmConstants {
    //数据库表常量
    public static final String USERTABLE = "user_inf";
    public static final String DEPTTABLE = "dept_inf";
    public static final String JOBTABLE = "job_inf";
    public static final String EMPLOYEETABLE = "employee_inf";
    public static final String NOTICETABLE = "notice_inf";
    public static final String DOCUMENTTABLE = "document_inf";
    //登录
    public static final String LOGIN = "loginForm";
    //用户的session对象
    public static final String USER_SESSION = "user_session";
    //默认每页4条数据
    public static int PAGE_DEFAULT_SIZE = 4;
}
```

#### 定义DAO接口

UserDao.java

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.*;
import top.zhoulis.www.dao.provider.UserDynaSqlProvider;
import top.zhoulis.www.domain.User;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.USERTABLE;

/**
 * 用户dao接口
 * @author zhou
 */
public interface UserDao {
    /**
     * 根据登录名和密码查询员工
     * @param loginname
     * @param password
     * @return
     */
    @Select("select * from " + USERTABLE + " where loginname=#{loginname} and password = #{password}")
    User selectByLoginnameAndPassword(
            @Param("loginname") String loginname,
            @Param("password") String password);

    /**
     * 根据id查询用户
     * @param id
     * @return
     */
    @Select("select * from " + USERTABLE + " where id = #{id}")
    User selectById(Integer id);

    /**
     * 根据id删除用户
     * @param id
     */
    @Delete("delete from " + USERTABLE + " where id = #{id}")
    void deleteById(Integer id);

    /**
     * 动态修改用户
     * @param user
     */
    @SelectProvider(type=UserDynaSqlProvider.class,method = "updateUser")
    void update(User user);

    /**
     * 动态查询
     * @param params
     * @return
     */
    @SelectProvider(type= UserDynaSqlProvider.class,method = "selectWhitParam")
    List<User> selectByPage(Map<String,Object> params);

    /**
     * 根据参数查询用户总数
     * @param params
     * @return
     */
    @SelectProvider(type=UserDynaSqlProvider.class,method = "count")
    Integer count(Map<String, Object> params);

    /**
     * 动态插入用户
     * @param user
     */
    @SelectProvider(type = UserDynaSqlProvider.class,method = "insertUser")
    void save(User user);
}
```

UserDynaSqlProvider.java

```java
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.User;

import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.USERTABLE;


/**
 * UserDao接口中动态SQL实现
 * @author zhou
 */
public class UserDynaSqlProvider {

    /**
     * 分页动态查询
     * @param params
     * @return
     */
    public String selectWhitParam(final Map<String,Object> params) {
        String sql = new SQL(){
            {
                SELECT("*");
                FROM(USERTABLE);
                if (params.get("user") != null) {
                    User user = (User) params.get("user");
                    if (user.getUsername() != null && !user.getUsername().equals("")) {
                        WHERE(" username like concat('%',#{user.username},'%') ");
                    }
                    if (user.getStatus() != null && !user.getStatus().equals("")) {
                        WHERE(" status like concat('%',#{user.status},'%') ");
                    }
                }
            }
        }.toString();
        if (params.get("pageModel") != null) {
            sql += " limit #{pageModel.firstLimitParam},#{pageModel.pageSize} ";
        }
        return sql;
    }

    /**
     * 动态查询总数量
     * @param params
     * @return
     */
    public String count(final Map<String, Object> params) {
        return new SQL(){
            {
                SELECT("COUNT(*)");
                FROM(USERTABLE);
                if (params.get("user") != null) {
                    User user = (User) params.get("user");
                    if (user.getUsername() != null && !user.getUsername().equals("")) {
                        WHERE(" username like concat('%',#{user.username},'%') ");
                    }
                    if (user.getStatus() != null && !user.getStatus().equals("")) {
                        WHERE(" status like concat('%',#{user.status},'%') ");
                    }
                }
            }
        }.toString();
    }

    /**
     * 动态插入
     * @param user
     * @return
     */
    public String insertUser(final User user) {
        return new SQL(){
            {
                INSERT_INTO(USERTABLE);
                if (user.getUsername() != null && !user.getUsername().equals("")) {
                    VALUES("username","#{username}");
                }
                if (user.getStatus() != null && !user.getStatus().equals("")) {
                    VALUES("status","#{status}");
                }
                if (user.getLoginname() != null && !user.getLoginname().equals("")) {
                    VALUES("loginname","#{loginname}");
                }
                if (user.getPassword() != null && !user.getPassword().equals("")) {
                    VALUES("password", "#{password}");
                }
            }
        }.toString();
    }

    /**
     * 动态更新
     * @param user
     * @return
     */
    public String updateUser(final User user) {
        return new SQL(){
            {
                UPDATE(USERTABLE);
                if (user.getUsername() != null) {
                    SET(" username = #{username} ");
                }
                if (user.getLoginname() != null) {
                    SET(" loginname = #{loginname} ");
                }
                if (user.getPassword() != null) {
                    SET(" password = #{password} ");
                }
                if (user.getStatus() != null) {
                    SET(" status = #{status} ");
                }
                if (user.getCreateDate() != null) {
                    SET(" create_date = #{createDate} ");
                }
                WHERE(" id = #{id}");
            }
        }.toString();
    }
}
```

DeptDao.java

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.*;
import top.zhoulis.www.dao.provider.DeptDynaSqlProvider;
import top.zhoulis.www.domain.Dept;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.DEPTTABLE;

/**
 * 部门DeptDao接口
 * @author zhou
 */
public interface DeptDao {

    /**
     * 动态查询
     * @param params
     * @return
     */
    @SelectProvider(type = DeptDynaSqlProvider.class, method = "selectWhitParam")
    List<Dept> selectByPage(Map<String, Object> params);

    /**
     * 动态查询总数
     * @param params
     * @return
     */
    @SelectProvider(type = DeptDynaSqlProvider.class, method = "count")
    Integer count(Map<String, Object> params);

    @Select("select * from "+DEPTTABLE+" ")
    List<Dept> selectAllDept();

    /**
     * 根据id查询部门
     * @param id
     * @return
     */
    @Select("select * from " + DEPTTABLE + " where id = #{id}")
    Dept selectById(int id);

    /**
     * 根据id删除部门
     * @param id
     */
    @Delete(" delete from " + DEPTTABLE + " where id = #{id}")
    void deleteById(Integer id);

    /**
     * 动态插入部门
     * @param dept
     */
    @SelectProvider(type = DeptDynaSqlProvider.class, method = "insertDept")
    void save(Dept dept);

    /**
     * 动态修改用户
     * @param dept
     */
    @SelectProvider(type = DeptDynaSqlProvider.class, method = "updateDept")
    void update(Dept dept);

}
```

```
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.Dept;

import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.DEPTTABLE;

public class DeptDynaSqlProvider {

    /**
     * 分页动态查询
     * @param params
     * @return
     */
    public String selectWhitParam(final Map<String, Object> params) {
        String sql = new SQL(){
            {
                SELECT("*");
                FROM(DEPTTABLE);
                if (params.get("dept") != null) {
                    Dept dept = (Dept) params.get("dept");
                    if (dept.getName() != null && !dept.getName().equals("")) {
                        WHERE(" name like concat('%',#{dept.name},'%')");
                    }
                }
            }
        }.toString();
        if (params.get("pageModel") != null) {
            sql += " limit #{pageModel.firstLimitParam}, #{pageModel.pageSize} ";
        }
        return sql;
    }

    /**
     * 动态查询总数量
     * @param params
     * @return
     */
    public String count(final Map<String, Object> params) {
        return new SQL(){
            {
                SELECT("count(*)");
                FROM(DEPTTABLE);
                if (params.get("dept") != null) {
                    Dept dept = (Dept) params.get("dept");
                    if (dept.getName() != null && !dept.getName().equals("")) {
                        WHERE(" name like concat('%',#{dept.name},'%') ");
                    }
                }
            }
        }.toString();
    }

    /**
     * 动态插入部门
     * @param dept
     * @return
     */
    public String insertDept(final Dept dept) {
        return new SQL(){
            {
                INSERT_INTO(DEPTTABLE);
                if (dept.getName() != null && !dept.getName().equals("")) {
                    VALUES("name","#{name}");
                }
                if (dept.getRemark() != null && !dept.getRemark().equals("")) {
                    VALUES("remark", "#{remark}");
                }
            }
        }.toString();
    }

    public String updateDept(final Dept dept) {
        return new SQL(){
            {
                UPDATE(DEPTTABLE);
                if (dept.getName() != null) {
                    SET(" name = #{name} ");
                }
                if (dept.getRemark() != null) {
                    SET(" remark = #{remark} ");
                }
                WHERE(" id = #{id} ");
            }
        }.toString();
    }
}
```

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.*;
import org.apache.ibatis.mapping.FetchType;
import top.zhoulis.www.dao.provider.DocumentDynaSqlProvider;
import top.zhoulis.www.domain.Document;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.DOCUMENTTABLE;

public interface DocumentDao {
    // 动态查询
    @SelectProvider(type=DocumentDynaSqlProvider.class,method="selectWhitParam")
    @Results({
            @Result(id=true,column="id",property="id"),
            @Result(column="CREATE_DATE",property="createDate",javaType=java.util.Date.class),
            @Result(column="USER_ID",property="user",
                    one=@One(select="org.fkit.hrm.dao.UserDao.selectById",
                            fetchType= FetchType.EAGER))
    })
    List<Document> selectByPage(Map<String, Object> params);

    @SelectProvider(type= DocumentDynaSqlProvider.class,method="count")
    Integer count(Map<String, Object> params);

    // 动态插入文档
    @SelectProvider(type=DocumentDynaSqlProvider.class,method="insertDocument")
    void save(Document document);

    @Select("select * from "+DOCUMENTTABLE+" where ID = #{id}")
    Document selectById(int id);

    // 根据id删除文档
    @Delete(" delete from "+DOCUMENTTABLE+" where id = #{id} ")
    void deleteById(Integer id);

    // 动态修改文档
    @SelectProvider(type=DocumentDynaSqlProvider.class,method="updateDocument")
    void update(Document document);
}
```

```java
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.Document;

import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.DOCUMENTTABLE;

public class DocumentDynaSqlProvider {
    // 分页动态查询
    public String selectWhitParam(final Map<String, Object> params){
        String sql =  new SQL(){
            {
                SELECT("*");
                FROM(DOCUMENTTABLE);
                if(params.get("document") != null){
                    Document document = (Document) params.get("document");
                    if(document.getTitle() != null && !document.getTitle().equals("")){
                        WHERE("  title LIKE CONCAT ('%',#{document.title},'%') ");
                    }
                }
            }
        }.toString();

        if(params.get("pageModel") != null){
            sql += " limit #{pageModel.firstLimitParam} , #{pageModel.pageSize}  ";
        }

        return sql;
    }
    // 动态查询总数量
    public String count(final Map<String, Object> params){
        return new SQL(){
            {
                SELECT("count(*)");
                FROM(DOCUMENTTABLE);
                if(params.get("document") != null){
                    Document document = (Document) params.get("document");
                    if(document.getTitle() != null && !document.getTitle().equals("")){
                        WHERE("  title LIKE CONCAT ('%',#{document.title},'%') ");
                    }
                }
            }
        }.toString();
    }
    // 动态插入
    public String insertDocument(final Document document){

        return new SQL(){
            {
                INSERT_INTO(DOCUMENTTABLE);
                if(document.getTitle() != null && !document.getTitle().equals("")){
                    VALUES("title", "#{title}");
                }
                if(document.getFileName() != null && !document.getFileName().equals("")){
                    VALUES("filename", "#{fileName}");
                }
                if(document.getRemark() != null && !document.getRemark().equals("")){
                    VALUES("remark", "#{remark}");
                }
                if(document.getUser() != null && document.getUser().getId() != null){
                    VALUES("user_id", "#{user.id}");
                }
            }
        }.toString();
    }

    // 动态更新
    public String updateDocument(final Document document){

        return new SQL(){
            {
                UPDATE(DOCUMENTTABLE);
                if(document.getTitle() != null && !document.getTitle().equals("")){
                    SET(" title = #{title} ");
                }
                if(document.getFileName() != null && !document.getFileName().equals("")){
                    SET(" filename = #{fileName} ");
                }
                if(document.getRemark() != null && !document.getRemark().equals("")){
                    SET("remark = #{remark}");
                }
                if(document.getUser() != null && document.getUser().getId() != null){
                    SET("user_id = #{user.id}");
                }
                WHERE(" id = #{id} ");
            }
        }.toString();
    }
}
```

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.*;
import org.apache.ibatis.mapping.FetchType;
import top.zhoulis.www.dao.provider.EmployeeDynaSqlProvider;
import top.zhoulis.www.domain.Employee;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.EMPLOYEETABLE;

public interface EmployeeDao {

    /**
     * 根据参数动态查询员工总数
     * @param params
     * @return
     */
    @SelectProvider(type = EmployeeDynaSqlProvider.class, method = "count")
    Integer count(Map<String, Object> params);

    /**
     * 根据参数动态查询员工
     * @param params
     * @return
     */
    @SelectProvider(type = EmployeeDynaSqlProvider.class, method = "selectWhitParam")
    @Results({
            @Result(id = true, column = "id", property = "id"),
            @Result(column = "CARD_ID", property = "cardId"),
            @Result(column = "POST_CODE", property = "postCode"),
            @Result(column = "QQ_NUM", property = "qqNum"),
            @Result(column = "BIRTHDAY", property = "birthday", javaType = java.util.Date.class),
            @Result(column = "CREATE_DATE", property = "createDate", javaType = java.util.Date.class),
            @Result(column = "DEPT_ID", property = "dept",
                    one = @One(select = "org.fkit.hrm.dao.DeptDao.selectById",
                            fetchType = FetchType.EAGER)),
            @Result(column = "JOB_ID", property = "job",
                    one = @One(select = "org.fkit.hrm.dao.JobDao.selectById",
                            fetchType = FetchType.EAGER))
    })
    List<Employee> selectByPage(Map<String, Object> params);

    /**
     * 动态插入员工
     * @param employee
     */
    @SelectProvider(type = EmployeeDynaSqlProvider.class, method = "insertEmployee")
    void save(Employee employee);

    /**
     * 根据id删除员工
     * @param id
     */
    @Delete(" delete from " + EMPLOYEETABLE + " where id = #{id} ")
    void deleteById(Integer id);

    /**
     * 根据id查询员工
     * @param id
     * @return
     */
    @Select("select * from " + EMPLOYEETABLE + " where id = #{id} ")
    @Results({
            @Result(id = true, column = "id", property = "id"),
            @Result(column = "CARD_ID", property = "cardId"),
            @Result(column = "POST_CODE", property = "postCode"),
            @Result(column = "QQ_NUM", property = "qqNum"),
            @Result(column = "BIRTHDAY", property = "birthday", javaType = java.util.Date.class),
            @Result(column = "CREATE_DATE", property = "createDate", javaType = java.util.Date.class),
            @Result(column = "DEPT_ID", property = "dept",
                    one = @One(select = "org.fkit.hrm.dao.DeptDao.selectById",
                            fetchType = FetchType.EAGER)),
            @Result(column = "JOB_ID", property = "job",
                    one = @One(select = "org.fkit.hrm.dao.JobDao.selectById",
                            fetchType = FetchType.EAGER))
    })
    Employee selectById(Integer id);

    @SelectProvider(type = EmployeeDynaSqlProvider.class, method = "updateEmployee")
    void update(Employee employee);
}
```

```java
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.Employee;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.EMPLOYEETABLE;

public class EmployeeDynaSqlProvider {
    // 分页动态查询
    public String selectWhitParam(final Map<String, Object> params) {
        String sql = new SQL() {
            {
                SELECT("*");
                FROM(EMPLOYEETABLE);
                if (params.get("employee") != null) {
                    Employee employee = (Employee) params.get("employee");
                    if (employee.getDept() != null && employee.getDept().getId() != null && employee.getDept().getId() != 0) {
                        WHERE(" DEPT_ID = #{employee.dept.id} ");
                    }
                    if (employee.getJob() != null && employee.getJob().getId() != null && employee.getJob().getId() != 0) {
                        WHERE(" JOB_ID = #{employee.job.id} ");
                    }
                    if (employee.getName() != null && !employee.getName().equals("")) {
                        WHERE("  NAME LIKE CONCAT ('%',#{employee.name},'%') ");
                    }
                    if (employee.getPhone() != null && !employee.getPhone().equals("")) {
                        WHERE(" phone LIKE CONCAT ('%',#{employee.phone},'%') ");
                    }
                    if (employee.getCardId() != null && !employee.getCardId().equals("")) {
                        WHERE(" card_id LIKE CONCAT ('%',#{employee.cardId},'%') ");
                    }
                    if (employee.getSex() != null && employee.getSex() != 0) {
                        WHERE("sex = #{employee.sex}");
                    }
                }
            }
        }.toString();

        if (params.get("pageModel") != null) {
            sql += " limit #{pageModel.firstLimitParam} , #{pageModel.pageSize}  ";
        }

        return sql;
    }

    // 动态查询总数量
    public String count(final Map<String, Object> params) {
        return new SQL() {
            {
                SELECT("count(*)");
                FROM(EMPLOYEETABLE);
                if (params.get("employee") != null) {
                    Employee employee = (Employee) params.get("employee");
                    if (employee.getDept() != null && employee.getDept().getId() != null && employee.getDept().getId() != 0) {
                        WHERE(" DEPT_ID = #{employee.dept.id} ");
                    }
                    if (employee.getJob() != null && employee.getJob().getId() != null && employee.getJob().getId() != 0) {
                        WHERE(" JOB_ID = #{employee.job.id} ");
                    }
                    if (employee.getName() != null && !employee.getName().equals("")) {
                        WHERE("  NAME LIKE CONCAT ('%',#{employee.name},'%') ");
                    }
                    if (employee.getPhone() != null && !employee.getPhone().equals("")) {
                        WHERE(" phone LIKE CONCAT ('%',#{employee.phone},'%') ");
                    }
                    if (employee.getCardId() != null && !employee.getCardId().equals("")) {
                        WHERE(" card_id LIKE CONCAT ('%',#{employee.cardId},'%') ");
                    }
                    if (employee.getSex() != null && employee.getSex() != 0) {
                        WHERE("sex = #{employee.sex}");
                    }
                }
            }
        }.toString();
    }

    // 动态插入
    public String insertEmployee(final Employee employee) {

        return new SQL() {
            {
                INSERT_INTO(EMPLOYEETABLE);
                if (employee.getName() != null) {
                    VALUES("name", "#{name}");
                }
                if (employee.getCardId() != null) {
                    VALUES("card_id", "#{cardId}");
                }
                if (employee.getAddress() != null) {
                    VALUES("address", "#{address}");
                }
                if (employee.getPostCode() != null) {
                    VALUES("post_code", "#{postCode}");
                }
                if (employee.getTel() != null) {
                    VALUES("tel", "#{tel}");
                }
                if (employee.getPhone() != null) {
                    VALUES("phone", "#{phone}");
                }
                if (employee.getQqNum() != null) {
                    VALUES("qq_num", "#{qqNum}");
                }
                if (employee.getEmail() != null) {
                    VALUES("email", "#{email}");
                }
                if (employee.getSex() != null) {
                    VALUES("sex", "#{sex}");
                }
                if (employee.getParty() != null) {
                    VALUES("party", "#{party}");
                }
                if (employee.getBirthday() != null) {
                    VALUES("birthday", "#{birthday}");
                }
                if (employee.getRace() != null) {
                    VALUES("race", "#{race}");
                }
                if (employee.getEducation() != null) {
                    VALUES("education", "#{education}");
                }
                if (employee.getSpeciality() != null) {
                    VALUES("speciality", "#{speciality}");
                }
                if (employee.getHobby() != null) {
                    VALUES("hobby", "#{hobby}");
                }
                if (employee.getRemark() != null) {
                    VALUES("remark", "#{remark}");
                }
                if (employee.getCreateDate() != null) {
                    VALUES("create_Date", "#{createDate}");
                }
                if (employee.getDept() != null) {
                    VALUES("dept_id", "#{dept.id}");
                }
                if (employee.getJob() != null) {
                    VALUES("job_id", "#{job.id}");
                }
            }
        }.toString();
    }

    // 动态更新
    public String updateEmployee(final Employee employee) {

        return new SQL() {
            {
                UPDATE(EMPLOYEETABLE);
                if (employee.getName() != null) {
                    SET(" name = #{name} ");
                }
                if (employee.getCardId() != null) {
                    SET(" card_id = #{cardId} ");
                }
                if (employee.getAddress() != null) {
                    SET(" address = #{address} ");
                }
                if (employee.getPostCode() != null) {
                    SET(" post_code = #{postCode} ");
                }
                if (employee.getTel() != null) {
                    SET(" tel = #{tel} ");
                }
                if (employee.getPhone() != null) {
                    SET(" phone = #{phone} ");
                }
                if (employee.getQqNum() != null) {
                    SET(" qq_num = #{qqNum} ");
                }
                if (employee.getEmail() != null) {
                    SET(" email = #{email} ");
                }
                if (employee.getSex() != null) {
                    SET(" sex = #{sex} ");
                }
                if (employee.getParty() != null) {
                    SET(" party = #{party} ");
                }
                if (employee.getBirthday() != null) {
                    SET(" birthday = #{birthday} ");
                }
                if (employee.getRace() != null) {
                    SET(" race = #{race} ");
                }
                if (employee.getEducation() != null) {
                    SET(" education = #{education} ");
                }
                if (employee.getSpeciality() != null) {
                    SET(" speciality = #{speciality} ");
                }
                if (employee.getHobby() != null) {
                    SET(" hobby = #{hobby} ");
                }
                if (employee.getRemark() != null) {
                    SET(" remark = #{remark} ");
                }
                if (employee.getCreateDate() != null) {
                    SET(" create_Date = #{createDate} ");
                }
                if (employee.getDept() != null) {
                    SET(" dept_id = #{dept.id} ");
                }
                if (employee.getJob() != null) {
                    SET(" job_id = #{job.id} ");
                }
                WHERE(" id = #{id} ");
            }
        }.toString();
    }
}
```

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.SelectProvider;
import top.zhoulis.www.dao.provider.JobDynaSqlProvider;
import top.zhoulis.www.domain.Job;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.JOBTABLE;

public interface JobDao {
    /**
     * 根据id查询职位
     * @param id
     * @return
     */
    @Select("select * from " + JOBTABLE + " WHERE id = #{id}")
    Job selectById(int id);

    /**
     * 查询所有的职位
     * @return
     */
    @Select("select * from " + JOBTABLE + " ")
    List<Job> selectAllJob();

    /**
     * 动态查询
     * @param params
     * @return
     */
    @SelectProvider(type = JobDynaSqlProvider.class, method = "selectWhitParam")
    List<Job> selectByPage(Map<String, Object> params);
    @SelectProvider(type = JobDynaSqlProvider.class, method = "count")
    Integer count(Map<String, Object> params);

    /**
     * 根据id删除job
     * @param id
     */
    @Delete(" delete  from " + JOBTABLE + " where id = #{id} ")
    void deleteById(Integer id);

    /**
     * 动态插入job
     * @param job
     */
    @SelectProvider(type = JobDynaSqlProvider.class, method = "insertJob")
    void save(Job job);

    /**
     * 动态更新
     * @param job
     */
    @SelectProvider(type = JobDynaSqlProvider.class, method = "updateJob")
    void update(Job job);
}
```

```java
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.Job;

import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.JOBTABLE;

public class JobDynaSqlProvider {

    /**
     * 分页动态查询
     * @param params
     * @return
     */
    public String selectWhitParam(final Map<String, Object> params) {
        String sql = new SQL(){
            {
                SELECT("*");
                FROM(JOBTABLE);
                if (params.get("job") != null) {
                    Job job = (Job) params.get("job");
                    if (job.getName() != null && job.getName().equals("")) {
                        WHERE(" name like concat('%',#{job.name},'%') ");
                    }
                }
            }
        }.toString();
        if (params.get("pageModel") != null) {
            sql += " limit #{pageModel.firstLimitParam}, #{pageModel.pageSize} ";
        }
        return sql;
    }

    /**
     * 动态查询总数量
     * @param params
     * @return
     */
    public String count(final Map<String, Object> params) {
        return new SQL(){
            {
                SELECT("count(*)");
                FROM(JOBTABLE);
                if (params.get("job") != null) {
                    Job job = (Job) params.get("job");
                    if (job.getName() != null && job.getName().equals("")) {
                        WHERE(" name like concat('%',#{job.name},'%') ");
                    }
                }
            }
        }.toString();
    }

    /**
     * 动态插入
     * @param job
     * @return
     */
    public String insertJob(final Job job) {
        return new SQL(){
            {
                INSERT_INTO(JOBTABLE);
                if (job.getName() != null && job.getName().equals("")) {
                    VALUES("name", "#{name}");
                }
                if (job.getRemark() != null && job.getRemark().equals("")) {
                    VALUES("remark", "#{remark}");
                }
            }
        }.toString();
    }

    /**
     * 动态更新
     * @param job
     * @return
     */
    public String updateJob(final Job job) {
        return new SQL(){
            {
                UPDATE(JOBTABLE);
                if (job.getName() != null) {
                    SET(" name = #{name} ");
                }
                if (job.getRemark() != null) {
                    SET(" remark = #{remark} ");
                }
                WHERE(" id = #{id} ");
            }
        }.toString();
    }
}
```

```java
package top.zhoulis.www.dao;

import org.apache.ibatis.annotations.*;
import org.apache.ibatis.mapping.FetchType;
import top.zhoulis.www.dao.provider.NoticeDynaSqlProvider;
import top.zhoulis.www.domain.Notice;

import java.util.List;
import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.NOTICETABLE;

public interface NoticeDao {

    // 动态查询
    @SelectProvider(type=NoticeDynaSqlProvider.class,method="selectWhitParam")
    @Results({
            @Result(id=true,column="id",property="id"),
            @Result(column="CREATE_DATE",property="createDate",javaType=java.util.Date.class),
            @Result(column="USER_ID",property="user",
                    one=@One(select="org.fkit.hrm.dao.UserDao.selectById",
                            fetchType= FetchType.EAGER))
    })
    List<Notice> selectByPage(Map<String, Object> params);

    @SelectProvider(type= NoticeDynaSqlProvider.class,method="count")
    Integer count(Map<String, Object> params);

    @Select("select * from "+NOTICETABLE+" where ID = #{id}")
    Notice selectById(int id);

    // 根据id删除公告
    @Delete(" delete from "+NOTICETABLE+" where id = #{id} ")
    void deleteById(Integer id);

    // 动态插入公告
    @SelectProvider(type=NoticeDynaSqlProvider.class,method="insertNotice")
    void save(Notice notice);

    // 动态修改公告
    @SelectProvider(type=NoticeDynaSqlProvider.class,method="updateNotice")
    void update(Notice notice);

}
```

```java
package top.zhoulis.www.dao.provider;

import org.apache.ibatis.jdbc.SQL;
import top.zhoulis.www.domain.Notice;

import java.util.Map;

import static top.zhoulis.www.util.common.HrmConstants.NOTICETABLE;

public class NoticeDynaSqlProvider {
    // 分页动态查询
    public String selectWhitParam(final Map<String, Object> params){
        String sql =  new SQL(){
            {
                SELECT("*");
                FROM(NOTICETABLE);
                if(params.get("notice") != null){
                    Notice notice = (Notice)params.get("notice");
                    if(notice.getTitle() != null && !notice.getTitle().equals("")){
                        WHERE("  title LIKE CONCAT ('%',#{notice.title},'%') ");
                    }
                    if(notice.getContent() != null && !notice.getContent().equals("")){
                        WHERE("  content LIKE CONCAT ('%',#{notice.content},'%') ");
                    }
                }
            }
        }.toString();

        if(params.get("pageModel") != null){
            sql += " limit #{pageModel.firstLimitParam} , #{pageModel.pageSize}  ";
        }

        return sql;
    }
    // 动态查询总数量
    public String count(final Map<String, Object> params){
        return new SQL(){
            {
                SELECT("count(*)");
                FROM(NOTICETABLE);
                if(params.get("notice") != null){
                    Notice notice = (Notice)params.get("notice");
                    if(notice.getTitle() != null && !notice.getTitle().equals("")){
                        WHERE("  title LIKE CONCAT ('%',#{notice.title},'%') ");
                    }
                    if(notice.getContent() != null && !notice.getContent().equals("")){
                        WHERE("  content LIKE CONCAT ('%',#{notice.content},'%') ");
                    }
                }
            }
        }.toString();
    }
    // 动态插入
    public String insertNotice(final Notice notice){

        return new SQL(){
            {
                INSERT_INTO(NOTICETABLE);
                if(notice.getTitle() != null && !notice.getTitle().equals("")){
                    VALUES("title", "#{title}");
                }
                if(notice.getContent() != null && !notice.getContent().equals("")){
                    VALUES("content", "#{content}");
                }
                if(notice.getUser() != null && notice.getUser().getId() != null){
                    VALUES("user_id", "#{user.id}");
                }
            }
        }.toString();
    }
    // 动态更新
    public String updateNotice(final Notice notice){

        return new SQL(){
            {
                UPDATE(NOTICETABLE);
                if(notice.getTitle() != null && !notice.getTitle().equals("")){
                    SET(" title = #{title} ");
                }
                if(notice.getContent() != null && !notice.getContent().equals("")){
                    SET(" content = #{content} ");
                }
                if(notice.getUser() != null && notice.getUser().getId() != null){
                    SET(" user_id = #{user.id} ");
                }
                WHERE(" id = #{id} ");
            }
        }.toString();
    }
}
```

