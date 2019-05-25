---
layout: post
title: 'Class类文件结构概述'
date: 2017-09-10
categories: Java虚拟机
tags: JVM 虚拟机
---
> 参考文献：周志明《深入理解Java虚拟机》第二版

## **关于Class文件**
class 文件应该是所有 Java 程序员都必定知道的文件，因为我们的 Java 源代码经过 javac 编译以后就会得到这个 class 文件。但是，它虽然与 Java 关系密切可它并不仅仅属于 Java，因为**它属于虚拟机，虚拟机能够执行的就是 class 文件**。所以我们 Java 虚拟机，其实就根本不认识 Java。
        不过，也正是因为 JVM 只认识这个 class 文件，才使得 Java 有了最NB的一大特色：“一次编译，到处运行”。JVM 无论在哪，只需要带着这个 class 文件就能跑代码。但是，其实这种平台无关性也并不是最NB的地方，因为 JVM 的设计者们想要实现的最终目标是 **语言无关性** ，就是让其他的语言也能跑在 JVM 上，真正的实现“大一统”，比如现在一些语言就推出了 JVM 版本，例：Jython（Python）, JRuby（Ruby）,  Rhino（JavaScript），Groovy 等。所以 Java 程序员们，不要担心 Java 会被替代，好好学 JVM ，以后一定前途无量啊！

好了，言归正传

<br>

## **Class文件结构概述**
因为 class 文件结构这里比较繁琐，而且大部分是固定的数据结构，所以我只简单的整理一下，详细的信息请从参考资料查阅

> **class 文件是一组以 8 位字节为基础单位的二进制流**，各个数据项目按照顺序紧凑地排列在 class 文件中，**中间没有添加任何分隔符**，这使得整个 class 文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。**当遇到需要占用 8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个 8 位字节进行存储**

class 文件只有两种数据类型：**无符号数，表**
无符号数是基本类型，以 u1, u2, u4, u8 分别代表 1 个字节，2 个字节，4 个字节， 8 个字节的无符号数
表是由多个无符号数或其他表作为数据项构成的复合数据类型，所有表都以“_info”结尾，甚至可以说整个 class 文件就是一张表
![这里写图片描述](http://img.blog.csdn.net/20170910000506799?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这张表就是 class 文件中包含的所有数据项
我们接下来逐一进行介绍

### **魔数（magic number）与版本（version）**
从表中可以看出，class 文件的第一个数据项是一个 u4 类型，占 4 个字节的 **魔数**，它的唯一作用是**用来确定该 class 文件是否可以被虚拟机接受**，这个作用就相当于文件的后缀名，.txt，.jpg，.gif，.mp4等等，那为什么不用后缀名呢？因为这样隐式的更加安全，class 文件的 magic number 很有意思，它的值为：**0xCAFFBABE** （咖啡宝贝），这个名字在 Java 还叫 Oak 的时候就确定了缘由是开发人员喜欢喝咖啡，这也是 Java 的 logo 的由来

接着后面是 **2 个字节的次版本号（minor version）**和 **2 个字节的主版本号（major version）**

<br>
### **常量池（constant_pool）**
再下来是就是 class 文件的资源仓库 **常量池** 了，它是 class 文件结构中与其他项目关联最多的数据类型，也是占用空间最大的数据项之一
因为常量池的大小不固定，它由程序决定，所以在它之前是一个 **u2 的常量池容量计数值（constant_pool_count）**（*后面的数据项都是一样，所有 _info 类型的数据项大小都是由程序决定，所以前面都有一个计数值*）
这里需要注意的是，**constant_pool_count 是从 1 开始计数的，空出 0 是为了表示“不引用任何一个常量池项目”**

常量池中主要存放两大类常量：**字面量**，**符号引用**
字面量：文本字符串，final常量等
符号引用：类和接口全限定名，字段的名称和描述，方法的名称和描述

**常量池中每一项常量都是一个表**，目前有这 14 种
![这里写图片描述](http://img.blog.csdn.net/20170910003232251?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这其中每一个类型的表都还有自己的结构
例：
![这里写图片描述](http://img.blog.csdn.net/20170910003410845?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170910003518615?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
所以这也是常量池占存大的原因

<br>
### **访问标志（access_flags）**
常量池结束后，就是 **u2 的访问标志**，**它用来识别一些类或接口层次的访问信息**
例：判断是 class 还是 interface，标记修饰符 public, protected, abstract等
![这里写图片描述](http://img.blog.csdn.net/20170910004125054?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>
###**类索引（this_class），父索引（super_class），接口索引集合（interfaces）**
类索引为 u2，父索引为 u2，接口索引集合为一个 u2 集合（前面还有一个 u2 的计数器 interface_count）
根据 Java 语言的规则，除了 java.lang.Object 之外，其他所有类的父类均不为 0 
![这里写图片描述](http://img.blog.csdn.net/20170910005511685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170910005541917?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
### **字段表集合（fields）**
**字段表用来描述接口或者类中声明的变量**，**字段包括类级变量和实例级变量，但不包括在方法内部声明的局部变量。**
![这里写图片描述](http://img.blog.csdn.net/20170910005947336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
access_flags 就是用来标记字段作用域，修饰符等信息
![这里写图片描述](http://img.blog.csdn.net/20170910010412489?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
注意，从表中我们可以看出，这些字段标记是严格按照 Java 规则制定的，例：
private int m 的 access_flags 就为：0002
public static int m 的 access_flags 就为：0001（public）+ 0008（static）= 0009
protected final double m 的 access_flags 就为：0004（protected）+ 0010（final）= 0014
**然后是 name_index 和 descriptor_index ，分别代表字段的简单名称和字段和方法的描述符**
描述符规则：基本类型用字母表示，数组用“[”表示，方法参数用“()”括起来
![这里写图片描述](http://img.blog.csdn.net/20170910011936354?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
例：
void inc()  描述符为：“()V”
java.lang.String.toString() 描述符为：“()Ljava/lang/String;”
int indexOf(char[]source,int sourceOffset,int sourceCount,char[]target,int targetOffset,int targetCount,int fromIndex) 描述符为：“([CII[CIII)I”

字段表集合中不会列出从超类或者父类接口中继承的字段，但可能列出原本 Java 代码中不存在的字段，如，在内部类中添加指向外部类实例的字段（保持外部访问性）
对 Java 来说，无论描述符如何，字段都不能重名
对字节码来讲，如果字段描述符不同，重名也是合法的

<br>
### **方法表集合（methods）**
这部分的结构与字段表几乎相同
![这里写图片描述](http://img.blog.csdn.net/20170910100502212?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里针对字段与方法的不同，在访问标志上有修改

![这里写图片描述](http://img.blog.csdn.net/20170910100559254?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

仔细查看方法表结构，你会发现，方法中的代码并不在其中，因为，方法里的  Java 代码，经过编译器编译成字节码指令后，存放在方法属性表集合（attributes）中一个名为“Code”的属性里

<br>
### **属性表集合（attributes）**
class 文件对于这一数据项的要求比较宽松，它不必有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表写入自己定义的属性信息（支持用户自主扩展）
在最新的《Java虚拟机规范（Java SE 7）》中有 21 项预定义的属性，
例：
![这里写图片描述](http://img.blog.csdn.net/20170910102051087?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
完整表格请查阅参考资料

所有预定义或是自定义的属性都应该符合属性表结构
![这里写图片描述](http://img.blog.csdn.net/20170910102300662?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这其中的每一个属性都有自己的结构表，里面包含了其所定义的一些数据