---
layout : post
title : "synchronized原理"
category : Java
tags : 同步 synchronized原理
---
* content
{:toc}


　　因为sleep和wait而引起的对synchronized的怀疑，再想起sleep和wait的不同，虽然两者都会使当前线程阻塞，如果当前线程持有锁，sleep是不会放弃锁，而wait会放弃。进而看到了关于synchronized的原理，就备注一下。




### synchronized

　　synchronized锁对象是通过获得对象实例的对象头，其实现如下：

```C++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL; //持有ObjectMonitor对象的线程
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0;
    _Responsible  = NULL;
    _succ         = NULL;
    _cxq          = NULL;
    FreeNext      = NULL;
    _EntryList    = NULL; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0;
    _SpinClock    = 0;
    OwnerIsThread = 0;
  }
```

>ObjectMonitor中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表( 每个等待锁的线程都会被封装成ObjectWaiter对象)，_owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合，当线程获取到对象的monitor 后进入 _Owner 区域并把monitor中的owner变量设置为当前线程同时monitor中的计数器count加1，若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSet集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。如下图所示： 

![ObjectMonitor](https://github.com/shiliewrain/shiliewrain.github.io/blob/master/img/synchronized_theory.png?raw=true)

　　以上内容都是引用，简单来说，就是当_owner = 某个线程，该对象就被锁住了。而synchronized有两种锁方式，一种是显示锁，直接锁住对象的，一种是隐式锁，锁方法。显示锁通过在同步块开始和结束的位置加上monitorenter和monitorexit表示同步开始和结束。隐式锁则是通过查看常量池中的方法表里这个方法是否包含ACC_SYNCHNOIZED标志，包含则去获取持有当前该方法的对象的对象中的锁。值得一提的是，synchronized始终都是锁住Java对象的，而所有的Java对象都可以被synchronized锁住。因此wait和notify这类方法才会出现在顶级类Object中。

### CAS

　　CAS（Compare and swap），即比较交换。包含V、E、N三个元素，V代表更新变量，E代表预期值，N代表新值。
如果使用CAS改变一个变量的值，则先比较V和E，相等则将V复制给N。而在Java程序中相应的实现是AtomicInteger，其中的compareAndSet()和getAndIncrement()等方法都是使用了Unsafe类的CAS操作。

### ABA

　　ABA是针对CAS的漏洞提出的，因为如果V初始为A，而后变成了B，最后又变回了A，这样是无法被CAS察觉的。对于ABA问题的解决方案有一种就是给变量加上版本号，每一次改变，都会是版本号加1。J.U.C包为了解决这个问题，提供了一个带有标记的原子引用类"AtomicStampedReference"，它可以通过控制变量值的版本来保证CAS的正确性。

### 参考

[Java并发编程（二）线程同步和等待唤醒机制](https://blog.csdn.net/huaxun66/article/details/77942389)