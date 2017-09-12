---
layout : post
title : "Java实现爬虫"
category : Java
tags : Java 爬虫 spider
---
* content
{:toc}

　　比较闲，所以让冯董立了个项目。项目在gitlab上，项目地址：[java的数据采集项目](https://gitlab.com/458/spider)。




### 网络爬虫

　　网络爬虫只是一种比喻，指的是通过模拟用户请求获得页面内容。一般都是爬取的数据量都很大，然后对这些页面进行解析，获取其中有用的数据，整理分析。

### Java实现爬虫

　　用Java实现爬虫就是要用Java模拟用户发送请求，获得服务器返回的数据。Java发送HTTP请求的工具类有很多，我使用的是org.jsoup包中的Jsoup类。

>　　jsoup 是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

　　以上一段引自百度百科。获得一个页面文档可以直接使用这一句：

```java
Document document = Jsoup.connect(url).post();
```

　　url即为你需要爬取的页面，通过post方式去访问获取，返回一个Document对象。当然，还可以使用其他的工具类去获取网页内容，可以保存为本地文件或者是字符串，都可以使用Jsoup.parse()方法进行解析。也有其他的解析工具类，或者也可以自己实现一个解析工具，这里就不多说了。

　　利用Jsoup解析Document文档，其实和JavaScript的选择器很像。通过样式、标签或者Id去获取到相应的元素，通过遍历、筛选等方法获取这些元素中的数据。方式是多种多样的，而且要根据不同的页面结构编写相应的解析代码。

### 示例

　　这里以冯董给出的需要爬取的地址：http://hr.tencent.com，下面的代码也根据这个网页的相应结构来爬取数据。

```java
//获取网页内容
public Document getHtml(String url) throws IOException {
        Document document = Jsoup.connect(url).post();
        return document;
    }

	//解析网页并打印出需要的数据
    public String printHtml(Document document){
        String cssSelect = "table.tablelist tr";
        Elements elements = document.select(cssSelect);
        for (Element element : elements){
            if (!element.attr("class").equals("f")){
                Elements elements1 = element.getAllElements();
                for (Element element1 : elements1){
                    String text = element1.ownText();
                    System.out.print(text + " ");
                }
                Elements aTag = element.select("a");
                String url = aTag.attr("href");
                System.out.println(url);
            }
        }
        //获取下一页的地址
        Element nextNode = document.getElementById("next");
        String url = nextNode.attr("href");
        return url;
    }

	//测试
    public static void main(String [] args){
        String url = "keywords=&lid=2218&tid=87";
        SpiderTest st = new SpiderTest();
        try {
            while (!url.equals("javascript:;")){
                url = "http://hr.tencent.com/position.php?"+ url;
                url = st.printHtml(st.getHtml(url));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

### 总结

　　其实爬虫就是模拟用户请求，去获得原始的数据。之后就是按不同的页面结构进行相应的解析，以获取需要的数据，然后进行分析。

### 参考

[java爬虫中jsoup的使用](http://www.cnblogs.com/Jims2016/p/5652493.html)

[Java爬虫项目实战（一）](http://www.cnblogs.com/Jims2016/p/5877300.html)

[java从零到变身爬虫大神（一）](http://www.cnblogs.com/TTyb/p/5784581.html)