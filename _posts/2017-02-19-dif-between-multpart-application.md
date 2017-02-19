---
layout : post
title : "form-data和x-www-form-urlencoded的区别"
categories : 前端
tags : 表单编码类型
---
* content
{:toc}

　　之前工作需要使用postman调用外部接口，需要传入相应的参数。postman传参有四种形式：form-data、x-www-form-urlencoded、raw和binary。查阅了一下相互之间的区别，记录一下。




## form-data

　　mutipart/form-data，将表单的数据处理为一条消息。以标签为单元，用分隔符分割。键值对形式，可以上传文件。

## x-www-form-urlencoded

　　application/x-www-form-urlencoded，将表单数据转化为键值对，用“&”连接。没指明enctype时的默认值。

## raw
　　raw，上传任意格式的文本。

## binary
　　binary，只能传递二进制数据，一般用于上传文件，非键值对，一次只能上传一个文件。

### 参考

[传送门](http://blog.csdn.net/ye1992/article/details/49998511)