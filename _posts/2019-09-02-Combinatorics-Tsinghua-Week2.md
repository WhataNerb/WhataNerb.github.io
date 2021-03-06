---
layout: post
title: '清华大学马昱春老师《组合数学》第二周作业题解'
date: 2019-09-02
categories: 数学
tags: 组合数学 数学 排列组合
---

> 课程链接：[《组合数学》马昱春副教授](http://www.xuetangx.com/courses/course-v1:TsinghuaX+60240013X+sp/about) 

## H1

### 小于10000的含1的正整数有___个?

分析：

首先确定可选范围，小于10000的数，且要求正整数，所以从1~9999中选择（0非正非负）。

那么“”含1“这个条件怎么处理呢？若是直接计算含1的数字会非常麻烦，所以我们不妨采用**减法法则**，即先计算出不含1的数的个数，再用整体减去这一部分就是答案了

接下来计算1~9999中不含1的正整数：

首先从一位数考虑，共有2\~9，一共8种取值

两位数：十位数的取值范围是2\~9，个位数是0、2\~9，一共8\*9=72种取值

三位数：百位数可取2\~9，十位、个位数可取0、2\~9，一共8\*9\*9=648种取值

四位数：千位数可取2\~9，百位、十位、个位数可取0、2\~9，一共8\*9\*9\*9=5832种取值

所以答案是共有9999-5832-648-72-8=**3439**个

### 小于10000的含0的正整数有___个？

分析：

仿照上一题的思路，我们计算1~9999之间不含0的正整数的个数：

一位数：可取1~9，一共9种取值

两位数：十位、个位可取1~9，一共 9 \* 9 = 81 种取值

三位数：百位、十位、个位可取1~9，一共 9 \* 9 \* 9 = 729 种取值

四位数：千位、百位、十位、个位可取1~9，一共 9 \* 9 \* 9 \* 9 = 6561 种取值

所以答案是共有 9999 - 6561 - 729 - 81 - 9 = **2619** 个

## H2

如图所示，从O点出发，到达P点，每次只能移动一个单位，

求满足下列条件的从O到P的最短路径数

1）路径必须经过A点

2）路径必须经过道路AB

3）路径必须经过A和C

![](https://img-blog.csdnimg.cn/20190902194446293.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)

分析：

首先，因为要求最短路径，所以每一步只能向右（用X表示）或是向上（用Y表示）走

举一个例子，若要计算从O到A的最短路径数，可以发现，从O到A的最短路径共由3个X和2个Y组成，那么问题就转化为了3个X和2个Y的组合问题了

接下来，继续对问题化简，3个X和2个Y的组合问题，可以化简为从5个元素中选取2个元素做Y（或选取3个元素做X）的问题，即 C(5, 2) = 10

有了以上的经验，我们接下来分析问题：

1）要求过点A，即先计算O到A的最短路径数再乘A到P的最短路径数：

C(5, 2) \* C(8, 3) = 10 \* 56 = **560**

2）C(5, 2) \* C(7, 3) = 10 * 35 = **350**

3）C(5, 2) \* C(4, 1) \* C(4, 2) = **240**

## H3

### 给三个孩子发水果,一共12个一样的苹果,每个孩子至少有一个苹果,问有____种分法

分析：

#### 解法一：

可以将问题想像成可重组合模型

![](https://img-blog.csdnimg.cn/20190902194503241.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)

根据表中的模型，其中把孩子当作水果（选出y个第x种水果拼果盘 == 选出y个水果给第x个小孩），套用公式：

12个水果分给三个孩子：n = 3，r = 12 即 C(3 + 12 - 1, 12) = C(14, 12) = 91

12个水果分给两个孩子：n = 2，r = 12 即 C(2 + 12 - 1, 12) = C(13, 12) = 13

因为上面两个结果中91和13中都包括了把所有水果都给一个孩子的情况，所以分别减去这两种情况

三个孩子，只给一个孩子水果，一共3种可能，所以91 - 3 = 88

两个孩子，只给一个孩子水果，一共2种可能，所以13 - 2 = 11

三个孩子，选出两个孩子分水果，一共C(3, 2) = 3种可能

所以最终的答案为：91 - 3 - (13 - 2) \* 3 = **55**

#### 解法二：

 使用隔板法，将12个水果摆开，因为要求每个孩子至少一个水果，所以，只考虑12个水果当中的11个空，使用2块隔板，填入11个空中将水果分为3份

此时题目化简为：从11个空中选出2个空放入隔板，即C(11, 2) = **55**

