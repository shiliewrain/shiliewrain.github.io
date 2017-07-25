---
layout : post
title : "自定义注解示例"
category : Java
tags : 注解 自定义
---

* content
{:toc}

　　最近看了Java视频的自定义注解介绍，然后根据视频的示例将代码简单copy了一遍。写代码的过程中一直在回忆视频中的内容，这里也记一些基本的东西加强记忆。




### 自定义Java注解

　　为了将数据库的表和实体类联系起来，我们在这里需要定义两个注解去表示table和Column。自定义Java注解的内容比较多，但是知道一些简单的东西就足够应用了。先看下面的示例Table：

```java
package com.shiliew.function.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {

    String value();
}
```

　　使用@interface去声明一个自定义注解，然后使用一些meta-annotation（元注解）去说明该自定义注解的一些属性。@Target用来表示该自定义注意的作用类型，它接收一个数组对象为参数，实际应该这样表示：@Target({ElementType.TYPE})。ElementType的枚举值包括：CONSTRUCTOR（用于描述构造器）、FIELD（用于描述域）、LOCAL_VARIABLE（用于描述局部变量）、METHOD（用于描述方法）、PACKAGE（用于描述包）、PARAMETER（用于描述参数）和TYPE（用于描述类、接口、注解或enum声明）。而@Retention(RetentionPolicy.RUNTIME)表示该自定义注解的生命周期会持续到程序运行时，另外RetentionPolicy还有两个枚举值：SOURCE:在源文件中有效（即源文件保留）；CLASS:在class文件中有效（即class保留）。这里没有写另外两个元注解@Documented和@Inherited，因为不需要。

　　在自定义注解中，只能声明public或者default的成员函数，实际上这就是其接收的参数，而返回类型就是其接收参数的类型。如果该自定义注解只接收一个参数，最好将方法名设置为value。自定义注解支持的数据类型包括所有基本的数据类型以及String、Class、enum、Annotation和这所有类型的数组。

　　按之前的写法再声明一个Column注解，如下：

```java
package com.shiliew.function.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {

    String value();
}
```

### 带自定义注解的实体类

　　在这里，我们并没有真正去实现一个数据库的表，模拟一下情景即可。定义一个user表，表名为user，包含三个字段：name、sex和age。然后定义User实体，String类型的name和sex，Integer类型的age。

```java
package com.shiliew.function.annotation;

@Table("user")
public class User {

    @Column("name")
    private String name;
    @Column("sex")
    private String sex;
    @Column("age")
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

### 测试

　　根据实体类对象带条件去生成对应的SQL查询语句，生成SQL的方法要是通用的，能自动根据实体类中的Table和Column注解去解析获得表中的字段。代码如下：

```java
package com.shiliew.function.annotation;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class MainTest {

    public static void main(String [] args){
        User user = new User();
        user.setName("Shiliew");
        user.setAge(26);
//        user.setSex("男");
        System.out.println(query(user));
    }

    public static String query(Object obj){
        StringBuilder sb = new StringBuilder();
        Class clazz = obj.getClass();
        boolean exist = clazz.isAnnotationPresent(Table.class);
        if(exist) {
            //获取表名
            String tableName = ((Table) clazz.getAnnotation(Table.class)).value();
            sb.append("select * from ").append(tableName).append(" where 1=1");
            //获取列名
            Field[] fields = clazz.getDeclaredFields();
            for (Field f : fields) {
                boolean fExist = f.isAnnotationPresent(Column.class);
                if (fExist) {
                    String columnName = f.getAnnotation(Column.class).value();
                    String methodName = "get" + f.getName().substring(0, 1).toUpperCase() + f.getName().substring(1);
                    try {
                        Method method = clazz.getDeclaredMethod(methodName);
                        Object object = method.invoke(obj);
                        if (object != null) {
                            sb.append(" and ").append(columnName).append("=");
                            if (object instanceof String) {
                                sb.append("'").append(object).append("'");
                            } else if (object instanceof Integer) {
                                sb.append(object);
                            }
                        }
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        return sb.toString();
    }

}
```

　　大致的原理是先根据实体类的对象获得其Class对象，判断其是否有Table注解。如果有，则获得该注解中的value值，通过StringBuilder去拼装。然后获得该对象中的所有属性，并获得其中有Column注解的注解value。拼装其get方法，利用反射机制执行获得该属性的值，将值不为null或者空串的属性作为条件进行SQL拼装。需要注意的地方是要判断通过instanceof去判断获得的属性值的类型，因为在SQL语句中，字符串类型的数据需要加单引号，而整型不需要。最后将拼装完的SQL语句返回，即完成最初的要求。

```java
select * from user where 1=1 and name='Shiliew' and age=26
```

### 总结

　　以上的内容是Java自定义注解最简单的使用，掺入了一点[Java反射](https://shiliewrain.github.io/2017/05/11/java-reflect-proxy/)的内容。在实际工作中如果需要使用到更高级的用法，则可以根据具体情况再去查阅相关资料。总的来说，通过一些如拦截的手段获得某些类的对象后，判断其是否含有特定的注解来处理不同的逻辑。

### 参考

[深入理解Java：注解（Annotation）自定义注解入门](http://www.cnblogs.com/peida/archive/2013/04/24/3036689.html)

[JAVA自定义注解](http://www.cnblogs.com/zenghong/p/3919332.html)