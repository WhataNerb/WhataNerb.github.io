---
layout: post
title: 'IDEA中Spring配置错误：class path resource [.xml] cannot be opened because it does not exist'
date: 2017-10-29
categories: debug
tags: debug IDEA
---
如果在运行 Spring 项目时出现了类似于：

~~~
class path resource [applicationContext.xml] cannot be opened because it does not exist
~~~
![这里写图片描述](http://img.blog.csdn.net/20171029154859972?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样的异常
意思就是没有找到你的 .xml 配置文件

### **原因**
我可以肯定你一定用的是

~~~java
 ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
~~~
来获取配置文件

ClassPathXmlApplicationContext( ) 方法是在其所在的目录中寻找 .xml 配置文件
*注意：*  **这里指的是编译后的 .class 文件所在的目录，不是 .java 文件**

出现异常时，我的项目结构时这样的：
![这里写图片描述](http://img.blog.csdn.net/20171029155849837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

请注意我用红线标注的地方：

此时 applicationContext.xml 文件在 src 目录下

造成异常的原因是 IDEA 默认的项目结构导致的，它将 **.java 文件和 .class 文件分开存放，.java文件存于 src 中，.class 文件存于 target 中**
因此，ClassPathXmlApplicationContext( ) 方法无法找到 applicationContext.xml

### **解决方法**

知道了原因，那么我们就需要把 .class 也放进 src 目录中
*注意：*不能直接把 applicationContext.xml 移至 target 目录下，因为 .xml 配置文件运行时也需要在 .java 文件中获取属性信息

点击 File -> Project Structure（或快捷键 Ctrl+Alt+Shift+S）
![这里写图片描述](http://img.blog.csdn.net/20171029160906620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后修改 Output Path 到 src 目录下即可：
![这里写图片描述](http://img.blog.csdn.net/20171029161150233?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)