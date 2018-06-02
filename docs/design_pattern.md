<a id = "jump">[首页](/README.md)</a>
<!-- TOC -->

- [单例模式](#单例模式)
    - [饿汉式单例](#饿汉式单例)
    - [懒汉式单例](#懒汉式单例)
    - [静态内部类](#静态内部类)
    - [双重检测同步延迟加载](#双重检测同步延迟加载)
        - [使用ThreadLocal修复双重检测](#使用threadlocal修复双重检测)
- [适配器模式](#适配器模式)
    - [类适配器模式](#类适配器模式)
    - [对象适配器模式](#对象适配器模式)
    - [接口适配器模式](#接口适配器模式)
    - [配器模式应用场景](#配器模式应用场景)
- [建造者模式（Builder）](#建造者模式builder)
- [装饰者模式](#装饰者模式)
- [工厂模式](#工厂模式)

<!-- /TOC -->

# 单例模式
## 饿汉式单例
饿汉式在类创建的同时就已经创建好一个静态的对象供系统使用，以后不再改变，所以天生是线程安全的

```java
//饿汉式单例类.在类初始化时，已经自行实例化   
public class Singleton1 {  
    private Singleton1() {}  
    private static final Singleton1 single = new Singleton1();  
    //静态工厂方法   
    public static Singleton1 getInstance() {  
        return single;  
    }  
}  
```

## 懒汉式单例

```java
//懒汉式单例类.在第一次调用的时候实例化自己   

public class Singleton {  
    private Singleton() {}  
    private static Singleton single=null;  
    //静态工厂方法   
    public static Singleton getInstance() {  
         if (single == null) {    
             single = new Singleton();  
         }    
        return single;  
    }  
}  
```

## 静态内部类
既实现了线程安全，又避免了同步带来的性能影响

```java
public class Singleton {    
    private static class LazyHolder {    
       private static final Singleton INSTANCE = new Singleton();    
    }    
    private Singleton (){}    
    public static final Singleton getInstance() {    
       return LazyHolder.INSTANCE;    
    }    
}    
```

## 双重检测同步延迟加载
为处理原版非延迟加载方式瓶颈问题，我们需要对 instance 进行第二次检查，目的是避开过多的同步（因为这里的同步只需在第一次创建实例时才同步，一旦创建成功，以后获取实例时就不需要同获取锁了），但在Java中行不通，因为同步块外面的if (instance == null)可能看到已存在，但不完整的实例。JDK5.0以后版本若instance为volatile则可行

```java
public class Singleton {  
 private volatile static Singleton instance = null;  
 private Singleton() {}  
 public static Singleton getInstance() {  
  if (instance == null) {  
   synchronized (Singleton.class) {// 1  
    if (instance == null) {// 2  
     instance = new Singleton();// 3  
    }  
   }  
  }  
  return instance;  
 }  
}  
```

双重检测锁定失败的问题并不归咎于 JVM 中的实现 bug，而是归咎于 Java 平台内存模型。内存模型允许所谓的“无序写入”，这也是失败的一个主要原因。
* 无序写入
为解释该问题，需要重新考察上述清单中的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 instance 来引用此对象。这行代码的问题是：在 Singleton 构造函数体执行之前，变量 instance 可能成为非 null 的，即赋值语句在对象实例化之前调用，此时别的线程得到的是一个还会初始化的对象，这样会导致系统崩溃。
什么？这一说法可能让您始料未及，但事实确实如此。在解释这个现象如何发生前，请先暂时接受这一事实，我们先来考察一下双重检查锁定是如何被破坏的。假设代码执行以下事件序列：

1) 线程 1 进入 getInstance() 方法。
2) 由于 instance 为 null，线程 1 在 //1 处进入 synchronized 块。 
3) 线程 1 前进到 //3 处，但在构造函数执行之前，使实例成为非 null。 
4) 线程 1 被线程 2 预占。
5) 线程 2 检查实例是否为 null。因为实例不为 null，线程 2 将 instance 引用返回给一个构造完整但部分初始化了的 Singleton 对象。 
6) 线程 2 被线程 1 预占。
7) 线程 1 通过运行 Singleton 对象的构造函数并将引用返回给它，来完成对该对象的初始化。

为展示此事件的发生情况，假设代码行 instance =new Singleton(); 执行了下列伪代码：

```java
mem = allocate();             //为单例对象分配内存空间.
instance = mem;               //注意，instance 引用现在是非空，但还未初始化
ctorSingleton(instance);    //为单例对象通过instance调用构造函数
```

这段伪代码不仅是可能的，而且是一些 JIT 编译器上真实发生的。执行的顺序是颠倒的，但鉴于当前的内存模型，这也是允许发生的。JIT 编译器的这一行为使双重检查锁定的问题只不过是一次学术实践而已。
确实,在JAVA2(以jdk1.2开始)以前对于实例字段是直接在主储区读写的.所以当一个线程对resource进行分配空间,
初始化和调用构造方法时,可能在其它线程中分配空间动作可见了,而初始化和调用构造方法还没有完成.
但是从JAVA2以后,JMM发生了根本的改变,分配空间,初始化,调用构造方法只会在线程的工作存储区完成,在没有
向主存储区复制赋值时,其它线程绝对不可能见到这个过程.而这个字段复制到主存区的过程,更不会有分配空间后
没有初始化或没有调用构造方法的可能.在JAVA中,一切都是按引用的值复制的.向主存储区同步其实就是把线程工作
存储区的这个已经构造好的对象有压缩堆地址值COPY给主存储区的那个变量.这个过程对于其它线程,要么是resource
为null,要么是完整的对象.绝对不会把一个已经分配空间却没有构造好的对象让其它线程可见.

参考1 : [用happen-before规则重新审视DCL](http://www.iteye.com/topic/260515)

参考2 : [关于double-check锁失效](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)

### 使用ThreadLocal修复双重检测
借助于ThreadLocal，将临界资源（需要同步的资源）线程局部化，具体到本例就是将双重检测的第一层检测条件 if (instance == null) 转换为了线程局部范围内来作。这里的ThreadLocal也只是用作标示而已，用来标示每个线程是否已访问过，如果访问过，则不再需要走同步块，这样就提高了一定的效率。但是ThreadLocal在1.4以前的版本都较慢，但这与volatile相比却是安全的。

```java
public class Singleton {  
 private static final ThreadLocal perThreadInstance = new ThreadLocal();  
 private static Singleton singleton ;  
 private Singleton() {}  
   
 public static Singleton  getInstance() {  
  if (perThreadInstance.get() == null){  
   // 每个线程第一次都会调用  
   createInstance();  
  }  
  return singleton;  
 }  
  
 private static  final void createInstance() {  
  synchronized (Singleton.class) {  
   if (singleton == null){  
    singleton = new Singleton();  
   }  
  }  
  perThreadInstance.set(perThreadInstance);  
 }  
}  
```    

[toTop](#jump)

# 适配器模式
适配器就是一种适配中间件，它存在于不匹配的二者之间，用于连接二者，将不匹配变得匹配，简单点理解就是平常所见的转接头，转换器之类的存在。
适配器模式有两种：类适配器、对象适配器、接口适配器
## 类适配器模式
当我们要访问的接口A中没有我们想要的方法 ，却在另一个接口B中发现了合适的方法，我们又不能改变访问接口A，在这种情况下，我们可以定义一个适配器p来进行中转，这个适配器p要实现我们访问的接口A，这样我们就能继续访问当前接口A中的方法（虽然它目前不是我们的菜），然后再继承接口B的实现类BB，这样我们可以在适配器P中访问接口B的方法了，这时我们在适配器P中的接口A方法中直接引用BB中的合适方法，这样就完成了一个简单的类适配器。
详见下方实例：我们以ps2与usb的转接为例
* ps2接口：Ps2

```java
public interface Ps2 {
     void isPs2();
}
```

* USB接口：Usb

```java
public interface Usb {
     void isUsb();
}
```

* USB接口实现类：Usber

```java
public class Usber implements Usb {

    @Override
    public void isUsb() {
        System.out.println("USB口");
    }

}
```

* 适配器：Adapter

```java
public class Adapter extends Usber implements Ps2 {

    @Override
    public void isPs2() {
        isUsb();
    }

}
```

* 测试方法：Clienter

```java
public class Clienter {

    public static void main(String[] args) {
        Ps2 p = new Adapter();
        p.isPs2();
    }

}
```

## 对象适配器模式
当我们要访问的接口A中没有我们想要的方法 ，却在另一个接口B中发现了合适的方法，我们又不能改变访问接口A，在这种情况下，我们可以定义一个适配器p来进行中转，这个适配器p要实现我们访问的接口A，这样我们就能继续访问当前接口A中的方法（虽然它目前不是我们的菜），然后在适配器P中定义私有变量C（对象）（B接口指向变量名），再定义一个带参数的构造器用来为对象C赋值，再在A接口的方法实现中使用对象C调用其来源于B接口的方法。
详见下方实例：我们仍然以ps2与usb的转接为例
* ps2接口：Ps2

```java
public interface Ps2 {
     void isPs2();
}
```

* USB接口：Usb
```java
public interface Usb {
     void isUsb();
}
```

* USB接口实现类：Usber

```java
public class Usber implements Usb {

    @Override
    public void isUsb() {
        System.out.println("USB口");
    }

}
```

* 适配器：Adapter

```java
public class Adapter implements Ps2 {
    
    private Usb usb;
    public Adapter(Usb usb){
        this.usb = usb;
    }
    @Override
    public void isPs2() {
        usb.isUsb();
    }

}
```

测试类：Clienter

```java
public class Clienter {

    public static void main(String[] args) {
        Ps2 p = new Adapter(new Usber());
        p.isPs2();
    }

}
```

## 接口适配器模式
当存在这样一个接口，其中定义了N多的方法，而我们现在却只想使用其中的一个到几个方法，如果我们直接实现接口，那么我们要对所有的方法进行实现，哪怕我们仅仅是对不需要的方法进行置空（只写一对大括号，不做具体方法实现）也会导致这个类变得臃肿，调用也不方便，这时我们可以使用一个抽象类作为中间件，即适配器，用这个抽象类实现接口，而在抽象类中所有的方法都进行置空，那么我们在创建抽象类的继承类，而且重写我们需要使用的那几个方法即可。
* 目标接口：A

```java
public interface A {
    void a();
    void b();
    void c();
    void d();
    void e();
    void f();
}
```

* 适配器：Adapter

```java
public abstract class Adapter implements A {
    public void a(){}
    public void b(){}
    public void c(){}
    public void d(){}
    public void e(){}
    public void f(){}
}
```

* 实现类：Ashili

```java
public class Ashili extends Adapter {
    public void a(){
        System.out.println("实现A方法被调用");
    }
    public void d(){
        System.out.println("实现d方法被调用");
    }
}
```

* 测试类：Clienter

```java
public class Clienter {

    public static void main(String[] args) {
        A a = new Ashili();
        a.a();
        a.d();
    }

}
```

## 配器模式应用场景
类适配器与对象适配器的使用场景一致，仅仅是实现手段稍有区别，二者主要用于如下场景：
1) 想要使用一个已经存在的类，但是它却不符合现有的接口规范，导致无法直接去访问，这时创建一个适配器就能间接去访问这个类中的方法。
2) 我们有一个类，想将其设计为可重用的类（可被多处访问），我们可以创建适配器来将这个类来适配其他没有提供合适接口的类。
以上两个场景其实就是从两个角度来描述一类问题，那就是要访问的方法不在合适的接口里，一个从接口出发（被访问），一个从访问出发（主动访问）。
接口适配器使用场景：
    1) 想要使用接口中的某个或某些方法，但是接口中有太多方法，我们要使用时必须实现接口并实现其中的所有方法，可以使用抽象类来实现接口，并不对方法进行实现（仅置空），然后我们再继承这个抽象类来通过重写想用的方法的方式来实现。这个抽象类就是适配器。 
      
[toTop](#jump)

# 建造者模式（Builder）
在了解之前，先假设有一个问题，我们需要创建一个学生对象，属性有name,number,class,sex,age,school等属性，如果每一个属性都可以为空，也就是说我们可以只用一个name,也可以用一个school,name,或者一个class,number，或者其他任意的赋值来创建一个学生对象，这时该怎么构造？

```java
public class Builder {

    static class Student{
        String name = null ;
        int number = -1 ;
        String sex = null ;
        int age = -1 ;
        String school = null ;

　　　　　//构建器，利用构建器作为参数来构建Student对象
        static class StudentBuilder{
            String name = null ;
            int number = -1 ;
            String sex = null ;
            int age = -1 ;
            String school = null ;
            public StudentBuilder setName(String name) {
                this.name = name;
                return  this ;
            }

            public StudentBuilder setNumber(int number) {
                this.number = number;
                return  this ;
            }

            public StudentBuilder setSex(String sex) {
                this.sex = sex;
                return  this ;
            }

            public StudentBuilder setAge(int age) {
                this.age = age;
                return  this ;
            }

            public StudentBuilder setSchool(String school) {
                this.school = school;
                return  this ;
            }
            public Student build() {
                return new Student(this);
            }
        }

        public Student(StudentBuilder builder){
            this.age = builder.age;
            this.name = builder.name;
            this.number = builder.number;
            this.school = builder.school ;
            this.sex = builder.sex ;
        }
    }

    public static void main( String[] args ){
        Student a = new Student.StudentBuilder().setAge(13).setName("LiHua").build();
        Student b = new Student.StudentBuilder().setSchool("sc").setSex("Male").setName("ZhangSan").build();
    }
}
```   

[toTop](#jump)

# 装饰者模式
装饰者模式可以动态地给一个对象添加一些额外的职责。
该模式的适用环境为：
1) 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
2) 处理那些可以撤消的职责。
3) 当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类。
* 简单实现
场景：在星巴克的咖啡销售系统中，提供咖啡和调料的组合，并且在用户选好了咖啡和调料之后自动计算价格

某客户点了 浓缩咖啡加摩卡加牛奶的组合

首先是饮料类的抽象类

```java
//饮料的抽象类  
public abstract class Beverage {  
    String description = "Unknown Beverage";  
    //描述   
    public String getdescribtion() {  
        return description;  
    }  
    //花 费  
    public abstract double cost();  
}  
```  

调料类的抽象类

```java
//调料类，也是装饰者类  
public abstract class CondimentDecorrator {  
    //这里要重写getDescribtion的原因  
    //所谓装饰者类 其实就是包含被装饰者的类  
    //那么被装饰者和装饰者中的共有的还没定义的方法在这里定义  
    public abstract String getDescribtion();  
}  
```

浓缩咖啡类

```java
//浓缩咖啡的饮料类的实现  
public class Espresso extends Beverage {  
    public Espresso() {  
        description = "这是一杯浓缩咖啡";  
    }  
    @Override  
    public double cost() {  
        return 1;  
    }  
}  
```

摩卡类

```java
public class Moka extends CondimentDecorrator {  
    Beverage beverage;  
  
    public Moka(Beverage beverage) {  
        this.beverage = beverage;  
    }  
  
    @Override  
    public String getDescribtion() {  
        return beverage.getdescribtion() + "加了摩卡,";  
    }  
  
    public double cost() {  
        return beverage.cost() + 2;  
    }  
}  
```

牛奶类

```java
public class Milk extends CondimentDecorrator {  
    Beverage beverage;  
    public Milk(Beverage beverage) {  
        this.beverage = beverage;  
    }  
    @Override  
    public String getDescribtion() {  
        return beverage.getdescribtion() + "加了牛奶,";  
    }  
    public double cost() {  
        return beverage.cost() + 3;  
    }  
}  
```

测试类

```java
public class Test3 {  
  
    public static void main(String[] args) {  
        // 点一杯浓缩咖啡  
        Beverage espresso = new Espresso();  
        // 加入摩卡  
        espresso = new Moka(espresso);  
        // 加入牛奶  
        espresso = new Milk(espresso);  
        System.out.println(espresso.getdescribtion() + "花费了" + espresso.cost());  
    }  
}  
```

[toTop](#jump)

# 工厂模式
