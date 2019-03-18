<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [SpringBoot启动过程](#springboot启动过程)
- [Spring Boot 提供了哪些核心功能](#spring-boot-提供了哪些核心功能)
- [springboot自动配置的原理](#springboot自动配置的原理)

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
