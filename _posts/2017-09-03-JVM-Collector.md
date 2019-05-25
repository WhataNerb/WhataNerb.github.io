---
layout: post
title: 'JVM垃圾收集器'
date: 2017-09-03
categories: Java虚拟机
tags: JVM 虚拟机
---
> 参考文献：周志明《深入理解Java虚拟机》第二版

垃圾收集器就是 GC 具体的实现，不同厂商，不同版本，针对虚拟机不同的分区都有不一样垃圾收集器

图. 基于JDK 1.7 update 14的 HotSpot虚拟机包含的收集器
![这里写图片描述](http://img.blog.csdn.net/20170903152816375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### **Serial 收集器**
这个收集器是一个**单线程**收集器，**在它进行收集工作时，必须暂停其他所有的工作线程，直到收集结束**，这种工作方式又被称为“Stop the World”
它采用**[复制算法](http://blog.csdn.net/baidu_32045201/article/details/77804085)**
因为单线程，所以会造成停顿，但也使得它简单而高效
适用于新生代 **Minor GC**
是运行在 Client 模式下的虚拟机的一个最佳选择
<br>
### **Serial Old 收集器**
它是 Serial 收集器的老年代版本，也是一个 **单线程** 收集器
应用**[标记-整理算法](http://blog.csdn.net/baidu_32045201/article/details/77804085)**
应用于老年代 **Full GC**
主要用于 Client 模式下

图. Serial / Serial Old收集器运行示意图
![Serial / Serial Old收集器运行示意图](http://img.blog.csdn.net/20170903154159738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br>

### **ParNew 收集器**
它是 Serial 收集器的 **多线程** 版

> 注意区分：
> 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态
> 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续执行，而垃圾收集程序运行在另一个CPU上

采用 **收集算法**
应用于新生代 **Minor GC**
适用于 **Server 模式**

图. ParNew / Serial Old 收集器运行示意图
![这里写图片描述](http://img.blog.csdn.net/20170903160553523?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
### **Parallel Scavenge 收集器**
它是一个 **多线程** 收集器
采用 **复制算法**
应用于新生代 **Minor GC**
特点：**可控制吞吐量**

> 吞吐量（Throughput）：CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值
> 吞吐量 = 运行代码时间 / （运行代码时间 + GC 时间）

提供设置参数：
-XX:MaxGCPauseMillis    控制最大 GC 停顿时间
-XX:GCTimeRatio             直接设置吞吐量大学
-XX:+UseAdaptiveSizePolic  动态自适应调整

它无法与 CMS 收集器配合工作，因为 CMS 收集器关注尽可能缩短 GC 时用户线程的停顿时间，与 Parallel Scavenge 的可控吞吐量不合
<br>
###**Parallel Old 收集器**
它是 Parallel Scavenge 收集器的老年代版本
是一个 **多线程** 收集器
采用 **标记-整理算法**
应用于老年代 **Full GC**
经常与 Parallel Scavenge 配合，以充分达到“吞吐量优先”的作用

图. Parallel Scavenge / Parallel Old 收集器运行示意图
![这里写图片描述](http://img.blog.csdn.net/20170903164715929?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
### **CMS 收集器**
CMS（Concurrent Mark Sweep）收集器是一种以**以获取最短回收停顿时间为目标**的收集器
应用 **标记-清除算法**
它是一种 **单线程与多线程结合使用** 的收集器
收集步骤：

 1. 初始标记（CMS initial mark）
 2. 并发标记（CMS concurrent mark）
 3. 重新标记（CMS remark）
 4. 并发清除（CMS concurrent sweep）

初始标记、重新标记仍然要“Stop the world”
CMS 收集器最大的优点就是 **大大减少了停顿时间**
但是，它也有三个明显的缺点：

 ① **CMS 收集器对 CPU 资源敏感**：它的并发清理虽然不会造成停顿，但是会占用 CPU 资源，导致用户程序变慢吞吐量低
 ② **CMS 收集器无法处理浮动垃圾**（Floating Garbage）：因为清理过程是并发的，所以在清理过程中用户也会产生垃圾，这些垃圾叫做浮动垃圾，CMS无法清理，只能等到下一次清理
 ③ CMS 应用“标记-清除算法”（为了快速），所以**会造成空间碎片**

图. CMS收集器运行示意图
![这里写图片描述](http://img.blog.csdn.net/20170903171542337?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
### **G1 收集器**
G1（Garbage-First）收集器可以说是目前最先进的收集器了，它具有如下特点：

 - 并行与并发
 - 分代收集
 - 空间整合
 - 可预测停顿

G1 收集器的先进之处在于，它几乎完全区别于以往收集器的思想，它改变了 Java 堆的格局，它将整个 Java 堆分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代与老年代不再是物理隔离的了，它们都是一部分 Region （不需要连续）的集合
G1 收集器能够预测停顿时间，是因为它可以有计划地避免在整个 Java 堆中进行全区域的垃圾收集，它跟踪各个 Region 里面的垃圾堆积的价值大小，在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region

收集步骤：

 1. 初始标记（Initial Marking）
 2. 并发标记（Concurrent Marking）
 3. 最终标记（Final Marking）
 4. 筛选回收（Live Data Counting and Evavcuation）

前几步与 CMS 收集器非常相似，最后的筛选回收，就是从优先列表中根据预测停顿时间选择回收

图. G1 收集器运行示意图
![这里写图片描述](http://img.blog.csdn.net/20170903172316631?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

G1 收集器非常先进而且非常复杂，可以说还处于试验阶段，并未完全成熟，目前主流的应用最多的收集器还是 CMS 收集器
