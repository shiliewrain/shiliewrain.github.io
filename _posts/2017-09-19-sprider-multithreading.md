---
layout : post
title : "多线程实现Java爬虫"
category : Java
tags : 爬虫 多线程
---
* content
{:toc}

　　之前实现了对网页数据的爬取和存储，然后准备将这些取得的数据进行分析。但是因为需要进行分词以及关键字提取，用到了[BosonNLP](http://docs.bosonnlp.com/tutorial_index.html)，无奈免费使用的部分还是有一些限制，再者分析并不是我学习的重点，那就先将分析放下。因为单线程爬虫的效率比较低，因此使用多线程实现Java爬虫。




### 背景

　　使用Java爬虫爬取[腾讯招聘](http://hr.tencent.com/position.php")的信息。先通过其地址“http://hr.tencent.com/position.php”爬取到各个地区和工作种类的url并保存，然后循环爬取这些url。原本使用的单线程循环遍历去爬取，效率比较低。现在需要使用多线程去竞争这些url，缩短爬取的时间。

```java
//获取需要爬取的url集合
public List<String> getUrl(String url) throws IOException {
        List<String> lidList = new ArrayList<>();
        List<String> tidList = new ArrayList<>();
        Document document = Jsoup.connect(url).post();
        List<String> urlList = new ArrayList<>();
        String css1 = "div#additems a";
        Elements elements1 = document.select(css1);
        String css2 = "div#searchrow3 div.left a";
        Elements elements2 = document.select(css2);
        for (Element element1 : elements1){
            String cityId = element1.attr("href");
            String lid = cityId.substring(cityId.lastIndexOf("=")+1);
            if (!lid.equals("0")){
                lidList.add(lid);
            }
        }
        for (Element element2 : elements2){
            String kindId = element2.attr("href");
            String tid = kindId.substring(kindId.lastIndexOf("=")+1);
            if (!tid.equals("0")){
                tidList.add(tid);
            }
        }
        for (String lid : lidList){
            for (String tid : tidList){
                urlList.add("position.php?keywords=&lid=" + lid + "&tid=" + tid);
            }
        }
        return urlList;
    }

//获取url的Document
public Document getHtml(String url) throws IOException {
        return Jsoup.connect(url).post();
    }

//保存并获得下一页的url
public String printHtml(Document document) throws IOException {
        SaveData saveData = new SaveData();
        saveData.saveDataByTxt(document);
        Element nextNode = document.getElementById("next");
        String url = "javascript:;";
        if (nextNode != null){
            url = nextNode.attr("href");
        }
        return url;
    }

//保存Document中需要的数据
public void saveDataByTxt(Document document) throws IOException {

        String css1 = "div#additems a.active span font";
        Element e1 = document.select(css1).get(0);
        String css2 = "div#searchrow3 div.left a.active span font";
        Element e2 = document.select(css2).get(0);
        String txtName = (e1.ownText() + "_" + e2.ownText() + ".txt").replace("/","or").replace("\\","or");
        File file = new File(path + txtName);
        FileWriter fileWriter = null;
        try {
            fileWriter = new FileWriter(file, true);
            String cssSelect = "table.tablelist tr";
            Elements elements = document.select(cssSelect);
            for (Element element : elements){
                if (!element.attr("class").equals("f")){
                    if (element.attr("class").equals("h") && file.length() > 0){
                        continue;
                    }
                    Elements elements1 = element.getAllElements();
                    for (Element element1 : elements1){
                        String text = element1.ownText().trim().replaceAll("\\u00A0","");
                        if(!StringUtil.isBlank(text)){
                            fileWriter.append(text + "\t");
                        }
                    }
                    Elements aTag = element.select("a");
                    String url = aTag.attr("href");
                    fileWriter.append(url + "\n");
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileWriter != null){
                fileWriter.flush();
                fileWriter.close();
            }
        }

    }
```

　　以上代码是爬取并保存相应的资源，可以略过。

### 单线程

　　单线程实现非常简单，只需要循环遍历即可。

```java
public static void main(String [] args) throws Exception {
        SpiderTest st = new SpiderTest();
        List<String> urlList = st.getUrl("http://hr.tencent.com/position.php");
        for (String url : urlList){
            while (!url.equals("javascript:;")){
                url = "http://hr.tencent.com/"+ url;
                try {
                    url = st.printHtml(st.getHtml(url));
                }catch (Exception e){
                    System.out.println("报错Url：" + url);
                    throw new Exception();
                }
            }
        }
    }
```

### 多线程

　　使用多线程，即思考需要并行的功能和它们竞争的共同资源，那么这里自然是多个爬虫并行竞争urlList里面的数据，而这个urlList肯定就是由多个爬虫线程共享。

```java
public class SpiderThread implements Runnable{

    private SpiderTest spiderTest;

    private List<String> list;

    private int no;

    //构造方法
    public SpiderThread(List<String> list, SpiderTest spiderTest, int no){
        this.list = list;
        this.spiderTest = spiderTest;
        this.no = no;
    }

    @Override
    public void run() {
        System.out.println("线程" + no +"启动了！");
        String url;
        while (!list.isEmpty()){
            synchronized (list){
                url = list.get(0);
                list.remove(url);
            }
            try {
                while (!url.equals("javascript:;")){
                    url = "http://hr.tencent.com/" + url;
                    url = spiderTest.printHtml(spiderTest.getHtml(url));
                }
            } catch (IOException e) {
                System.out.println("报错Url：" + url);
                e.printStackTrace();
            }
        }
    }
}
```

　　每当爬虫获取list中的一条数据，则将该数据删除，以免其他线程再获得该数据，重复爬取相同的内容。使用synchronized关键字给list加锁，保证线程安全，避免多个线程删除同一条数据。

```java
public static void main(String [] args) throws Exception {
        SpiderTest st = new SpiderTest();
        //多线程启动
        List<String> urlList = st.getUrl("http://hr.tencent.com/position.php");
        for (int i = 0; i < 10 ; i++){
            new Thread(new SpiderThread(urlList, st, i)).start();
        }
    }
```

### 总结

　　多线程的实现需要知道哪些功能是需要并行执行的，并且并行的线程是否需要访问共享的资源。如果需要，则必须保证该资源的线程安全。