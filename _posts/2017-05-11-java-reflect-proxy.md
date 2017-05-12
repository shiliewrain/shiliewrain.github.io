---
layout : post
title : "Java反射机制与动态代理"
categories : Java
tags : Java 反射 代理
---

* content
{:toc}

　　一直对Java的反射机制停留在Class.forName()的阶段，最近工作略闲，就想着将这部分知识重新学一遍，让自己有更深刻的印象，而且Java的反射和动态代理也是Spring框架的实现原理之一，有必要深入理解一下。




## 反射

　　在Java运行时，获取任意一个对象的类的名称、构造器、属性和方法，并且可以生成该类的实例，并且调用其中的方法，这是Java反射机制提供给我们的能力。如果不使用反射，我们只能在编译之前完成我们想要的功能。例如，输入一个类的名称，想要获得该类的所有属性和方法。下面通过一些小示例来具体看看Java反射机制的应用。

#### 获取对象的类名

```java
ReflectionTest rt = new ReflectionTest();
        System.out.println(rt.getClass().getName());
        System.out.println(ReflectionTest.class.getName());
```

　　自定义了一个类，然后获取该类的方法名，两句打印语句都会输出ReflectionTest的包含包名的完整名称。

#### 获取Class的实例

```java
Class<?> class1 = Class.forName("com.shiliew.function.reflection.ReflectionTest");
        Class<?> class2 = new ReflectionTest().getClass();
        Class<?> class3 = ReflectionTest.class;

        System.out.println(class1.getName());
        System.out.println(class2.getName());
        System.out.println(class3.getName());
```

　　三条打印语句的结果都是“com.shiliew.function.reflection.ReflectionTest”。

#### 获取对象的父类与实现的接口

```java
		Class<?> clazz = Class.forName("com.shiliew.function.reflection.ReflectionTest");
        Class<?> parentClass = clazz.getSuperclass();
        System.out.println(parentClass.getName());

        Class<?> [] intes = clazz.getInterfaces();
        for (Class inte : intes){
            System.out.println(inte.getName());
        }
```

#### 反射实现实例化

```java
		Class<?> clazz = Class.forName("com.shiliew.function.reflection.User");
        User user = (User) clazz.newInstance();
```

　　存在User类，调用newInstance()方法可以通过反射机制实例化获得到的class对象。       

#### 获取对象的构造函数

```java
		Class<?> clazz = Class.forName("com.shiliew.function.reflection.User");
        Constructor<?> [] constructors = clazz.getConstructors();
        for (Constructor<?> constructor : constructors){
            Class<?> [] classes = constructor.getParameterTypes();
            System.out.println(constructor.getName());
            for (Class c : classes){
                System.out.println(c.getName());
            }
        }
        User user = (User) constructors[0].newInstance("Shiliew", 27);
```

　　存在User类，里面有三个构造函数：一个无参，一个参数为String类型，一个参数为String类型和int类型，在操作过程中，发现getConstructors()方法如同入栈一样将构造器按书写顺序放入数组，读取的时候是先进后出。示例中将两个入参的构造器写在其他构造器之后，于是获得的构造器数组中，它变成了第一个。

#### 获取一个类的全部属性

```java
		Class<?> clazz = Class.forName("com.shiliew.function.reflection.SubUser");
        Field [] fields = clazz.getDeclaredFields();
        for (Field field : fields){
            int mo = field.getModifiers();
            String priv = Modifier.toString(mo);

            Class<?> type = field.getType();
            System.out.println(priv + " " + type.getName() + " " + field.getName() + ";");
        }
        System.out.println("=======================");
        Field [] fields1 = clazz.getFields();
        for (Field field : fields1){
            int mo = field.getModifiers();
            String priv = Modifier.toString(mo);

            Class<?> type = field.getType();
            System.out.println(priv + " " + type.getName() + " " + field.getName() + ";");
        }
```

　　上面的示例中有两个获取类的属性的方法：getDeclaredFields()和getFields()，这两个方法的区别在于getDeclaredFields()能获取类本身的所有属性，不管private还是public，而getFields()只能获取类中所有public的属性，包括从父类继承过来的和从接口实现的。类似的方法还有获取类的方法的getDeclaredMethods()和getMethods()以及getConstructors()和getDeclaredConstructors()。

#### 在泛型为Integer的ArrayList中存在一个String类型的对象

```java
		ArrayList<Integer> list = new ArrayList<Integer>();
        Method method = list.getClass().getMethod("add", Object.class);
        method.invoke(list, "Java反射机制");
        System.out.println(list.get(0));
```

