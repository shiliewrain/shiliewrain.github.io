---
layout : post
title : "JavaWeb批量下载HTTP文件"
categories : javaweb
tags : I/O 批量下载 
---

* content
{:toc}

　　工作中碰到需要批量下载HTTP文件的需求，在实现过程中出现了一些问题，如下载的文件损坏、循环下载多个文件却只能下载到第一个文件。分析了一下问题，但是研究得不够深入，先记录。





## 下载方法的实现（字节流）

　　Java下载文件，原理就是inputStream输入流读需要下载的文件，outputStream输出流输出inputStream读到的内容。JavaWeb下载HTTP文件，即inputStream读HTTP文件，通过response中的outputStream将文件流输出。Java以字节流方式下载文件的基本代码如下：

```java

/**
     * java下载文件
     * @throws FileNotFoundException
     */
    public void downloadFile() throws FileNotFoundException {
        File resource = new File("c:resource.txt");
        File target = new File("d:target.txt");
        //文件输入流，传入需要下载的文件对象
        FileInputStream inputStream = new FileInputStream(resource);
        //文件输出流，传入需要写入数据的文件对象
        FileOutputStream outputStream = new FileOutputStream(target);
        //用来装输入流读取的字节
        byte [] bytes = new byte[1024];
        //记录读取的字节长度
        int len;
        try {
            //inputStream将输入流中的字节放入bytes中，读完所有字节则返回-1
            while ((len = inputStream.read(bytes)) != -1){
                outputStream.write(bytes, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            //关闭输入输出流
            try {
                if(inputStream != null){
                    inputStream.close();
                }
                if (outputStream != null){
                    outputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
```

### read和write

　　InputStream中有三个read方法，分别是read()、read(byte[] b)和read(byte[] b, int off, int len)。

* read()  该方法从输入流中读取数据的下一个字节，返回0到255范围内的int字节值。若无可读的字节，则返回-1。

* read(byte[] b)  该方法读取输入流中最多b.length个字节，并将其写入该数组中，返回实际读取到的字节数。若无可读的字节，则返回-1。

* read(byte[] b, int off, int len)  该方法从本次读取的起始位置开始偏移off个字节，然后最多读取len个字节写入b中，返回实际读取到的字节数。若无可读的字节，则返回-1。

　　OutputStream中有三个write方法，分别是write(byte[] b)、write(byte[] b, int off, int len)和write(int b)。

* write(byte[] b)  该方法将b写到输出流中。

* write(byte[] b, int off, int len)  该方法从b中偏移off个字节开始，往输出流中写入len个字节。若off+len>b.length，则抛出IndexOutOfBoundsException异常。

* write(int b)  该方法向输出流写入指定的字节b。

### 问题一：下载的文件损坏

　　可以肯定的是，如果程序执行没有报错，而下载的文件已损坏，则肯定是文件下载不完整。而文件不完整，则说明在字节输入流读取源文件或者字节输出流写目标文件有字节丢失或者写入了多余错误的字节。