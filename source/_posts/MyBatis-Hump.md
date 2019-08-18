---
title: MyBatis拦截器之下画线键值转驼峰
date: 2019-08-18 20:47:00
author: 周李
top: true
cover: true
mathjax: false
categories: MyBatis
tags:
  - java
  - MyBatis
---

# MyBatis拦截器之下画线键值转驼峰

​		在使用MyBatis时，为了方便扩展而使用Map类型的返回值。使用Map作为返回值时，Map中的键值就是查询结果中的列名，而列名一般都是大写小写字母或者下画线形式，和Java中使用的驼峰形式不一致，为了保证在使用map时属性一致，可以对结果进行特殊处理------使用拦截器将不同格式的列名转换为Java中的驼峰形式。实现代码如下：

``` java
package tk.mybatis.simple.plugin;


import org.apache.ibatis.executor.resultset.ResultSetHandler;
import org.apache.ibatis.plugin.*;

import java.sql.Statement;
import java.util.*;

/**
 * MyBatis Map类型下划线key转小写驼峰形式
 *
 * @author zhouli
 */
@Intercepts(   //ResultSetHandler接口方法签名。在查询方法中被调用
        @Signature(
                type = ResultSetHandler.class,
                method = "handleResultSets",
                args = {Statement.class})
)
@SuppressWarnings({"unchecked", "rawtypes" })
public class CameHumpInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        List<Object> list = (List<Object>) invocation.proceed();
        for (Object object : list) {
            //如果时map类型，就对map的key转换
            if (object instanceof Map) {
                processMap((Map)object);
            } else {
                break;
            }
        }
        return list;
    }

    /**
     * 处理Map类型
     * @param map
     */
    private void processMap(Map<String, Object> map) {
        Set<String> keySet = new HashSet<>(map.keySet());
        for (String key : keySet) {
            //将以大写开头的字符串转换为小写，如果不包含下划线也会处理为驼峰
            //此处只通过这两个简单的标识来判断是否转换
            if ((key.charAt(0) >= 'A' && key.charAt(0) <= 'Z')
                    || key.indexOf("_") >= 0) {
                Object value = map.get(key);
                map.remove(key);
                map.put(underlineToCamelhump(key), value);
            }
        }
    }

    /**
     * 将下划线风格替换为驼峰风格
     * @param inputString
     * @return
     */
    public static String underlineToCamelhump(String inputString) {
        StringBuilder sb = new StringBuilder();
        boolean nextUpperCase = false;
        for (int i = 0; i < inputString.length(); i++) {
            char c = inputString.charAt(i);
            if (c == '_') {
                if (sb.length() > 0) {
                    nextUpperCase = true;
                }
            } else {
                if (nextUpperCase){
                    sb.append(Character.toUpperCase(c));
                }else {
                    sb.append(Character.toLowerCase(c));
                }
            }
        }
        return sb.toString();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }
}

```

这个插件很简单，就是循环判断结果。如果是Map类型的结果，就对Map的key进行处理。当数据经过这个拦截器插件处理后，就可以保证在任何数据库中以Map作为结果值类型时，都有一致的key值，可以统一取值。想要使用该插件需要在mybatis-config.xml中配置该插件。

```
//拦截器配置
<plugins>
  <plugin interceptor="该拦截器的全限定类名"/>
</plugins>

```

在上面拦截器代码的26行，**invocation.proceed()执行的结果被强制转换为List类型**。这是因为拦截器接口**ResultSetHandler**的**handleResultSets**方法的返回值为List类型。写拦截器时，**要根据被拦截的方法来确定返回值的类型。**