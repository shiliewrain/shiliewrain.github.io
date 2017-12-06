---
layout : post
title : "单例模式"
category : 设计模式
tags : 单例 线程安全 序列化
---
* content
{:toc}

　　单例模式是Spring中常见的一种设计模式，而最近看到了关于这种设计模式的一些文章，发现其中有很大的学问，简单记录一下。




### 单例模式

　　单例模式就是类的实例只有一个，其他类使用到的该类的实例永远都是同一个。思想是很简单的，但是想要写出完全正确的单例模式却不简单，而且之前在学校的教材里看到的示例代码都是有错误的。单例模式分为饿汉模式和懒汉模式两种，下面都会介绍一下。

#### 饿汉单例模式

　　饿汉模式就是在类加载的时候就完成初始化工作，应不同的要求，写法上也有一些区别，但无非就是获得实例的方式是直接访问还是通过方法访问。这里就只记录通过方法访问的，直接访问无非就是直接访问类的属性。

```java
public class Singleton {

    private static Singleton singleton = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return singleton;
    }
}
```

#### 懒汉单例模式

　　懒汉模式就是在第一次需要使用该实例的时候才完成初始化工作，相较于饿汉模式，其好处是资源利用率高，不会从最开始就占据内存。最简单的懒汉模式如下：

```java
public class Singleton {

    private static Singleton singleton = null;

    private Singleton(){}

    public static Singleton getInstance(){
        if (singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

　　印象中，那本教材上也是这样写的。在单线程的程序中，这样写也是没有问题的，但是在多线程的环境中，这样写不是线程安全的。这也是饿汉模式比懒汉优秀的地方，饿汉模式因为在类加载的时候就完成了初始化，所以饿汉模式是线程安全的。

　　为了保证线程安全，给getInstance方法加上synchronized关键字，但这样会造成性能的严重下降。因为锁的粒度太大了，即使singleton已经初始化了，但是获得该实例的操作仍有锁。因此，给初始化的代码加锁，减小锁的粒度，保证方法返回singleton的操作不会被锁。代码如下：

```java
public class Singleton {

    private static Singleton singleton = null;

    private Singleton(){}

    public static Singleton getInstance(){
        if (singleton == null){
            synchronized (Singleton.class){
                if (singleton == null)
                    singleton = new Singleton();
            }
        }
        return singleton;
    }
}
```

　　上面这种写法被称为Double-checked locking，不过智能的idea已经直接指出这样的写法不是线程安全的，其中的主要原因就是编译器的指令重排序。因为编译器自身对指令的优化可能会改变某些指令的顺序，在单线程情况下，这样是没有问题的，但是在多线程环境中，却可能发生一些意想不到的错误。

　　对于singleton = new Singleton()，简单地可以被编译器编译为如下指令：

　　singleton=allocate();//1.分配对象的内存空间

　　ctorInstance(memory);//2.初始化对象

　　singleton=memory;//3.singleton指向刚分配的内存

　　从执行结果上来说，2和3互换也能得到正确的结果，但这就可能造成在多线程环境中，某个线程获得一个不为null，但是却为初始化的对象。那么解决方法也很简单，就是使用volite修饰singleton禁止指令重排。

　　如此这般，懒汉模式的线程安全就被解决了。但这依然不是最安全的单例模式，因为使用反射或者序列化手段可以获得多个singleton的实例。

　　（关于反射和序列化如何破坏单例，这里暂时就不介绍了，保留《反射破坏单例》和《序列化破坏单例》的位置）

　　那么，最后介绍的就是网上非常推荐的一种单例模式的写法，使用枚举。

#### 枚举实现单例

　　这是令我非常惊奇的一种写法，但非常推荐。

```java
public enum SingletonEnum {
    SINGLETON_ENUM;

    public void test(){
        System.out.println("枚举实现单例");
    }
}
public class Test {

    public static void main(String [] args){
        SingletonEnum singletonEnum = SingletonEnum.SINGLETON_ENUM;
        singletonEnum.test();
    }
}
```

### 总结

　　饿汉模式是线程安全的，但资源利用率低，会一直占据内存。懒汉模式相较于饿汉模式资料利用率高一点，但是需要注意线程安全。最后，为了防止被反射和序列化破坏，最推荐使用枚举来实现单例模式。

### 参考

[漫画：什么是单例设计模式？](https://www.itcodemonkey.com/article/1357.html)

[单例模式－－反射－－防止序列化破坏单例模式](https://www.cnblogs.com/ttylinux/p/6498822.html?utm_source=itdadao&utm_medium=referral)

[如何防止单例模式被JAVA反射攻击](http://blog.csdn.net/u013256816/article/details/50525335)

[java单例模式之readResolve()](http://blog.csdn.net/u011499747/article/details/50982956)

[单例模式中，饿汉式和懒汉式有什么区别？各适合用在哪里？为什么说推荐用饿汉模式？](http://blog.csdn.net/abc19900828/article/details/39479377)

[单例饿汉式和饱汉式各自的有缺点](http://blog.csdn.net/u010061060/article/details/51147576)

[Java 对象序列化](https://www.ibm.com/developerworks/cn/java/j-5things1/)