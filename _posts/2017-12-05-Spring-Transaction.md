---
layout: post
title: 'Spring中的事务传播行为'
date: 2017-12-05
categories: Web开发 Spring
tags: Java Spring 后端 Web开发
---
在 Spring 中可以使用 @Transactional 注解，将一个方法标记为一个事务方法
事务方法具有事务的 [ACID 四大特性](http://blog.csdn.net/baidu_32045201/article/details/77588591)

其中 @Transactional 有一个属性 propagation 用来标记该事务的传播行为
Spring 支持的传播行为有这样几种：
![这里写图片描述](http://img.blog.csdn.net/20171205153955329?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
*默认情况是：REQUIRED*

接下来举例说明：

~~~java
/**
 * 购买一本书
 * 事务传播行为: REQUIRED
 */
@Transactional(propagation = Propagation.REQUIRED)
public void purchase(String username, String isbn) {

	//从数据库中找到书本单价
    int price = bookShopDao.findBookPriceByIsbn(isbn);

	//将该书的库存减一
    bookShopDao.updateBookStock(isbn);

	//从用户的余额中减去书的单价
    bookShopDao.updateUserAccount(username, price);
}
~~~
~~~java
/**
 * 结账
 * 根据购书列表，循环调用 purchase 方法
 */
@Transactional
public void checkout(String username, List<String> isbns) {
    for (String isbn : isbns){
        bookShopService.purchase(username, isbn);
    }
}
~~~

假设用户要买 3 本书， 可用户的余额仅够买前 2 本书，那么在执行 checkout 结账事务时，就会抛出“余额不足”的异常

那么根据事务的传播行为会有这样的情况：
我们之前声明了 purchase 的事务传播行为是 **REQUIRED**，所以在 checkout 事务调用 purchase 时，purchase 发现有事务正在运行，所以就加入事务
按照顺序，前 2 本书的 purchase 都顺利完成了，可是第 3 个 purchase 出现了异常，所以导致整个 checkout 事务失败，则 3 个purchase 全部回滚。此时，用户余额不变，书的库存也不变

![这里写图片描述](http://img.blog.csdn.net/20171205162616899?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果声明 purchase 的事务传播行为是 **REQUIRED_NEW**， 那么就会是这样的：
checkout 调用 purchase，此时 checkout 事务正在运行，但是 purchase 会将 checkout 事务挂起并重新建立一个自己的事务，自己的事务完成后，会重新唤醒 checkout 事务，如此循环，直到成功或是出现异常则停止
这样以后，用户在 checkout 结账后，账户余额会减去前 2 本书的单价，并且前 2 本书的库存也会减 1， 但第 3 本书的库存不会变
![这里写图片描述](http://img.blog.csdn.net/20171205162638595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)