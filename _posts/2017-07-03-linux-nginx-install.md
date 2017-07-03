---
layout : post
title : "Ubuntu安装nginx"
category : nginx
tags : Linux ubuntu nginx
---

* content 
{:toc}


　　Nginx是一个高性能的逆向代理服务器，在Ubuntu里面安装nginx有两种方式，一种apt install，一种源码安装。





### apt install安装

　　这是在ubuntu中安装nginx最简单的方法，直接使用'apt install nginx'命令进行安装，在服务器可以访问网络的情况下就推荐使用这种方式。

![apt_install_nginx](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/apt_install_nginx.png?raw=true)

### nginx源码安装

　　因为apt安装nginx会自动下载安装相应的依赖包，而源码安装不会，所以使用源码安装的时候，我们需要自己去下载相应的依赖包：

　　* apt install gcc

　　* apt install libpcre3

　　* apt install zlib1g

　　* apt install openssl

　　安装了以上依赖包之后，可以使用'wget &lt;url&gt;'或者'apt source nginx'来下载nginx源码包，然后使用tar命令解压，解压之后，进入到解压的nginx文件夹中，使用'./configure'命令配置环境变量，然后使用'make'和'make install'完成安装。最后执行'./usr/local/nginx/sbin/nginx -t'测试nginx是否正常工作，使用'./usr/local/nginx/sbin/nginx'启动nginx，访问该服务器地址的80端口，如果出现nginx的欢迎页面则启动成功。


### nginx卸载

　　不同的安装方式需要不同的卸载方式：

　　1. apt install安装：使用'apt remove nginx*'命令卸载所有nginx软件，并随后使用'apt autoremove'方法。有时候，如果再安装nginx报错，可能是由于之前卸载不完整，可以使用'apt purge nginx*'命令，然后再执行'apt autoremove'。

　　2. nginx源码安装：如果nginx正在工作，就先关闭nginx。然后找到nginx文件夹，将其使用'rm -rf'命令全部删除。

### 参考

[ubuntu下安装nginx](http://blog.csdn.net/u013140542/article/details/36070521)

[在Ubuntu系统上安装Nginx服务器的简单方法](http://www.jb51.net/article/71384.htm)

[Ubuntu下安装Nginx详细步骤](http://www.cnblogs.com/hzh19870110/p/6100674.html)

[170217、nginx 安装时候报错：make: *** No rule to make target	&acute;build', needed by  	&acute;default'. Stop.](http://www.cnblogs.com/zrbfree/p/6419043.html)

[apt-get命令详解(中文),以及实例](http://blog.51yip.com/linux/1176.html)

[Nginx是什么？Nginx介绍及Nginx的优点](https://lnmp.org/nginx.html)

[写给Web开发人员看的Nginx介绍](https://fraserxu.me/2013/06/22/Nginx-for-developers/)