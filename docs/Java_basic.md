<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [8个基本类型](#8个基本类型)
- [Object,String类的方法](#objectstring类的方法)
- [static](#static)
- [异常](#异常)
- [HTTP GET和POST区别](#http-get和post区别)
- [Java中private、protected、public和default的区别](#java中privateprotectedpublic和default的区别)
- [sleep wait区别](#sleep-wait区别)
- [String 的intern()方法](#string-的intern方法)
- [分派](#分派)
    - [静态分派](#静态分派)

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

# String 的intern()方法

一个初始时为空的字符串池，它由类 String 私有地维护。

当调用 intern 方法时，如果池已经包含一个等于此 String 对象的字符串（该对象由 equals(Object) 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中，并且返回此 String 对象的引用。

它遵循对于任何两个字符串 s 和 t，当且仅当 s.equals(t) 为 true 时，s.intern() == t.intern() 才为 true。

所有字面值字符串和字符串赋值常量表达式都是内部的。

代码示例:

```java
public class Test {
	public static void main(String argv[])
	{
		String s1 = "HelloWorld";
		String s2 = new String("HelloWorld");
		String s3 = "Hello";
		String s4 = "World";
		String s5 = "Hello" + "World";
		String s6 = s3 + s4;
		
		System.out.println(s1 == s2);//false
		System.out.println(s1 == s5);//true
		System.out.println(s1 == s6);//false
		System.out.println(s1 == s6.intern());//true
		System.out.println(s2 == s2.intern());//false
	}
}
```

s1 创建的 HelloWorld 存在于方法区中的常量池其中的字符串池，而 s2 创建的 HelloWorld 存在于堆中，故第一条 false 。

s5 的创建是先进行右边表达式的字符串连接，然后才对 s5 进行赋值。赋值的时候会搜索池中是否有 HelloWorld 字符串，若有则把 s5 引用指向该字符串，若没有则在池中新增该字符串。显然 s5 的创建是用了 s1 已经创建好的字面量，故 true 。

第三个比较容易弄错，s6 = s3 + s4; 其实相当于 s6 = new String(s3 + s4); s6 是存放于堆中的，不是字面量。所以 s1 不等于 s6 。

第四个 s6.intern() 首先获取了 s6 的 HelloWorld 字符串，然后在字符串池中查找该字符串，找到了 s1 的 HelloWorld 并返回。这里如果事前字符串池中没有 HelloWorld 字符串，那么还是会在字符串池中创建一个 HelloWorld 字符串再返回。总之返回的不是堆中的 s6 那个字符串。

第四条也能解释为什么第五条是 false 。s2是堆中的 HelloWorld，s2.intern() 是字符串池中的 HelloWorld 。

如果把
```java
String s6 = s3 + s4;
```
改成
```java
String s6 = (s3 + s4).intern();
```
s6 存储的 HelloWorld 是存放字符串池中

[toTop](#jump)


# 分派
## 静态分派

```java
static abstract class A{}

static  class B extends A{}

static class C extends A{}

public static void sayHello(A a){
	System.out.println("a");
}

public static void sayHello(B a){
	System.out.println("b");
}

public static void sayHello(C a){
	System.out.println("c");
}

public static void main(String args[]) {
		A a = new B();
        A b = new C();
        sayHello(a);
        sayHello(b);	
}
```
* 输出

```java
a
a
```
* 解析

在代码
```java
A a = new B();
```
中A类型为变量的静态类型(Static Type),或者叫做外观模型(Apparent Type), 后面的B类型为变量的实际类型(Actual Type)。虚拟机在重载时通过参数的静态类型而不是实际类型作为判断依据。


[toTop](#jump)

