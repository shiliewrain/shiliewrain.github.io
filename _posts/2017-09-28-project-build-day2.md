---
layout : post
title : "项目搭建问题（二）"
category : JavaWeb
tags : 项目实战 buildNumber jdk1.5
---
* content
{:toc}

　　今天碰到了两个问题，一个是buildNumber，另外一个是项目update之后jdk变为1.5。




### pom文件内容

　　今天弄的依然是pom文件中的标签，首先是不太了解buildNumberPropertyName标签的含义，另外就是在update项目之后，发现其中一个项目的jdk变为了1.5，进而项目报错。

#### buildNumberPropertyName

　　pom中有如下一段：

```xml
<plugins>
		<plugin>
			<groupId>org.codehaus.mojo</groupId>
			<artifactId>buildnumber-maven-plugin</artifactId>
			<version>1.3</version> 
			<executions>
				<execution>
					<phase>validate</phase>
					<goals>
						<goal>create</goal>
					</goals>
				</execution>
			</executions>
			<configuration>
				<format>{0,date,yyyy-MM-dd HH:mm:ss}</format>
				<items>
					<item>timestamp</item>
				</items>
				<buildNumberPropertyName>current.time</buildNumberPropertyName>
			</configuration>
		</plugin>
	</plugins>
```

　　我刚开始不知道buildNumberPropertyName标签的作用，但其中的current.time是在其他文件中引用的值。不同于properties中直接使用标签作为key，这里的current.time是key，而这个current.time对应的value则为format和items组合而得到的值。在format标签中，大括号中的内容是会被item替换的。大括号中的第一位是一个数字，代表选择第几个item，从0开始。第二位是数据类型，第三位是数据格式，这两位都是可选的。那么以上一段配置表示在项目进行validate的时候，执行buildnumber插件的create目标（貌似可以直白理解为执行buildNumber中的create方法），将current.time替换为格式为“yyyy-MM-dd HH:mm:ss”的当前时间。

　　那么这个buildNumber插件就是用来为每次项目构建时，根据自己的需要定制规则，生成相应的标识。可能主要的目的是为每次的项目构建生成一个唯一标识，比如版本号。值得注意的是，该插件要与scm插件一起使用，我将pom中的scm配置删除，再次构建项目就会出错。至于是否有方法可以单独使用buildNumber插件，后面再说了。

#### jdk变为1.5

　　在使用Maven update项目之后，其中一个项目的jdk由1.7变为了1.5，造成项目报错。去网上查找，发现该问题挺常见的。产生问题的原因，我就直接贴在下面了。

> Maven官方文档有如下描述：
>
> 　　编译器插件用来编译项目的源文件.从3.0版本开始, 用来编译Java源文件的默认编译器是javax.tools.JavaCompiler (如果你是用的是java 1.6) . 如果你想强制性的让插件使用javac,你必须配置插件选项 forceJavacCompilerUse. 同时需要注意的是目前source选项和target 选项的默认设置都是1.5, 与运行Maven时的JDK版本无关.如果你想要改变这些默认设置, 可以参考 Setting the -source and -target of the Java Compiler中的描述来设置 source 和target 选项。
>
> 　　这是Maven已知的一个特性。除非在你的POM文件中显示的指定一个版本，否则会使用编译器默认的source/target版本1.5。主要还是在于Eclipse中Maven的集成方式起到了关键作用, 它会从POM文件中生成项目的.project,.classpath以及.settings, 因此除非POM文件指定了正确的JDK版本, 否则你每次更新项目配置的时候它都会重置到1.5版本。

　　如果使用的Maven版本为3.0，那么有三种方式来解决。

*1. 在项目的pom文件中使用maven-compiler-plugin插件，配置如下：

```xml
<build>
		<plugins>
		    <plugin>
		        <groupId>org.apache.maven.plugins</groupId>
		        <artifactId>maven-compiler-plugin</artifactId>
		        <version>3.5.1</version>
		        <configuration>
		            <source>1.7</source>
		            <target>1.7</target>
		        </configuration>
		    </plugin>
		</plugins>
	</build>
```

*2. 在项目的pom文件中加入如下属性：

```xml
<properties>  
      <maven.compiler.source>1.7</maven.compiler.source>  
      <maven.compiler.target>1.7</maven.compiler.target>
</properties>
```

*3. 修改setting文件，加入或者修改其中关于jdk的配置：

```xml
<profile>
      <id>jdk-1.7</id>
      <activation>
        <jdk>1.7</jdk>
      </activation>
      <properties>
            <maven.compiler.source>1.7</maven.compiler.source>
            <maven.compiler.target>1.7</maven.compiler.target>
            <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
      </properties>
</profile>
```

### 参考

[Maven插件之buildnumber](http://www.360doc.com/content/13/1028/17/9318309_324837708.shtml)

[解决maven update project 后项目jdk变成1.5的问题](http://www.jb51.net/article/98174.htm)

[eclipse中解决update maven之后jre被改成1.5的问题](http://www.cnblogs.com/ScvQ/p/6880066.html)