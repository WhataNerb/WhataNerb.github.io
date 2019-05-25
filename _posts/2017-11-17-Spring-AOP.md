---
layout: post
title: 'AOP面向切编程及在Spring中的使用方法'
date: 2017-11-17
categories: Web开发 Spring
tags: Java Spring 后端 Web开发
---
## AOP 简介
AOP(Aspect-Oriented Programming, 面向切面编程): 是一种新的方法论, 是对传统的 OOP(Object-Oriented Programming, 面向对象编程)的补充

AOP 的主要编程对象是**切面(aspect)**

在应用 AOP 编程时, 仍然需要**定义公共功能**, 但可以明确的定义这个功能在哪里, 以什么方式应用, **并且不必修改受影响的类**

AOP 的好处：

- 每个事物逻辑位于一个位置, 代码不分散, 便于维护和升级
- 业务模块更简洁, 只包含核心业务代码

### 通过实例讲解AOP
单纯的术语也许不能让你清楚的明白 AOP，那么接下来我会通过一个实例来更加清晰的描述：

假设我们现在来实现一个计算器，我们可以很容易的写出：

~~~java
package com.dht.aop_annotation;

/**
 * @author dht925nerd@126.com
 */
public interface Calculator {

    int add(int x, int y);
    int sub(int x, int y);

    int mul(int x, int y);
    int div(int x, int y);

}
~~~

~~~java
package com.dht.aop_annotation;

/**
 * @author dht925nerd@126.com
 */
public class CalculatorImpl implements Calculator {

    private int result;

    @Override
    public int add(int x, int y) {
        result = x + y;
        return result;
    }
    @Override
    public int sub(int x, int y) {
        result = x - y;
        return result;
    }
    @Override
    public int mul(int x, int y) {
        result = x * y;
        return result;
    }
    @Override
    public int div(int x, int y) {
        result = x / y;
        return result;
    }
}
~~~
这样简单的代码就实现了一个基础的计算器

