---
layout: post
title: 'Java中的等待-通知机制'
date: 2020-06-17
categories: Java
tags: Java基础 并发 互斥锁 synchronized
---
# 为什么有等待-通知机制？

首先，设想这样一种场景：一个线程的执行需要满足某些条件，当条件不满足时就通过一个循环不断尝试，直到条件满足。

这个场景下存在一个明显的缺点，就是线程不断地尝试获取所需的条件，这个循环的过程会白白浪费CPU资源，降低系统性能。

等待-通知机制是一种优化策略，其核心思想就是：**当线程所需条件不满足时，就阻塞该线程，之后当条件满足时再通知线程，以此提高硬件资源的利用率**

# Java 中的实现

在 Java 中实现等待-通知机制，一种经典的做法是使用 **synchronized + wait() + notify() / notifyAll()**

我们可以通过一个经典的面试题来讲解这种用法：

> 请使用等待-通知机制实现两个线程交替打印 1~100

这个问题的解法有很多，我这里展示一下我自己的方法：

```java
public class AlternatelyPrint {

    private int curNum = 0;

    synchronized void printNum(int n) {
        // 用 n - curNum 判断是否轮到当前打印数字
        while (n - curNum != 1) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName() + ":\t" + n);
        curNum += 1;
        notifyAll();
    }

    public static void main(String[] args) {
        AlternatelyPrint alternatelyPrint = new AlternatelyPrint();

        // OddThread 打印奇数
        new Thread(() -> {
            for (int i = 1; i <= 100; i += 2) {
                alternatelyPrint.printNum(i);
            }
        }, "OddThread").start();

        // EvenThread 打印偶数
        new Thread(() -> {
            for (int i = 2; i <= 100; i += 2) {
                alternatelyPrint.printNum(i);
            }
        }, "EvenThread").start();
    }

}
```

这段代码创建了两个线程 `oddThread` 和 `evenThread` 它们分别负责输出 1~100 之间的奇数和偶数。

`AlternatelyPrint` 中的 `curNum` 是共享变量，用于记录当前整个程序输出的位置，随着两个线程的交替输出，`curNum` 会随之递增

`printNum()` 为同步方法，每次只允许一个线程进入。在进入方法时线程首先通过 `n - curNum` 来判断当前是否轮到自己输出，如果不是则调用 `wait()` 将自己阻塞并且解锁释放资源，让其他线程来执行；如果是则正常执行方法，执行完成后调用 `notifyAll` 通知等待队列中正在等待的线程来执行。

其实，如果你熟悉操作系统的话应该知道，Java 中 `synchronized` 其实就是操作系统中**管程**的一种实现，`synchronized` 属于 Java 语言层面的管程实现。但是它的使用不灵活，而且效率较低，因此 JDK 中提供了更加灵活，更加高效的管程实现 `Lock` 和 `Condition`

**Lock & Condition 的优势**

- 支持公平锁与非公平锁
- 支持中断响应、超时、非阻塞式获取锁
- 支持关联多个条件队列

# Tips

**1. 总是在 synchronized 内部调用 wait() + notify() + notifyAll()**

阻塞与唤醒线程的前提一定是必须先持有互斥锁，有锁我们才有权利决定是否释放共享资源。其次还要注意，必须清楚当前的互斥锁的对象是谁，即 synchronized 锁的范围是什么，默认如果不加指定的话就是 `this` 对象，如果试图在 synchronized 锁定的范围外调用阻塞或唤醒方法，将会导致 `java.lang.IllegalMonitorStateException`

**2. 总是在循环内使用 wait()**

官方在 `Object.wait()` 的注释中给出了这样的建议。因为如果不适用循环可能会导致**虚假唤醒**的问题。`notify()` 和 `notifyAll()` 只会告诉被阻塞的线程它所需要的条件**曾经**满足过，而被通知线程的执行时间点和通知的时间点基本上不会重合，所以当线程执行的时候，很可能条件已经不满足了（保不齐有其他线程插队）。**当被阻塞线程被唤醒后，是从 `wait()` 方法之后开始继续运行的**，如果此时已经被其他线程插队而导致条件不满足，将会导致程序执行出错。

**3. 尽量使用 notifyAll()**

**notify() 会随机地通知等待队列中的一个线程，而 notifyAll() 会通知等待队列中的所有线程。**因为 `notify()` 是随机唤醒一个线程的，因此可能会导致某个线程一直得不到唤醒的情况。建议当满足以下三个条件时使用 `notify()`:

- 所有等待线程拥有相同的等待条件
- 所有等待线程被唤醒后，执行相同的操作
- 只需要唤醒一个线程



> 这篇文章是我学习王宝令老师讲授的《Java并发编程实战》的笔记和总结
>
> [极客时间 王宝令 《Java并发编程实战》](https://time.geekbang.org/column/intro/159)

