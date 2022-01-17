<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [ESAPI](#esapi)
- [如何避免 sql 注入？](#如何避免-sql-注入)
- [什么是 XSS 攻击，如何避免？](#什么是-xss-攻击如何避免)
- [什么是 CSRF 攻击，如何避免？](#什么是-csrf-攻击如何避免)
- [缺少X-Frame-Options头部信息](#缺少x-frame-options头部信息)
- [Spring Boot应对Log4j2注入漏洞](#spring-boot应对log4j2注入漏洞)

<!-- /TOC -->

# ESAPI

# 如何避免 sql 注入？

1) PreparedStatement（简单又有效的方法）
2) 使用正则表达式过滤传入的参数
3) 字符串过滤
4) JSP中调用该函数检查是否包函非法字符
5) JSP页面判断代码

[toTop](#jump)

# 什么是 XSS 攻击，如何避免？

XSS攻击又称CSS,全称Cross Site Script  （跨站脚本攻击），其原理是攻击者向有XSS漏洞的网站中输入恶意的 HTML 代码，当用户浏览该网站时，这段 HTML 代码会自动执行，从而达到攻击的目的。XSS 攻击类似于 SQL 注入攻击，SQL注入攻击中以SQL语句作为用户输入，从而达到查询/修改/删除数据的目的，而在xss攻击中，通过插入恶意脚本，实现对用户游览器的控制，获取用户的一些信息。 XSS是 Web 程序中常见的漏洞，XSS 属于被动式且用于客户端的攻击方式。

**XSS防范**的总体思路是：对输入(和URL参数)进行过滤，对输出进行编码。

[toTop](#jump)

#  什么是 CSRF 攻击，如何避免？

CSRF（Cross-site request forgery）也被称为 one-click attack或者 session riding，中文全称是叫跨站请求伪造。一般来说，攻击者通过伪造用户的浏览器的请求，向访问一个用户自己曾经认证访问过的网站发送出去，使目标网站接收并误以为是用户的真实操作而去执行命令。常用于盗取账号、转账、发送虚假消息等。攻击者利用网站对请求的验证漏洞而实现这样的攻击行为，网站能够确认请求来源于用户的浏览器，却不能验证请求是否源于用户的真实意愿下的操作行为。

如何避免：

1. 验证 HTTP Referer 字段

    HTTP头中的Referer字段记录了该 HTTP 请求的来源地址。在通常情况下，访问一个安全受限页面的请求来自于同一个网站，而如果黑客要对其实施 CSRF
    攻击，他一般只能在他自己的网站构造请求。因此，可以通过验证Referer值来防御CSRF 攻击。

2. 使用验证码

    关键操作页面加上验证码，后台收到请求后通过判断验证码可以防御CSRF。但这种方法对用户不太友好。

3. 在请求地址中添加token并验证

    CSRF 攻击之所以能够成功，是因为黑客可以完全伪造用户的请求，该请求中所有的用户验证信息都是存在于cookie中，因此黑客可以在不知道这些验证信息的情况下直接利用用户自己的cookie 来通过安全验证。要抵御 CSRF，关键在于在请求中放入黑客所不能伪造的信息，并且该信息不存在于 cookie 之中。可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有token或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。这种方法要比检查 Referer 要安全一些，token 可以在用户登陆后产生并放于session之中，然后在每次请求时把token 从 session 中拿出，与请求中的 token 进行比对，但这种方法的难点在于如何把 token 以参数的形式加入请求。
    对于 GET 请求，token 将附在请求地址之后，这样 URL 就变成 http://url?csrftoken=tokenvalue。
    而对于 POST 请求来说，要在 form 的最后加上 <input type="hidden" name="csrftoken" value="tokenvalue"/>，这样就把token以参数的形式加入请求了。

4. 在HTTP 头中自定义属性并验证

    这种方法也是使用 token 并进行验证，和上一种方法不同的是，这里并不是把 token 以参数的形式置于 HTTP 请求之中，而是把它放到 HTTP 头中自定义的属性里。通过 XMLHttpRequest 这个类，可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，并把 token 值放入其中。这样解决了上种方法在请求中加入 token 的不便，同时，通过 XMLHttpRequest 请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会透过 Referer 泄露到其他网站中去。

[toTop](#jump)

#  缺少X-Frame-Options头部信息
Apache tomcat 7.0.90 和 tomcat 8以上都有HttpHeaderSecurityFilter

可以在tomcat下的conf里的web.xml中增加以下过滤器

```xml
<filter>
  <filter-name>httpHeaderSecurity</filter-name>
  <filter-class>org.apache.catalina.filters.HttpHeaderSecurityFilter</filter-class>
  <init-param>
    <param-name>antiClickJackingEnabled</param-name>
    <param-value>true</param-value>
  </init-param>
  <init-param>
    <param-name>antiClickJackingOption</param-name>
    <param-value>SAMEORIGIN</param-value>
  </init-param>
  <async-supported>true</async-supported>
</filter>
<filter-mapping>
  <filter-name>httpHeaderSecurity</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

 如果没有HttpHeaderSecurityFilter，需要自己写过滤器，添加如下代码，自己在项目中配置拦截。

 ```java
HttpServletResponse response = (HttpServletResponse) sResponse;
response.addHeader("x-frame-options","SAMEORIGIN"); 
 ```

 X-Frame-Options 有三个值:

**DENY**

表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。

**SAMEORIGIN**

表示该页面可以在相同域名页面的 frame 中展示。

**ALLOW-FROM**

表示该页面可以在指定来源的 frame 中展示。

换一句话说，如果设置为 DENY，不光在别人的网站 frame 嵌入时会无法加载，在同域名页面中同样会无法加载。

另一方面，如果设置为 SAMEORIGIN，那么页面就可以在同域名页面的 frame 中嵌套。


* Apache配置
配置 Apache 在所有页面上发送 X-Frame-Options 响应头，需要把下面这行添加到 'site' 的配置中:
1、打开httpd.conf 找到LoadModule headers_module modules/mod_headers.so模块，去掉前面的# 号 启用该模块
2、在此处加上 ``Header always append X-Frame-Options SAMEORIGIN``

```xml
<IfModule headers_module>
RequestHeader unset DNT env=bad_DNT
Header always append X-Frame-Options SAMEORIGIN
</IfModule>
```
重启完就OK。

* Nginx
配置 nginx 发送 X-Frame-Options 响应头，把下面这行添加到 'http', 'server' 或者 'location' 的配置中:

```
add_header X-Frame-Options SAMEORIGIN;
```

* IIS
配置 IIS 发送 X-Frame-Options 响应头，添加下面的配置到 Web.config 文件中

```xml

<system.webServer>
  ...
​
  <httpProtocol>
    <customHeaders>
      <add name="X-Frame-Options" value="SAMEORIGIN" />
    </customHeaders>
  </httpProtocol>
​
  ...
</system.webServer>
```


[toTop](#jump)


# Spring Boot应对Log4j2注入漏洞
默认配置不受影响

Spring Boot默认日志组件是``logback``，开发者通过日志门面``Slf4j``进行集成对接。Spring Boot 用户只有在将默认日志系统切换到 Log4J2 时才会受到此漏洞的影响。Spring Boot包含的``log4j-to-slf4j``
和``log4j-api``
、``spring-boot-starter-logging``
不能独立利用。只有``log4j-core``
在日志消息中使用和包含用户输入的应用程序容易受到攻击。

也就是说Spring Boot现在包含Log4J2的依赖只要你不启用是不会触发漏洞的。
下版本更新补丁

Spring Boot将在2021 年 12 月 23 日后发布的 2.5.8
和 2.6.2
版本将采用打了补丁的Log4J v2.15.0，但由于这是一个极其严重的漏洞，一定要覆盖我们的依赖项管理并尽快升级您的 Log4J2 依赖项。
* Maven用户

对于 Maven 用户，您可以通过覆盖自己项目中pom.xml
的版本号配置属性来修改该依赖的版本号。提升Log4J2到安全版本只需要：

```xml
<properties>
    <log4j2.version>2.15.0</log4j2.version>
</properties>
```

然后使用
```shell
./mvnw dependency:list | grep log4j
```
命令运行以检查版本是否为 2.15.0

* Gradle用户

    对于大多数用户来说，设置``log4j2.version``
    属性就足够了：

```xml
ext['log4j2.version'] = '2.15.0'
```

如果你的Gradle并没有直接对Spring Boot进行依赖管理，你可以添加Log4J BOM
依赖项:

```xml
implementation(platform("org.apache.logging.log4j:log4j-bom:2.15.0"))
```

万金油”的方法是声明一个Gradle的resolutionStrategy

```xml
configurations.all {
 resolutionStrategy.eachDependency { DependencyResolveDetails details ->
  if (details.requested.group == 'org.apache.logging.log4j') {
   details.useVersion '2.15.0'
  }
 }
}
```

上面三种方法无论你使用哪种，安全起见都需要使用下面的命令进行检查确认：

```shell
./gradlew dependencyInsight --dependency log4j-core
```


[toTop](#jump)