　　通过反射可以在Java里做些“违法”的事，如上面的示例，获取到ArrayList中的add()方法，然后调用时传入String类型的参数，保存成功。

#### 通过反射操作数组对象

```java
		Object obj = Array.newInstance(String, length);
		String [] str = (String []) obj; 
```

　　使用反射机制生成数组有略微一点不同，就是要使用Array类。通过Array的newInstance方法反射生成一个type类型和length长度的数组对象，然后强制转换。

## 代理

　　说完了反射，再来讲讲代理。当我们不想A类和B类直接发生耦合的时候，可以为A类创建一个代理类C，通过C类代理A类去和B类产生联系。简单地说，这就是代理。而要实现代理，则需要为A类和C类创建一个共同的接口，接口中定义要被代理的方法。而代理分为静态代理和动态代理，静态代理常规实现，动态代理由反射实现。

### 静态代理

```java
//代理接口
interface ProxyInterface{
    public void say();
}


//被代理者
class RealObject implements ProxyInterface{
    //实现接口方法
    @Override
    public void say() {
        System.out.println("say");
    }
    
}

//代理者
class ProxyObject implements ProxyInterface{
    @Override
    public void say() {
        System.out.println("hello proxy");
        new RealObject().say();
        System.out.println("this is method end");
    }
}

public class Main {
    //这里传入的是接口类型的对象，方便向上转型，实现多态
    public static void consumer(ProxyInterface pi){
        pi.say();
    }
    public static void main(String[] args) {
        consumer(new ProxyObject());
    }
}
```

　　静态代理的示例很简单，一个共同的代理接口ProxyInterface，里面有一个public的say()方法，代理类和被代理类都实现了该接口的say()方法，然后调用代理类的say()方法则可以达到调用被代理类的say()方法的目的。

### 动态代理

　　静态代理类的很大一个缺点是要为每一个被代理类创建一个代理类，如果有几十个代理接口，那么类的数量会急剧增加。另外，我们必须知道被代理类才能为其创建代理类，否则无法使用代理。为了解决这些问题，我们需要使用动态代理模式。

　　要使用动态代理，需要实现InvocationHandler接口，这个接口中只定义了一个public方法invoke，该方法接受三个参数：Object、Method和Object[]，返回一个Object对象。第一个参数一般指代理类，第二个参数指被代理的方法，第三个参数为被代理方法的参数数组。实现动态代理的步骤如下：

1. 创建一个实现了接口InvocationHandler的类，并且实现invoke方法；

2. 创建被代理的类以及相应的接口；

3. 调用Proxy类中的静态方法newProxyInstance(ClassLoader loader,Class[] interfaces,InvocationHandler h)，创建出一个代理类；

4. 通过代理类调用被代理的方法。

	示例代码如下：

```java
class MyInvocationHandler implements InvocationHandler{

    private Object object = null;

    public MyInvocationHandler(Object object){
        this.object = object;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object temp = method.invoke(this.object, args);
        return temp;
    }
}

interface Subject {
    String say(String name, int age);
}

class RealSubject implements Subject{
    public String say(String name, int age) {
        return name + " " + age;
    }
}

public class Main {
    public static void main(String[] args) {
        //反射机制动态代理
        RealSubject realSubject = new RealSubject();
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler(realSubject);
        Subject subject = (Subject) Proxy.newProxyInstance(Subject.class.getClassLoader(), new Class[] {Subject.class}, myInvocationHandler);
        String info = subject.say("shiliew", 20);
        System.out.println(info);
    }
}
```		

　　通过上面的示例看出，我们不必再为每一个被代理的类去创建一个新的代理类，通过实现了接口InvocationHandler的MyInvocationHandler类和Proxy.newProxyInstance()方法就能够动态创建一个代理对象。

## 总结

　　要更好地去理解反射和代理，可能是要多去了解它们的应用场景。我了解的并不多，反射可以用在通过配置文件去动态生成类的对象，以及运行时获取对象的属性或者方法。感觉获取JSONObject对象中的属性其实挺类似的，暂时不清楚是不是也用到了反射，但反射是绝对能达到同样效果的。另外代理模式，可以参考翻墙，一个代理就可以达到A不能直接访问B，却可以通过能访问B的C间接访问B的目的。

## 参考

[JAVA的反射机制和动态代理](https://my.oschina.net/lyp3314/blog/136589)

[Java反射机制详解](http://www.cnblogs.com/lzq198754/p/5780331.html)

[为什么要使用代理模式](http://blog.163.com/love_wangchao/blog/static/21251930820138811412715/)

[java反射机制应用场景](http://blog.csdn.net/yaowj2/article/details/8496581)