---
layout: post
title: '关于Java互斥锁'
date: 2020-06-11
categories: Java
tags: Java基础 并发 互斥锁 synchronized
---
一切从一个例子开始：

```java
class SafeCalc {
    long value = 0L;
    
    long get() {
        return value;
    }
    
    synchronized void addOne() {
        for (int i = 0; i < 3; i++) {
            value += 1;
        }
    }
}
```

现在假设我们启动 5 个**写线程**执行 `addOne()`，启动 3 个**读线程**执行 `get()`

请问，存在并发安全问题吗？

答案：**存在**，因为 `value` 值对于 `get()` 方法的可见性是无法保证的，我们只对 `addOne` 加了互斥锁，所以只能保证所有执行 `addOne()` 线程的同步，而执行 `get()` 的线程没有加锁，所以它可以在任意时刻访问 `value`

为代码添加输出后，观察结果：

![QQ截图20200611163701](https://pic1.zhimg.com/80/v2-6ba50d09cd35cd88669797c69e035dfc_720w.jpg)

可以看到，`get()` 会在 `addOne()` 还未结束就读取 `value`，所以会读到一个中间值（即非 3 的倍数）

为 `get()` 添加 `synchronized` 关键字可以修复这个同步问题。

![img](https://static001.geekbang.org/resource/image/26/f6/26a84ffe2b4a6ae67c8093d29473e1f6.png)

你可以尝试观察修复后的执行结果，无论执行多少次，`get()` 读到的 `value` 值总是 3 的倍数（即每个线程的操作是同步进行的）

那么下面对代码再做一次修改：

```java
class SafeCalc {
    static long value = 0L;
    
    synchronized long get() {
        return value;
    }
    
    synchronized static void addOne() {
        for (int i = 0; i < 3; i++) {
            value += 1;
        }
    }
}
```

请问，现在还存在并发安全问题吗？

答案：**存在**

这就涉及到 Java 中类锁与实例锁了，对于 synchronized 关键字 Java 中有一条隐式规则：

> 当修饰静态方法时，锁定的是当前类的 Class 对象
>
> 当修饰非静态方法时，锁定的是当前实例对象的 this

其实简单地说，类锁保证类中静态方法和变量的线程安全，而实例锁保证非静态方法和变量的线程安全

这并不难理解，因为对于静态方法和变量，在 JVM 中只会在方法区中保存一份由所有的实例对象共享，而非静态的变量和方法就是每一个实例私有的

![img](https://static001.geekbang.org/resource/image/60/be/60551e006fca96f581f3dc25424226be.png)

因此，这里出现的问题就是，**给每个方法都上一个锁，那等于整体没上锁**，因为 `get()` 并不需要等待 `addOne()` 的锁释放，所以还是会出现读到中间值的情况

上面的实例最终的解决方案是对 `get()` 和 `addOne()` 添加同一个锁（即对 this 实例加锁），这种加锁方式是一种 **N:1** 的方式，就是多个共享资源由同一把锁保护的方式。

这种方式实现简单，只需要在每一个受保护的资源上加一个 `synchronized` 即可。但它还有一个很明显的缺点就是**效率低下**。

举个例子，一个银行的转账系统包含四个操作：取款、查看余额、修改密码、查看密码。

如果使用同一把锁来管理这四个操作，会使得整套操作都是串行进行的。而我们不难看出，在这四个操作中：（取款、查看余额），（修改密码、查看密码）两组之间并不存在竞争关系，因此我们可以对两组操作分别采用两把锁管理，这样可以两组操作并行执行，大大提高效率。**这种对特定资源精细化管理的锁，也可以称之为细粒度锁**

但是，实际业务中事情往往不是那么简单的！**锁的粒度究竟能够细化多少，需要看对象之间的业务联系**

例如，如果新增一个转账业务，那么这个如何保证用户之间转账操作的**原子性**呢？

>“**原子性”的本质是什么？**其实不是不可分割，不可分割只是外在表现，其本质是多个资源间有一致性的要求，**操作的中间状态对外不可见**。例如，在 32 位的机器上写 long 型变量有中间状态（只写了 64 位中的 32 位），在银行转账的操作中也有中间状态（账户 A 减少了 100，账户 B 还没来得及发生变化）。**所以解决原子性问题，是要保证中间状态对外不可见。**

```java
class Account {
    private int balance;
    // 转账
    void transfer(
        Account target, int amt){
        if (this.balance > amt) {
            this.balance -= amt;
            target.balance += amt;
        }
    } 
}
```

也许第一直觉会是这样的：

```java
class Account {
    private int balance;
    // 转账
    synchronized void transfer(
        Account target, int amt){
        if (this.balance > amt) {
            this.balance -= amt;
            target.balance += amt;
        }
    } 
}
```

可惜，这样根本行不通，因为这里 `synchronized` 是一个实例锁，它只能保护 `this` 实例，就相当于我们不能用自己家的锁去锁别人家的门。

因此，这里就需要提高 `synchronized` 的粒度，解决方案可以是这样的：

```java
class Account {
    private int balance;
    // 转账
    void transfer(Account target, int amt){
        synchronized(Account.class) {
            if (this.balance > amt) {
                this.balance -= amt;
                target.balance += amt;
            }
        }
    } 
}
```

这里将锁的粒度提升至类锁，Account.class 对所有的 Account 实例是唯一且共享的，因此即便是不同的实例对象运行到这里时都会使用同一把锁，这样就保证了不同实例之间的线程安全。

做了这么多的工作，看起来已经不错了，不过很可惜，还是不能上线，还是因为锁的粒度太大导致性能太差了！使用类锁，等同于硬生生地把转账操作变成了串行，设想银行系统每天上亿用户执行转账操作，那么就意味着这上亿的操作必须串行执行，这显然是不可接受的！还需要优化，那就是 `wait - notify` 机制。

（先写这么多吧:grimacing:）

> 这篇文章是我学习王宝令老师讲授的《Java并发编程实战》的笔记和总结
>
> [极客时间 王宝令 《Java并发编程实战》](https://time.geekbang.org/column/intro/159)