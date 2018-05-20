# 基础
## 8个基本类型
byte boolean 1字节  
short char 2字节  
int float 4字节  
double long 8字节  

## Object,String类的方法
Object:getClass(),hashCode(),equals(),toString(),wait(),  notify(),notifyAll()  
String:equals(),replace(),split(),subString()  

## static
可用于修饰成员变量和成员函数，静态随类的加载而加载，可以直接用类进行访问。静态方法不能访问非静态成员变量和非静态成员方法，因为非静态成员方法/变量必须依赖具体的对象才能被调用。Java中Static方法不能被覆盖。

## 异常
Throwable是所有异常的父类，包含Error和Exception。  
Error：OutofMemory，StackOverflow。  
Exception包括受检异常和非受检异常(RuntimeException)  
受检异常包括：SQLException,IOException，JSONException  
非受检异常包括：NullPointException,IndexOutOfBoundsException,FileNotFoundException,SecurityException,IllegalArgumentException,NumberFormatException

## HTTP GET和POST区别
GET：是请求资源。安全，幂等（同一个请求多次和一次效果完全相同），可缓存，参数长度有限制（受限于url长度）  
POST：根据报文主题对指定资源做出处理，不安全，不幂等，不可缓存

## Java中private、protected、public和default的区别
![](/img/javaclass_type.png)


