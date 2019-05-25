---
layout: post
title: 'Java泛型详解（上）'
date: 2017-07-19
categories: Java
tags: Java 泛型 面向对象
---
## **一. 什么是泛型**
泛型是一种程序设计手段（机制），使用泛型可以让你的代码被很多不同类型的对象所重用，提高代码的**重用性**，还可以提高代码的**可读性**和**安全性**

比如，我们经常使用的**ArrayList**类，就是一个泛型类，也正因如此，它可以接受很多不同类型的对象

~~~java
/*
可以根据需要存储不同类型的对象
*/
ArrayList<Integer> arraylist = new ArrayList<Integer>();
ArrayList<String> arraylist = new ArrayList<String>();
ArrayList<File> arraylist = new ArrayList<File>();
......
/*
在Java SE 7及以后的版本中，可以省略构造函数中的泛型类型，即可写为：

ArrayList<Integer> arraylist = new ArrayList<>();

编译器会根据变量的类型推断出泛型类型
*/
~~~
<br>
## **二. 为什么要用泛型**
如上所说，使用泛型可以提高代码对不同类型的重用性，但是实际上，在泛型出现之前，Java也有对不同类型重用代码的运行机制，那就是使用**[Object类](http://blog.csdn.net/baidu_32045201/article/details/75453295)**，例如，ArrayList类就只维护一个Object引用的数组

~~~java
//泛型机制之前
public class ArrayList { 
	private Object[] elementData;
	...
	public Object get(int i){...}
	public void add(Object o){...}
}
~~~
这样的做法会带来两个问题：

①. 当获取一个值时，必须进行强制类型转换

~~~java
Array files = new ArrayList();
...
String filename = (String)files.get(0);
~~~

②. 没有错误检查

~~~java
files.add(new file("...")); 
 //不会报错,但是在获取filename转换成String是就会出错
~~~
然而在泛型机制中，提供了**类型参数**来解决这一问题，并且提高了代码的可读性和安全性

~~~java
ArrayList<String> files = new ArrayList<String>();
~~~
<br>
## **三. 定义泛型类**

~~~java
public class Pair<T> {  //在类名后加类型变量T, 用尖括号<>括起来
	private T first;
	private T second;

	public Pair() {
		first = null;
		second = null;
	}
	public Pair(T first, T second) {
		this.first = first;
		this.second = second;
	}
	
	public getFirst() { return this.first; }
	public getSecond() { return this.second; }

	public setFirst(T first) { this.first = first; }
	public setSecond(T second) { this.second = second; }
}
~~~
类型变量使用大写形式，且比较短。在Java库中：

**E** 表示**集合元素类型**
**K 和 V** 表示表的**关键字与值的类型**
**T** 表示“**任意类型**”（在需要多个类型变量时，还可以使用**U和S**）

<br>

## **四. 定义泛型方法**
![图片来自百度百科](https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0=baike150,5,5,150,50/sign=31869f5ed31373f0e13267cdc566209e/8ad4b31c8701a18b6713156a9c2f07082838fe1f.jpg)

**泛型方法可以定义在普通类中，也可以定义在泛型类中**

当我们调用泛型类时，在尖括号中可以指明放入参数的具体类型，但是也可以不直接写出，因为编译器会推断出参数的类型

~~~java
public static <T> T getMiddle(T... a) {
	return a[a.length / 2];
}

String middle = <String>getMiddle("hello", ",", "world"); //OK
String middle = getMiddle("hello", ",", "world"); //also OK
~~~

在大多数情况下，这样调用泛型方法没有任何问题，但是这里面还是有坑的，比如：

~~~java
double middle = getMiddle(3.14, 1592, 6);
//这样，编译器就会自动将参数打包为一个Double，两个Integer
~~~
解决方法就是将所有的参数都写成double

注：当你要用的数据是浮点数类型时，即便它是个整数，也最好带上小数点，这样既可以防止出错，也可以让别人阅读代码时明白这是浮点数
<br>
## **五. 类型变量的限定**
在一些情况下，我们必须要对我们的泛型类或方法的类型变量做一些限定才能保证程序的正确执行，比如：

~~~java
class ArrayAlg {
	public static <T> T min(T... a) {
		if (a == null || a.length == 0)
			return null;
		T smallest = a[0];
		for (int i = 1; i < a.length; i++)
			if(smallest.compareTo(a[i]) > 0) //不是所有类都有compareTo方法
				smallest = a[i]
		return smallest;
	}
}
~~~
在上面的例子中，不是所有的类都有compareTo方法，为了保证程序的正确执行，我们就必须限制所有调用此方法的类型变量 T 都实现了Comparable接口（只含一个方法compareTo的标准接口）

我们可以这样写：

~~~java
public static <T extends Comparable> T min(T... a){...}
~~~
这样一来，如果调用这个方法的类没有实现Comparable接口，就会出现编译错误

**注意：**这里的extends并不是继承的意思，而是绑定的意思。我们也可以对类型变量做多个限定，在多个限定中用“&”作间隔

~~~java
<T extends Comparable & Serializable>
~~~
<br>
## **六. 类型擦除**
Java代码都是跑在虚拟机里的，这个大家都知道，但是，**在虚拟机里并没有泛型类的对象**，一切的对象都是普通类，泛型机制只不过是一种方便我们重用代码的技术手段而已，那么泛型代码在虚拟机中使如何解释执行的呢？

无论何时定义一个泛型类型，都会自动提供一个相应的**原始类型**，原始类型的名字就是删去类型参数后的泛型类型名。擦除类型变量，**并替换为所限定的类型**（如果没有限定类型就用Object类），比如：

~~~java
public class Pair {
	private Object first; //对 T 没有做限定所以用Object类替换
	private Object second;

	public Pair(Object first, Object second) {
		this.first = first;
		this.second = second;
	}

	public Object getFirst() { return this.first; }
	public Object getSecond() { return this.second; }

	public void setFirst(Object first) { this.first = first; }
	public void setSecond(Object second) { this.second = second; }
}
~~~
如果有限定类型，就用第一个限定类型替换：

~~~java
public class Interval <T extends Comparable & Serializable>
			implements Serializable {
	private T first;
	private T second;
	...
	public Interval(T first, T second){...} 
}

//替换后
public class Interval implements Serializable {
	private Comparable first;
	private Comparable second;
	...
	public Interval(Comparable first, Comparable second){...} 
}
~~~
<br>
## **七. 桥方法**
类型擦除也会出现在泛型方法中
然而，类型擦除与Java的多态之间会有一些小矛盾，比如：

~~~java
class DateInterval extends Pair<LocalDate> {
	public void setSecond(LocalDate second) {
		if (second.compareTo(getFirst()) >= 0)
			super.setSecond(second);
		}
	...
}
	
~~~
在上面的例子中Pair类也有一个setSecond方法，而它的setSecond方法和DateInterval类中的不一样

Pair类中的setSecond方法类型擦除后:

~~~java
public void setSecond(Object second) {
	this.second = second;
}
~~~
那么这时，在我们调用setSecond方法时，就会出现冲突（因为Object类是一切类的父类，所以无法简单地根据参数类型选择调用的函数）

为了解决这一问题，编译器会在DateInterval类中生成一个**桥方法**:

~~~java
public void setSecond(Object second) { 
	setSecond((Date) second);
}
~~~
也就是说，桥方法会根据所引用的对象进行强制类型转换，来调用最合适的那个方法

假设DateInterval方法也覆盖了getSecond方法：

~~~java
class DataInterval extends Pair<LocalDate> {
	public LocalDate getSecond() {
		return (Date) super.getSecond().clone();
		//这里调用clone方法是为了防止原数据被修改
	}
}
~~~
总结一下，关于Java泛型转换，我们要记住：

 - 虚拟机中没有泛型，只有普通的类和方法 
 - 所有的类型参数都用它们的限定类型替换
 -  桥方法被合成来保持多态
 - 为保持类型安全性，必要时插入强制类型转换

<br>
## **八. 泛型的约束与局限性**
### 1. 不能用基本类型实例化类型参数
不能将基本类型写入类型参数

~~~java
//错误！！！
Pair<int> ， Pair<float>, Pair<double>, Pair<boolean>, ...

//正确
Pair<Integer>, Pair<Float>, Pair<Double>, Pair<Boolean>, ...
~~~
### 2. 运行时类型查询只适用于原始类型
虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型

无论使用instanceof, getClass()或是强制类型转换都会导致错误

~~~java
if (a instanceof Pair<String>) //Error
if (a instanceof Pair<T>) //Error

Pair<String> p = (Pair<String>)A; 
//Warning--can only test that A is a Pair

Pair<String> stringPair = ...;
Pair<Integer> integerPair = ...;
if (stringPair.getClass() == integerPair.getClass())
//always equal， 因为两次调用getClass都返回的是Pair.class
~~~
### 3. 不能创建参数化类型的数组

~~~java
Pair<String>[] table = new Pair<String>[10]; //Error
~~~
以上代码的错误在于，在类型擦除后，table的类型是Pair[]，可以把它转换为Object[]

~~~java
Object[] objarray = table;
~~~

数组会记住它的元素类型，如果试图存储其他类型的元素时，就会抛出一个ArrayStoreException异常

~~~java
objarray[0] = "Hello"; //Error--component type is Pair
~~~
当然，如果你很机智的这样写

~~~java
objarray[0] = new Pair<String>();
~~~
这样的确会通过数组存储检查，但是还是会导致一个类型错误
那么，如果需要收集参数化类型对象，只有一种安全而有效的方法：使用ArrayList

~~~java
ArrayList<Pair<String>>
~~~
### 4. Varargs警告
我们来看这样一个方法

~~~java
public static <T> void addAll(Collection<T> coll, T... ts) {
	for (t : ts) coll.add(t);
}
~~~
如果我们想调用这样一个方法来对一些泛型类操作，遵循之前的原则，我们要这样写

~~~java
Collection<Pair<String>> table = ...;
Pair<String> pair_1 = ...;
Pair<Stirng> pair_2 = ...;
...
addAll(table, pair_1, pair_2, ...);
~~~
但是，为了调用这个方法，Java虚拟机必须建立一个Pair &lt;String>数组，虽然这违反了之前的规则，但是这并不会导致错误，你只会得到一个警告，有两种方法抑制这个警告：
① 为包含addAll调用的方法增加注解 @SuppressWarnings("unchecked")
② 在Java SE 7之后，还可以用@SafeVarargs直接标注addAll方法

~~~java
@SafeVarargs
public static <T> void addAll(Collection<T> coll, T... ts)
~~~
### 5. 不能实例化类型变量
不能使用像new T(...)，new T[...] 或 T.class这样的表达式中的类型变量。例如，下面的Pair&lt;T>构造器就是非法的

~~~java
public Pair() { first = new T(); second = new T(); } //Error
~~~
### 6. 不能构造泛型数组 
同样由于类型擦除，你无法保证你所构造的泛型数组在虚拟机内是你需要的类型。如果你只将数组作为一个类的私有实例域，就可以将这个数组声明为Object[]，并在获取元素时进行类型转换，但是这其中还是会有不少的安全隐患
### 7. 泛型类的静态上下文中的类型变量无效
### 8. 不能抛出或捕获泛型类的实例
泛型类扩展Throwable是不合法的

~~~java
public class Problem<T> extends Exception {...} //Error
~~~
catch子句中不能使用类型变量。例如，以下方法不能通过编译:

~~~java
public static <T extends Throwable> void doWork(Class<T> t) {
	try
	{
		...
	}
	catch (T e) //Error--can't catch type variable
	{
		...
	}
}
~~~
但是，在异常规范中使用类型变量是允许的。以下方法是合法的:

~~~java
public static <T extends Throwable> void doWork(T t) throws T //OK{
	try
	{
		...
	}
	catch (Throwable realCause)
	{
		...
	}
}	
~~~

### 9. 可以消除对受查异常的检查
### 10. 注意擦除后的冲突

> 参考文献 [美]Cay S.Horstmann著 《Java核心技术 卷I》（第十版）