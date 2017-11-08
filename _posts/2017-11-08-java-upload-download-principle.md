---
layout : post
title : "Java上传下载文件原理"
category : Java
tags : 上传 下载 原理
---
* content
{:toc}

　　在工作中常碰到需要上传下载文件的需求，主要通过框架的去实现。不同的框架，在实现上有些许不同，但究其原理实际都是相同的。




### Java实现上传下载原理

　　在页面上通过form表单提交文件，http通过流将文件传输到服务器。后台程序通过获取request中的输入流来解析出文件，然后通过outputStream保存在服务器上，这就是Java上传文件的原理。

　　在页面上发送请求下载文件，程序通过inputStream读取服务器上的文件，然后通过写入response的OutputStream，完成文件的下载。

　　以上就是对Java上传下载文件的原理简单的介绍了一下。

#### Java上传

　　在Java项目中上传文件需要使用form表单（当然也可以使用一些上传的js库），需要写出method=post和enctype=mutipart/form-data。示例如下：

```html
<form method="post" enctype="mutipart/form-data">
	<input type="file">
</form>
```

　　form表单提交到服务器后，就能使用request.getInputStream()获取流数据，然后可以对其中的数据进行解析，使用OutputStream将文件保存在服务器上。需要注意的是，流里面的数据有一些并不是源文件的数据，需要去除。而在使用框架或者一些工具类时，可以免去这一部分的解析工作。例如在springmvc中常使用的MultipartHttpServletRequest：

```java
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
Map<String, MultipartFile> fileMap = multipartRequest.getFileMap();
```

　　接下来就只需要使用OutputStream将文件保存在服务器中就可以了。

#### Java下载

　　Java下载文件的原理很简单，就是把文件写入response的输出流中就可以了。需要注意的是要设置好response中的contentType和header，attachment表示以附件的方式下载文件，filename为文件全名。关于contentType的具体内容可以自行百度。

```java
	OutputStream os = new BufferedOutputStream(response.getOutputStream());
	response.setHeader("Content-Disposition", "attachment; filename="+fileName);
    response.setContentType("application/force-download");
    byte[] bytes = new byte[1024];
	int len = 0;
	while((len = bis.read(bytes)) != -1){
		os.write(bytes, 0, len);
    }
	bis.close();
    os.flush();
    os.close();
```

　　以上的代码是下载单个文件，如果需要下载多个文件，给出的建议是在服务器端将文件打包一起下载。尽量避免循环下载，而且一次连接，输出流只能输出一次。

### 总结

　　对Java的IO有基本的了解后，使用Java实现上传和下载就比较简单。重点在于对文件流的读出和写入，上传就是将request中的输入流读出保存在服务器上，下载就是将服务器上的文件写入response的输出流中。

### 参考

[Content-type 的说明](http://blog.sina.com.cn/s/blog_5d8945610100corv.html)

[SpringMVC 文件上传配置，多文件上传，使用的MultipartFile](http://blog.csdn.net/swingpyzf/article/details/20230865)

[java Servlet上传下载文件http协议原理详解](http://www.zuidaima.com/share/2185806059785216.htm)

[JavaWeb学习总结(五十)——文件上传和下载](http://www.cnblogs.com/xdp-gacl/p/4200090.html)