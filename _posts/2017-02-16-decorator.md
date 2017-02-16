---
layout : post
title : 观察者模式
categories : 设计模式
---
* content
{:toc}
　　今天继续浏览了Mybatis的二级缓存，因为里面涉及到装饰者模式，就顺着去复习了下装饰者模式。虽然以前学了两遍设计模式，但总是忘得很干净，现在又开始重新学习，希望能有更深刻的印象吧。




## 装饰者模式

[好文传送门](http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html)

### 目的

　　动态地给对象添加额外的职责。

### 类图

![decorator uml](http://my.csdn.net/uploads/201205/03/1336034007_4657.jpg)

### 角色

* 抽象组件（Component）：接口，
* 具体组件（ConcreteComponent）：类，
* 抽象装饰（Decorator）：抽象类，
* 具体装饰（ConcreteDecorator）：类，

### 代码实现

#### 抽象组件（Component）

```java
public interface Component{
	public void operation();
}
```

#### 具体组件（ConcreteComponent）

```java
public class ConcreteComponent implements Component{
	
	@Override
	public void operation(){
		System.out.println("穿衣服");
	}
}
```

#### 抽象装饰（Decorator）

```java
public abstract class Decorator implements Component{
	public Component component;

	public void setComponent(Component component){
		this.component = component
	}

	@Override
	public void operation(){
		component.operation();
	}
}
```

#### 具体装饰（ConcreteDecorator）

```java
public class ConcreteDecoratorA extends Decorator{
	
	@Override
	public void operation(){
		System.out.println("涂防晒霜");
		component.operation();
	}
}

public class ConcreteDecoratorB extends Decorator{
	
	@Override
	public void operation(){
		component.operation();
		System.out.println("戴太阳镜");
	}
}
```

#### 测试

```java
public class Test{
	
	public static void main(String[] args){
		ConcreteComponent cc = new ConcreteComponent();
		ConcreteDecoratorA cda = new ConcreteDecoratorA();
		ConcreteDecoratorB cdb = new ConcreteDecoratorB();
		cda.setComponent(cc);
		cdb.setComponent(cda);
		cdb.operation();
	}
}
```

### 自我理解

　　首先理解继承的是类型，而不是行为，行为主要由装饰者提供。装饰者一定要继承组件的类型，主要是为了满足setComponent(component)，这即是对传入的对象进行装饰，能使得装饰一直进行。感觉装饰者模式的核心思想是将原来的对象进行扩展，在保证基本功能实现的同时动态添加一些额外的职责。而对于全透明的要求，则需要将具体装饰类ConcreteDecorator向上转型声明为Component；半透明则允许声明为ConcreteDecorator类型的，以满足更高的职责需求。