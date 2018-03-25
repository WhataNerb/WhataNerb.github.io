---
layout: post
title: 'IDEA手动配置JUnit'
date: 2017-11-17
categories: debug
tags: debug
---
## 安装JUnit
使用快捷键 ctrl + shift + s 或点击 File->Settings
点击 Plugins 查看插件
点击下方 Browse repositories... 查找插件

![这里写图片描述](http://img.blog.csdn.net/20171117185425564?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
在搜索栏输入 JUnitGenerator V2.0，点击 install 安装 ，如果没有 install 按钮证明你已经安装过了，可以在之前的 Plugins 中查看
![这里写图片描述](http://img.blog.csdn.net/20171117183843274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

安装后，重启 IDEA

## 导入 jar 包
在项目目录下新建 lib 文件夹

打开电脑中 IntelliJ IDEA 的安装路径，找到并打开 lib 文件夹，在里面找到这三个 jar 包
![这里写图片描述](http://img.blog.csdn.net/20171117184409223?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
将这三个 jar 包复制到项目目录下的 lib 文件夹中

如果 jar 包旁边没有出现展开的小箭头
![这里写图片描述](http://img.blog.csdn.net/20171117185231163?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
就右键选择 Add as Library...
![这里写图片描述](http://img.blog.csdn.net/20171117185322875?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至此，导入成功，按下 Ctrl + Shift + T 可以快速新建测试类
![这里写图片描述](http://img.blog.csdn.net/20171117190046437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20171117190137359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其实也可以通过 IDEA 的 Project Structrue... 进行配置，但是我个人还是习惯直接将 jar 包复制进项目中