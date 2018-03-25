---
layout: post
title: 'HTTP,HTTPS和HSTS详解'
date: 2017-09-23
categories: 网络
tags: 网络
---
> 参考资料：
> [《从HTTP到HTTPS再到HSTS》作者：又拍云](http://www.cnblogs.com/upyun/p/7447977.html)
> 
>   《计算机网络：自顶向下方法》（第四版）作者：（美）James F.Kurose, Keith W.Ross
>   
>    wikipedia：[HTTP](https://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)，[HTTPS](https://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)，[HSTS](https://zh.wikipedia.org/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)
>     
> [《HTTPS科普扫盲贴》](https://div.io/topic/1614)作者：程序猿小卡
> 
> 《一个故事讲完https》微信公众号：码农翻身 作者：刘欣
> 
>[《你所不知道的 HSTS》](http://www.barretlee.com/blog/2015/10/22/hsts-intro/)作者：小胡子哥

HTTP（HyperText Transfer Protocol，超文本传输协议）
HTTPS（HTTP Secure 或 HTTP on TLS，超文本传输安全协议）
HSTS（HTTP Strict Transport Security，HTTP严格传输安全协议）

随着互联网的迅速发展，网络安全面临着越来越大的挑战，人们也越来越注重网络安全的重要性，于是诞生了从 HTTP 到 HTTPS 再到 HSTS 这样一条不断加强安全性的道路

**HTTP 是基础，HTTPS 和 HSTS 都是后来对它进行的安全性升级**
<br>
## **HTTP**
### **概述**
HTTP是一种用于 **分布式、协作式** 和 **超媒体信息系统** 的 **应用层** 协议
它是万维网的数据通信的基础，Web的核心。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。
![这里写图片描述](http://img.blog.csdn.net/20170923201503362?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### **请求访问过程**
HTTP 使用 **[TCP](http://blog.csdn.net/baidu_32045201/article/details/78021536)** 作为它的支撑运输层协议。但是，实际上HTTP协议中，并没有规定必须使用它或它支持的层，**HTTP只是假定其下层协议提供可靠的传输**，所以它也就选择了 TCP/IP 族中的 TCP
也正是因为 HTTP 采用 TCP 作为传输层协议，所以**HTTP协议不用担心数据丢失**
**HTTP 请求的默认端口是 80 号端口**，HTTP的访问过程如下：

 1. HTTP客户端在端口80向目标服务器发起TCP连接
 2. 连接成功后，HTTP客户端经它的Socket向服务器发送一个HTTP请求报文，其中包含请求访问路径
 3. HTTP服务器接受请求报文，在存储器里按照路径找到文件，然后包装成HTTP响应报文，通过Socket发送给客户端
 4. 传输结束后，服务器发起TCP断开连接通知
 5. 客户端接受完文件后，响应TCP断开连接

我们还需要知道，**HTTP 是一个无状态协议**，HTTP 服务器不会记住客户端的请求信息，即使你连续两次请求同一文件，服务器都会当做第一次请求，并执行完整的响应步骤


### **连接类型**
#### **非持久性连接**
这种连接方式是，**每传输一个文件都要经历上述的五个步骤**
比如，我们需要访问一个Web页面包含**10个JPEG图片和一个HTML文件**，它的HTML文件的URL地址是： www.someSchool.edu/someDepartment/home.index
如果应用非持久性连接请求，就会这样：
![这里写图片描述](http://img.blog.csdn.net/20170923204523413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这种连接方式无疑会非常消耗服务器资源

#### **持久性连接**
这种连接方式就是对非持久性的改进
它会在客户端与服务器之间**保持TCP连接的打开**，所有的请求文件都通过这一个TCP连接进行传输
一般来说，**如果一个连接经过一定的时间间隔（可配置）仍未被使用，HTTP服务器就会关闭该TCP连接**
![这里写图片描述](http://img.blog.csdn.net/20170923205436474?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

对于持久性连接，还有两种类型：
**无流水线的持久性连接：**客户端只有收到前一个响应后才发送新请求
**有流水线的持久性连接：**客户端不管响应回复，尽快地发送新请求

### **报文格式**
HTTP报文有两种：**请求报文，响应报文**
#### **请求报文**
HTTP 请求报文由请求行、请求头部、空行 和 请求包体 4 个部分组成，如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20170923214028438?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
实例：

~~~
GET /somedir/page.html HTTP/1.1  // 使用GET方法，请求/somedir/page.html，浏览器实现HTTP/1.1版本
Host: www.someschool.edu         // 主机名
Connection: close                // 不使用持久性连接
User-agent: Mozilla/4.0          // 浏览器类型
Accept-language: fr              // 对象语言版本（fr为法语）
~~~

#### **响应报文**
HTTP 响应报文由状态行、响应头部、空行 和 响应包体 4 个部分组成，如下图所示：
![这里写图片描述](http://img.blog.csdn.net/20170923215149300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
实例：

~~~
HTTP/1.1 200 OK                                 // HTTP/1.1协议， 状态200一切正常
Connection: close                               // 结束后关闭TCP连接
Date: Thu, 03 Jul 2003 12:00:15 GMT             // 服务器发送日期
Server: Apache/1.3.0 (Unix)                     // 服务器版本
Last-Modified: Sun, 6 May 2007 09:23:24 GMT     // 对象最后修改时间
Content-Length: 6821                            // 对象字节数
Content-Type: text/html                         // 对象文件类型

(data data data data data ...)                  //对象实体
~~~

### **版本**
HTTP已经演化出了很多版本，它们中的大部分都是向下兼容的。在 RFC 2145 中描述了HTTP版本号的用法。客户端在请求的开始告诉服务器它采用的协议版本号，而后者则在响应中采用相同或者更早的协议版本。
#### **HTTP/0.9**
已过时。只接受GET一种请求方法，没有在通讯中指定版本号，且不支持请求头。由于该版本不支持POST方法，因此客户端无法向服务器传递太多信息。
#### **HTTP/1.0**
这是第一个在通讯中指定版本号的HTTP协议版本，至今仍被广泛采用，特别是在代理服务器中。
**HTTP/0.9和HTTP/1.0都采用非持久性连接**

#### **HTTP/1.1**
**持久性连接被默认采用**，并能很好地配合代理服务器工作。还支持以管道方式在同时发送多个请求，以便降低线路负载，提高传输速度。
HTTP/1.1相较于HTTP/1.0协议的区别主要体现在：

 - 缓存处理 
 - 带宽优化及网络连接的使用
 -  错误通知的管理
 -  消息在网络中的发送
 - 互联网地址的维护
 - 安全性及完整性


#### **HTTP/2.0**
在 HTTP/2 的第一版草案（对 SPDY 协议的复刻）中，新增的性能改进不仅包括HTTP/1.1中已有的多路复用，修复队头阻塞问题，允许设置设定请求优先级，还包含了一个头部压缩算法(HPACK)。此外， HTTP/2 采用了二进制而非明文来打包、传输客户端—服务器间的数据。

### **状态码**
所有HTTP响应的第一行都是状态行，依次是当前HTTP版本号，3位数字组成的状态代码，以及描述状态的短语，彼此由空格分隔。
状态代码的第一个数字代表当前响应的类型：

 - 1xx消息——请求已被服务器接收，继续处理
 - 2xx成功——请求已成功被服务器接收、理解、并接受
 - 3xx重定向——需要后续操作才能完成这一请求
 - 4xx请求错误——请求含有词法错误或者无法被执行

### **特点**
综上所述，HTTP 具有如下特点：

 -  简单、快速、灵活
 - 无连接、无状态
 - 管线化和内容编码

<br>
## **从HTTP到HTTPS**
有了 HTTP 我们就可以方便，高效地浏览网页，但是，HTTP 是**明文传输**，我们（客户端）与服务器之间的通信内容是完全公开的，这就意味着如果通信内容包含一些隐私的内容，那么网络中的任何人都可以随意查看或修改内容，这肯定不行！
为了保证传输内容的私密性和完整性，我们需要对我们的消息内容进行加密处理，于是就诞生了 HTTPS
<br>

## **HTTPS**
HTTPS的主要思想是在不安全的网络上创建一安全信道，并可在使用适当的加密包和服务器证书可被验证且可被信任时，对窃听和中间人攻击提供合理的防护。
**HTTPS 的默认端口是 443**
![这里写图片描述](http://img.blog.csdn.net/20170923221826991?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

HTTPS 对信息内容进行加密的方式就是，在 HTTP 报文从应用层发送至传输层之前，对 HTTP 报文进行加密处理，**加密使用的协议是 SSL/TLS** ，我们暂且称它为“安全层”，利用 HTTPS 传输信息时，**发出的信息会先通过安全层加密，接受加密信息时，会经过安全层解密**
![这里写图片描述](http://img.blog.csdn.net/20171001185801909?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


SSL（Secure Sockets Layer）是 TLS（Transport Layer Security）的前身。网景公司（Netscape）在1994年推出首版网页浏览器，网景导航者时，推出HTTPS协议，以SSL进行加密，这是SSL的起源。IETF将SSL进行标准化，1999年公布第一版TLS标准文件。

那么 HTTPS 究竟是如何对数据进行加密的呢？

### **对称加密**
对称加密简单地说，就是通信双方使用的密钥是相同的，这种方法非常简单高效，但是却存在着致命的问题：
![这里写图片描述](http://img.blog.csdn.net/20171001200153054?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
首先，双方必须要商量好这个共用的密钥，那么密钥该怎么传输呢？
如果先使用明文传输交换密钥，那么密钥要是被劫持了，岂不是功亏一篑
就算安全地交换了密钥，如果通信过程中有多个个体，那么两两之间协商一次密钥会极大地降低效率，也许你会问：难道不能多人共用一把密钥吗？如果这样的话，那不就失去了这个加密算法的意义吗

### **非对称加密**
这个方法就解决了对称加密存在的问题
非对称加密约定，首先通信双方都需要维持两把密钥，一把是**公钥（对外公开）**，一把是**私钥（自己保密）**
然后遵循这样的约定：**通过公钥加密的数据，只能通过私钥解开。通过私钥加密的数据，只能通过公钥解开**
![这里写图片描述](http://img.blog.csdn.net/20171001201814473?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如上图所示，以张大胖向Bill发送信息的过程为例，张大胖先用Bill公开的公钥将信息加密，然后Bill收到后，再用自己的私钥解开信息，这样就可以保证信息安全了，反过来Bill向张大胖发送信息也是一样的道理

但是，非对称加密也不是绝对安全的，它存在这样的问题：

 1. 我该怎么样确定对方就是我想通信的人，如果在中间被劫持掉了，中间人将自己的公钥冒充的发送给我，那么不是直接与黑客通信了吗？
 2. 例如社交软件一样，具有隐私信息且用户量成千上万的系统，它的公钥是统一的，所有的用户都用这一种公钥加密，如果中间代理器获得公钥，那么不是换了个方式裸奔吗？

这就是**中间人问题**，为了解决这一问题，对加密算法又做了如下改进

### **数字证书认证机构（CA）**
为了防止有人冒充服务端，人们设立了数字证书认证机构（CA），它就是一个可以值得信赖的第三方机构，所有合法的网站都会在这个机构认证自己的身份并将自己的公钥交给该机构，我们的浏览器（客户端）中就带有认证机构的认证名单，当我们访问一个网站时，浏览器会在认证名单中查询该网站是否合法，如果合法就会直接从认证机构处拿到该网站的公钥，如果不合法就会发出警告

图. Chrome浏览器的认证标志
![这里写图片描述](http://img.blog.csdn.net/20171001204608043?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171001204737179?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

那么第 2 个问题怎么解决呢？
大致流程是这样的：

 1. 用户访问XX，XX将自己的证书给到用户（其实是给到浏览器，用户不会有感知）
 2. 浏览器从证书中拿到XX的公钥A
 3. 浏览器生成一个只有用户自己的对称密钥B，用公钥A加密，并传给XX（其实是有协商的过程，这里为了便于理解先简化）
 4. XX通过私钥解密，拿到对称密钥B
 5. 浏览器、XX 之后的数据通信，都用密钥B进行加密

<br>
## **从HTTPS到HSTS**

> 但是当网站传输协议从 HTTP 到 HTTPS 之后，数据传输真的安全了吗？
> 
> 由于用户习惯，通常准备访问某个网站时，在浏览器中只会输入一个域名，而不会在域名前面加上 http:// 或者 https://，而是由浏览器自动填充，当前所有浏览器默认填充的都是http://。一般情况网站管理员会采用了 301/302 跳转的方式由 HTTP 跳转到 HTTPS，但是这个过程总使用到 HTTP 因此容易发生劫持，受到第三方的攻击。
> 
> 这个时候就需要用到 HSTS（HTTP 严格安全传输）。

![这里写图片描述](http://img.blog.csdn.net/20171001210105729?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## **HSTS**
HTTP严格传输安全（英语：HTTP Strict Transport Security，缩写：HSTS）是一套由互联网工程任务组发布的互联网安全策略机制。网站可以选择使用HSTS策略，来让浏览器强制使用HTTPS与网站进行通信，以减少会话劫持风险

HSTS 的运行流程大致如此：
![这里写图片描述](http://img.blog.csdn.net/20171001210433712?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至此我们为了能够安全的传输信息做了这么多工作，看似已经无懈可击，十分安全了，可是，遗憾的告诉你 HSTS 也是存在不足的：

> 用户首次访问某网站是不受HSTS保护的。这是因为首次访问时，浏览器还未收到HSTS，所以仍有可能通过明文HTTP来访问。解决这个不足目前有两种方案，一是浏览器预置HSTS域名列表，Google、Chrome、Firefox、Internet Explorer和Microsoft Edge实现了这一方案。二是将HSTS信息加入到域名系统记录中。但这需要保证DNS的安全性，也就是需要部署域名系统安全扩展。截至2016年这一方案没有大规模部署。
> 由于HSTS会在一定时间后失效（有效期由max-age指定），所以浏览器是否强制HSTS策略取决于当前系统时间。部分操作系统经常通过网络时间协议更新系统时间，如Ubuntu每次连接网络时，OS X Lion每隔9分钟会自动连接时间服务器。攻击者可以通过伪造NTP信息，设置错误时间来绕过HSTS。解决方法是认证NTP信息，或者禁止NTP大幅度增减时间。比如Windows 8每7天更新一次时间，并且要求每次NTP设置的时间与当前时间不得超过15小时。


<br>
## **总结**
通过上述的介绍，我们可以看出，网络安全一直以来都是一个不可忽视的大问题，也是互联网发展需要面临的一大难题，尽管科学家和技术人员们利用他们的聪明才智不断地在保护互联网的安全，但是，随着技术的发展，这项工作永远不能停止，必须要不断地完善和创新才能保护每一个用户信息的安全。所以，我们一定要记住，抵制网络犯罪，共同维护一个和平安全的网络环境！

