---
title: MyBatis缓存
date: 2019-08-17 21:25:00
author: 周李
top: true
cover: true
toc: true
mathjax: false
categories: MyBatis
tags:
  - java
  - MyBatis
---

## MyBatis缓存

 #### 一级缓存

> 一级缓存（也叫本地缓存）默认会启用，而且不能控制

先看一个示例

``` java
public class CacheTest extends BaseMapperTest {

    @Test
    public void testL1Cache() {
        SqlSession sqlSession = getSqlSession();
        SysUser user1 = null;
        try {
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            user1 = userMapper.selectById(1l);
            user1.setUserName("New Name");
            SysUser user2 = userMapper.selectById(1l);
            Assert.assertEquals("New Name",user2.getUserName());
            Assert.assertEquals(user1,user2);
            System.out.println(user1 == user2);
            System.out.println(user1.equals(user2));
        }finally {
            sqlSession.close()
        }
        System.out.println("开启新的sqlSession");
        sqlSession = getSqlSession();
        try {
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            SysUser user2 = userMapper.selectById(1l);
            Assert.assertNotEquals("New Name",user2.getUserName());
            Assert.assertNotEquals(user1,user2);
            userMapper.deleteUserById(2L);
            SysUser user3 = userMapper.selectById(1l);
            Assert.assertNotEquals(user2,user3);
            System.out.println(user1 == user2);
            System.out.println(user1.equals(user2));
        }finally {
            sqlSession.close();
        }
    }
}

```

测试该代码输出日志，结合日志分析代码

``` 
DEBUG [main] -==>  Preparing: select *from sys_user where id = ? 
DEBUG [main] -==> Parameters: 1(Long)
TRACE [main] -<==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] -<==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-06-07 01:11:12
DEBUG [main] -<==      Total: 1
true
true
开启新的sqlSession
DEBUG [main] -==>  Preparing: select *from sys_user where id = ? 
DEBUG [main] -==> Parameters: 1(Long)
TRACE [main] -<==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] -<==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-06-07 01:11:12
DEBUG [main] -<==      Total: 1
DEBUG [main] -==>  Preparing: {call delete_user_by_id(?)} 
DEBUG [main] -==> Parameters: 2(Long)
DEBUG [main] -<==    Updates: 0
DEBUG [main] -==>  Preparing: select *from sys_user where id = ? 
DEBUG [main] -==> Parameters: 1(Long)
TRACE [main] -<==    Columns: id, user_name, user_password, user_email, user_info, head_img, create_time
TRACE [main] -<==        Row: 1, admin, 123456, admin@mybatis.tk, <<BLOB>>, <<BLOB>>, 2016-06-07 01:11:12
DEBUG [main] -<==      Total: 1
false
false

```

​	可以看出在第一次执行selectById方法获取SysUser数据时，真正的执行了数据库的查询，得到了user1的结果，第二次执行获取user2的时候，可以看出只有一次查询 ，也就是第二次并没有执行数据库操作 。 也就是说明了user1和user2**指向同一处地址 也就是同一个对象**。即user1改变会影响到user2的改变



Mybatis的一级缓存存在于SqlSession的生命周期中，在同一个SqlSession中查询时，会把执行的方法和参数通过算法生成缓存的键值，存入到Map对象中。 反复执行同一个方法时，总是返回**同一个对象**，因此就会出现user1的重新赋值会影响到user2. 在使用中要注意避免出现如上错误。 那要怎么解决这个问题呢？

``` java
    <select id="selectById" resultMap="userMap" flushCache="true">
            select * from sys_user where id = #{id}
    </select>
```

加上**flushCache=“true”**，这个属性后会清空当前一级缓存。因此每次都会重新从数据库中查询数据，此时的user2和user1就会成为两个不同的实例了。**一级缓存只存在于sqlsession的生命周期中。 insert，update，delete操作都会清空一级缓存**

#### 二级缓存

> MyBatis的二级缓存不同于一级缓存只存在于SqlSession的生命周期中，而是理解为存在于SqlSessionFactory的生命周期中。

##### 配置二级缓存

