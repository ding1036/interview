<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [AOP](#aop)
    - [AOP执行顺序](#aop执行顺序)
    - [JDK代理和CGLIB代理](#jdk代理和cglib代理)
- [@ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常](#controlleradvice--exceptionhandler-全局处理-controller-层异常)
- [Spring MVC Controller单例还是多例](#spring-mvc-controller单例还是多例)
- [@Resource与@Autowired注解的区别](#resource与autowired注解的区别)

<!-- /TOC -->


# AOP
通过预编译和运行期代理实现在不修改源代码的情况下给程序动态统一添加功能。
默认使用JDK代理来创建AOP代理
当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理

## AOP执行顺序
* 配置AOP执行顺序的三种方式

1) 通过实现org.springframework.core.Ordered接口

```java
@Component  
@Aspect  
@Slf4j  
public class MessageQueueAopAspect1 implements Ordered{@Override  
    public int getOrder() {  
        // TODO Auto-generated method stub  
        return 2;  
    }  
      
} 
```

2) 通过注解

```java
@Component  
@Aspect  
@Slf4j  
@Order(1)  
public class MessageQueueAopAspect1{  
      
    ...  
}  
```

3) 通过配置文件配置

```java
<aop:config expose-proxy="true">  
    <aop:aspect ref="aopBean" order="0">    
        <aop:pointcut id="testPointcut"  expression="@annotation(xxx.xxx.xxx.annotation.xxx)"/>    
        <aop:around pointcut-ref="testPointcut" method="doAround" />    
        </aop:aspect>    
</aop:config>
```

order越小越是最先执行，但更重要的是最先执行的最后结束。

* 在一个方法只被一个aspect类拦截时，aspect类内部的 advice 将按照以下的顺序进行执行

正常情况：

![](/img/aop_1)

异常情况：

![](/img/aop_2)

* 同一个方法被多个Aspect类拦截
有些情况下，对于两个不同的aspect类，不管它们的advice使用的是同一个pointcut，还是不同的pointcut，都有可能导致同一个方法被多个aspect类拦截。

如何指定每个 aspect 的执行顺序呢？ 
方法有两种：

    1) 实现org.springframework.core.Ordered接口，实现它的getOrder()方法
    2) 给aspect添加@Order注解，该注解全称为：org.springframework.core.annotation.Order
    不管采用上面的哪种方法，都是值越小的 aspect 越先执行。

比如，我们为 apsect1 和 aspect2 分别添加 @Order 注解，如下:

```java

@Order(5)
@Component
@Aspect
public class Aspect1 {
    // ...
}

@Order(6)
@Component
@Aspect
public class Aspect2 {
    // ...
}
```

这样修改之后，可保证不管在任何情况下， aspect1 中的 advice 总是比 aspect2 中的 advice 先执行。

![](/img/aop_3)

* 注意点

如果在同一个 aspect 类中，针对同一个 pointcut，定义了两个相同的 advice(比如，定义了两个 @Before)，那么这两个 advice 的执行顺序是无法确定的，哪怕你给这两个 advice 添加了 @Order 这个注解，也不行。

对于@Around这个advice，不管它有没有返回值，但是必须要方法内部，调用一下 pjp.proceed();否则，Controller 中的接口将没有机会被执行，从而也导致了 @Before这个advice不会被触发。

参考1 ：[Spring AOP @Before @Around @After 等 advice 的执行顺序](https://blog.csdn.net/rainbow702/article/details/52185827)

[toTop](#jump)

## JDK代理和CGLIB代理

JDK动态代理是面向接口，在创建代理实现类时比CGLib要快，创建代理速度快。

CGLib动态代理是通过字节码底层继承要代理类来实现（如果被代理类被final关键字所修饰，那么抱歉会失败），在创建代理这一块没有JDK动态代理快，但是运行速度比JDK动态代理要快。

如果要被代理的对象是个实现类（实现接口），那么Spring会使用JDK动态代理来完成操作（Spirng默认采用JDK动态代理实现机制）

如果要被代理的对象不是个实现类那么，Spring会强制使用CGLib来实现动态代理。

* 选择的使用代理机制

```java
<aop:config proxy-target-class="true">
    <!--切面详细配置-->
</aop:config>    
```

通过配置Spring的中<aop:config>标签来显示的指定使用动态代理机制 proxy-target-class=true表示使用CGLib代理，如果为false就是默认使用JDK动态代理




[toTop](#jump)


# @ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常

参考1 ：[@ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常](https://blog.csdn.net/kinginblue/article/details/70186586)

[toTop](#jump)

#  Spring MVC Controller单例还是多例

* Spring MVC Controller默认是单例的：

单例的原因有二：

1) 为了性能。单例不用每次都new.

2) 不需要多例。，如果你给controller中定义很多的属性，那么单例肯定会出现竞争访问了。因此，只要controller中不定义属性，那么单例完全是安全的。下面给个例子说明下：

```java
package com.lavasoft.demo.web.controller.lsh.ch5;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/demo")
@Scope("prototype") //定义为多例
public class MultViewController {
    private static int st = 0;      //静态的
    private int index = 0;          //非静态
    @RequestMapping("/show")
    public String toShow(ModelMap model) {
        User user = new User();
        user.setUserName("testuname");
        user.setAge("23");
        model.put("user", user);
        return "/show";
    }
    @RequestMapping("/test")
    public String test() {
        System.out.println(st++ + " | " + index++);
        return "/test";
    }
}
```

单例结果

```java
0 | 0

1 | 1

2 | 2

3 | 3

4 | 4
```

多例结果

```java
0 | 0

1 | 0

2 | 0

3 | 0

4 | 0
```

从此可见，单例是不安全的，会导致属性重复使用。

 

最佳实践：

1) 不要在controller中定义成员变量。

2) 万一必须要定义一个非静态成员变量时候，则通过注解@Scope("prototype")，将其设置为多例模式。


[toTop](#jump)

# @Resource与@Autowired注解的区别

* @Resource默认按照名称方式进行bean匹配，@Autowired默认按照类型方式进行bean匹配
* @Resource(import javax.annotation.Resource;)是J2EE的注解，@Autowired( import org.springframework.beans.factory.annotation.Autowired;)是Spring的注解

```java
//区别使用类
@Resource(name = "xxxImpl") 
private Human human;
//一般是类名首字母小写，因为使用@Service，容器为我们创建bean时默认类名首字母小写
```

如果使用@Autowire
要配上@Qualifier("xxxImpl")
```java
@Autowired
@Qualifier("manImpl")
private Human human;
```

参考1 :[@Autowired 与@Resource的区别](https://blog.csdn.net/weixin_40423597/article/details/80643990)

参考1 : [@Resource与@Autowired注解的区别](https://blog.csdn.net/wangzuojia001/article/details/54312074/)

[toTop](#jump)