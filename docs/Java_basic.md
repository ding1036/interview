<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [8个基本类型](#8个基本类型)
- [Object,String类的方法](#objectstring类的方法)
- [static](#static)
- [异常](#异常)
- [HTTP GET和POST区别](#http-get和post区别)
- [Java中private、protected、public和default的区别](#java中privateprotectedpublic和default的区别)
- [sleep wait区别](#sleep-wait区别)

<!-- /TOC -->

# 8个基本类型
byte boolean 1字节  -128~127（-2^8~2^8-1）1字节8位
short char 2字节  
int float 4字节  
double long 8字节  

[toTop](#jump)


# Object,String类的方法
Object:getClass(),hashCode(),equals(),toString(),wait(),  notify(),notifyAll()  
String:equals(),replace(),split(),subString() 

[toTop](#jump) 

# static
可用于修饰成员变量和成员函数，静态随类的加载而加载，可以直接用类进行访问。静态方法不能访问非静态成员变量和非静态成员方法，因为非静态成员方法/变量必须依赖具体的对象才能被调用。Java中Static方法不能被覆盖。  

[toTop](#jump)

# 异常
Throwable是所有异常的父类，包含Error和Exception。  
Error：OutofMemory，StackOverflow。  
Exception包括受检异常和非受检异常(RuntimeException)  
受检异常包括：SQLException,IOException，JSONException  
非受检异常包括：NullPointException,IndexOutOfBoundsException,FileNotFoundException,SecurityException,IllegalArgumentException,NumberFormatException  

[toTop](#jump)

# HTTP GET和POST区别
GET：是请求资源。安全，幂等（同一个请求多次和一次效果完全相同），可缓存，参数长度有限制（受限于url长度）  
POST：根据报文主题对指定资源做出处理，不安全，不幂等，不可缓存 

[toTop](#jump)

# Java中private、protected、public和default的区别
![](/img/javaclass_type.png)  

# sleep wait区别
sleep：Thread类的方法，必须带一个时间参数。会让当前线程休眠进入阻塞状态并释放CPU，提供其他线程运行的机会且不考虑优先级，**但如果有同步锁则sleep不会释放锁即其他线程无法获得同步锁**

wait：Object类的方法，**必须放在循环体和同步代码块中，执行该方法的线程会释放锁**，进入线程等待池中等待被再次唤醒(notify随机唤醒，notifyAll全部唤醒，线程结束自动唤醒)即放入锁池中竞争同步锁

[toTop](#jump)


