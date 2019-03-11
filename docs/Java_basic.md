<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [8个基本类型](#8个基本类型)
- [Object,String类的方法](#objectstring类的方法)
- [static](#static)
- [异常](#异常)
- [HTTP GET和POST区别](#http-get和post区别)
- [Java中private、protected、public和default的区别](#java中privateprotectedpublic和default的区别)
- [sleep wait区别](#sleep-wait区别)
- [java一些常用包](#java一些常用包)
- [String 的intern()方法](#string-的intern方法)
- [分派](#分派)
    - [静态分派](#静态分派)
- [java问题排查](#java问题排查)
    - [linux 命令类](#linux-命令类)
        - [tail](#tail)
        - [grep](#grep)
        - [awk](#awk)
        - [find](#find)
        - [pgm](#pgm)
        - [tsar](#tsar)
        - [top](#top)
        - [其他](#其他)
    - [排查利器](#排查利器)
        - [btrace](#btrace)
        - [Greys](#greys)
        - [javOSize](#javosize)
        - [JProfiler](#jprofiler)
    - [java自带工具](#java自带工具)
        - [jps](#jps)
        - [jstack](#jstack)
        - [jinfo](#jinfo)
        - [jmap](#jmap)
        - [jstat](#jstat)
        - [jdb](#jdb)
        - [CHLSDB](#chlsdb)
    - [VM options](#vm-options)
    - [jar包冲突](#jar包冲突)
    - [其他](#其他-1)
        - [dmesg](#dmesg)
        - [Arthas](#arthas)
- [泛型](#泛型)
    - [集合与泛型](#集合与泛型)
- [String StringBuffer StringBuilder](#string-stringbuffer-stringbuilder)
    - [String](#string)
    - [StringBuffer](#stringbuffer)
    - [StringBuilder](#stringbuilder)
- [反射](#反射)

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

# java一些常用包
java.lang 	
该包提供了Java编程的基础类，例如 Object、Math、String、StringBuffer、System、Thread等，不使用该包就很难编写Java代码了。

java.util 	
该包提供了包含集合框架、遗留的集合类、事件模型、日期和时间实施、国际化和各种实用工具类（字符串标记生成器、随机数生成器和位数组）。

java.io 	
该包通过文件系统、数据流和序列化提供系统的输入与输出。

java.net 	
该包提供实现网络应用与开发的类。

java.sql 	
该包提供了使用Java语言访问并处理存储在数据源（通常是一个关系型数据库）中的数据API。

java.awt 	
这两个包提供了GUI设计与开发的类。java.awt包提供了创建界面和绘制图形图像的所有类，而javax.swing包提供了一组“轻量级”的组件，尽量让这些组件在所有平台上的工作方式相同。

java.text 	
提供了与自然语言无关的方式来处理文本、日期、数字和消息的类和接口。

 [java.lang](http://beyond429.iteye.com/blog/344024)

 [java.util](http://blog.csdn.net/abeetle/article/details/1497706)

 [java.io](http://blog.csdn.net/yczz/article/details/38761237)

[java.net](https://zhidao.baidu.com/question/87061018.html)

[java.sql](http://www.360doc.com/content/12/0329/09/1200324_198823027.shtml)

[java.awt](http://blog.csdn.net/u013147587/article/details/49815031)

[javax.swing](http://blog.sina.com.cn/s/blog_4a7979120100087g.html)

[java.text](http://www.cnblogs.com/beibeibao/p/3411750.html)

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


# java问题排查

## linux 命令类

### tail

```shell
tail -300f shopbase.log #倒数300行并进入实时监听文件写入模式
```
### grep

```shell
grep forest f.txt     #文件查找
grep forest f.txt cpf.txt #多文件查找
grep 'log' /home/admin -r -n #目录下查找所有符合关键字的文件
cat f.txt | grep -i shopbase    
grep 'shopbase' /home/admin -r -n --include *.{vm,java} #指定文件后缀
grep 'shopbase' /home/admin -r -n --exclude *.{vm,java} #反匹配
seq 10 | grep 5 -A 3    #上匹配
seq 10 | grep 5 -B 3    #下匹配
seq 10 | grep 5 -C 3    #上下匹配，平时用这个就妥了
cat f.txt | grep -c 'SHOPBASE'
```

### awk
1 基础命令

```shell
awk '{print $4,$6}' f.txt
awk '{print NR,$0}' f.txt cpf.txt    
awk '{print FNR,$0}' f.txt cpf.txt
awk '{print FNR,FILENAME,$0}' f.txt cpf.txt
awk '{print FILENAME,"NR="NR,"FNR="FNR,"$"NF"="$NF}' f.txt cpf.txt
echo 1:2:3:4 | awk -F: '{print $1,$2,$3,$4}'
```
2 匹配

```shell
awk '/ldb/ {print}' f.txt   #匹配ldb
awk '!/ldb/ {print}' f.txt  #不匹配ldb
awk '/ldb/ && /LISTEN/ {print}' f.txt   #匹配ldb和LISTEN
awk '$5 ~ /ldb/ {print}' f.txt #第五列匹配ldb
```

3 内建变量
NR:NR表示从awk开始执行后，按照记录分隔符读取的数据次数，默认的记录分隔符为换行符，因此默认的就是读取的数据行数，NR可以理解为Number of Record的缩写。

FNR:在awk处理多个输入文件的时候，在处理完第一个文件后，NR并不会从1开始，而是继续累加，因此就出现了FNR，每当处理一个新文件的时候，FNR就从1开始计数，FNR可以理解为File Number of Record。

NF: NF表示目前的记录被分割的字段的数目，NF可以理解为Number of Field。


### find

```shell
sudo -u admin find /home/admin /tmp /usr -name \*.log(多个目录去找)
find . -iname \*.txt(大小写都匹配)
find . -type d(当前目录下的所有子目录)
find /usr -type l(当前目录下所有的符号链接)
find /usr -type l -name "z*" -ls(符号链接的详细信息 eg:inode,目录)
find /home/admin -size +250000k(超过250000k的文件，当然+改成-就是小于了)
find /home/admin f -perm 777 -exec ls -l {} \; (按照权限查询文件)
find /home/admin -atime -1  1天内访问过的文件
find /home/admin -ctime -1  1天内状态改变过的文件    
find /home/admin -mtime -1  1天内修改过的文件
find /home/admin -amin -1  1分钟内访问过的文件
find /home/admin -cmin -1  1分钟内状态改变过的文件    
find /home/admin -mmin -1  1分钟内修改过的文件
```

### pgm

批量查询vm-shopbase满足条件的日志
```shell
pgm -A -f vm-shopbase 'cat /home/admin/shopbase/logs/shopbase.log.2017-01-17|grep 2069861630'

```

### tsar

tsar是采集工具。很好用, 将历史收集到的数据持久化在磁盘上，可以快速来查询历史的系统数据。当然实时的应用情况也是可以查询的啦。大部分机器上都有安装。

```shell
tsar  ###可以查看最近一天的各项指标
```
![](/img/tsar.jpg)


```shell
tsar --live ###可以查看实时指标，默认五秒一刷
```
![](/img/tsar_live.jpg)


```shell
tsar -d 20161218 ###指定查看某天的数据，貌似最多只能看四个月的数据
```
![](/img/tsar_d.jpg)

```shell
tsar --mem
tsar --load
tsar --cpu
###当然这个也可以和-d参数配合来查询某天的单个指标的情况 
```
![](/img/tsar_mem.jpg)
![](/img/tsar_load.jpg)
![](/img/tsar_cpu.jpg)

### top

top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了

```shell
ps -ef | grep java
top -H -p pid
```
获得线程10进制转16进制后jstack去抓看这个线程到底在干啥

### 其他

```shell
netstat -nat|awk  '{print $6}'|sort|uniq -c|sort -rn 
#查看当前连接，注意close_wait偏高的情况，比如如下
```
![](/img/netstat.jpg)

## 排查利器

### btrace
1) 查看当前谁调用了ArrayList的add方法，同时只打印当前ArrayList的size大于500的线程调用栈

```java
@OnMethod(clazz = "java.util.ArrayList", method="add", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/"))
public static void m(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod, @TargetInstance Object instance, @TargetMethodOrField String method) {
   if(getInt(field("java.util.ArrayList", "size"), instance) > 479){
       println("check who ArrayList.add method:" + probeClass + "#" + probeMethod  + ", method:" + method + ", size:" + getInt(field("java.util.ArrayList", "size"), instance));
       jstack();
       println();
       println("===========================");
       println();
   }
}
```

2) 监控当前服务方法被调用时返回的值以及请求的参数

```java
@OnMethod(clazz = "com.taobao.sellerhome.transfer.biz.impl.C2CApplyerServiceImpl", method="nav", location = @Location(value = Kind.RETURN))
public static void mt(long userId, int current, int relation, String check, String redirectUrl, @Return AnyType result) {
   println("parameter# userId:" + userId + ", current:" + current + ", relation:" + relation + ", check:" + check + ", redirectUrl:" + redirectUrl + ", result:" + result);
}
```

更多：[https://github.com/btraceio/btrace](https://github.com/btraceio/btrac)

注意:

经过观察，1.3.9的release输出不稳定，要多触发几次才能看到正确的结果
正则表达式匹配trace类时范围一定要控制，否则极有可能出现跑满CPU导致应用卡死的情况
由于是字节码注入的原理，想要应用恢复到正常情况，需要重启应用。

### Greys

说几个挺棒的功能(部分功能和btrace重合):

sc -df xxx: 输出当前类的详情,包括源码位置和classloader结构

trace class method: 相当喜欢这个功能! 很早前可以早JProfiler看到这个功能。打印出当前方法调用的耗时情况，细分到每个方法。

### javOSize

就说一个功能
classes：通过修改了字节码，改变了类的内容，即时生效。 所以可以做到快速的在某个地方打个日志看看输出，缺点是对代码的侵入性太大。但是如果自己知道自己在干嘛，的确是不错的玩意儿。

其他功能Greys和btrace都能很轻易做的到，不说了。

### JProfiler

之前判断许多问题要通过JProfiler，但是现在Greys和btrace基本都能搞定了。再加上出问题的基本上都是生产环境(网络隔离)，所以基本不怎么使用了，但是还是要标记一下。
[官网](https://www.ej-technologies.com/products/jprofiler/overview.html)

## java自带工具

### jps
```shell
sudo -u admin /opt/taobao/java/bin/jps -mlvV
```

### jstack

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jstack 2815
```
native+java栈:

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jstack -m 2815
```
### jinfo
可看系统启动的参数
```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jinfo -flags 2815
```

### jmap

1.查看堆的情况

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jmap -heap 2815
```
2.dump

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jmap -dump:live,format=b,file=/tmp/heap2.bin 2815
```
或者

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jmap -dump:format=b,file=/tmp/heap3.bin 2815
```

3.看看堆都被谁占了? 再配合zprofiler和btrace

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jmap -histo 2815 | head -10
```

### jstat
jstat参数众多，但是使用一个就够了

```shell
sudo -u admin /opt/taobao/install/ajdk-8_1_1_fp1-b52/bin/jstat -gcutil 2815 1000 
```
### jdb

jdb可以用来预发debug,假设你预发的java_home是/opt/taobao/java/，远程调试端口是8000.那么
```shell
sudo -u admin /opt/taobao/java/bin/jdb -attach 8000.
```
出现以上代表jdb启动成功。后续可以进行设置断点进行调试。
具体参数可见[oracle官方说明](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/jdb.html)

### CHLSDB

CHLSDB感觉很多情况下可以看到更好玩的东西，不详细叙述了。 查询资料听说jstack和jmap等工具就是基于它的。

```sell
sudo -u admin /opt/taobao/java/bin/java -classpath /opt/taobao/java/lib/sa-jdi.jar sun.jvm.hotspot.CLHSDB
```
[详细](http://rednaxelafx.iteye.com/blog/1847971)


## VM options
1、你的类到底是从哪个文件加载进来的？

```
-XX:+TraceClassLoading
结果形如[Loaded java.lang.invoke.MethodHandleImpl$Lazy from D:\programme\jdk\jdk8U74\jre\lib\rt.jar]
```

2、应用挂了输出dump文件
```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof
```

## jar包冲突

```shell
mvn dependency:tree > ~/dependency.txt
打出所有依赖

mvn dependency:tree -Dverbose -Dincludes=groupId:artifactId
只打出指定groupId和artifactId的依赖关系

-XX:+TraceClassLoading
vm启动脚本加入。在tomcat启动脚本中可见加载类的详细信息

-verbose
vm启动脚本加入。在tomcat启动脚本中可见加载类的详细信息

greys:sc
greys的sc命令也能清晰的看到当前类是从哪里加载过来的

tomcat-classloader-locate
通过以下url可以获知当前类是从哪里加载的
curl http://localhost:8006/classloader/locate?class=org.apache.xerces.xs.XSObjec

```

## 其他

### dmesg

如果发现自己的java进程悄无声息的消失了，几乎没有留下任何线索，那么dmesg一发，很有可能有你想要的。

```shell
sudo dmesg|grep -i kill|less
```
去找关键字oom_killer。找到的结果类似如下:

```
[6710782.021013] java invoked oom-killer: gfp_mask=0xd0, order=0, oom_adj=0, oom_scoe_adj=0
[6710782.070639] [<ffffffff81118898>] ? oom_kill_process+0x68/0x140 
[6710782.257588] Task in /LXC011175068174 killed as a result of limit of /LXC011175068174 
[6710784.698347] Memory cgroup out of memory: Kill process 215701 (java) score 854 or sacrifice child 
[6710784.707978] Killed process 215701, UID 679, (java) total-vm:11017300kB, anon-rss:7152432kB, file-rss:1232kB
```

以上表明，对应的java进程被系统的OOM Killer给干掉了，得分为854.
解释一下OOM killer（Out-Of-Memory killer），该机制会监控机器的内存资源消耗。当机器内存耗尽前，该机制会扫描所有的进程（按照一定规则计算，内存占用，时间等），挑选出得分最高的进程，然后杀死，从而保护机器。

dmesg日志时间转换公式:
```
log实际时间=格林威治1970-01-01+(当前时间秒数-系统启动至今的秒数+dmesg打印的log时间)秒数：
```

```
date -d "1970-01-01 UTC `echo "$(date +%s)-$(cat /proc/uptime|cut -f 1 -d' ')+12288812.926194"|bc ` seconds"
```

剩下的，就是看看为什么内存这么大，触发了OOM-Killer了。

### Arthas

[GitHub地址](https://github.com/alibaba/arthas)
[用户文档](https://alibaba.github.io/arthas/)
[简单使用说明](https://mp.weixin.qq.com/s/5Yj6UckTabrQbgJ9TLV1gQ)

[toTop](#jump)

# 泛型

## 集合与泛型

![](/img/generic1.png)
![](/img/generic2.png)
![](/img/generic3.png)
![](/img/generic4.png)
![](/img/generic5.png)

补充：Array不支持泛型

参考:码出效率6.5节

[toTop](#jump)

# String StringBuffer StringBuilder
## String
1、String 类是一个final 修饰的类所以这个类是不能继承的，也就没有子类。

2、String 类的成员变量都是final类型的并且没有提供任何方法可以来修改引用变量所引用的对象的内容，所以一旦这个对象被创建并且成员变量初始化后这个对象就不能再改变了，所以说String 对象是一个不可变对象。

3、使用“+”连接字符串的过程产生了很多String 对象和StringBuffer 对象所以效率相比直接使用StringBuffer 对象的append（） 方法来连接字符效率低很多。

4、引用变量是存在java虚拟机栈内存中的，它里面存放的不是对象，而是对象的地址或句柄地址。

5、对象是存在java heap(堆内存)中的值

6、引用变量的值改变指的是栈内存中的这个引用变量的值的改变是，对象地址的改变或句柄地址的改变，而对象的改变指的是存放在Java heap（堆内存）中的对象内容的改变和引用变量的地址和句柄没有关系。

## StringBuffer
1、StringBuffer 类被final 修饰所以不能继承没有子类

2、StringBuffer 对象是可变对象，因为父类的 value [] char 没有被final修饰所以可以进行引用的改变，而且还提供了方法可以修改被引用对象的内容即修改了数组内容。

3、在使用StringBuffer对象的时候尽量指定大小这样会减少扩容的次数，也就是会减少创建字符数组对象的次数和数据复制的次数，当然效率也会提升。

4.线程安全，涉及到synchronized 

## StringBuilder
基本和StringBuffer相同，但是不是线程安全

[toTop](#jump)

# 反射
java.lang.reflect.Proxy：这是 Java 动态代理机制的主类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象

java.lang.reflect.InvocationHandler：这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。

* 通过 Class 类获取成员变量、成员方法、接口、超类、构造方法等

```java
//获得类完整的名字
String className = c2.getName();
System.out.println(className);//输出com.ys.reflex.Person
        
//获得类的public类型的属性。
Field[] fields = c2.getFields();
for(Field field : fields){
   System.out.println(field.getName());//age
}
        
//获得类的所有属性。包括私有的
Field [] allFields = c2.getDeclaredFields();
for(Field field : allFields){
    System.out.println(field.getName());//name    age
}
        
//获得类的public类型的方法。这里包括 Object 类的一些方法
Method [] methods = c2.getMethods();
for(Method method : methods){
    System.out.println(method.getName());//work waid equls toString hashCode等
}
        
//获得类的所有方法。
Method [] allMethods = c2.getDeclaredMethods();
for(Method method : allMethods){
    System.out.println(method.getName());//work say
}
        
//获得指定的属性
Field f1 = c2.getField("age");
System.out.println(f1);
//获得指定的私有属性
Field f2 = c2.getDeclaredField("name");
//启用和禁用访问安全检查的开关，值为 true，则表示反射的对象在使用时应该取消 java 语言的访问检查；反之不取消
f2.setAccessible(true);
System.out.println(f2);
                
//创建这个类的一个对象
Object p2 =  c2.newInstance();
//将 p2 对象的  f2 属性赋值为 Bob，f2 属性即为 私有属性 name
f2.set(p2,"Bob");
//使用反射机制可以打破封装性，导致了java对象的属性不安全。 
System.out.println(f2.get(p2)); //Bob
        
//获取构造方法
Constructor [] constructors = c2.getConstructors();
for(Constructor constructor : constructors){
    System.out.println(constructor.toString());//public com.ys.reflex.Person()
}
```

[toTop](#jump)

