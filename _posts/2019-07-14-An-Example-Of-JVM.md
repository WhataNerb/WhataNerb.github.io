---
layout: post
title: '一个关于JVM类加载过程的案例'
date: 2019-07-14
categories: Java虚拟机
tags: Java虚拟机 
---

通过谜题探索答案，是一个很好的学习过程:point_up:。首先，我们就从一个案例出发：

```java
public class Test {
	public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        
		System.out.println("[main]num1: " + num1);
        System.out.println("[main]num2: " + num2);
	}
}

class Singleton {
	
	public static int num1;
    public static int num2 = 0;
	
	private static Singleton singleton = new Singleton();
	
	private Singleton() {
		num1++;
		num2++;
		
		System.out.println("[cons]num1: " + num1);
		System.out.println("[cons]num2: " + num2);
	}
	
	public static Singleton getInstance() {
		return singleton;
	}
	
}
```

请观察上面的代码，你觉得输出会是什么？:thinking:

也许你会觉得这报错了吧？:thinking:

其实不会，它的输出如下：

```
[cons]num1: 1
[cons]num2: 1
[main]num1: 1
[main]num2: 1
```

:open_mouth::exclamation:为什么不报错呢？num1不是没有赋初值么？

不要着急，我们一起来探索答案:mag:

我们知道，Java是一个半编译半解释的语言，Java代码先编译成class文件，然后再由虚拟机解释运行。其中，把class文件中的数据加载到内存中准备执行的过程就叫做JVM的类加载机制。

JVM的类加载机制，有以下几个阶段：

![类的生命周期](https://img-blog.csdnimg.cn/20190714165122310.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)

关于类加载机制的具体过程是一个长篇大论的内容，本文只关注于我们谜题中涉及的部分。

我们要知道，在上图的 **准备（Preparation）** 阶段，虚拟机会为每一个**静态变量**赋予默认的初始值（也称**零值**）

:point_up:*注意*:exclamation:，*此时的虚拟机并不关心程序员是否为此变量赋初始值或赋了什么值*

| 数据类型 |   零值    | 数据类型  | 零值  |
| :------: | :-------: | :-------: | :---: |
|   int    |     0     |  boolean  | false |
|   long   |    0L     |   float   | 0.0f  |
|  short   | (short) 0 |  double   | 0.0d  |
|   char   | '\u0000'  | reference | null  |
|   byte   | (byte) 0  |           |       |

因此，之前的谜题中，num1即使没有赋值虚拟机也会为它赋上零值。

到了 **初始化（Initialization）** 阶段时，虚拟机才会去关注程序员所赋予的初始值！而且还要注意一点：**所有的java虚拟机实现必须在每个类或接口被java程序首次主动使用时才初始化他们**，关于这点本文不详谈。

大牛们:cow:到这里可能已经觉得不耐烦了，等等别走:wave:！，真正的好戏才刚刚开始:dark_sunglasses:！

如果把上面代码中的一行换个顺序：

```java
public class Test {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        
        System.out.println("[main]num1: " + num1);
        System.out.println("[main]num2: " + num2);
    }
}

class Singleton {
    
    public static int num1;
    
    //我刚才在这里
    //public static int num2 = 0;
    
    private static Singleton singleton = new Singleton();
    
    private Singleton() {
        num1++;
        num2++;
        
        System.out.println("[cons]num1: " + num1);
        System.out.println("[cons]num2: " + num2);
    }
    
    //现在到这儿来啦！
    public static int num2 = 0;
    
    public static Singleton getInstance() {
        return singleton;
    }
    
}
```

这样，输出又是什么呢？:thinking:

直接甩答案：

```
[cons]num1: 1
[cons]num2: 1
[main]num1: 1
[main]num2: 0    //ATTENTION!!!
```

:thinking:嗯~~有点儿意思

这其中的陷阱在于我们一定要记住，类加载时，对变量的准备和初始化都是按照**顺序执行！！！**的

如上代码中，变量的顺序为：:one:num1，:two:singleton，:three:num2

我们按照类加载机制和变量的顺序捋一遍这段代码的加载过程

首先在准备阶段：

1. num1赋予零值，num1 == 0

2. singleton赋予零值，singleton == null
3. num2赋予零值，num2 == 0

接下来到了初始化阶段，注意顺序还是那个顺序：

1. num1没有人为赋值，num1 == 0

2. singleton == new Singleton()，此时执行构造方法Singleton()。在构造方法中num1++; num2++;因此会输出

   ```
   [cons]num1: 1
   [cons]num2: 1
   ```

3. 最后到了num2初始化，它被人为初始化为0，所以最终num2 == 0，输出

   ```
   [main]num1: 1
   [main]num2: 0
   ```

好了！谜底揭晓了！:tada:希望这篇文章能够让你学到知识！当然，JVM的类加载机制远不止这些，还有很多有趣的谜题等着我们去探索！:grinning:

最后，一个小练习，上面的代码如果这样修改，会输出什么？

```java
public class Test {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        
        System.out.println("[main]num1: " + num1);
        System.out.println("[main]num2: " + num2);
    }
}

class Singleton {
    
    public static int num1 = 1;

    private static Singleton singleton = new Singleton();
    
    private Singleton() {
        num1++;
        num2++;
        
        System.out.println("[cons]num1: " + num1);
        System.out.println("[cons]num2: " + num2);
    }
    
    //现在到这儿来啦！
    public static int num2 = 0;
    
    public static Singleton getInstance() {
        return singleton;
    }
    
}
```

聪明的你一定能给出答案，正确答案就是：

```
[cons]num1: 2
[cons]num2: 1
[main]num1: 2
[main]num2: 0
```

> 参考资料：
>
> [1] 张龙，《深入理解JVM》，http://www.iprogramming.cn/
>
> [2] 周志明，《深入理解java虚拟机：JVM高级特性与最佳实践》（第2版）

