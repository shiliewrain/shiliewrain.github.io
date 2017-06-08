---
layout : post
title : "JQuery常用的一些选择器"
categories : 前端
tags : JQuery 选择器
---

* content
{:toc}


　　工作中常常会用到一些JQuery选择器，除了常用的id、类、标签选择器，还会用到children、parent、next等选择器，因此整理一下它们基本的用法，以便查阅。有一点理解先记在前面：JQuery选择器会寻找当前dom中所有匹配括号中表达式的元素。




#### 基础选择器

　　id、类和标签选择器是最常用的三种选择器，用法也很简单，然后在此基础之上，可以进行一些组合，看下面的示例：

```html
<body>
	<div id="div1" class="div">
		<div id="div2" class="div">
			<div id="div3" class="div">
				<ul id="ul">
					<li id="li1" class="li">li1</li>
					<li id="li2" class="li">li2</li>
					<li id="li3" class="li">li3</li>
					<li id="li4" class="li">li4</li>
				</ul>
			</div>
		</div>
	</div>
</body>
<script type="text/javascript">
	$(function(){
		$("#li1").html("this is li1");//id=li1的元素
		$(".li").html("this is li");//所有class=li的
		$("li").each(function(index){//所有li标签
			$(this).html("li" + index);
		});
		$("#div1 li").html("li");//id=div1的里的所有li标签
		$("#div1 > li").html("li");//id=div1的直接子元素li，不包括子元素的子元素
		$("#li2 + li").html("li222");//id=li2的下一个元素为li，选择该li

		$("li:first").html("first");//第一个li元素
		$("li:last").html("last");//最后一个li元素

		$("li:even").html("even");//选择第0、2、4...个li元素
		$("li:odd").html("odd");//选择第1、3、5...个li元素

		$("li:eq(2)").html("eq");//序号为2的li元素
		$("li:gt(2)").html("gt");//序号大于2的li元素
		$("li:lt(2)").html("lt");//序号小于2的li元素

		$("li[id=li3]").html("id=li3");//id=li3的li元素
	});
</script>
```

　　类似于li:XXX的选择样式还有很多，列举不完，就暂时先列举上面这些比较常用的吧。

#### 选择器方法

　　在平时的编码中，我们会遇到需要查找父元素、子元素或者同级元素的要求。我们也可以用一些选择器方法去实现这些功能，操作上要比写表达式简单一些。

```html
<html>
<head>
	<title></title>
	<script type="text/javascript" src="jquery.js"></script>
</head>

<body>
	<div id="div1" class="div">
		<div id="div2" class="div">
			<div id="div3" class="div">
				<ul id="ul">
					<li id="li1" class="li">li1</li>
					<li id="li2" class="li">li2</li>
					<li id="li3" class="li">li3</li>
					<li id="li4" class="li">li4</li>
				</ul>
			</div>
		</div>
	</div>
</body>
<script type="text/javascript">
	$(function(){
		$("#ul").parent();//找到id=ul元素的父元素div3
		$("#ul").parents();//找到id=ul元素的所有祖先元素，一直到<html>
		$("#ul").parent("#div3");//parent()方法中可以传入选择器参数，但一般不需要，因为父元素就一个
		$("#ul").parents("#div2");//parents()方法中同样可以传参，能够寻找特定的祖先元素
		$("#div1").children();//返回所有的直接子节点，不包括子节点的子节点
		$("#div1").children(".div");//返回直接子节点中样式为div的元素
		$("#div3").find("li");//返回子孙节点中为li的元素，查找的是元素里的所有元素
		$("#li4").prev();//返回id=li4的前一个兄弟节点
		$("#li4").prevAll();//返回id=li4的所有前面的兄弟节点
		$("#li1").next();//返回id=li4的后一个兄弟节点
		$("#li1").nextAll();//返回id=li4的后面所有兄弟节点
	});
</script>
</html>
```

　　常用的就这么多吧，另外，因为返回的也是元素对象，因此可以采用多种组合来花式获取元素节点，获取的元素节点是不会重复的，如下面的例子：
```html
<html>
<head>
	<title></title>
	<script type="text/javascript" src="jquery.js"></script>
</head>

<body>
	<div id="div1" class="div">
		<div id="div2" class="div">
			<input class="test">
			<div id="div3" class="div">
				<ul id="ul">
					<li id="li1" class="test">li1</li>
					<li id="li2" class="li">li2</li>
					<li id="li3" class="li">li3</li>
					<li id="li4" class="li">li4</li>
				</ul>
			</div>
		</div>
	</div>
</body>
<script type="text/javascript">
	$(function(){
		$("#div1").find(".test").parents().append("345");
	});
</script>
</html>
```

　　input和li拥有共同的祖先元素，但是并没有获得重复的元素节点。

### 总结

　　JQuery选择器十分强大，你总可以通过某些手段获取dom中任意一个你想获得的元素，只不过因为你自己的掌握程度而采取的方式不一样。记住以上常用的一些就应该足够日常的工作需要了。

### 参考

　　[jQuery选择器总结](http://www.cnblogs.com/onlys/articles/jQuery.html)

　　[jquery获取父级元素、子级元素、兄弟元素的方法](http://blog.csdn.net/witsmakemen/article/details/20912893)