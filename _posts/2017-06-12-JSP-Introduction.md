---
layout: post
title: '简单了解JSP'
date: 2017-06-12
categories: JavaWEB
tags: JSP JavaWEB 后端 
---


引用来自wikipedia
>JSP（全称JavaServer Pages）是由Sun Microsystems公司倡导和许多公司参与共同创建的一种使软件开发者可以响应客户端请求，而动态生成HTML、XML或其他格式文档的Web网页的技术标准。JSP技术是以Java语言作为脚本语言的，JSP网页为整个服务器端的Java库单元提供了一个接口来服务于HTTP的应用程序。
首先一定要明确，**JSP是一种网页技术**，它可以实现在传统的html页面中嵌入Java代码使得静态页面具有动态特性

使用方法也很简单就是在html代码中用`<%  %>`嵌入Java代码
就像这样:

~~~html
<!DOCTYPE html>
<html>
    <head>
           <title>传统Frist code</title>
    </head>
    <body>
           <%
                  out.println("Hello World！");
           %>
    </body>
</html>
~~~

### **JSP与Servlet**
JSP与Servlet一样都是Sun公司推出的用于JavaWeb开发的技术产品，Servlet的推出要先于JSP
客观来说，这是两种不同的技术产品，但是**JSP是以Servlet作为技术基础的**，我们先来看一看JSP的处理过程

1. 客户端发出调用http请求给服务器
2. 服务器识别出是JSP网页的请求，于是交给**JSP引擎**
3. JSP引擎载入对应的.jsp文件**并将JSP代码转为Servlet代码（即将.jsp文件转为.java文件）并编译成为.class文件**
4. JVM将运行结果通过Response返回给客户端

**注意：在执行第3步时，JSP引擎会检查JSP文件对应的servlet是否已经存在，并且检查JSP文件的修改日期是否早于servlet。如果JSP文件的修改日期早于对应的servlet，那么容器就可以确定JSP文件没有被修改过并且servlet有效，这样的做法使得JSP更加的快捷高效**

由上述的过程我们可以看出，JSP实际上就是一种用来弥补Servlet在应用层劣势的技术。

说了这么多，也许你对JSP究竟是如何优化Servlet的还是一头雾水，那么我们就来说说这个（如果你没有用过Servlet，建议你先学着写一个简单的Servlet例子，再回来看这个，否则你可能还是懵逼的）：

例如我们现在正在做一个登录界面
按照**纯Servlet**的做法：html做界面 + Servlet做后台
当用户提交登录表单后，需要重新跳转到Servlet的映射去,Servlet获取表单内容，然后再将结果信息在转换为html后print至客户端

这样的做法无疑麻烦又耗时
如果**用JSP来做**的话，可以直接将Java代码嵌入在html中，那么，在处理用户输入的数据后就可以直接用html格式显示出来无限再去调用其他文件或是做转换，这样大大提高了动态网页的开发效率

其实我觉得JSP就是Java和html生的孩子（贵圈真乱）

但是，Servlet并不是一无是处的，Servlet与JSP各有所长，**Servlet擅长逻辑控制，JSP擅长视图展示**，在**M**(modle)**V**(view)**C**(control)设计模式中，**Servlet应用于C(control)模块中，JSP应用于V(view)模块中**

### **JSP与ASP**
ASP（Active Server Pages）是**微软**开发的服务器端运行的**脚本平台**
它运行在**IIS**（Internet Information Services）中，如果你使用Windows，那么你可以在控制板面->程序->启动或关闭Windows功能中找到并开启它
ASP实现可以**在html中嵌入VBScript 、 JavaScript等脚本语言**（JSP嵌入的是Java）
因为它是微软的产品，所以它也遵循着只能在微软产品中应用的优良传统（这是一个坑点）
JSP，ASP，PHP这三者都是用来做动态网页开发的，所以它们经常被拿来比较，但是首先一定要明确，**JSP，ASP都是一种技术手段并不是编程语言，而PHP是服务端脚本语言**

**论执行效率，JSP要高于ASP**，但由上文我们知道了，服务器在处理JSP请求时，会先将JSP转为Servlet再由JVM来执行，因此，**JSP第一次执行时会非常的缓慢**，所以JSP的第一次通常都给了系统管理员

### **JSP与JavaScript**
**JSP运行在服务器端，JavaScript运行在客户端**
JavaScript几乎不会与服务器端产生交互，因此它无法执行较为复杂的操作，例如连接数据库等，它主要用来动态地处理客户端提交的数据，对数据做一些简单的操作以减轻服务器负担
JavaScript是前端开发的一项重要技术，它决定了的网页的行为
