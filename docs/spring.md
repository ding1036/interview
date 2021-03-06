<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [AOP](#aop)
    - [AOP执行顺序](#aop执行顺序)
- [IOC](#ioc)
    - [IOC注入方式](#ioc注入方式)
    - [IoC容器](#ioc容器)
        - [BeanFactory](#beanfactory)
        - [ApplicationContext](#applicationcontext)
    - [IoC容器的初始化](#ioc容器的初始化)
    - [JDK代理和CGLIB代理](#jdk代理和cglib代理)
- [@ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常](#controlleradvice--exceptionhandler-全局处理-controller-层异常)
- [@Resource与@Autowired注解的区别](#resource与autowired注解的区别)
- [自定义注解](#自定义注解)
- [Spring事务失效原因](#spring事务失效原因)
- [Bean的作用域](#bean的作用域)
- [Spring生命周期](#spring生命周期)
- [spring bean初始化销毁过程](#spring-bean初始化销毁过程)
- [BeanFactory和FactoryBean的区别](#beanfactory和factorybean的区别)

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

例子

```java
@Aspect
@Component("maintainHistoryAspect")
public class MaintainHistoryAspect {

@Before("execution(public * com.mc.dbra.web.service.impl..*.*ServiceImpl.update*(..)) &&  args(resourcesList,request,..)")
	public void appendUpdateListInfo(List<Resources> resourcesList,HttpServletRequest request) {
		
	}
}

@Before("(execution(public * com.mc.dbra.web.service.impl..*.*ServiceImpl.add*(..))) &&  args(database,..)" +
			"|| (execution(public * com.mc.dbra.web.service.impl..*.*ServiceImpl.updateResource*(..))) &&  args(database,..)")
	public void encodePasswordInfo(Database database) {
		
	}

@AfterReturning(value="execution(public * com.mc.dbra.web.service.impl..*.*ServiceImpl.get*(..)) &&  args(database,..)",
			returning="result")
	public void decodePasswordInfo(Database database,Object result) {
		try {
			if (result instanceof Database) {
				Database resultVO = (Database) result;
				resultVO.setPassword(XXX));
			}
		} catch (Exception e) {
			logger.error(e.getMessage());
		}
	}    

```

例子1：[https://blog.csdn.net/u010502101/article/details/78823056](https://blog.csdn.net/u010502101/article/details/78823056)

例子2：[spring统一日志管理，切面（@Aspect），注解式日志管理](https://www.cnblogs.com/chihirotan/p/6228337.html)


[toTop](#jump)

# IOC

## IOC注入方式

1) 接口注入。从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式，基本处于“退
役状态”。因为它强制被注入对象实现不必要的接口，带有侵入性。

2) 构造方法注入。
优点:对象在构造完成之后，即已进入就绪状态，可以马上使用。
缺点:当**依赖对象比较多的时候，构造方法的参数列表会比较长**。而通过反
射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。而且
在Java中，构造方法无法被继承，无法设置默认值。对于非必须的依赖处理，可能需要引入多个构造方法，而参数数量的变动可能造成维护上的不便。

3) setter方法注入。因为方法可以命名，所以setter方法注入在描述性上要比构造方法注入好一些。 另外，setter方法可以被继承，允许设置默认值，而且有良好的IDE支持。缺点当然就是**对象无法在构造完成后马上进入就绪状态**。


## IoC容器

Spring中提供了两种IoC容器：

1) BeanFactory
2) ApplicationContext

`ApplicationContext`是`BeanFactory`的子类

### BeanFactory
基础类型IoC容器，默认采用延迟初始化策略`lazy-load`。**只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作**。
适合场景： 容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景

### ApplicationContext
ApplicationContext在BeanFactory的基础上构建，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性
1.  支持信息源，可以实现国际化。（实现MessageSource接口）
2.  访问资源。(实现ResourcePatternResolver接口)
3.  支持应用事件。(实现ApplicationEventPublisher接口)

`ApplicationContext`所管理的对象，在该类型容器启动之后，**默认全部初始化并绑定完成**。所以，相对于`BeanFactory`来说，`ApplicationContext`要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器**启动时间较之BeanFactory也会长一些**。
适合场景： 在那些系统资源充足，并且要求更多功能的场景中


## IoC容器的初始化
 IoC容器的初始化包括BeanDefinition的Resource定位、载入和注册这三个基本的过程。



参考1 : [Spring IOC原理解读 面试必读](https://blog.csdn.net/qq_34173549/article/details/79929071)

参考1 : [细说Spring——IoC详解](https://www.jianshu.com/p/4007079cb6c0)
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




# 自定义注解

1.注解的定义：Java文件叫做Annotation，用``@interface``表示。

2.元注解：``@interface``上面按需要注解上一些东西，包括``@Retention``、``@Target``、``@Document``、``@Inherited``四种。

3.注解的保留策略：
```java
@Retention(RetentionPolicy.SOURCE)   // 注解仅存在于源码中，在class字节码文件中不包含

@Retention(RetentionPolicy.CLASS)     // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得

@Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到
```

4.注解的作用目标：
```java
@Target(ElementType.TYPE) // 接口、类、枚举、注解

@Target(ElementType.FIELD) // 字段、枚举的常量

@Target(ElementType.METHOD) // 方法

@Target(ElementType.PARAMETER) // 方法参数

@Target(ElementType.CONSTRUCTOR) // 构造函数

@Target(ElementType.LOCAL_VARIABLE) // 局部变量

@Target(ElementType.ANNOTATION_TYPE) // 注解

@Target(ElementType.PACKAGE)  // 包
```

5.注解包含在javadoc中：``@Documented``

6.注解可以被继承：``@Inherited``

7.注解解析器：用来解析自定义注解。

[toTop](#jump)

# Spring事务失效原因
1.如使用mysql且引擎是``MyISAM``，则事务会不起作用，原因是``MyISAM``不支持事务，可以改成``InnoDB``

2. 如果使用了spring+mvc，则``context:component-scan``重复扫描问题可能会引起事务失败。(即父子容器)

3. ``@Transactional`` 注解开启配置，必须放到``listener``里加载，如果放到``DispatcherServlet``的配置里，事务也是不起作用的。

4. ``@Transactional`` 注解只能应用到 ``public`` 可见度的方法上。 如果你在 protected、private 或者 package-visible 的方法上使用 ``@Transactional`` 注解，它也不会报错，事务也会失效。 

5. Spring团队建议在具体的类（或类的方法）上使用 ``@Transactional`` 注解，而不要使用在类所要实现的任何接口上。在接口上使用 ``@Transactional`` 注解，只能当你设置了基于接口的代理时它才生效。因为注解是 不能继承 的，这就意味着如果正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装。

6. 在业务层捕捉异常后，发现事务不生效。
在业务层手工捕捉并处理了异常（try..catch）等于把异常“吃”掉了，Spring自然不知道这里有错，更不会主动去回滚数据。推荐做法是在业务层统一抛出异常，然后在控制层统一处理。

7.遇到非检测异常时，事务不开启，也无法回滚。
因为Spring的默认的事务规则是遇到运行异常（``RuntimeException``）和程序错误（``Error``）才会回滚。如果想针对非检测异常进行事务回滚，可以在``@Transactional ``注解里使用``rollbackFor`` 属性明确指定异常。

8.自调用无法回滚。spring的数据库事务约定的实现原理是AOP，而AOP的原理是动态代理，在自调用的过程中，是类自身的调用，而不是代理对象去调用，那么不会产生AOP，这样spring就不能把你的代码植入到约定的流程中，于是就产生了失败场景。
解决方案：
用一个service去调用另一个service，这样就是代理对象的调用。


参考1： [Spring事务失效的原因](https://blog.csdn.net/paincupid/article/details/51822599)

参考2： [Spring事务失效事件（非常规原因）](https://www.jianshu.com/p/d4c3634447d0)

参考3： [SpringBoot 快速开启事务中 @Transaction注解不生效的问题](https://blog.csdn.net/qq_21508727/article/details/82705028)

参考4： [springboot @Transactional 自调用失效问题](https://blog.csdn.net/qq_33696896/article/details/82013095)


[toTop](#jump)

# Bean的作用域

singleton单例：是spring默认缺省的，全局只有一个对象。

prototype原型：每次都是新的Bean实例，有状态的Bean建议用此类型。

request：一次Http请求中，容器返回同一实例Bean，仅在当前Http Request内有效

session：一次Http Session中，容器返回同一实例Bean，仅在当前Session内有效。

global session：一个全局的Http Session中，容器返回同一个实例Bean。

[toTop](#jump)

# Spring生命周期


1) 实例化bean对象(通过构造方法或者工厂方法)

2) 设置对象属性(setter等)（依赖注入）

3) 如果Bean实现了``BeanNameAware``接口，工厂调用Bean的``setBeanName()``方法传递Bean的ID。（和下面的一条均属于检查Aware接口）

4) 如果Bean实现了``BeanFactoryAware``接口，工厂调用``setBeanFactory()``方法传入工厂自身

5) 将Bean实例传递给Bean的前置处理器

```java
postProcessBeforeInitialization(Object bean, String beanname)
```

6) 调用Bean的初始化方法

7) 将Bean实例传递给Bean的后置处理器

```java
postProcessAfterInitialization(Object bean, String beanname)
```

8) 使用Bean

9) 容器关闭之前，调用Bean的销毁方法

参考1 :[Spring 了解Bean的一生(生命周期)](https://blog.csdn.net/w_linux/article/details/80086950)


[toTop](#jump)

# spring bean初始化销毁过程
Spring Bean 的初始化流程如下：

1. 实例化 Bean 对象

    1.1 Spring 容器根据配置中的 Bean Definition(定义)中实例化 Bean 对象。
    ``Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。``

    1.2 Spring 使用依赖注入填充所有属性，如 Bean 中所定义的配置。

2. Aware 相关的属性，注入到 Bean 对象
    2.1 如果 Bean 实现 ``BeanNameAware`` 接口，则工厂通过传递 Bean 的 beanName 来调用 ``setBeanName(String name)`` 方法。

    2.2 如果 Bean 实现 ``BeanFactoryAware`` 接口，工厂通过传递自身的实例来调用 ``setBeanFactory(BeanFactory beanFactory)`` 方法。

3. 调用相应的方法，进一步初始化 Bean 对象
    3.1 如果存在与 Bean 关联的任何 ``BeanPostProcessor`` 们，则调用 ``preProcessBeforeInitialization(Object bean, String beanName)`` 方法。

    3.2 如果 Bean 实现 ``InitializingBean`` 接口，则会调用 ``afterPropertiesSet()`` 方法。

    3.3 如果为 Bean 指定了 ``init`` 方法（例如 ``<bean />`` 的 ``init-method`` 属性），那么将调用该方法。

    3.4 如果存在与 Bean 关联的任何 ``BeanPostProcessor`` 们，则将调用 ``postProcessAfterInitialization(Object bean, String beanName)`` 方法。

Spring Bean 的销毁流程如下：

1. 如果 Bean 实现 ``DisposableBean`` 接口，当 spring 容器关闭时，会调用 ``destroy()`` 方法。

2. 如果为 bean 指定了 ``destroy`` 方法（例如 ``<bean />`` 的 ``destroy-method`` 属性），那么将调用该方法。

整体图
![](/img/spring_bean_life.jpg)

[toTop](#jump)


# BeanFactory和FactoryBean的区别

BeanFactory:
BeanFactory是个Factory，也就是IOC容器或对象工厂，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 ``DefaultListableBeanFactory``、``XmlBeanFactory``、``ApplicationContext``等，其中``XmlBeanFactory``就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。``XmlBeanFactory``类将持有此XML配置元数据，并用它来构建一个完全可配置的系统或应用。   原始的``BeanFactory``无法支持spring的许多插件，如AOP功能、Web应用等。 
``ApplicationContext``包含``BeanFactory``的所有功能，通常建议比``BeanFactory``优先 

``ApplicationContext``以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，``ApplicationContext``包还提供了以下的功能： 
• MessageSource, 提供国际化的消息访问 
• 资源访问，如URL和文件 
• 事件传播 
• 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层; 

FactoryBean:
FactoryBean是个Bean。对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似,它是实现了``FactoryBean<T>``接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的``getObject()``返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个``&``符号来获取。
例子：
当从IOC容器中获取FactoryBeanPojo对象的时候，用```getBean(String BeanName)```获取的确是Student对象，可以看到在``FactoryBeanPojo``中的type属性设置为student的时候，会在``getObject()``方法中返回Student对象。所以说从IOC容器获取实现了FactoryBean的实现类时，返回的却是实现类中的``getObject``方法返回的对象，要想获取FactoryBean的实现类，得在``getBean(String BeanName)``中的BeanName之前加上     ,写成``getBean(String &BeanName)``

参考1 :[BeanFactory 简介以及它 和FactoryBean的区别](https://www.cnblogs.com/aspirant/p/9082858.html)

参考2 :[BeanFactory和FactoryBean的区别](https://blog.csdn.net/wangbiao007/article/details/53183764)

参考3 :[spring中BeanFactory和FactoryBean的区别](https://blog.csdn.net/qiesheng/article/details/72875315)

参考4 :[Spring BeanFactory与FactoryBean的区别及其各自的详细介绍于用法](https://www.cnblogs.com/redcool/p/6413461.html)

[toTop](#jump)