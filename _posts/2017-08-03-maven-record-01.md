---
layout : post
title : "Maven学习笔记（一）"
category : 后台
tags : Maven 
---

* content
{:toc}


　　最近在看《Maven实战》，全书有近400面。估计自己并不需要对Maven研究得那么深，只需要了解一些核心概念和常规用法即可，于是粗略地过了一遍，记录一下。



### Maven简介

　　Maven是一个项目管理工具，能帮助你完成项目从构建、编译、测试、打包到发布的一系列任务。它能够通过pom.xml文件很好地帮你解决去各个不同的网站下载依赖jar包的问题，这是我最主要的感受。至于其他的功能，因为并没有体会过使用Maven之前各种操作的繁琐，所以也并未对其带来的便利有多深的体会。当然，作为目前主流的项目管理工具，Maven自然是十分强大的。

　　作为一个纯使用者，我觉得在平时工作中能够读懂pom.xml文件、了解仓库的概念和熟练使用clean、package、install及deploy这几个命令就够了。如果有更多的兴趣和时间，也可以去更深入研究Maven。

### pom.xml文件

　　在pom.xml文件中，最重要的两个概念就是坐标和依赖。还有其他如：生命周期、继承、插件等概念可以去阅读《Maven实战》。

#### 坐标

　　每一个Maven项目都有其自己的坐标，这是被其他Maven项目所依赖的根本。来看下面一段代码：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.fire.light</groupId>
  <artifactId>light-demo</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>light-demo Maven Webapp</name>
  <url>http://maven.apache.org</url>
</project>
```

　　一个最简单的pom.xml文件就如上所示，里面标明坐标的三个标签为：groupId、artifactId和version。

* groupId：定义当前Maven项目所属的实际项目。例如一个公司名为“fire”，其拥有一个实际项目名为“light”，那就可以定义此Maven项目的groupId为“com.fire.light”。

* artifactId：定义实际项目中的一个Maven项目。例如在实际项目light中有一个名为“demo”的Maven项目，则定义次Maven项目的artifactId为“light-demo”。

* version：定义Maven项目当前的版本，含有“SNAPSHOT”的代表快照版本，即不稳定版本。而纯数字或者含有“released”代表稳定版本。

　　在Maven中，约定大于配置。关于上面的关于项目坐标的配置也都是约定俗成的，你也可以不按照约定去配置，坏处可能就是没那么直观表示该Maven项目的身份和版本状态。另外，packaging这个标签也需要注意一下，它表示Maven项目打包的方式，常用的有war、jar和pom，更多详细的内容可以自行百度。

#### 依赖

　　依赖是基于坐标的，只有知道了坐标，才能获得该依赖。来看下面的代码：

```xml
<dependencies>
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
  </dependency>
</dependencies>
```

　　在一个dependencies中，含有一个或多个dependency，每一个dependency代表了一个项目依赖。只需要把需要依赖的包的groupId、artifactId和version写入，即写入需要依赖的包的坐标，Maven就能自动去引入该依赖。如果要去寻找某个依赖包的坐标，可以去Maven的中央仓库去找，[传送门在此](https://mvnrepository.com/)。另外，scope标签表示该依赖包的作用范围，test表示只在测试的时候生效，compile表示只作用于编译时。其它更详细的内容可以自行百度。

#### 仓库

　　仓库就是Maven用来存在依赖包的地方。仓库分为两类：本地仓库和远程仓库。当你在pom.xml文件中声明依赖的时候，Maven会根据你所写入的坐标去本地仓库里寻找，若找不到，则会去远程仓库寻找，若还是找不到则报错。安装Maven时，如果没有修改过默认的仓库位置，则可以在${user.home}/.m2/中找到setting.xml的文件，通过修改该文件中的&lt;localRepository&gt;中的值，可以修改默认本地仓库的位置。在&lt;repository&gt;中可以声明远程仓库的地址，而&lt;repositories&gt;中可以包含多个&lt;repository&gt;，这说明可以声明多个远程仓库。而远程仓库又可以分为中央仓库、私服和其他公共仓库。私服的主要作用是为了降低中央仓库的负载、节省自己的外网带宽、加速Maven的构建。

　　Maven根据pom.xml文件中声明的依赖先去本地仓库中寻找，找到则引入。若找不到，则去远程仓库寻找，找到了则下载到本地仓库，然后引入。若还找不到，则报错。而自己的打包的Maven项目也可以通过install安装到本地仓库，通过deploy安装到远程仓库。当然关于远程仓库的声明和认证请自行百度。

#### 常用的Maven命令

　　在平常的工作中，最常用的命令可能也就clean、package、test、install和deploy这几个，我目前用得最多的也就clean、package和deploy这三个命令。

* clean：清空项目中的target目录中的内容。

* package：给项目打包。

* deploy：将项目部署到远程仓库。

* install：将项目部署到本地仓库。

　　Maven中的所有命令是按照生命周期来执行的，分为三段：clean、default和sit。执行package会把本段内在其之前的所有命令都执行一遍。比如在default阶段，test在package之前，则执行package时，也会执行test。当然，test是一个可以跳过的步骤，还有一些步骤是不能跳过的，如compile。

### 随笔

　　其实还想写点关于Maven自定义插件的内容，但因为没有理解够，所以感觉也没太大意义。记住几个重点：Maven所有的生命周期中的阶段都和Maven插件绑定了，这些插件能完成相应的功能，如compile插件绑定在了compile阶段。自定义插件中三个重要点：继承AbstractMojo、提供@goal、实现execute()方法。记下这一段代码供自己回忆：

```xml
<build>
    <finalName>webTest-demo</finalName>
    <plugins>
      <plugin>
        <groupId>com.shiliew</groupId>
        <artifactId>maven.plugins.countsPlugins</artifactId>
        <version>1.0-SNAPSHOT</version>
        <executions>
          <execution>
          <!--绑定到编译阶段-->
            <phase>compile</phase>
            <goals>
              <goal>count</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

### 总结

　　关于Maven的使用，知道以上的内容，就足够应付平时的工作了。而平时工作更多的内容贴近业务，对于知识技术的学习深度就要自行取舍了。而把适合的技术应用到对应的业务中，才是最好的程序员。


