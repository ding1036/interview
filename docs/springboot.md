<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [SpringBoot启动过程](#springboot启动过程)

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