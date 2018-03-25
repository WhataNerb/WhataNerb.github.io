---
layout: post
title: 'TCP与UDP通信协议及Java实现'
date: 2017-09-18
categories: Java 网络
tags: Java 网络
---
## **概述**
TCP （Transmission Control Protocol）：传输控制协议
UDP（User Datagram Protocol）：用户数据报协议

TCP 与 UDP 都是 **运输层（Transport Layer）**上的因特网协议，运输层协议的功能就是为运行在不同主机上的应用进程之间提供 **逻辑通信** ，使得运行不同进程的主机即使分隔于地球两侧，也能像是直接相连一样。
而具体做法是，它为来自应用层的报文**添加上运输层首部**来创建运输层报文段，这个首部中包含了以下信息
![这里写图片描述](http://img.blog.csdn.net/20170918130005976?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
通过这些首部字段，不同主机之间的进程就可以通信了

**UDP** 为调用它的应用程序提供了一种 **不可靠的 无连接** 服务
**TCP** 为调用它的应用程序提供了一种 **可靠的 面向连接** 服务
<br>
## **UDP**
UDP 可以说是一种极简的运输层协议，它的功能仅仅包括运输层必备的功能，即：**多路复用/多路分解，差错检测**
在后面的讲解中我们会知道 TCP 进行通信时，发送方与接收方先要进行“三次握手”来确认连接，而 UDP 则没有这一流程，所以说 UDP 是**无连接**的 
在 TCP 中，如果消息没有成功发送，发送方是会收到发送失败的反馈的，而 UDP 中，发送方发出数据后无法确认是否成功发送，只能靠接收方是否回复信息来确认上一次发送成功与否，因此在应用 UDP 时，通常会加入一个延时，如果超过额定时间未收到回复，就重新发送
**DNS（Domain Name System，域名系统）就是一个应用 UDP 的例子**，当我们输入网址访问网站时，DNS 就是通过 UDP 发送域名查询报文的。所以，如果长时间没有得到回复，我们就会看到“连接超时”的错误页面。
![这里写图片描述](http://img.blog.csdn.net/20170918140926043?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
UDP 检验和，就是用来做差错检验的数据。
方法实现是，发送方的 UDP 对报文段中的所有 16 比特字的和进行反码运算，求和时遇到溢出要回卷。
~~~
例：有 3 个 16 比特字
		0110 0110 0110 0000
		0101 0101 0101 0101
		1000 1111 0000 1100

	①：前两个相加
		0110 0110 0110 0000
		0101 0101 0101 0101
       ——————————————————————
		1011 1011 1011 0101
	②：和与第三个相加
		1011 1011 1011 0101
		1000 1111 0000 1100
       ———————————————————————
            (1) 0100 1010 1100 0001

   ③：有溢出，进行回卷
	   0100 1010 1100 0001 + 1 = 0100 1010 1100 0010
   ④：转为反码，存入检验和
	   1011 0101 0011 1101
~~~
接收端收到数据后，会再次将数据取和，再与检验和相加，**若结果为1111 1111 1111 1111则表示无差错，若出现 0 则表示有差错**

UDP 虽然提供差错检测，但是**不会进行错误修复**，它要么直接丢弃错误段，要么将错误段上报应用程序

看到这里，不由得会产生一个疑问：UDP 这么的精简，功能不够强大，为什么还要用它呢？
原因是：

 1. 应用层能更好的控制要发送的数据和发送时间，实时性
 2. 无需建立连接，反应迅速
 3. 无连接状态，不需要维护连接状态，节省资源
 4. 分组首部开销小，节省数据段空间

<br>
## **TCP**
应用 TCP 时，一个应用进程可以开始向另一个应用进程发送数据之前，这两个进程必须先进行“握手”，所以说，TCP 是**面向连接的**
TCP 连接提供的是 **点对点，全双工**服务
TCP 提供的是 **可靠数据传输**
TCP 通过使用 **流量控制、序号、确认和定时器** 等技术，确保正确地、按序地将数据从发送进程交付给接收进程
TCP 还提供了 **拥塞控制** ，它允许 TCP 连接通过一条拥塞的网络链路，平等地共享网络链路带宽。

TCP 建立连接有三次握手
![这里写图片描述](http://img.blog.csdn.net/20170918151341119?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

终止连接有四次握手

![这里写图片描述](http://img.blog.csdn.net/20170918152042339?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

TCP 报文段结构
![这里写图片描述](http://img.blog.csdn.net/20170918153436185?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
## **代码示例**
### UDP
UDPClient.java

~~~java
package test;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/** 
 * UDPCLient deom
 * @author <dht925nerd@126.com>
 */
public class UDPClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket clientSocket = new DatagramSocket();
        BufferedReader inFromUser =
                new BufferedReader(
                        new InputStreamReader(System.in)
                );
		//获取本地 IP 地址
        InetAddress IPAddress = InetAddress.getLocalHost();
        byte[] sendData;
        byte[] receiveData = new byte[1024];
        System.out.println("请输入一句英文，服务器会返回其大写形式（输入exit退出）");
        while (true) {
            String sentence = inFromUser.readLine();
            if (sentence.equals("exit")) break;
            sendData = sentence.getBytes();
            //创建发送数据报包，并标注源地址#，目的地址#
            DatagramPacket sendPacket =
                    new DatagramPacket(sendData, sendData.length, IPAddress, 9876);
            //发送数据报包
            clientSocket.send(sendPacket);
            //创建接收数据报包
            DatagramPacket receivePacket =
                    new DatagramPacket(receiveData, receiveData.length);
            //接收服务器的数据报包
            clientSocket.receive(receivePacket);
            String modifiedSentence = new String(receivePacket.getData());
            System.out.println("FROM SERVER: " + modifiedSentence);
        }
        clientSocket.close();
    }
}

~~~
UDPServer.java

~~~java
package test;

import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * UDPServer Demo
 * @author <dht925nerd@126.com>
 */
public class UDPServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket serverSocket =
                new DatagramSocket(9876);
        byte[] sendData;
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        while (true) {
            byte[] receiveData = new byte[1024];
            //创建接收数据报包
            DatagramPacket receivePacket =
                    new DatagramPacket(receiveData, receiveData.length);
            //接收客户端数据报包
            serverSocket.receive(receivePacket);
            String sentence = new String(receivePacket.getData());
            //获取客户端地址
            InetAddress IPAddress = receivePacket.getAddress();
            if (sentence != null) System.out.println(df.format(new Date()) + " from " + IPAddress + ": " + sentence );
            //获得客户端端口号
            int port = receivePacket.getPort();
            String capitalizedSentence = sentence.toUpperCase();
            sendData = capitalizedSentence.getBytes();
            DatagramPacket sendPacket =
                    new DatagramPacket(sendData, sendData.length, IPAddress, port);
            //向客户端发送数据报包
            serverSocket.send(sendPacket);
        }
    }
}

~~~
测试方法：
① 先编译运行UDPServer.java
②  再运行 UDPClient.java 
③ 在UDPClient的控制板面输入字符串，回车
④ 在UDPClient 和 UDPServer 控制板面观察运行信息

运行信息：
客户端
![这里写图片描述](http://img.blog.csdn.net/20170918171544600?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
服务器端
![这里写图片描述](http://img.blog.csdn.net/20170918172149197?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### TCP
TCPClient.java

~~~java
package test;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.InputStreamReader;
import java.net.Socket;

/**
 * TCPClient demo
 * @author <dht925nerd@126.com>
 */
public class TCPClient {
    public static void main(String[] args) throws Exception {
        String sentence;
        String modifiedSentence;
        System.out.println("请输入一个英文字符串，服务器将返回其大写形式（输入exit退出）");
        while (true) {
            BufferedReader inFromUser = new BufferedReader(new InputStreamReader(System.in));
            //创建客户端 Socket 并指明需要连接的服务器端的主机名及端口号
            Socket clientSocket = new Socket("localhost", 6789);
            DataOutputStream outToServer = new DataOutputStream(clientSocket.getOutputStream());
            BufferedReader inFromServer = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
            sentence = inFromUser.readLine();
            if (sentence.equals("exit")) break;
            //向服务器发送数据
            outToServer.writeBytes(sentence + '\n');
            //接收服务器返回数据
            modifiedSentence = inFromServer.readLine();
            System.out.println("FROM SERVER: " + modifiedSentence);
            clientSocket.close();
        }
    }
}

~~~
TCPServer.java

~~~java
package test;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * TCPServer demo
 * @author <dht925nerd@126.com>
 */
public class TCPServer {
    public static void main(String[] args) throws Exception {
        String clientSentence;
        String capitalizedSentence;
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //创建服务器端 Socket 并指明端口号
        ServerSocket welcomeSocket = new ServerSocket(6789);
        while(true) {
            //接收客户端连接
            Socket connectionSocket = welcomeSocket.accept();
            BufferedReader inFromClient = new BufferedReader(new InputStreamReader(connectionSocket.getInputStream()));
            DataOutputStream outToClient = new DataOutputStream(connectionSocket.getOutputStream());
            //获取客户端传入的字符串
            clientSentence = inFromClient.readLine();
            if (clientSentence != null)
                System.out.println(df.format(new Date()) + " from " + connectionSocket.getInetAddress() + ": " + clientSentence );
            capitalizedSentence = clientSentence.toUpperCase() + '\n';
            //向客户端发送修改后的字符串
            outToClient.writeBytes(capitalizedSentence);
        }
    }
}

~~~
测试方法：同上

	