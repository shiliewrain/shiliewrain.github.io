---
layout : post
title : "List.remove()的使用注意"
categories : java
tags : java异常 数组循环删除
---

* content
{:toc}


　　今天修改一个bug，需要取一个List和一个Set的交集，使用了双重循环。想着提高循环效率，每加入一个交集中的元素，就将List中的元素删除，减少不必要的循环。结果直接调用了List的remove()方法，抛出了java.util.ConcurrentModificationException异常。这时才忽然记起之前看过的List循环中使用remove()方法要特别注意，尤其是forEach的循环。





## 不使用forEach的循环

　　使用常规的for循环写法，代码如下：

```java

    public static void main(String [] args){
        List<String> list = new ArrayList<String>();
        list.add("111");
        list.add("111");
        list.add("111");
        list.add("333");
        list.add("333");

        for (int i = 0; i < list.size(); i++){
            if ("111".equals(list.get(i))){
                list.remove("111");
//                i--;不加这句会少删除一个111
            }
            System.out.println(list.size());
        }
    }
```

　　先说说list的remove()方法，该方法有两个，一个是remove(Object obj)，另一个是remove(int index)。根据参数很容易理解，而这里要说的是remove(obj)会删除list中的第一个该元素，remove(index)会删除该下标的元素。调用一次remove方法，会使list中的所有该删除元素后面的元素前移。看看一个remove(Object obj)的源码：

```java

    public boolean remove(Object o) {  
        if (o == null) {  
            for (int index = 0; index < size; index++)  
                if (elementData[index] == null) {  
                    fastRemove(index);  
                    return true;  
                }  
        } else {  
            for (int index = 0; index < size; index++)  
                if (o.equals(elementData[index])) {  
                    fastRemove(index);  
                    return true;  
                }  
        }  
        return false;  
    }
```

　　remove是可以删除null的，但一般情况都是走else路径，再看看faseRemove方法：

```java

	private void fastRemove(int index) {  
	    modCount++;  
	    int numMoved = size - index - 1;  
	    if (numMoved > 0)  
	        System.arraycopy(elementData, index+1, elementData, index, numMoved);  
	    elementData[--size] = null; // Let gc do its work  
	}
```

　　System.arraycopy就是将list中删除元素的后面迁移，暂时没继续深入了解。总得来说，这样的操作是因为没有对变化的list做出相应的下标处理而产生了错误的结果，但是并没有产生异常。


## 使用forEach循环

　　错误的使用代码如下：

```java

public static void main(String [] args){
        List<String> list = new ArrayList<String>();
        list.add("111");
        list.add("111");
        list.add("111");
        list.add("333");
        list.add("333");
        for (String str : list){
            if("111".equals(str)){
                list.remove(str);//抛出异常java.util.ConcurrentModificationException
            }
        }
    }
```

　　这样使用会抛出java.util.ConcurrentModificationException。在forEach中，遍历的集合都必须实现Iterable接口（数组除外）。而forEach的写法实际是对Iterator遍历的简写，类似于以下代码：

```java

public void display(){  
        for(String s : strings){  
            System.out.println(s);  
        }  
          
        Iterator<String> iterator = strings.iterator();  
        while(iterator.hasNext()){  
            String s = iterator.next();  
            System.out.println(s);  
        }  
    }
```

　　上面的forEach和下面的Iterator迭代器效果一样，涉及到编译原理的一些内容，就不深究了。可以理解为forEach将其中的集合转为了迭代器进行遍历，而该迭代器内部有一个next()方法，代码如下：

```java

public E next() {  
    checkForComodification();  
    try {  
        E next = get(cursor);  
        lastRet = cursor++;  
        return next;  
    } catch (IndexOutOfBoundsException e) {  
        checkForComodification();  
        throw new NoSuchElementException();  
    }  
} 

final void checkForComodification() {  
    if (modCount != expectedModCount)  
        throw new ConcurrentModificationException();  
}
```

　　可以看到在使用next()方法获取下一个元素的之前会先检查迭代器修改次数，在我们使用ArrayList的remove()方法删除元素时，实际只修改了modCount，这样就会造成modCount和expectedModCount不相等，从而抛出异常。而使用迭代器本身的remove()方法则不会，因为Iterator本身的remove()方法会同时修改modCount和expectedModCount。

　　引用一段网上的解释：

>　　Iterator是工作在一个独立的线程中，并且拥有一个mutex锁。 Iterator被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照fail-fast原则Iterator会马上抛出java.util.ConcurrentModificationException异常。所以Iterator在工作的时候是不允许被迭代的对象被改变的。但你可以使用Iterator本身的方法remove()来删除对象，Iterator.remove() 方法会在删除当前迭代对象的同时维护索引的一致性。

　　最后总结一下就是，forEach将List转为了Iterator，删除元素就需要使用Iterator的remove()方法，错误地使用了List.remove()方法就会抛出异常。


## 参考

　　[Java中ArrayList循环遍历并删除元素的陷阱](http://tyrion.iteye.com/blog/2203335)

　　[List集合remove元素的问题](http://www.cnblogs.com/doudouxiaoye/p/5669481.html)

　　[Java语法糖1：可变长度参数以及foreach循环原理](http://www.cnblogs.com/xrq730/p/4868465.html)

　　[java forEach实现原理](http://blog.csdn.net/cq1982/article/details/49121879)