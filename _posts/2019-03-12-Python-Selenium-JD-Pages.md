---
layout: post
title: 'Python+Selenium爬取京东商城，翻页功能实现'
date: 2019-03-12
categories: 爬虫
tags: 爬虫
---



## 问题描述

最近学习爬虫，爬京东页面做翻页功能时遇到了问题
我的初始想法是这样的：
*这个是京东的转页模块*
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114170902984.jpg)
1. 修改页码输入的文本框，填入要跳转到页数
2. 点击“确定”按钮，实现跳转

代码如下：

```python
def next_page(page_number):
    try:
        # 获取页码文本框
        input = wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR, '#J_bottomPage > span.p-skip > input'))
        )
        # 获取“确认”按钮
        submit = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#J_bottomPage > span.p-skip > a')))
        input.clear() # 清空文本框
        input.send_keys(page_number) #输入跳转页码
        submit.click()
        # 检查当前页码, 确认是否成功跳转
        wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR, '#J_bottomPage > span.p-num > a.curr'), str(page_number)))
    except TimeoutException:
        print('翻页请求错误, 正在重新执行...')
        return next_page(page_number)
```
可是在执行时出现了这样的 Error:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114172143467.jpg)
错误信息：

```
selenium.common.exceptions.StaleElementReferenceException: Message: stale element reference: element is not attached to the page document
```
百度了一下，出现这样的错误是由于页面刷新导致失效，仔细分析京东的页面后发现，京东的页面是**分两段动态生成的**，先显示一半的结果，当你下拉页面后，再显示后一半的结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114175139679.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114175156389.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)
每次下拉一半时，都会生成一个新的**s_new.php?....**，同时请注意请求参数中的 **page** 数的变化情况
由此可以得出，网页中的一页，实际上是 **2 个 page** 组成的，那么出现这样的错误就可以解释了

当刚刚加载出页面时，此时页面中只有 page: 1，而整个页面框架也刚刚加载出来，所以此时的**页面跳转模块**在page：1的下面，而当selenium选择页面跳转模块时，页面就已经滚动到下方了，于是Ajax又动态加载了page: 2，页面因此发生了改变，所以原先选择的元素就失效了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114221934741.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMDQ1MjAx,size_16,color_FFFFFF,t_70)

搞清楚了原因，就有了解决方法
## 解决方法
等初始页面加载出来后，通过调用 javascript 动作将页面下拉至底部直接把page都加载出来，然后再获取元素执行后续操作

```python
def next_page(page_number):
    try:
        browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
        time.sleep(1) # 等待加载完成
        input = wait.until(
            EC.presence_of_element_located((By.CSS_SELECTOR, '#J_bottomPage > span.p-skip > input'))
        )
        submit = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#J_bottomPage > span.p-skip > a')))
        input.clear()
        input.send_keys(page_number)
        submit.click()
        wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR, '#J_bottomPage > span.p-num > a.curr'), str(page_number)))
    except TimeoutException:
        print('翻页请求错误, 正在重新执行...')
        return next_page(page_number)

```

