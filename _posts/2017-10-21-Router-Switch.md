---
layout: post
title: '路由器与交换机的区别与联系'
date: 2017-10-21
categories: 网络
tags: 网络
---
相信有很多人在学习网络的过程中，都会对路由器与交换机的区别与联系感到疑惑不解，因为这两台设备的功能看起来似乎一样。然而，其实路由器与交换机大有不同，下面是我对此的一些理解，希望能够帮助到你！

### **它们在哪里工作？**
根据 OSI模型的网络体系划分，自底向上，**路由器* 工作在第三层（网络层）**，而我们常说的***交换机* 工作在第二层（链路层）**（目前有更加高级的三层交换机，四层交换机，甚至还有七层交换机）
![这里写图片描述](http://img.blog.csdn.net/20171021153730816?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
### **它们怎么工作？**
它们的主要工作如下：
**路由器：寻址，转发（依靠 IP 地址）**
**交换机：过滤，转发（依靠 MAC 地址）**

我们可以看出这两者的主要工作就是转发数据，但是不同之处是，依靠的地址不同，这是一个根本区别！

路由器内有一份**路由表**，里面有它的寻址信息（就像是一张地图），它收到网络层的**数据报**后，会根据路由表和选路算法将数据报转发到下一站（可能是路由器、交换机、目的主机）

交换机内有一张**MAC表**，里面存放着和它相连的所有设备的MAC地址，它会根据收到的**数据帧**的首部信息内的目的MAC地址在自己的表中查找，如果有就转发，如果没有就放弃

我们来看一个网络拓扑图例子：
![这里写图片描述](http://img.blog.csdn.net/20171021165847599?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过拓扑图我们应该知道：
**每一个路由器与其之下连接的设备，其实构成一个局域网**
交换机工作在路由器之下，就是也就是**交换机工作在局域网内**
交换机用于**局域网内网的数据转发**
路由器用于**连接局域网和外网**

举个例子：
我们每个人相当于主机，路由器相当于快递员，宿管大爷相当于交换机，学校是一个局域网
快递员根据学校地址（IP）把包裹送到学校，再根据公寓号（子网IP）把快递交给这个公寓的宿管大爷，宿管大爷根据你的名字（MAC）交给你

<br>
### **它们两个可不可以少一个？**
交换机在局域网内工作，它根据 MAC 地址转发数据，如果没有了路由器在网络层寻址，那么我们的数据就不能发送到其他网络终端上去了

路由器内集成了交换机的功能，主机与路由器相连也可以实现数据转发，但是不足之处是：
可扩展的接口不如交换机多
交换机通常由硬件加速转发，路由器主要靠软件寻址，速度慢

<br>
### **实际网络数据转发过程**

> 此处参考：微信公众号：码农翻身，作者：刘欣

通过一个实际网络数据转发的过程，我们可以更好的理解路由器与交换机的区别所在

假设你使用电脑访问www.baidu.com
过程大致如下：
![这里写图片描述](http://img.blog.csdn.net/20171021190018212?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

你的电脑先在应用层打包一个 HTTP报文，然后在传输层在打包成 TCP报文，然后再根据 DNS 查到的 IP 在网络层打包成 IP数据报，然后在通过链路层打包成以太网数据帧，发送给你的交换机：
![这里写图片描述](http://img.blog.csdn.net/20171021185231498?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

你的交换机收到后，重新包装数据帧，再发送给你的路由器：
![这里写图片描述](http://img.blog.csdn.net/20171021190401424?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
你的路由器利用 NAT，将 IP 改为自己的对外的 IP，再根据路由表选择一条合适的路径：
![这里写图片描述](http://img.blog.csdn.net/20171021190539539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在接下来的过程中，每个节点都只改变 MAC 地址，然后在网络中一路向着目的地发送


