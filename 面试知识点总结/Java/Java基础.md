#### Java和C++的的区别？

- 都是面向对象的语言，支持封装，继承和多态
- Java不提供指针直接访问内存
- Java类是单继承的，C++是多继承的
- Java有自己的内存管理机制，不需要手动释放
- C语言字符串和字符数组都会以'/0'表示结束

#### Java基本概念

##### JDK和JRE

JDK功能齐全的 Java SDK，拥有JRE拥有的一切，还有编译器（javac）和工具，能够创建和编译程序。

JRE是Java运行环境，他是运行已编译Java程序所需要的全部集合，包括JVM，类库，还有其他一些基础构件

##### 为什么Java是编译和解释共存

 Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（*.class 文件），这种字节码必须由 Java 解释器来解释执行。

#### Java基础

##### static关键字，final关键字

static关键字

static用于创建类的静态变量和静态方法。

static修饰变量，随着类的加载（第一次加载初始化）而存在，类消失而消失。而实例变量生命周期跟随对象创建和消失。

static修饰方法，不依赖对象就可以访问，不可以访问非静态方法和变量

static静态代码块，在类初次加载会执行一次

> static静态内部类，静态内部类的创建不依赖于外部类，静态内部类可以访问外部类的静态变量，而不可访问外部类的非静态变量；

final关键字

final修饰变量表示常量，只能被赋值一次。直接赋值或者在构造方法里初始化

##### 泛型

泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数，提供了编译时类型安全检测机制。

- 泛型类
- 泛型接口
- 泛型方法

![](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200825215050131.png)

![image-20200825215351547](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200825215351547.png)

**泛型擦除**

编译器在编译过程会移除泛型信息，会被擦除到他的第一个边界（extends添加边界），没有指明边界则会擦除到Object。

##### Object对象通用方法

###### equals

###### hashcode

> 如果两个对象相等，则 hashcode 一定也是相同的
>
> 两个对象有相同的 hashcode 值，它们也不一定是相等的
>
> **equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**

###### toString

默认返回类名+@+散列码十六进制表示

###### clone

Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

##### 深拷贝浅拷贝

1. **浅拷贝**：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
2. **深拷贝**：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

##### 装箱，拆箱

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

**Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False**

```java
  Integer i1 = 40;		//装箱Integer.valueOf  拆箱Integer的intValue方法
  Integer i2 = new Integer(40);
  System.out.println(i1==i2);//输出 false
```

```java
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);
  
  System.out.println("i1=i2   " + (i1 == i2));	//true
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3)); //true
  System.out.println("i1=i4   " + (i1 == i4)); //false
  System.out.println("i4=i5   " + (i4 == i5)); //false
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6)); // true  
  System.out.println("40=i5+i6   " + (40 == i5 + i6)); // true
//语句 i4 == i5 + i6，因为+这个操作符不适用于 Integer 对象，首先 i5 和 i6 进行自动拆箱操作，进行数值相加，即 i4 == 40。然后 Integer 对象无法与数值进行直接比较，所以 i4 自动拆箱转为 int 值 40，最终这条语句转为 40 == 40 进行数值比较。
```

##### 方法重载，重写

- 重载：同一个方法名，参数类型不同，个数不同，顺序不同，返回值类型不同，访问修饰符不同
- 重写：重写是子类对父类允许访问的方法实现过程的重新编写

#### 面向对象

##### 封装

封装分为属性的封装和方法的封装。将对象不想让外界访问的属性和方法私有化，提供公有方法访问这些属性和方法。

##### 继承

继承是使用已存在的类作为基础创建新类的技术，新类可以新增数据和功能，也可以使用父类的功能，提高代码的复用性。

##### 多态

同一消息可以根据发送的对象的不同而采用多种不同的行为，依靠动态绑定，在执行期间根据对象的类型，调用相应的方法。

消除类型之间的耦合关系

##### 抽象类和接口

抽象类和抽象方法使用abstract声明，抽象类不能实例化，必须继承使用

接口可以看成是一个完全抽象的类，1.8之前的接口不允许有方法的实现，接口的成员默认都是public修饰的，不能是private和protected，都是static和final

#### String，StringBuffer，StringBuilder

String内部使用final修饰字符数组，不可变

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

##### String Pool

字符串常量池保存所有字符串字面量，这些字面量在编译时期已经确定

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。

##### intern方法

当一个字符串调用 **intern()** 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

在JDK 1.6中，intern()会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，**而JDK 1.7的intern()实现不会复制实例，只是在常量池中记录首次出现的实例引用**

##### new String("abc")

- "abc"字面量，在编译期会在String Pool中创建一个字符串对象，指向这个字面量
- new 会在堆中创建一个String对象

##### 场景问题

```java
String a = "hello2"; 
String b = "hello" + 2;
System.out.println((a == b)); //true	"hello"+2 在编译期间就已经被优化成"hello2"

String a = "hello2"; 
String b = "hello";    
String c = b + 2;      
System.out.println((a == c)); //false   编译期无法确定，优化为StringBuilder.append，然后返回String对象

String a = "hello2";   　 
final String b = "hello";       
String c = b + 2;       
System.out.println((a == c)); //true

public static void main(String[] args) {
        String a = "hello2";
        final String b = getHello();
        String c = b + 2;
        System.out.println((a == c)); //false
    }
     
    public static String getHello() {
        return "hello";
    }
```

```java
String a = "hello";
String b =  new String("hello");
String c =  new String("hello");
String d = b.intern();  
System.out.println(a==b);		//false
System.out.println(b==c);		//false
System.out.println(b==d);		//false
System.out.println(a==d);		//true


     String s1 = new String("12") + new String("34");	//
     String s2 = s1.intern();
     String s3 = "1234";				//jdk 1.7		//jdk 1.6
     System.out.println(s1 == s2);		//true			//false
     System.out.println(s1 == s3);		//true			//false
     System.out.println(s2 == s3);		//true			//true
```

#### Java异常

![image-20200820120803908](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200820120803908.png)

异常也是一种对象，java当中定义了许多异常类，并且定义了基类java.lang.Throwable作为所有异常的超类。

Java语言设计者将异常划分为两类：Error和Exception。

1、Error（错误）：是程序中无法处理的错误，表示运行应用程序中出现了严重的错误。此类错误一般表示代码运行时JVM出现问题。通常有Virtual MachineError（虚拟机运行错误）、NoClassDefFoundError（类定义错误）等。**StackOverflowError**，**OutOfMemoryError**。

2、Exception（异常）：**程序本身可以捕获并且可以处理的异常。**

Exception这种异常又分为两类：运行时异常和编译异常。

- 运行时异常(**免检异常**)：RuntimeException类极其子类表示JVM在运行期间可能出现的错误。比如说试图使用空值对象的引用（**NullPointerException**）、数组下标越界（**ArrayIndexOutBoundException**）。此类异常属于不可查异常，一般是由**程序逻辑错误引起的，在程序中可以选择捕获处理，也可以不处理**。ArithmeticException，ClassCastException，NumberFormatException，IllegalArgumentException。

- 编译异常(**受检异常**)：Exception中除RuntimeException及其子类之外的异常。如果程序中出现此类异常，比如说**IOException**，必须对该异常进行处理，否则编译不通过。**在程序中，通常不会自定义该类异常，而是直接使用系统提供的异常类**。ClassNotFoundException，InterruptedException。