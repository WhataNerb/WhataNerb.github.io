---
layout: post
title: 'ArrayList与LinkedList'
date: 2017-07-30
categories: Java基础
tags: Java ArrayList LinkedList 集合
---

在Java中有两种用来存储列表的集合类：ArrayList与LinkedList，我们来讨论一下它们两者之间的区别
<br>
### **ArrayList**
ArrayList维护着对象引用的一个内部数组，它是可以 **动态增长** 的
动态增长方式：ArrayList **默认大小是10**，当调用add方法且内部数组已经满了，数组列表就将自动创建一个更大的数组，并将所有的对象从原来的数组中拷贝到新的更大的数组中

ArrayList增长方式源码：

~~~java
// 若ArrayList的容量不足以容纳当前的全部元素，设置新的容量 = (原始容量x3)/2 + 1
public void ensureCapacity(int minCapacity) {
    modCount++;
    int oldCapacity = elementData.length;
    if (minCapacity > oldCapacity) {
        Object oldData[] = elementData;
        int newCapacity = (oldCapacity * 3)/2 + 1;
        if (newCapacity < minCapacity)
            newCapacity = minCapacity;
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
~~~
**注意：**数组列表的容量与数组的大小有一个非常重要的区别，如果为数组分配100个元素的存储空间，数组就有100个空位置可以使用，而容量为100个元素的数组列表只是拥有保存100个元素的潜力（实际上，重新分配空间的话，将会超过100），但是在最初，甚至完成初始化之后，数组列表根本就不含有任何元素
因为ArrayList是依靠数组形式存储数据，所以它最大的 **缺点** 就是：
执行从中间插入与删除元素操作代价很大，因为数组中间的任何一个位置出现变动，所有后面的元素都要移动位置
**优点** 是：
它支持高效率的随机访问，你可以通过数组下标快速的访问任意元素
<br>
### **LinkedList**
LinkedList是依靠链表实现列表的存储的，实际上，**在Java中，所有的链表都是双向链接的，即每个结点除指向下一个结点的next引用，还存放着指向前驱结点的previous引用**

LinkedList正好与ArrayList相反，**它在插入与删除元素上表现的很高效，但是，在访问元素上就表现的并不高效**

LinkedList随机访问的效率非常的低，它依靠[迭代器](http://blog.csdn.net/baidu_32045201/article/details/76271800)ListIterator类从前后两个方向遍历链表中的元素，并可以添加、删除元素，所以如果想要删除链表当中的元素，或是从中插入新元素，都必须从头或从尾逐个遍历过去

在LinkedList中先添加3个元素，然后再将第2个元素删除：

~~~java
List<String> staff = new LinkedList<>();
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
Iterator iter = staff.iterator();
String first = iter.next();  // first == "Amy"
String second = iter.next();  // second == "Bob"
iter.remove();   // remove Bob
~~~

在调用next之后，**remove方法会删除迭代器左侧的元素，如果调用previous就会删除右侧的元素，并且不能连续两次调用remove**

**add方法只依赖于迭代器的位置，而remove方法依赖于迭代器的状态**

在使用迭代器对LinkedList进行读写操作时，会有一些安全隐患，比如，如果有很多迭代器对同一个LinkedList进行操作，前一个迭代器将下一个迭代器要访问的元素删除了，那么就会出错，为了避免发生并发修改异常，**我们需要遵循以下原则**：
**可以给容器附加多个可读迭代器，但是一个容器最好只附加一个可写迭代器**

有一种简单的方法可以检测到并发修改的问题。集合可以跟踪改写操作的次数，每个迭代器都维护一个独立的计数值，在每个迭代器方法的开始处检查自己改写操作的计数值是否与集合改写操作计数值一致，如果不一致，抛出一个ConcurrentModificationException

> 参考文献：[美]Cay S. Horsimann著 《Java 核心技术 卷I》(第十版)