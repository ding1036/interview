<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [SpringBoot启动过程](#springboot启动过程)
- [Spring Boot 提供了哪些核心功能](#spring-boot-提供了哪些核心功能)
- [springboot自动配置的原理](#springboot自动配置的原理)
- [springboot加载配置类过程](#springboot加载配置类过程)
- [Spring Boot运行原理](#spring-boot运行原理)
    - [元注解](#元注解)
    - [条件注解@Conditional](#条件注解conditional)
    - [@SpringBootApplication](#springbootapplication)

<!-- /TOC -->

# SpringBoot启动过程
1.构造SpringApplication的实例

2.调用SpringApplication.run()方法

构造SpringApplicationRunListeners 实例
发布ApplicationStartedEvent事件
SpringApplicationRunListeners 实例准备环境信息
创建ApplicationContext对象
ApplicationContext实例准备环境信息
刷新的上下文


参考1 :[SpringBoot启动过程原理](https://blog.csdn.net/u010811939/article/details/80592461)

[toTop](#jump)

# Spring Boot 提供了哪些核心功能


1、独立运行 Spring 项目

Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需要通过 ``java -jar xx.jar`` 来运行。

2、内嵌 Servlet 容器

Spring Boot 可以选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

3、提供 Starter 简化 Maven 配置

Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。

4、自动配置 Spring Bean

Spring Boot 检测到特定类的存在，就会针对这个应用做一定的配置，进行自动配置 Bean ，这样会极大地减少我们要使用的配置。

当然，Spring Boot 只考虑大多数的开发场景，并不是所有的场景，若在实际开发中我们需要配置Bean ，而 Spring Boot 没有提供支持，则可以自定义自动配置进行解决。

5、准生产的应用监控

Spring Boot 提供基于 HTTP、JMX、SSH 对运行时的项目进行监控。

6、无代码生成和 XML 配置

Spring Boot 没有引入任何形式的代码生成，它是使用的 Spring 4.0 的条件 ``@Condition ``注解以实现根据条件进行配置。同时使用了 Maven /Gradle 的依赖传递解析机制来实现 Spring 应用里面的自动配置。


[toTop](#jump)

# springboot自动配置的原理

在spring程序main方法中 添加``@SpringBootApplication``或者``@EnableAutoConfiguration``

会自动去maven中读取每个starter中的``spring.factories``文件  该文件里配置了所有需要被创建spring容器中的bean


参考1 ：[springboot+springcloud相关面试题](https://blog.csdn.net/panhaigang123/article/details/79587612)

参考2 ：[一个面试题引起的SpringBoot启动解析](https://juejin.im/post/5b679fbc5188251aad213110)

[toTop](#jump)

# springboot加载配置类过程


参考1 :[springboot启动时是如何加载配置文件application.yml文件](https://blog.csdn.net/chengkui1990/article/details/79866499)

[toTop](#jump)

# Spring Boot运行原理

**Spring Boot框架本质上就是通过组合注解的方式实现了诸多Spring注解的组合**

## 元注解
元注解，**就是可以注解到其他注解上的注解**，被注解的注解就是**组合注解**,``@Configuration``就是一个这样的组合注解

## 条件注解@Conditional
**根据满足某一个特定条件与否来决定是否创建某个特定的Bean**，例如，某个依赖包jar在一个类路径的时候，自动配置一个或多个Bean时，可以通过@Conditional注解来实现只有某个Bean被创建时才会创建另外一个Bean，这样就可以依据特定的条件来控制Bean的创建行为，这样的话我们就可以利用这样一个特性来实现一些自动的配置。
对于Spring Boot实现自动配置来说是一个核心的基础能力

## @SpringBootApplication

``@SpringBootApplication``是一个复合注解，包括``@ComponentScan``，和``@SpringBootConfiguration``，``@EnableAutoConfiguration``

``@SpringBootConfiguration``继承自``@Configuration``，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以``@Bean``注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。

``@EnableAutoConfiguration``的作用启动自动的配置，``@EnableAutoConfiguration``注解的意思就是Springboot根据你添加的jar包来配置你项目的默认配置，比如根据``spring-boot-starter-web`` ，来判断你的项目是否需要添加了webmvc和tomcat，就会自动的帮你配置web项目中所需要的默认配置。

``@ComponentScan``，扫描当前包及其子包下被``@Component``，``@Controller``，``@Service``，``@Repository``注解标记的类并纳入到spring容器中进行管理。是以前的``<context:component-scan>``（以前使用在xml中使用的标签，用来扫描包配置的平行支持）。


```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
....
}
```



参考1 :[Spring Boot运行原理](https://mp.weixin.qq.com/s/4yQZszewKsAARAUO6-K-og)

参考2 :[@ComponentScan源码分析](https://blog.csdn.net/qq_20597727/article/details/82713306)