​	在MyBatis的全局配置settings中有一个参数cacheEnabled，这个参数是二级缓存的全局开关，默认值是true。所以不用配置，如果想要配置，可以在mybatis-config.xml中添加一下代码。

``` 
 <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
```

MyBatis的二级缓存是和命名空间绑定的，即要配置在Mapper.xml映射文件中，或配置在Mapper.java接口中。只需在UserMapper.xml中添加<cache/>元素即可。

**默认的二级缓存会有如下效果。**

- 映射语句文件中的所有 SELECT 语句将会被缓存 。

- 映射语句文件中的所有 时SERT 、 UPDATE 、 DELETE 语句会刷新缓存 。
  缓存会使用 Least Recently Used ( LRU，最近最少使用的）算法来收回 。

- 根据时间表（如 no Flush Interval ，没有刷新间隔），缓存不会以任何时间顺序来刷新 。
  缓存会存储集合或对象（无论查询方法返回什么类型的值）的 1024 个引用。

- 缓存会被视为 read/write （可读／可写）的 ， 意味着对象检索不是共享的，而且可以安全
  地被调用者修改，而不干扰其他调用者或线程所做的潜在修改 。

  所有的这些属性都可以通过缓存元素的属性来修改：

  ```
  <cache
  eviction＝ ” FIFO ”  //FIFO(先进先出) LRU(最近最少使用) SOFT(软引用) WEAK(弱引用)
  flushlnterval=” 60000”
  size=” 512 ”
  readOnly=” true " />  
  ```


##### 使用二级缓存

​	先写一个测试（要求pojo需要实现Serializable接口）

```java
@Test
    public void test2Cache(){
        SqlSession sqlSession = getSqlSession();
        SysRole role1= null;
        try {
            RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
            role1 = roleMapper.selectById(1l);
            role1.setRoleName("New Name");
            SysRole role2 = roleMapper.selectById(1l);
            Assert.assertEquals("New Name", role2.getRoleName());
            Assert.assertEquals(role1,role2);
        }finally {
            sqlSession.close();
        }
        System.out.println("开启新的sqlSession");
        sqlSession = getSqlSession();
        try {
            RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
            SysRole role2 = roleMapper.selectById(1l);
            Assert.assertEquals("New Name",role2.getRoleName());
            Assert.assertNotEquals(role1,role2);
            System.out.println(role1.getRoleName() + "------" + role2.getRoleName());
            SysRole role3 = roleMapper.selectById(1l);
            Assert.assertNotEquals(role2,role3);
            System.out.println(role2.getRoleName() + "------" + role3.getRoleName());
        }finally {
            sqlSession.close();
        }
    }
```

执行测试输出日志，内容如下

```
DEBUG [main] -Cache Hit Ratio [tk.mybatis.simple.mapper.RoleMapper]: 0.0
DEBUG [main] -==>  Preparing: select *from sys_role where id = ? 
DEBUG [main] -==> Parameters: 1(Long)
TRACE [main] -<==    Columns: id, role_name, enabled, create_by, create_time
TRACE [main] -<==        Row: 1, 管理员, 1, 1, 2016-04-01 17:02:14
DEBUG [main] -<==      Total: 1
DEBUG [main] -Cache Hit Ratio [tk.mybatis.simple.mapper.RoleMapper]: 0.0
开启新的sqlSession
DEBUG [main] -Cache Hit Ratio [tk.mybatis.simple.mapper.RoleMapper]: 0.3333333333333333
New Name------New Name
DEBUG [main] -Cache Hit Ratio [tk.mybatis.simple.mapper.RoleMapper]: 0.5
New Name------New Name
```

在第一个try块中。在一个sqlsession中所以是一级缓存。当调用close方法关闭sqlsession时才会保存查询数据到二级缓存中。

在第二个try块中，再次获取role2时，日志并没有输出数据库查询，而是输出了命中率0.333333333333。这是第3次查询，并且得到了缓存值，一次被命中，所以命中率为三分之一。后面以此类推。



> ***Mybatis默认提供缓存实现是基于Map实现的内存缓存。所以当需要缓存大量数据时就要选择一些类似Ehcache的框架或Redis缓存来保存二级缓存数据了。***

