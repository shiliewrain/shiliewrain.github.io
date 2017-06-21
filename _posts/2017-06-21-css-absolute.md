---
layout : post
title : "关于绝对定位absolute的一点学习记录"
category : CSS
tags : CSS 前端 绝对定位
---

* content
{:toc}


　　虽然自己平时的工作学习会用到CSS的知识，但确实没有系统的研究过。因为浏览器的多样性，CSS样式的复杂性，想要像使用Java那样有一个语法标准去编写不那么容易。对于CSS的学习掌握，需要非常大量的实践，所以我只能说是学习记录，而不敢说详解。




### 参考

　　先把参考文献贴出来吧，主要是根据这一篇文章来写的：[CSS绝对定位absolute详解](http://www.jianshu.com/p/a3da5e27d22b)

　　参考文献里面已经对绝对定位absolute做了详解了，下面的内容就记录一下自己的学习。

### 包裹性

　　给元素加上absolute或者float就相当于给元素加上display:block，然后就可以给元素设置width了。因为有些内联元素的宽度是自适应的，设置width是不生效的。如果存在display:block的样式，就可以设置width了。看下面一段代码和图：

```html
<div style="border:4px solid blue;">
  <img src="1.png" />
</div>
<div style="border:4px solid red; position: absolute;">
  <img src="1.png" />
</div>
```

![1]()

### 高度欺骗

　　这个是我以前没注意到的，之前好像看到过说加上绝对定位的元素会独立于父元素的文档流，从实验看来，绝对定位的元素依旧会在父元素的文档流中，但是该元素却不占用文档流中的位置。就是说，在同一个父元素中，绝对定位的元素会跟着它前面的元素出现，但是它之后的元素却出现在了它所占用的位置上。感觉就是绝对定位的元素在他出现的位置上飘起来了，如果不去设置它的位置，它就飘在前面元素的后面，同时，跟在它后面的元素就出现在了它下面。而父元素如果是自适应的高度，因为绝对定位的元素飘上去了，就不知道它的高度，就不会去适应它的高度了。来看下一段代码和图：

```html
<div style="border:4px solid red;">
  <img style="position:absolute;" src="1.png" />
  <span>我是一个绝对定位的absolute元素我是一个绝对定位的absolute元素</span>
</div>
```

![2]()

### 定位

　　关于绝对定位和相对定位，简而言之，如果absolute元素没有position:static以外的父元素，绝对定位是针对整个文档定位，而相对定位是相对于该元素在文档流中的位置定位。依据left、top、right、bottom给定的坐标，绝对定位的元素就出现在页面的该位置上，而相对定位的元素会依据自己原来在文档流的位置进行偏移。看下一段代码和图：

```html
<div style="border:4px solid red;margin-top:50px;">
  <img style="position:absolute;right:100px;top:100px;border:1px solid blue;" src="1.png" />
  <img style="position:relative;right:100px;top:100px;border:1px solid blue;" src="1.png" />
</div>
```

![3]()

　　根据上图可以知道，相对定位是相对与自己在文档流中的位置偏移，绝对定位则已跳出了文档流，按left、top、right、bottom定位在相应的窗口位置，没有设置的坐标则依据文档流的位置去默认设置。值得一提的是，如果父元素设置了相对定位，那么绝对定位的子元素就只能相对父元素去定位了。

### 小示例

　　主要是自己实践了一把参考文献中的示例，记住一些小技巧。

#### 右上角提示

　　一种实现思想是，用一个相对定位的父元素div包裹要提示的图标，然后利用绝对定位的元素定位到父元素的右上角。重点在于绝对定位的子元素和相对定位的父元素，以及父元素的display:inline-block使父元素刚好包裹住图片。代码如下：

```html
<style type="text/css">
.tipIcon {
  background-color: #f00;
  color: #fff;
  border-radius:50%;
  text-align: center;
  position: absolute;
  width: 20px;
  height: 20px;
  right:-10px;
  top:-10px;
}
</style>

<div style="display: inline-block; position:relative; border:1px solid blue;">
  <img src="1.png"/>
  <span class="tipIcon">6</span>
</div>
```

　　实际上，这样做就已经达到效果了，但是因为提示的元素是相对于父元素定位的，那么父元素变化之后，提示也可能发生变化。下面有一种父元素宽度足够的前提下，提示一直跟着图片的实现：

```html
<style type="text/css">
.tipIcon {
  background-color: #f00;
  color: #fff;
  border-radius:50%;
  text-align: center;
  position: absolute;
  width: 20px;
  height: 20px;
  margin: -10px 0 0 -10px;
}
</style>
<div style="display: inline-block; border:1px solid blue;">
  <img src="1.png"/><!--
  --><span class="tipIcon">6</span>
</div>
<!--img和span间的HTML注释的目的是消除换行符，你也可以将span紧跟在img后面写到一行里-->
```

　　去掉了父元素的相对定位，并且把top和right变为margin。这是由绝对定位的元素在文档流中的位置来设置margin，以达到提示始终在图片右上角的目的。前面提到了父元素的宽度足够，若不足够，则提示的子元素会在换行，因为该元素在文档流的位置就如此。当然，没有人会刻意去设置一些奇怪的宽度值。

#### 滤镜

　　滤镜，或者说遮罩，在工作中，会因为一些需求，而给页面加上一个遮罩，阻止用户对页面进行操作，除非用户操作了遮罩上的提示框。那么遮罩可以通过绝对定位来实现。看下面的代码：

```html
<style type="text/css">
.cover {
    position: absolute;
    left: 0;right: 0;top: 0;bottom: 0;
    background-color: #fff;
    opacity: .5;filter: alpha(opacity=50);
}
</style>
<div style="display: inline-block; border:1px solid blue;">
  <img src="1.png"/><span class="tipIcon">6</span>
</div>
<!-- 现在是全屏滤镜时间 -->
<span class="cover"></span>
```

　　全屏遮罩的原理是利用绝对定位，将滤镜元素通过left、right、top和bottom进行拉伸，覆盖整个页面。又根据父元素相对定位，子元素就只能相对于父元素进行绝对定位，则我们可以达到只覆盖图片的需求，代码如下：

```html
<style type="text/css">
.cover {
    position: absolute;
    left: 0;right: 0;top: 0;bottom: 0;
    background-color: #fff;
    opacity: .5;filter: alpha(opacity=50);
}
</style>
<div style="display: inline-block; border:1px solid blue;position:relative;">
  <span class="cover"></span>
  <img src="1.png"/><span class="tipIcon">6</span>
</div>
```

　　只是将滤镜元素变成div的子元素，然后给父元素div加上相对定位，就达到了只遮罩图片的目的。随便在这里写一下在遮罩上加一个垂直居中的提示框的样式，实现方法肯定有很多，这里就只记录了两个：

```html
<style type="text/css">
.div1{
	width: 100px;
	height: 100px;
	position: absolute;
	margin: auto;
	border:4px solid red;
	left: 0;
	right: 0;
	top: 0;
	bottom: 0;
}
.div2{
	width: 100px;
	height: 100px;
	position: absolute;
	left: 50%;
	top: 50%;
	margin: -54px 0 0 -54px;
	border:4px solid red;
}
</style>
```

　　两种方法，div1通过margin，div2通过left和top。div1将元素拉伸为整个窗口，利用margin:auto自动居中。div2利用定位，将元素定位到屏幕中间点，再利用偏移将元素移动到居中的位置。值得一提的是，div1的盒子模型占满了整个页面，四周都是margin，而div2的盒子模型只占用了中间一点空间。