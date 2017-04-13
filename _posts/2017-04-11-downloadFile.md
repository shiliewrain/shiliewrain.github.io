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

　　可以肯定的是，如果程序执行没有报错，而下载的文件已损坏，则肯定是文件下载不完整。而文件不完整，则说明在字节输入流读取源文件或者字节输出流写目标文件有字节丢失或者写入了多余错误的字节。在编写代码的过程中，我用到了两种字节输入输出流的写法，一种是一次读出全部字节，并一次写入输出流，另一种是一次读取指定长度的字节，并写入输出流，循环直至写完所有字节。两种写法如下：

```java

//全部字节一次读，一次写
byte[] bytes = new byte[inputStream.available()];
        inputStream.read(bytes);
        outputStream.write(bytes);

//每次读1024个字节，将读出的字节全部写入，直至所有内容读完写完
byte[] bytes = new byte[1024];
            while ((len = in.read(bytes)) != -1){
                out.write(bytes, 0, len);
            }
```

　　我是因为第一种方式下载的文件被损坏，才换了第二种方式，从而获得了正确的结果。结果实验，发现在执行本地java程序进行文件下载时，两种方式都可以获得正确的结果，而在本地的JavaWeb程序进行文件下载时，第一种方式下载的文件不完整，只有1KB，而第二种方式能获得完整正确的文件。通过网上查阅资料，大致得出的结论为：
　　	在使用read(byte[] b)方法时，该方法最多能读取b.length个字节，但不能保证会读到这么多，特别是在下载非当前服务器中资源的时候，因为网络通讯不够稳定，传输过来的字节可能不全，可能顺序不对，因此就无法保证文件的正确性，下载得到的文件为损坏的文件。通过将文件分组读出，可以降低出错的概率。

## JavaWeb下载Http文件

　　因为Java的I/O流使用了装饰者模式，在java下载文件的基础上修改输入输出流就可以实现JavaWeb下载Http文件。将文件输入流由FileInputStream变为HttpURLConnection.getInputStream()，将文件输出流由FileOutputStream变为response.getOutputStream()。示例代码如下：

```java

public void downloadFile(HttpServletResponse response, String path, String fileName){
        OutputStream out = null;
        URL url;
        InputStream in = null;
        int len = 0;
        try{
            //取得HTTP文件的URL对象
            url = new URL(path);
            //获取该URL对象的连接
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            //连接的相关属性设置
            connection.setConnectTimeout(3 * 1000);
            connection.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");
            //获取该连接的字节输入流
            in = connection.getInputStream();
            //设置响应头相关属性
            response.setHeader("Content-Disposition", "attachment; filename=" + fileName);
            response.setContentType("application/force-download");
            //获取响应头的字节输出流，装饰为缓冲流提高读写效率
            out = new BufferedOutputStream(response.getOutputStream());
            byte[] bytes = new byte[1024];
            while ((len = in.read(bytes)) != -1){
                out.write(bytes, 0, len);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                if (in != null) {
                    in.close();
                }
                if (out != null){
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

　　基本内容就这些，毕竟Java下载文件的原理简单来说就是，将源文件放入输入流，输入流将文件写到临时数组中，输出流从该临时数组中读出文件，写入目标文件中。

### 问题二：JavaWeb批量下载多个文件，只能得到第一个。

　　对于这个问题，目前我不能确定原因，通过网上的相关解释来看，问题出在response.getOutputStream()中。因为HTTP是无状态的，客户端不知道响应是否结束，是否需要保持连接。所以，当调用了一次response.getOutputStream()方法后，完成了输出流的写入，客户端就认为响应已经结束，那么连接就已断开，则之后的文件就不会被下载下来。一次request对应一次response，response执行过一次之后，则客户端认为请求已完成，则断开连接。

　　目前要解决这个问题，我能想到的，也是最为通用的方法就是将多个文件打包下载。

## JavaWeb打包下载文件

　　简单说说实现打包下载的过程：

1. 生成一个压缩文件，将其放入文件输出流，然后用压缩输出流装饰。

2. 循环将文件放入输入流，然后通过压缩输出流写入压缩文件。

3. 将压缩输出流关闭，将压缩文件放入输入流，通过response的输出流写出，最后删除服务器上的压缩文件。

　　示例代码如下：

```java

public void downloadHttpFileByZip(HttpServletResponse response, List<String> paths, List<String> fileNames) throws IOException{
		//需要生成的压缩文件名
		String fileName = UUID.randomUUID().toString() + ".zip";
		String outFilePath = "d:" + fileName;
		//生成该压缩文件
		File file = new File(outFilePath);
		FileOutputStream outStream;
		try {
			//将压缩文件放入文件输出流
			outStream = new FileOutputStream(file);
			//使用压缩输出流装饰文件输出流
			ZipOutputStream zipStream = new ZipOutputStream(outStream);
			zipFile(paths, fileNames, zipStream);
			//一定要先关闭压缩输出流，再使用压缩文件，否则压缩文件会不正确
			zipStream.close();
			outStream.close();
			
			//同JavaWeb下载文件
			BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file.getPath()));
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
		    //关闭流之后再删除文件，否则会删除失败
		    file.delete();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
	}
	
	//循环获取HTTP文件，并写入压缩输出流
	private void zipFile(List<String> paths, List<String> fileNames, ZipOutputStream os) throws IOException{
		URL url;
		InputStream inputStream = null;
		for(int i = 0 ; i < paths.size(); i++){
			url = new URL(paths.get(i));
			HttpURLConnection conn = (HttpURLConnection)url.openConnection();
		    conn.setConnectTimeout(3*1000);
		    inputStream = conn.getInputStream();
		    zipFile(inputStream, os, fileNames.get(i));
		}
	}
	
	//将输入流中的字节通过压缩输出流写入压缩文件中
	private void zipFile(InputStream is, ZipOutputStream os, String fileName) throws IOException{
		if(is != null){
			BufferedInputStream bis = new BufferedInputStream(is);
			//压缩包内的文件对象
			ZipEntry entry = new ZipEntry(fileName);
			//将该文件对象放入压缩输出流
			os.putNextEntry(entry);			
			byte[] bytes = new byte[1024];
			int len = 0;
			//将输入流中的字节写入输出流中的文件对象
			while((len = bis.read(bytes)) != -1){  
				os.write(bytes, 0, len);  
            }
            //写完就关闭流，保证文件的正确性
			os.closeEntry();
			bis.close();
			is.close();
		}
	}
```

### 问题三：下载的压缩包解压报错“不可预料的压缩文件末端”，但是解压出来文件是正确的。

　　这个问题是由于没有正确关闭压缩输出流造成的，并且在使用该压缩文件之前，一定要关闭压缩输出流，否则就无法得到正确的文件。

## 结语

　　通过一次实践，对文件输入输出流有了更全的认识，目前来说，使用是没有多大的问题了。通过理解InputStream(源文件)->bytes[字节数组]->OutputStream(目标文件)，就可以理解Java I/O的使用方法，然后通过各种输入输出流装饰类来满足需求。