但是，重点来了！如果现在客户提出了新要求：**要求输出计算日志（log）**，就像这样：
![这里写图片描述](http://img.blog.csdn.net/20171117165519371?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
为此，我们可能需要这样为每一个方法都加上输出日志的内容：

~~~java
public int add(int x, int y) {
    System.out.println("the method begins with: " + x + ", " + y);
    result = x + y;
    return result;
}
~~~
这样做无疑使得我们会写很多重复冗余的代码，极大的降低了效率而且非常不利于维护
像这样多出来的重复性的无关功能逻辑的代码，我们就称其为**切面（Aspect）**

面向切面编程（AOP）需要解决的问题就是：**将切面独立出来**
![这里写图片描述](http://img.blog.csdn.net/20171117171604240?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIwNDUyMDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## AOP 术语
- **切面(Aspect)**: 横切关注点(跨越应用程序多个模块的功能)被模块化的特殊对象
- **通知(Adive)**: 切面必须要完成的工作
- **目标(Target)**: 被通知的对象
- **代理(Proxy)**: 向目标对象应用通知之后创建的对象
- **连接点(Joinpoint)**: 程序执行的某个特定位置
- **切点(Cutpoint)**: 每个类都拥有多个连接点, AOP 通过切点定位到特定的连接点. 类比: 连接点相当于数据库中的记录, 切点相当于查询条件

## 基于注解的切面配置. (需要导入 AspectJ 包)
~~~java
@Aspect
~~~
### 五种注解：

 - @Before 前置通知
 - @After 后置通知
 - @AfterReturning 返回通知
 - @AfterThrowing 异常通知
 - @Around 环绕通知
 
~~~java
package com.dht.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * @author dht925nerd@126.com
 */

@Aspect
@Component
public class LogAspect {

    /**
     * 前置通知: 方法执行前执行的通知
     * @param joinPoint
     */
    @Before("execution(public int com.dht.aop.Calculator.*(int, int))")
    public void logBefore(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println("the method " + name + " is Logging...");
    }

    /**
     * 后置通知: 方法执行后执行的通知, 不能获得返回值
     * 无论方法会不会抛出异常都执行该通知
     * @param joinPoint
     */
    @After("execution(public int com.dht.aop.Calculator.*(int, int))")
    public void logAfter(JoinPoint joinPoint){
        String name = joinPoint.getSignature().getName();
        System.out.println("the method " + name + " is ending...");
    }

    /**
     * 返回通知: 方法正常结束后的通知, 可以获得返回值
     * @param joinPoint
     * @param result
     */
    @AfterReturning(value = "execution(public int com.dht.aop.Calculator.*(int, int))",
                     returning = "result")
    public void returnLog(JoinPoint joinPoint, Object result){
        String name = joinPoint.getSignature().getName();
        System.out.println("the method " + name + " end with " + result);
    }

    /**
     * 异常通知: 在目标方法出现异常时执行的通知
     * 可以访问到异常对象, 可以指定在出现特定异常时再执行通知
     * @param joinPoint
     * @param ex
     */
    @AfterThrowing(value = "execution(public int com.dht.aop.Calculator.*(int, int))",
                    throwing = "ex")
    public void afterException(JoinPoint joinPoint, Exception ex){
        String name = joinPoint.getSignature().getName();
        System.out.println("the method " + name + " occurs exception : " + ex);
    }

    /**
     * 环绕通知需要携带 ProceedingJoinPoint 类型的参数
     * 环绕通知类似于动态代理的全过程: ProceedingJoinPoint 类型的参数可以决定是否执行目标方法
     * 且环绕通知必须有返回值, 返回值即为目标方法的返回值
     * @param joinPoint
     * @return
     */
    @Around("execution(public int com.dht.aop.Calculator.*(int, int))")
    public Object around(ProceedingJoinPoint joinPoint){
        Object result = null;
        String methodname = joinPoint.getSignature().getName();

        try {
            //前置通知
            System.out.println("The method " + methodname + " begins with " + Arrays.asList(joinPoint.getArgs()));
            //执行目标方法
            result = joinPoint.proceed();
            //返回通知
            System.out.println("The method " + methodname + " ends with " + result);
        } catch (Throwable throwable) {
            //异常通知
            throwable.printStackTrace();
        }
        //后置通知
        System.out.println("The method " + methodname + "ended");

        return null;
    }
}

~~~

### 切面优先级
~~~java
/**
 * 使用 @Order 可以设置切面的优先级, 数字越小优先级越高
 */
@Order(1)
@Before("execution(public int com.dht.aop.Calculator.*(int, int))")
public void logBefore(JoinPoint joinPoint){
    String name = joinPoint.getSignature().getName();
    System.out.println("the method " + name + " is Logging...");
}
```

### 重用切入点路径
```
/**
 * 定义一个方法, 用于声明切点表达式, 一般地, 该方法不填入其他代码
 */
@Pointcut("execution(public int com.dht.aop.Calculator.*(..))")
public void declearPath(){}

@Order(1)
@Before("declearPath()")  // 直接调用方法
public void logBefore(JoinPoint joinPoint){
    String name = joinPoint.getSignature().getName();
    System.out.println("the method " + name + " is Logging...");
}
~~~

## 基于 xml 的切面配置

~~~xml
<bean id="calculatorImpl" class="com.dht.aop_xml.CalculatorImpl"/>

<!-- 配置切面的 Bean -->
<bean id="logAspect" class="com.dht.aop_xml.LogAspect"/>

<!-- 配置 AOP -->
<aop:config>
    <!-- 配置切点表达式 -->
    <aop:pointcut id="pointcut" expression="execution(* com.dht.aop_xml.Calculator.*(..))"/>
    <!-- 配置切面及通知 -->
    <aop:aspect id="aspect" ref="logAspect">
        <aop:after method="logAfter" pointcut-ref="pointcut"/>
        <aop:before method="logBefore" pointcut-ref="pointcut"/>
        <aop:after-returning method="returnLog" pointcut-ref="pointcut" returning="result"/>
        <aop:after-throwing method="afterException" pointcut-ref="pointcut" throwing="ex"/>
    </aop:aspect>
</aop:config>

~~~
