---
layout : post
title : 观察者模式
categories : 设计模式
---
* content
{:toc}
　　今天继续浏览了Mybatis的二级缓存，因为里面涉及到装饰者模式，就顺着去复习了下装饰者模式。虽然以前学了两遍设计模式，但总是忘得很干净，现在又开始重新学习，希望能有更深刻的印象吧。




## 装饰者模式

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

	public void getComponent(Component component){
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
		super.operation();
	}
}

public class ConcreteDecoratorB extends Decorator{
	
	@Override
	public void operation(){
		super.operation();
		System.out.println("戴太阳镜");
	}
}
```