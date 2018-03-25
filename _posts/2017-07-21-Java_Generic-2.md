---
layout: post
title: 'Java泛型详解（下）'
date: 2017-07-21
categories: Java基础
tags: Java 泛型 面向对象
---

## **九. 泛型类型的继承规则**
假设现在有一个类Employee和它的子类Manager

现在问题来了：
**Pair&lt;Manager>是Pair&lt;Employee>的子类吗？**

答案是：**不是**

例如，下面的代码将不会编译成功：

~~~java
Manager[] topHonchos = ...;
Pair<Employee> result = ArrayAlg.minmax(topHonchos); //Error
//minmax方法返回Pair<Manager>, 而不是Pair<Employee>, 这样的赋值是不合法的
~~~
也就是说，**无论 S 与 T 有什么联系，通常，Pair&lt;S>与Pair&lt;T>没有什么联系**
也可以这么理解，无论赋给类型变量的类型之间有什么联系，在泛型类里，这些关系都不存在了

另外，泛型类可以扩展或实现其他的泛型类，例如：ArrayList&lt;T>类实现了List&lt;T>接口，这意味着，一个ArrayList&lt;Manager>可以被转换为一个List&lt;Manager>。但是，一个ArrayList&lt;Manager>不是一个ArrayList&lt;Employee>或List&lt;Employee>
<br>
## **十. 通配符类型**
### **1. 通配符的概念：**
假设我们现在需要打印一对员工的方法：

~~~java
public static void printBuddies(Pair<Employee> p) {
	Employee first = p.getFirst();
	Employee second = p.getSecond();
	System.out.println(first.getName() + "and" + second.getName() + "are buddies.");
}
~~~
现在问题来了，Manager虽然不同于普通员工，但他也算是员工，他是员工的子类，但是根据之前说的泛型类继承规则，Pair&lt;Manager>与Pair&lt;Employee>没有任何关系，所以这个方法就不能接受Pair&lt;Manager>类了，难道Manager与普通员工就不能做朋友了吗？不，为了解决这个问题，我们使用**通配符类型**：

~~~java
public static void printBuddies(Pair<? extends Employee> p)
~~~
类型Pair&lt;Manager>是Pair&lt;？ extends Employee>的子类型
可以认为，通配符为泛型类之间添加了一层联系，然而这种联系并不纯粹，我们来看一个例子：

~~~java
Pair<Manager> managerBuddies = new Pair<>(CEO, CFO); //JDK7之后可以省略构造函数的类型变量，由编译器根据语句自己翻译
Pair<? extends Employee> wildcBuddies; //OK
wildcardBuddies.setFirst(lowlyEmployee); //这步就会报错，compile-time error

~~~
究其原因，我们来看对于类型Pair&lt;? extends Employee>内部对 setFirst 和 getFirst 方法的调用，它是这样的：

~~~java
? extends Employee getFirst() //将getFirst返回值赋给一个Employee的引用完全合法
void setFirst(? extends Employee) //出现类型错误，因为编译器只知道需要某个Employee的子类型，但不知道具体是什么类型
~~~
<br>
### **2. 通配符的超类型限定**
通配符还有一个比较强的功能就是可以指定一个超类，像这样：

~~~java
? super Manager //super关键字限制通配符为Manager的所有超类（父类）
~~~
这样的做法与之前的子类型限定恰好相反。这样可以为方法提供参数（setFirst），但不能使用返回值（getFirst）

~~~java
void setFirst(? super Manager)
? super Manager getFirst() 
~~~
直观的讲，**带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对象读取**
<br>
### **3. 无限定通配符**
还可以使用无限定通配符，例如，Pair&lt;?>，初看起来，这好像与原始的Pair类型一样，实际上却有很大的不同，类型Pair&lt;?>有以下的方法:

~~~java
? getFirst()
void setFirst(?)
~~~
getFisrt的返回值只能赋给一个Object。setFirst方法不能被调用，甚至不能用Object调用。Pair&lt;?>和Pair的本质不同在于：可以用任意Object对象调用原始Pair类的setObject方法
<br>
### **4. 通配符捕获**
<br>

> 参考文献：[美]Cay S. Horsimann著 《Java 核心技术 卷I》(第十版)