---
layout : post
title : "Java序列化与单例模式"
category : Java
tags : 序列化 单例模式
---
* content
{:toc}

　　简单来说，序列化是指将实现了Serializable接口的类的实例连同其状态通过ObjectOutputStream转变为字节数组保存下来，然后可以通过ObjectInputStream将其读取出来。Java提供的这种保存对象的方式可以用在对象持久化、对象传输等方面。

### 序列化

　　因为平时的工作中，用到Java对象序列化的地方不多，因此我只是比较简单的了解了一下序列化。把其中比较重要的几点内容总结一下：

1. Java序列化可以保存实例属性的状态，但是不会保存方法，因为方法没有状态。

2. Java序列化不会保存static变量，因为static变量是类的状态，序列化保存的是对象，而不是类。

3. transient关键字修饰的变量也不会被序列化保存，反序列化时，该变量会被设置为默认值。

4. 父类实现了Serializable接口，其子类会继承，从而子类不需要显示实现Serializable接口。

5. 子类实现了Serializable接口，其父类如未实现该接口，子类序列化不会保存父类属性的状态。

6. 为了保证序列化和反序列化成功，对象必须有相同的序列化ID。

　　暂时能总结的也只有这么多，接下来就聊聊序列化与单例模式的关系。

### 序列化与单例模式

　　如果不是采用枚举的方式实现的单例模式，那么是可能被序列化破坏的。看看下面的例子：

```java
public class Singleton implements Serializable{

    private static volatile Singleton singleton = null;

    private String s;

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

    public String getS() {
        return s;
    }

    public void setS(String s) {
        this.s = s;
    }
}
```

　　以上代码是一个单例模式的实现，Singleton实现了Serializable接口，并且保证了线程安全。有一个私有属性s，通过set和get方法进行赋值取值操作。下面来看序列化与反序列化：

```java
public class SerializableTest{

    public void serialzableSingleton(String file) throws IOException {
        //序列化
        ObjectOutputStream objectOutputStream = null;
        FileOutputStream fileOutputStream = null;
        Singleton singleton1 = Singleton.getInstance();
        try {
            fileOutputStream = new FileOutputStream(file);
            objectOutputStream = new ObjectOutputStream(fileOutputStream);
            singleton1.setS("This is a serializable test!");
            objectOutputStream.writeObject(singleton1);//1. 序列化
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileOutputStream != null){
                fileOutputStream.close();
            }
            if (objectOutputStream != null){
                objectOutputStream.close();
            }
        }
        //反序列化
        ObjectInputStream objectInputStream = null;
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream(file);
            objectInputStream = new ObjectInputStream(fileInputStream);
            Singleton singleton2 = (Singleton) objectInputStream.readObject();//2. 反序列化
            System.out.println(singleton2.getS());
            System.out.println(singleton1 == singleton2);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null){
                fileInputStream.close();
            }
            if (objectInputStream != null){
                objectInputStream.close();
            }
        }
    }
}
```

　　为了代码的规范，因此内容稍微有点多，但主要内容其实就是try里面的两段，最重要的就是注释1和2的两句。测试的结果为：This is a serializable test!和false。这说明序列化确实保存了属性的状态，并且反序列化后的对象singleton2和序列化之前的singleton1不是同一个对象，即单例模式被破坏了。那么对ObjectInputStream的readObject方法进行跟踪，发现了以下代码：

```java
private Object readOrdinaryObject(boolean unshared)
        throws IOException
    {
        if (bin.readByte() != TC_OBJECT) {
            throw new InternalError();
        }

        ObjectStreamClass desc = readClassDesc(false);
        desc.checkDeserialize();

        Class<?> cl = desc.forClass();
        if (cl == String.class || cl == Class.class
                || cl == ObjectStreamClass.class) {
            throw new InvalidClassException("invalid class descriptor");
        }

        Object obj;
        try {
            obj = desc.isInstantiable() ? desc.newInstance() : null;//1.对象是否可以进行实例化
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        passHandle = handles.assign(unshared ? unsharedMarker : obj);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(passHandle, resolveEx);
        }

        if (desc.isExternalizable()) {
            readExternalData((Externalizable) obj, desc);
        } else {
            readSerialData(obj, desc);
        }

        handles.finish(passHandle);

        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);//2. 执行readResolve方法
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                handles.setObject(passHandle, obj = rep);//3. 将需要返回的obj指向rep
            }
        }

        return obj;
    }
```

　　虽然代码有点长，但我标注出了其中最重要的两句。注释1表示如果对象可以在运行时被实例化，则使用反射新建一个实例，否则返回null。因此，序列化也是使用反射新建了一个实例，破坏了单例模式。那么注释2就是避免序列化破坏单例的一种方式。我们只需要在单例的类中实现readResolve方法，利用它返回其实例，那么序列化得到的对象就是那个唯一的实例。

```java
private Object readResolve(){
        return singleton;
    }
```

　　如果singleton存在readResolve方法，序列化的时候就会执行该方法，在该方法中返回该唯一的实例，就避免了反序列化返回一个新的Singleton实例。

### 总结

　　这里的重点是序列化通过反射生成新的实例破坏了单例模式，以及如何通过readResolve方法避免单例模式被破坏。只是稍微读了一下其源码，读懂的感觉真是爽，读源码还真是非常好的学习方法。

### 参考

[深入理解JAVA序列化](https://www.cnblogs.com/wxgblogs/p/5849951.html)

[我对Java Serializable（序列化）的理解和总结](http://xiebh.iteye.com/blog/121311)

[单例与序列化的那些事儿](http://www.importnew.com/18030.html)

[Java 对象序列化](https://www.ibm.com/developerworks/cn/java/j-5things1/)