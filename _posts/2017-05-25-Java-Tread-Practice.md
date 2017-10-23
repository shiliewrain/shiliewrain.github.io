---
layout : post
title : "Java线程学习杂记"
categories : Java
tags : Java Thread 线程
---

* content
{:toc}


　　最近看了一些关于Java线程的文章[链接在此](http://blog.csdn.net/ghsau/article/details/7421217)，结果却不知道能该记录些什么，那么就将前面几篇关于Java线程的文章的知识点稍微整理总结一下。





### synchronized关键字

　　synchronized能修饰方法或者代码块，修饰方法时，实际是给拥有该方法的对象加锁。synchronized修饰的变量或方法一定要是线程间的共享对象，否则无效。每一个线程都有自己私有的工作内存，使用synchronized进行同步的工作流程如下：

　　1. 获得同步锁；

　　2. 清空工作内存；

　　3. 从主内存拷贝对象副本到工作内存；

　　4. 执行代码；

　　5. 刷新主内存数据；

　　6. 释放同步锁。

### volatile关键字

　　volatile只能修饰变量，作用是强制是各线程读取主内存中的变量值，其工作流程如下：

　　1. 将变量从主内存拷贝到工作内存；

　　2. 执行代码；

　　3. 刷新主内存的数据。

　　感觉上volatile是强制要求所有的线程都从主内存中取值，避免线程一直访问自己私用内存中的变量。

### wait()和notify()

　　wait()阻塞当前线程，notify()唤醒阻塞队列中的线程。在Object对象中，通过wait()、notify()和notifyAll()这几个方法来实现线程间的通信。总共5个方法必须都在同步块或者同步方法中执行，这是因为调用某个对象的这些方法必须获得该对象的锁。

#### wait()

　　wait()方法会阻塞当前的线程，直至被该线程被唤醒或者中断。在调用该方法之前，必须获得该对象的锁，否则抛出IllegalMonitorStateException异常。

　　在调用wait()方法后，锁就被释放了。因此wait()后面的代码直到被唤醒或者中断才有可能继续执行。

#### notify()和notifyAll()

　　这两个方法的作用都是唤醒阻塞的线程，差别在于后者唤醒全部阻塞中的线程。同样，如果调用该方法时没有获得当前对象的锁，则抛出IllegalMonitorStateException异常。

　　值得注意的是，notify不会立刻释放掉锁，它只是唤醒了某个线程，让该线程等待其退出同步块或者同步方法之后才获得该对象锁，notifyAll则是让所有被唤醒的线程去竞争该对象锁。

### 锁对象

　　锁对象Lock和synchronized关键字的效果一样，需要注意的是加锁后也需要解锁。示例如下：

```java
package com.shiliew.function.threadTest;

import java.util.Random;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ThreadTest1 {

    public static void main(String [] args){
        final Data data = new Data();
        for (int i = 0; i < 3; i++){
            new Thread(new Runnable() {
                public void run() {
                    for (int j = 0; j < 5; j++){
                        data.setData(new Random().nextInt(10));
                    }
                }
            }).start();
        }
        for (int i = 0; i < 3; i++){
            new Thread(new Runnable() {
                public void run() {
                    for (int j = 0; j < 5; j++){
                        data.getData();
                    }
                }
            }).start();
        }
    }
}

class  Data{

    private int data;

    private ReadWriteLock lock = new ReentrantReadWriteLock();

    public void setData(int data){
        lock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "准备写入数据");
            try{
                Thread.sleep(20);
            } catch (InterruptedException e){
                e.printStackTrace();
            }
            this.data = data;
            System.out.println(Thread.currentThread().getName() + "写入" + this.data);
        }finally {
            lock.writeLock().unlock();
        }
    }

    public void getData(){
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "准备读取数据");
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "读取" + this.data);
        }finally {
            lock.readLock().unlock();
        }
    }
}
```

### 参考

[【Java并发编程】之十：使用wait/notify/notifyAll实现线程间通信的几点重要说明](http://blog.csdn.net/ns_code/article/details/17225469)