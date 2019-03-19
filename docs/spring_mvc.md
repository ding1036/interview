<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [Spring MVC Controller单例还是多例](#spring-mvc-controller单例还是多例)
- [springMVC请求流程详解](#springmvc请求流程详解)
- [SpringMVC的拦截器（Interceptor）和过滤器（Filter）的区别与联系](#springmvc的拦截器interceptor和过滤器filter的区别与联系)
    - [过滤器](#过滤器)
    - [拦截器](#拦截器)
    - [过滤器和拦截器的区别：](#过滤器和拦截器的区别)
    - [触发时机](#触发时机)

<!-- /TOC -->

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

#  springMVC请求流程详解

SpringMVC核心处理流程：

1) DispatcherServlet前端控制器接收发过来的请求，交给HandlerMapping处理器映射器

2) HandlerMapping处理器映射器，根据请求路径找到相应的HandlerAdapter处理器适配器（处理器适配器就是那些拦截器或Controller）

3) HandlerAdapter处理器适配器，处理一些功能请求，返回一个ModelAndView对象（包括模型数据、逻辑视图名）

4) ViewResolver视图解析器，先根据ModelAndView中设置的View解析具体视图

5) 然后再将Model模型中的数据渲染到View上

![](/img/springmvc_flow.png)

[toTop](#jump)

# SpringMVC的拦截器（Interceptor）和过滤器（Filter）的区别与联系

## 过滤器

**依赖于servlet容器**,是JavaEE标准,是在请求进入容器之后，还未进入Servlet之前进行预处理，并且在请求结束返回给前端这之间进行后期处理。在实现上基于函数回调，可以对几乎所有请求进行过滤，但是缺点是一个过滤器实例**只能在容器初始化时调用一次**。使用过滤器的目的是用来做一些过滤操作，获取我们想要获取的数据，比如：在过滤器中修改字符编码；在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等

* filter功能，它使用户可以改变一个 request和修改一个response. Filter 不是一个servlet,它不能产生一个response,它能够在一个request到达servlet之前预处理request,也可以在离开 servlet时处理response.换种说法,filter其实是一个”servlet chaining”(servlet 链).

一个Filter包括：
1) 在servlet被调用之前截获;
2) 在servlet被调用之前检查servlet request;
3) 根据需要修改request头和request数据;
4) 根据需要修改response头和response数据;
5) 在servlet被调用之后截获.

```java

 @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("before...");
        chain.doFilter(request, response);
        System.out.println("after...");
    }
```

chain.doFilter(request, response);这个方法的调用作为分水岭。事实上调用Servlet的doService()方法是在chain.doFilter(request, response);这个方法中进行的。

## 拦截器

拦截器不依赖与servlet容器，**依赖于web框架**，在SpringMVC中就是依赖于SpringMVC框架。在实现上**基于Java的反射机制**，属于面向切面编程（AOP）的一种运用。由于拦截器是基于web框架的调用，因此可以使用spring的依赖注入（DI）获取IOC容器中的各个bean,进行一些业务操作，同时一个拦截器实例在一个controller生命周期之内**可以多次调用**。但是缺点是**只能对controller请求进行拦截**，
1) 请求还没有到controller层时进行拦截，
2) 请求走出controller层次，还没有到渲染时图层时进行拦截，
3) 结束视图渲染，但是还没有到servlet的结束时进行拦截。对其他的一些比如直接访问静态资源的请求则没办法进行拦截处理,拦截器功在对请求权限鉴定方面确实很有用处

![](/img/filterAndIntercept.png)

```java

 @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
```

　a.preHandle()这个方法是在过滤器的chain.doFilter(request, response)方法的前一步执行，也就是在 [System.out.println("before...")][chain.doFilter(request, response)]之间执行。

 

　　b.preHandle()方法之后，在return ModelAndView之前进行，可以操控Controller的ModelAndView内容。

 

　　c.afterCompletion()方法是在过滤器返回给前端前一步执行，也就是在[chain.doFilter(request, response)][System.out.println("after...")]之间执行。

## 过滤器和拦截器的区别：

1) 拦截器是基于java的反射机制的，而过滤器是基于函数回调。

2) 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。

3) 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。

4) 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。

5) 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。

6) 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

拦截器可以获取ioc中的service bean实现业务逻辑，拦截器可以获取ioc中的service bean实现业务逻辑，拦截器可以获取ioc中的service bean实现业务逻辑，


## 触发时机
过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前。

过滤器的触发时机是容器后，servlet之前，所以过滤器的
```java
doFilter(
ServletRequest request, ServletResponse response, FilterChain chain
)
```

的入参是``ServletRequest`` ，而不是``httpservletrequest``。因为过滤器是在``httpservlet``之前。
![](/img/filter.png)

![](/img/filterAndIntercept2.png)

过滤器实现

```java
public class LogCostFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();

        filterChain.doFilter(servletRequest,servletResponse);
        
        System.out.println("Execute cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void destroy() {
 
    }
}

```
Spring boot中，我们需要``FilterRegistrationBean``来完成配置。其实现过程如下：

```java
@Configuration
public class FilterConfig {
 
    @Bean
    public FilterRegistrationBean registFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new LogCostFilter());
        registration.addUrlPatterns("/*");
        registration.setName("LogCostFilter");
        registration.setOrder(1);
        return registration;
    }
 
}
```

除了通过 ``FilterRegistrationBean`` 来配置以外，还有一种更直接的办法，直接通过注解就可以完成了：

```java
@WebFilter(urlPatterns = "/*", filterName = "logFilter2")
public class LogCostFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("LogFilter2 Execute cost=" + (System.currentTimeMillis() - start));
    }
 
    @Override
    public void destroy() {
 
    }
}
```

这里直接用``@WebFilter``就可以进行配置，同样，可以设置url匹配模式，过滤器名称等。这里需要注意一点的是``@WebFilter``这个注解是``Servlet3.0``的规范，并不是Spring boot提供的。除了这个注解以外，我们还需在配置类中加另外一个注解：``@ServletComponetScan``，指定扫描的包。

```java

@SpringBootApplication
@MapperScan("com.pandy.blog.dao")
@ServletComponentScan("com.pandy.blog.filters")
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```

拦截器实现

```java
public class LogCostInterceptor implements HandlerInterceptor {
    long start = System.currentTimeMillis();
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        start = System.currentTimeMillis();
        return true;
    }
 
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }
}
```

需要实现``HandlerInterceptor``这个接口，这个接口包括三个方法，``preHandle``是请求执行前执行的，``postHandler``是请求结束执行的，但只有``preHandle``方法返回``true``的时候才会执行，``afterCompletion``是视图渲染完成后才执行，同样需要``preHandle``返回``true``，该方法通常用于清理资源等工作。除了实现上面的接口外，我们还需对其进行配置：

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogCostInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}

```
    Spring 5.0 以后WebMvcConfigurerAdapter会取消掉,可以直接实现WebMvcConfigurer替代


参考 1 :[spring过滤器和拦截器的区别和联系](https://www.cnblogs.com/nizuimeiabc1/p/6774073.html)

[toTop](#jump)