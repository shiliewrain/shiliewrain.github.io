---
layout : post
title : "maven项目多环境配置"
category : Maven
tags : maven 多环境 配置文件
---
* content
{:toc}

　　好久没有写了，那...就随便写点吧。



### maven项目获取配置

　　我接触的有两种方式，一种是在tomcat配置中声明，一种是在pom文件中声明。最终都是由Properties这个类读取其中的配置，因为Properties这个类继承至HashTable，所以能明白为什么配置中都是键值对。

#### 在tomcat中声明配置

　　在tomcat的配置中加入```-Dglobal.config.path="{配置文件路径}"```，那么tomcat启动之后，在系统的环境变量中就会出现global.config.path这个变量。通过System.getProperty("global.config.path")读取该变量，就能获得配置文件的路径。然后就该文件转为文件输入流，通过Properties类的load(inputStream)方法就能将配置的变量放入内存中读取。

![tomcat中声明配置](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/tomcat_var_config.png?raw=true)

#### 在pom中声明配置

　　在pom文件中，有profiles标签，里面可以定义若干个profile标签。profile标签可以定义任何属性，在这里我就只说定义环境变量了。每个profile标签都有一个id子标签，值唯一。多环境配置示例代码如下：

```xml
<profile>
			<id>dev</id>
			<properties>
				<config.path>classpath:config/dev</config.path>
			</properties>
		</profile>
		<profile>
			<id>beta</id>
			<properties>
				<config.path>file:/home/webdata/project/webroot/config</config.path>
			</properties>
		</profile>
		<profile>
			<id>preview</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<properties>
				<config.path>file:/home/webdata/project/webroot/config</config.path>
			</properties>
		</profile>
```

　　通过定义```<activeByDefault>true</activeByDefault>```可以指定默认的激活环境配置，通过获取```<config.path>```的值可以获得配置文件路径。剩下的读取文件就和上面一样了。值得一提，如果在pom文件中修改了配置，记得需要update一下，否则配置不生效。

　　总的来说，配置环境的读取是多路径，同入口。我可以通过很多种方式获得配置文件，然后将其通过Properties类加载，此时就可以在程序中通过getProperty方法获得相应的配置。

### maven项目启动

　　除了平时将项目部署到tomcat中，然后启动tomcat去启动项目之外，还可以通过maven的方式去启动项目。我用的不多，只晓得设置启动命令：tomcat:run (-P[id])。-P表示激活某个profile，id就是该profile的id。关于其他的一些设置，可以去参考一些资料。我看到文章中说需要在pom中设置tomcat插件，但我去掉该插件一样可以，猜想是因为启动时候使用了eclipse中的配置的tomcat。

### 参考

[Maven 进行多环境配置，使用profile文件进行配置](http://www.cnblogs.com/guoqingqu/p/4530438.html)

[使用maven profile实现多环境可移植构建](http://blog.csdn.net/mhmyqn/article/details/24501281)

[Java 读写Properties配置文件](http://www.cnblogs.com/xudong-bupt/p/3758136.html)

[在Eclipse中用Maven创建Web工程（tomcat:run 启动）](http://blog.csdn.net/shenhaiwen/article/details/68923192)
