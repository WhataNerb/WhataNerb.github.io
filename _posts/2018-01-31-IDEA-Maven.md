---
layout: post
title: 'Maven项目中无法找到 xml文件或 properties文件等配置文件的解决方法'
date: 2018-01-31
categories: debug Maven
tags: debug Maven
---

在初次使用 Maven 项目的时候总是会遇到一些比较奇异的问题
就比如说总是会报错：项目里的 \*\*.xml 或 \*\*.properties 配置文件无法找到
这时你去项目中的 classes 生成文件（target 或 out）中找，确实这些配置文件没有被编译
这是因为 Maven 通常会忽略掉标记为 Sources 的文件夹中的配置文件

这是 Maven 项目的目录结构：
![这里写图片描述](http://img.blog.csdn.net/20180131234321691?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

有两种解决方案：
一、 将配置文件放入 resources 文件夹中

![这里写图片描述](http://img.blog.csdn.net/20180131234811740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


二、在 Maven 的配置文件 pom.xml 文件中添加以下代码:

```
<build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```