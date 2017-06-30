---
layout : post
title : "Linux配置JDK和Tomcat"
category : Linux
tags : Linux JDK Tomcat
---

* content
{:toc}


　　先说点题外话，从2016年7月1日正式开始工作到今天2017年6月30号，正好是一整年，巧得是去年7月1号是周五，而今天也是周五。其实也没啥可总结的，可能有进步，也可能退步了，简单点，keep smile！





### Linux查看版本

　　呈呈大神说新手最好使用ubuntu这个版本，那就选这个版本吧。那查看自己使用的Linux版本的呢？查看内核版本可用：(1)cat /proc/version;(2)uname -a;(3)uname -r。查看Linux版本可用：(1)lsb_release -a;(2)cat /etc/issue。如下图：

![查看Linux内核和版本]()

　　当然，Linux博大精深，肯定还有很多其他的方式可以查询，而关于命令的参数，可以使用man命令自己查看一下。

### Linux配置JDK

　　先使用'java -version'命令查看本机是否安装了JDK，如果已安装了，可以先卸载。如果是apt方式安装，就需要使用apt方式卸载，如果是自己解压安装，就直接去删除相应的文件夹。另外注意需要删除/etc/profile相应的设置。

![查看和卸载JDK]()

　　当Linux未安装JDK时，使用java命令，会有如下提示：

![使用java命令]()

　　在ubuntu中，会有一些相关软件提示，而centos中没有，其他的版本暂时不考虑。我们可以使用apt install <software package>来安装相关软件。如果出现下图的错误，可以先使用一次apt update命令。

![apt install无法安装1]()

　　apt update命令会更新软件源，然后再安装，就OK了。安装完成后就可以使用java命令了。然后，将jdk的安装路径加入到JAVA_HOME中。使用vi /etc/profile命令，在文件最后加入JAVA_HOME：

```txt
#set java environment
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME CLASSPATH PATH
```

　　加入以上内容后，退出保存，使用source /etc/profile命令是配置生效。

### Linux配置Tomcat

　　先去Apache官网去获取tomcat的包地址，或者也可以自己下载后传到Linux上。建议获取地址，直接用wget命令直接下载到Linux上。

![下载tomcat]()

　　然后使用tar命令解压下载的tar.gz的tomcat包，如果是zip，需要使用unzip解压。

![解压tomcat]()

　　可以使用mv将解压后的tomcat文件夹名修改为tomcat，简短一点的文件名更方便使用。

```txt
mv apache-tomcat-8.5.16 tomcat
```

　　然后将tomcat的路径加入到环境变量中，以便后面使用。

```txt
vi /etc/profile

#tomcat
CATALINA_HOME=/usr/local/tomcat
export CATALINA_HOME
```

　　使用source /etc/profile命令使配置生效，可以使用export命令查看命令是否生效。

![检查tomcat配置]()

　　然后修改tomcat文件夹bin中的catalina.sh文件，在其中加入jdk的路径。
```txt
vi $CATALINA_HOME/bin
```

![tomcat加入jdk路径]()

　　退出保存，然后使用$CATALINA_HOME/bin/catalina.sh start启动tomcat服务，在浏览器上输入对应的Linux服务器地址，默认端口8080访问tomcat服务器首页。

![tomcat首页]()

　　如果访问不成功，原因可能有很多种，比如：
1. 未关闭防火墙；[ubuntu关闭防火墙](http://www.jianshu.com/p/a1a4455ff8fd)
2. tomcat启动失败；（去查看tomcat中的log文件夹中的catalina.out文件）
3. 端口被占用；[Linux查看程序端口占用情况](http://www.cnblogs.com/benio/archive/2010/09/15/1826728.html)

### 参考

[在linux上通过yum安装JDK](https://my.oschina.net/andyfeng/blog/601291)

[linux配置java环境变量(详细)](http://www.cnblogs.com/samcn/archive/2011/03/16/1986248.html)

[Linux下Tomcat的安装配置](http://blog.csdn.net/zhuying_linux/article/details/6583096)

[Linux服务器安装配置tomcat](http://www.jianshu.com/p/b71296e8b9a7)

[解决java.net.BindException: Address already in use问题](http://www.programgo.com/article/68533223626/)

[Ubuntu下JDK的安装、配置与卸载](http://blog.csdn.net/wenqisun/article/details/7422875)

[Ubuntu安装配置和卸载JDK](http://zyjustin9.iteye.com/blog/2176556)

[Ubuntu中软件的安装、卸载以及查看的方法总结](http://www.jianshu.com/p/03035ff6bb80)