---
layout: post
title: 'String，StringBuffer，StringBuilder'
date: 2017-10-14
categories: Java基础
tags: Java String
---
String，StringBuffer，StringBuilder 这三者的区别是 Java 学习中的一个基础知识，也是在面试中经常会问到的一个知识点

## **String**
String 也许是 Java 中最常被用到的类了，关于它，有很多值得一提的地方

首先，Java 中的 String 是一个 char 类型序列：

~~~java
char[] data = {'a', 'b', 'c'};

// 等价于

String str = "abc";
~~~
不过，更加严格地定义 String , 它应该是 **Unicode 字符序列**（char 数据类型就是一个采用 UTF-16 编码表示 Unicode 码点的代码单元）
关于 Unicode 字符我们需要知道，大多数常用的 Unicode 字符使用一个代码单元就可以表示，而一些特殊字符则需要两个代码单元

关于 String 我们一定要记住的最重要的特性是，**String 是不可变的常量**
当你 new 一个 String 对象以后，它的值就确定了，不能做修改，String 类中也没有提供可供修改其内容的方法。这样做的主要目的是为了实现 **字符串共享** 从而提高效率
在详细讲述这点前，我们需要注意一点：你也许会把 String 想象成一个 char 数组（char[]），这种认识是错误的，String 应该类似于是一个 char 指针（char*）

我们来做一下验证：

~~~java
public class Test {
	public static void main(String[] args) {
		String str_1 = "hello";
		String str_2 = "hello";
		
		System.out.println("地址相同？" + (str_1 == str_2));
		System.out.println("内容相同？" + str_1.equals(str_2));
	}
}
~~~
如果说“==”是用来比较地址的，这个说法不够严格，其实“==”就是比较内容的，因为 String 变量是一个引用，它的存储的内容就是对象的内存地址。而 equals() 方法是用来比较对象内的数据内容的。
上面代码的输出结果是：
![这里写图片描述](http://img.blog.csdn.net/20171014165428779?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，str_1 和 str_2 引用着同一个数据，它们**共享**一个“hello”

如果我们修改一下代码：

~~~java
public class Test {
	public static void main(String[] args) {
		String str_1 = "hello";
		String str_2 = new String("hello"); // new 出一个 str_2
		
		System.out.println("地址相同？" + (str_1 == str_2));
		System.out.println("内容相同？" + str_1.equals(str_2));
	}
}
~~~
这次的结果是：
![这里写图片描述](http://img.blog.csdn.net/20171014165837450?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这次就指向了不同的数据（但是内容相同）

在这里你也许还会有一个疑问：如果 String 不可变，每次需要一个新的 String 时就要重新 new 一个出来，那么不是很浪费空间？
没有关系，这个工作只需要交给 Java 的自动垃圾回收机制（[GC](http://blog.csdn.net/baidu_32045201/article/details/77804085)）去做，你不用担心

通过上面的讲述，我们还可以看出一个 String 的特性，那就是 **可以直接使用 “+”拼接字符串**，当然不仅可以字符串之间拼接，**当其他类型的数据和字符串之间用 “+”连接就会自动转化为字符串类型**



<br>
## **StringBuffer**
前面我们说过了，String 不可变，每 new 一个新 String 出来后，旧 String 的回收交给 JVM 去做，这样一来，虽然我们不用担心内存溢出的问题，但是这无疑增加了工作量，会极大的降低性能，因此我们有了 StringBuffer

我们可以把 StringBuffer 看做是一个 **可修改的 String**
而且 **StringBuffer 是线程安全的**，也就是说它适用于多线程的环境下，在任意时刻，只能有一个线程对 StringBuffer 进行操作，它的每一个方法都被 [synchronized](http://blog.csdn.net/baidu_32045201/article/details/77434996) 所修饰

StringBuffer 中最主要的两个方法是 append 和 insert
~~~java
//append 方法用于将修改的字符串添至原字符串末尾
StringBuffer str = new StringBuffer("hello");
str.append("abc");
System.out.println(str);

//结果：helloabc
~~~

~~~java
//insert 方法用于将修改的字符串添加至指定的位置
StringBuffer str = new StringBuffer("hello");
str.insert(3, "abc");
System.out.println(str);

//结果：helabclo
~~~


<br>
## **StringBuilder**
StringBuilder 是在 JDK 5 中新加入的，它可以看做是 StringBuffer 的单线程版本，**它与 StringBuffer 唯一的区别就是取消了线程同步机制**

因为 StringBuilder 没有线程同步机制，所以 **StringBuilder 的性能要优于 StringBuffer**

因此，如果不是在多线程操作字符串的情况下，**尽量使用 StringBuilder**

其实 StringBuffer 是一个极少被用到的类，因为几乎不会存在多线程修改字符串的情况



