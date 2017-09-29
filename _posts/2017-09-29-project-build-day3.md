---
layout : post
title : "项目搭建问题（三）"
category : JavaWeb
tags : 项目实战 Spring mybatis
---
* content
{:toc}

　　今天完成了项目中的登录功能，用Mybatis去完成了一些基本的查询功能，然后在项目启动的过程中，主要碰到了两个问题。一个是Mybatis中的dao实例化bean失败，一个是mybatis-config.xml文件的配置要求。




#### 实例化dao失败

　　贴一点关键的错误信息：

```
No qualifying bean of type [com.xxx.mybatis.dao.IXXXDao] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

　　要实例化的IXXXDao如下：

```java
public interface IXXXDao {

	public List<XXX> queryList(XXX xxx);
	public XXX queryXXXById(long id);
}
```

　　错误原因并不在此，而是在applicationContext.xml中的一段内容：

```xml
<bean id="sqlSessionFactory_feiniu" class="org.mybatis.spring.SqlSessionFactoryBean"> 
		<property name="dataSource" ref="dataSource"/> 
		<property name="configLocation" value="classpath:mybatis-config.xml" />
		<property name="mapperLocations" value="classpath*:com/xxx/mybatis/xml/*.xml" />
	</bean>
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property> 
		<property name="basePackage" value="com.xxx.mybatis.dao" />	 
 	</bean>
```

　　错误原因就在于我将mapperLactions和basePackage对应的value的路径给写错了。那么查错的过程中，排错的过程大致如下：

　　1. 类上面缺少@Repository或者@Component这样标识的注解，但是dao是接口，接口不会被注释也不会生成bean。

　　2. 缺少```xml<context:annotation-config /><context:component-scan base-package="com.xxx" />```。applicationContext.xml中存在该配置，并且dao是接口，与该配置没关系。

　　3. 使用了Mybatis去生成接口dao的相应bean，必须配置正确的文件路径。

#### mybatis-config配置顺序

　　在mybatis-config.xml文件中使用了别名配置，以免在dao的mapper.xml文件中写类的全名。报错如下：

```
The content of element type "configuration" must match "(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".
```

　　百度之后说是mybatis-config.xml文件对标签的配置也有要求，而顺序就是上面括号中的排列顺序。

### 参考

[expected at least 1 bean which qualifies as autowire candidate for this dependency. ](http://blog.csdn.net/zhanghe687/article/details/71214166)

[SSM整合出现not found for dependency: expected at least 1 bean which qualifies as autowire错误](http://blog.csdn.net/grit_icpc/article/details/58141424)

[mybatis:"configuration" must match "(properties?,settings?,typeAliase..... ](http://blog.csdn.net/zhaifengmin/article/details/44707351)

[Mybatis MapperScannerConfigurer 自动扫描 将Mapper接口生成代理注入到Spring](http://www.cnblogs.com/daxin/p/3545040.html)