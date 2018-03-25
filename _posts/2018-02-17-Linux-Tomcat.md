---
layout: post
title: '从主机 Windows 上无法远程访问 Linux 的 Tomcat 服务器解决方法'
date: 2018-02-17
categories: Linux Tomcat debug
tags: Linux Tomcat debug
---
当在 Linux 上配置好 Tomcat 服务器后，如果从主机中无法访问到 Linux 中的 Tomcat 服务器时，需要做如下的检查：
### 1. Tomcat 是否启动成功
在控制台输入：

~~~bash
ps -ef | grep tomcat
~~~
*命令含义*：从当前所有进程中查找是否含有 tomcat 进程

如果有内容显示，则说明 Tomcat 启动成功
![这里写图片描述](http://img.blog.csdn.net/20180217143232289?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

否则，重新启动 Tomcat

### 2. 检查能否从 Linux 本地中访问到 Tomcat
从控制台输入命令：

~~~bash
wget http://localhost:8080
~~~
*命令含义*：访问 http://localhost:8080
![这里写图片描述](http://img.blog.csdn.net/20180217160138699?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
否则，检查 Tomcat 端口号是否正确

### 3. 检查 Tomcat 启动端口号
Tomcat 默认的启动端口号是 8080，如果你没有对 Tomcat 的配置文件做修改的话应该是没有问题的
输入命令：

~~~bash
ps -ef | grep tomcat
~~~
*命令含义*：查看 tomcat 进程信息
![这里写图片描述](http://img.blog.csdn.net/20180217160321236?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
查看进程号（图中画圈位置）

接着输入命令：

~~~bash
netstat -apn | grep 4091
~~~
（*注*：grep 后跟的就是上一步所查的进程号）
*命令含义*：查看 4091 进程占用的端口号
![这里写图片描述](http://img.blog.csdn.net/20180217160846152?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
图中画圈位置即是你的 Tomcat 的启动端口号

### 4. 检查远程访问的 ip 地址是否正确
如果从 Linux 本地可以成功访问 Tomcat 服务器，而从 Windows（主机） 上无法访问，那么首先检查远程访问的 ip 地址是否正确
在 Liunx 控制台上输入命令:

~~~bash
ifconfig
~~~
![这里写图片描述](http://img.blog.csdn.net/20180217161341267?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
图中位置即是 Linux 的 ip 地址

### 5. 检查 Linux 防火墙是否开放 Tomcat 端口号
如果你没有修改过 Linux 防火墙配置的话，那么 Tomcat 端口号一定是被禁用了
因为 Linux 防火墙默认只开启 22 号端口

你需要设置防火墙配置，开放 Tomcat 的端口号
*注：网上有其他解决方法说直接关闭防火墙，这种方法很不可取*

我的 Linux 版本是 CentOS 7
开放 8080 端口号只需输入命令：

~~~bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
~~~

然后重启防火墙：

~~~bash
firewall-cmd --reload
~~~

