---
layout: post
title: '线程状态及属性'
date: 2017-08-16
categories: 并发
tags: 并发 Thread 线程
---

## 线程一共有六种状态

- New （新创建）
- Runnable （可运行） 
-  Blocked （被阻塞）    
- Waiting （等待）    
-  Timed waiting （计时等待）    
-  Terminated （被终止）

**可以通过调用 getState 方法，获取一个线程的当前状态**
<br>
### New
基础且常用的创建线程的方式有两种：**继承 Thread 类** 和 **实现 Runnable 接口**
这两种方法各有所长，不过不推荐使用继承 Thread 类的方法，因为：
**Java单继承机制使得继承 Thread 类后无法再继承其他父类**
**继承方法不利于多线程之间共享资源**
但是，继承 Thread 方法相较于实现 Runnable 方法的好处便于获取线程，实现 Runnable 必须通过调用静态方法 Thread.currentThread()来获取当前线程
~~~java
//继承 Thread 类
class MyThread extends Thread {
	public void run(){
		//code
	}
} 

//实现 Runnable 接口
class MyThread implements Runnable {
	public void run(){
		//code
	}
}
//建议这样写
Runnable r = () -> {
	try{
		...
		while (!Thread.currentThread().isInterrupted()){
			...
		}
	} 
	catch(InterruptedException e) {
		...
	} 
	finally {
		...
	}
};
Thread t = new Thread(r);
~~~
<br>
### Runnable
一个线程调用 start 方法后，就处于 runnable 状态
**在任意时刻处于可运行的状态的线程，有可能在运行，也有可能没在运行，这就是为什么叫 *可运行状态* 而不是 *运行状态***
至于Runnable状态线程到底什么时候运行，这取决于操作系统的调度系统
**有三种常见的调度方式：前后台式，抢占式，协作式**
**前后台式：**严格按照前来后到的原则调度线程
**抢占式：**给每个线程分配时间片，当正在执行的线程的时间片用完后，立刻终止运行权，并按照优先级选择下一个线程，而且如果有一个比正在运行的线程优先级高线程可运行时，正在运行的线程立刻停止运行让出运行权
**协作式：**按照优先级调度线程，但是只有正在运行的线程主动停止运行时才会让出运行权
<br>
### Blocked，Waiting
当线程处于 Blocked 或 Waiting 状态时， 它暂时不活动，也不运行任何代码而且消耗最少的资源，直到调度器重新激活它。
以下几种情况会导致线程进入非活动状态：

 - 当一个线程试图获取一个内部的对象锁，而该锁被其他线程持有，则线程进入 *Blocked 状态*，当所有其他线程释放该锁，并且线程调度器允许本线程持有它的时候才会变成 *非阻塞状态*
 - 当线程等待另一个线程通知调度器一个条件时，它自己进入 *等待状态*，在调用 Object.wait 方法或是 Thread.join 方法时，或是等待 java.util.concurrent 库中的 Lock 或 Condition 时，就会出现这种情况
 - 有几个方法有一个超时参数，调用它们导致线程进入 *计时等待状态* ，这一状态将一直保持到超时期满或接受到适当的通知，带有超时参数的方法有 Thread.sleep，Object.wait，Thread.join，Lock.tryLock 及 Condition.await 的计时版
<br>

### Terminated
以下两种原因会导致线程终止：
 ①  run 方法正常退出的自然死亡 
 ②  因为没有捕获异常而终止 run 方法的意外死亡

![这里写图片描述](http://img.blog.csdn.net/20170816174146563?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>
## 线程属性
线程属性包括：**线程优先级，守护线程，线程组，处理未捕获异常的处理器**
<br>
## 优先级
在 Java 中，每个线程都有一个优先级，默认情况下，一个线程继承它的父线程的优先级。可以用 setPriority 方法设置线程的优先级高低，优先级范围在 MIN_PRIORITY（Thread类中为 1）到 MAX_PRIORITY（10）之间
**但是线程的优先级是高度依赖于系统的**，在不同的平台下，优先级设置可能不同
**如果确定要使用优先级，就一定要注意，在高优先级线程完成之前，低优先级线程永远不会运行，因此如果代码出错，或是调度有问题，低优先级线程可能会被饿死**
<br>
## 守护线程
通过调用 t.setDaemon(true)，可以将线程转换为守护线程。
**守护线程唯一的用途是为其他线程提供服务**，例如，计时线程
当只剩下守护线程时，虚拟机就退出了
**守护线程永远不应该访问固有资源，如文件，数据库，因为它们可能随时退出，这会很危险**
