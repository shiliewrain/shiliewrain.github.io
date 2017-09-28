---
layout : post
title : "项目搭建问题（一）"
category : JavaWeb
tags : 项目实战 resources scm
---
* content
{:toc}


　　在公司第一次自己搭建真实项目，不详细记录其中的过程，只记录其中的问题。




### pom文件内容

　　使用maven直接构建了一个JavaWeb项目，然后在pom文件中写一些依赖和插件。因为和平时的学习练手的项目还是有很大的区别，所以参照了公司的另外一个项目。

#### resources

```xml
<resources>
		<resource>
			<filtering>true</filtering>
			<directory>src/main/resources</directory>
			<includes>
				<include>checksrv.properties</include>
			</includes>
		</resource>
		<resource>
			<filtering>false</filtering>
			<directory>src/main/resources</directory>
			<includes>
				<include>**/*.xml</include>
				<include>**/*.properties</include>
			</includes>
		</resource>
		<resource>
			<filtering>false</filtering>
			<directory>src/main/java</directory>
			<includes>
				<include>**/*.xml</include>
				<include>**/*.properties</include>
			</includes>
		</resource>
	</resources>
```

　　虽然从标签名大概可以看出含义，但不太确定。查阅过后，知道这些标签是针对项目中的配置文件的。resources包含所有的资源（resource），而每个resource下面都包含自己的信息。

* filtering表示是否过滤。

* directory表示文件夹。

* includes中的include表示包含的文件

* excludes中的exclude表示不包含的文件

　　那上面配置中第一个resource的含义即为过滤掉src/main/resources中的checksrc.properties文件。第二个resource即为不过滤该文件夹中的xml和Properties文件。这两个resource就把除了checksrc.properties以外的文件都包含了，然后替换掉checksrc.properties中的所有${key}这样的值，而替换这些key的值就在pom文件的properties中。

#### scm

　　scm（software configuration management），软件配置管理，了解不多，在我看来是一个支持svn、cvs等软件的插件，便于我们使用类似scm:branch、scm:update等命令进行版本控制。这里直接copy一份说明吧。

```xml
<!--SCM(Source Control Management)标签允许你配置你的代码库，供Maven web站点和其它插件使用。-->     
    <scm>     
        <!--SCM的URL,该URL描述了版本库和如何连接到版本库。欲知详情，请看SCMs提供的URL格式和列表。该连接只读。-->     
        <connection>     
            scm:svn:http://svn.baidu.com/banseon/maven/banseon/banseon-maven2-trunk(dao-trunk)      
        </connection>     
        <!--给开发者使用的，类似connection元素。即该连接不仅仅只读-->    
        <developerConnection>     
            scm:svn:http://svn.baidu.com/banseon/maven/banseon/dao-trunk      
        </developerConnection>    
        <!--当前代码的标签，在开发阶段默认为HEAD-->    
        <tag/>           
        <!--指向项目的可浏览SCM库（例如ViewVC或者Fisheye）的URL。-->     
        <url>http://svn.baidu.com/banseon</url>     
    </scm>
```

### 参考

[maven的resources介绍](http://jjhpeopl.iteye.com/blog/2325375)

[maven POM.xml 标签详解](http://blog.csdn.net/sunzhenhua0608/article/details/32938533)

[maven的scm插件介绍及使用示例](http://blog.csdn.net/fenglibing/article/details/16842645)

[利用maven的resources、filter和profile实现不同环境使用不同配置文件](http://zhaoshijie.iteye.com/blog/2094478)