---
title: jvm(6)--string
date: 2020-7-17 23:37:46
categories: 
- java基础
tag:
- java
- jvm
---

## String的基本特性

<!--more-->

- String：字符串，使用一对 ”” 引起来表示
  - String s1 = “mogublog” ; // 字面量的定义方式
  - String s2 = new String(“moxi”);
- string声明为**final**的，不可被继承
- String实现了Serializable接口：表示字符串是支持序列化的。实现了Comparable接口：表示string可以比较大小
- **string在jdk8及以前内部定义了final char[] value用于存储字符串数据。JDK9时改为byte[]**

### 为什么JDK9改变了结构

String类的当前实现将字符存储在char数组中，每个字符使用两个字节(16位)。从许多不同的应用程序收集的数据表明，**字符串是堆使用的主要组成部分**，而且，**大多数字符串对象只包含拉丁字符。这些字符只需要一个字节的存储空间，**因此这些字符串对象的内部char数组中有一半的空间将不会使用。
String再也不用char[] 来存储了，改成了byte [] 加上**编码标记**，节约了一些空间

同时基于String的数据结构，例如StringBuffer和StringBuilder也同样做了修改

### String的不可变性

当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
当调用string的replace（）方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。

```java
/**
 * String的不可变性
 */
public class StringTest1 {

    public static void test1() {
        // 字面量定义的方式，“abc”存储在字符串常量池中
        String s1 = "abc";
        String s2 = "abc";
        System.out.println(s1 == s2);//true
        s1 = "hello";
        System.out.println(s1 == s2);//false
        System.out.println(s1);//hello
        System.out.println(s2);//abc
    }

    public static void test2() {
        String s1 = "abc";
        String s2 = "abc";
        // 只要进行了修改，就会重新创建一个对象，这就是不可变性
        s2 += "def";
        System.out.println(s1);//abc
        System.out.println(s2);//abcdef
    }

    public static void test3() {
        String s1 = "abc";
        String s2 = s1.replace('a', 'm');
        System.out.println(s1);//abc
        System.out.println(s2);//mbc
    }
}

```

```java

public class StringExer {
    String str = new String("good");
    char [] ch = {'t','e','s','t'};

    public void change(String str, char ch []) {
        str = "test ok";
        ch[0] = 'b';
    }

    public static void main(String[] args) {
        StringExer ex = new StringExer();
        ex.change(ex.str, ex.ch);
        System.out.println(ex.str);
        System.out.println(ex.ch);
        //good
        //best
    }
}

```

**字符串常量池是不会存储相同内容的字符串的**

String的string Pool是一个固定大小的Hashtable，默认值大小长度是1009。如果放进string Pool的**string非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用string.intern时性能会大幅下降。**

使用-XX:StringTablesize可设置stringTab1e的长度

在jdk6中stringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。stringTablesize设置没有要求

在jdk7中，stringTable的长度默认值是60013，

在JDK8中，StringTable可以设置的最小值为1009


**data.intern()  如果字符串常量池中没有对应data的字符串，则在常量池中生成**

## String的内存分配

在Java语言中有8种基本数据类型和一种比较特殊的类型string。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念。

**常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，string类型的常量池比较特殊。它的主要使用方法有两种。**

直接使用双引号声明出来的String对象会直接存储在常量池中。

- 比如：string info=“atguigu.com”；
- 如果不是用双引号声明的string对象，可以使用string提供的intern（）方法。

Java 6及以前，字符串常量池存放在永久代

Java 7中 oracle的工程师对字符串池的逻辑做了很大的改变，即将**字符串常量池的位置调整到Java堆内**

Java8元空间，字符串常量在堆

### 为什么StringTable从永久代调整到堆中

- 永久代的默认比较小
- 永久代垃圾回收频率低

## String的基本操作

Java语言规范里要求完全相同的字符串字面量，应该包含同样的Unicode字符序列（包含同一份码点序列的常量），并且必须是指向同一个String类实例。

## 字符串拼接操作

- 常量与常量的拼接结果在常量池，原理是**编译期优化**
- 常量池中**不会存在相同内容的变量**
- **只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder**
- 如果拼接的结果调用**intern()**方法，则**主动将常量池中还没有的字符串对象放入池中**，并返回此对象地址

```java
    public static void test1() {
        String s1 = "a" + "b" + "c";  // 得到 abc的常量池,编译期优化
        String s2 = "abc"; // abc存放在常量池，直接将常量池的地址返回
        /**
         * 最终java编译成.class，再执行.class
         */
        System.out.println(s1 == s2); // true，因为存放在字符串常量池
        System.out.println(s1.equals(s2)); // true
    }

	public static void test2() {
        String s1 = "javaEE";
        String s2 = "hadoop";
        String s3 = "javaEEhadoop";
        String s4 = "javaEE" + "hadoop";    
        String s5 = s1 + "hadoop";
        String s6 = "javaEE" + s2;
        String s7 = s1 + s2;
		
        System.out.println(s3 == s4); // true
        System.out.println(s3 == s5); // false
        System.out.println(s3 == s6); // false
        System.out.println(s3 == s7); // false
        System.out.println(s5 == s6); // false
        System.out.println(s5 == s7); // false
        System.out.println(s6 == s7); // false
		//只要有变量，就在堆空间中new一个String
        String s8 = s6.intern();
        //判断字符串常量池中是否存在，存在返回地址，不存在就创建
        System.out.println(s3 == s8); // true
    }

```

### 底层原理

拼接操作的底层其实使用了StringBuilder

s1+s2执行细节

- new StringBuilder
- s\.append("a")
- s.append("b")
- s.toString()  **类似于new String**

在JDK5之后，使用的是StringBuilder，在JDK5之前使用的是StringBuffer

注意，我们**左右两边如果是变量的话，就是需要new StringBuilder进行拼接**

但是**如果使用的是final修饰，则是从常量池中获取。**所以说拼接符号左右两边都是字**符串常量或常量引用 则仍然使用编译器优化**，即费StringBuilder的方式。也就是说被final修饰的变量，将会变成常量，类和方法将不能被继承、

- 在开发中，能够使用final的时候，建议使用上

```java
public static void test4() {
    final String s1 = "a";//被final修饰的变量拜年成了常量
    final String s2 = "b";
    String s3 = "ab";
    String s4 = s1 + s2;//从常量池中取出常量s1和s2，所以他不是变量，也就不会采用new StringBuilder方式拼接
    System.out.println(s3 == s4); //true
}

```

### 拼接操作和append性能对比

```java
public static void method1(int highLevel) {
    String src = "";
    for (int i = 0; i < highLevel; i++) {
        src += "a"; // 每次循环都会创建一个StringBuilder对象 String对象
    }
}

public static void method2(int highLevel) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < highLevel; i++) {
        sb.append("a");
    }
}
```
方法1耗费的时间：4005ms，方法2消耗时间：7m

结论：

- 通过StringBuilder的append()方式添加字符串的效率，要远远高于String的字符串拼接方法

好处

- StringBuilder的append的方式，自始至终**只创建一个StringBuilder的对象**

  对于字符串拼接的方式，还需要创建很多StringBuilder对象和 调用toString时候创建的String对象

- 内存中由于创建了较多的StringBuilder和String对象，内存占用过大，**如果进行GC那么将会耗费更多的时间**

改进的空间

- 我们使用的是StringBuilder的空参构造器，默认的字符串容量是16，然后将原来的字符串拷贝到新的字符串中， 我们也可以默认初始化更大的长度，减少扩容的次数
- **因此在实际开发中，我们能够确定，前前后后需要添加的字符串不高于某个限定值，那么建议使用构造器创建一个阈值的长度**

|                          **String**                          |                       **StringBuffer**                       | **StringBuilder** |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :---------------: |
| String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，不仅效率低下，而且浪费大量优先的内存空间 | StringBuffer是可变类，和线程安全的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。每个StringBuffer对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量 | 可变类，速度更快  |
|                            不可变                            |                             可变                             |       可变        |
|                                                              |                           线程安全                           |    线程不安全     |
|                                                              |                       多线程操作字符串                       | 单线程操作字符串  |

## intern()的使用

intern是一个native方法，调用的是底层C的方法

字符串池最初是空的，由String类私有地维护。在调用intern方法时，如果池中已经包含了由equals(object)方法确定的与该字符串对象相等的字符串，则返回池中的字符串。否则，该字符串对象将被添加到池中，并返回对该字符串对象的引用。

如果不是用双引号声明的string对象，可以使用string提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

```java
String myInfo = new string("I love atguigu").intern();
```

如果在任意字符串上调用string.intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true

```java
（"a"+"b"+"c"）.intern（）=="abc"
```

```
//如何保证s指向字符串常量池
方式一 :	String s = " afsadf"
方式二 :	String s = new String().intern;
		  String s = new StringBuilder("gfgs").toString().intern();
```

## 面试题

### new String(“ab”)会创建几个对象

```
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}

```

这里面就是两个对象

- 一个对象是：new关键字在堆空间中创建
- 另一个对象：字符串常量池中的对象

证明字节码

```text
 0 new #2 <java/lang/String>
 3 dup
 4 ldc #3 <ab>
 6 invokespecial #4 <java/lang/String.<init>>
 9 astore_1
10 return

```

### new String(“a”) + new String(“b”) 会创建几个对象

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}

```

我们创建了6个对象

- 对象1：new StringBuilder()
- 对象2：new String(“a”)
- 对象3：常量池的 a
- 对象4：new String(“b”)
- 对象5：常量池的 b
- 对象6：toString中会创建一个 new String(“ab”)
  **调用toString方法，不会在常量池中生成ab**


```text
 0 new #2 <java/lang/StringBuilder>
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>>
 7 new #4 <java/lang/String>
10 dup
11 ldc #5 <a>
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>
22 dup
23 ldc #8 <b>
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>
34 astore_1
35 return

```

### intern的使用：JDK6和JDK7

#### JDK6中

```java
String s = new String("1");  // 在常量池中已经有了 造了两个对象，一个是因为new在堆空间 另一个在字符串常量池1
s.intern(); // 将该对象放入到常量池。但是调用此方法没有太多的区别，因为已经存在了1
			//不会改变s的引用
String s2 = "1";//这个就是第一行new的时候再堆中的地址，而s是在常量池中的地址，显然不是同一个地址
System.out.println(s == s2); // false//如果第二行中改为s=s.inter();这里就是true了

String s3 = new String("1") + new String("1");//表示new String("11")在堆区开辟了一个内存空间，并把其地址赋给了s3
//执行完上一行代码以后，字符串常量池中，是否存在"11"呢？不存在！
s3.intern();//在字符串常量池中生成"11" jdk6中：创建了一个新的对象"11"，也就有了新的地址
String s4 = "11";//在上一行执行时，在常量池中生成的"11"的地址赋给了s4
System.out.println(s3 == s4); // false s3是堆区内存空间的地址，s4是常量池中"11"的地址，所以为false

```

false
false

为什么对象会不一样呢？

- 一个是new创建的对象，一个是常量池中的对象，显然不是同一个

  

  s=s.intern() 就是true

  

#### JDK7/8中

```java
String s = new String("1");
s.intern();
String s2 = "1";
System.out.println(s == s2); // false

String s3 = new String("1") + new String("1");
s3.intern();//和jdk6不一样的就在这里，jdk7/8中，并不会创建一个新的对象"11"而是指向上一行中new String("11")的地址
String s4 = "11";//s4指向的是上一行中常量池中的对象"11" 但是他也是指向堆空间new String("11")的地址，所以两者一致
System.out.println(s3 == s4); // true
```

false
true

**s3.intern();//和jdk6不一样的就在这里，jdk7/8中，并不会创建一个新的对象"11"而是指向上一行中new String("11")的地址**

在JDK7中，在JDK7中，并没有创新一个新对象，而是指向常量池中的新对象



### 扩展

```java
String s3 = new String("1") + new String("1");//表示new String("11")在堆区开辟了一个内存空间，并把其地址赋给了s3
String s4 = "11";  // 在常量池中生成的字符串
String s5 = s3.intern();  // 然后s3就会从常量池中找，发现有了，就什么事情都不做

System.out.println(s3 == s4); //false
System.out.println(s5 == s4); //false
```

总结
总结string的intern（）的使用：

JDK1.6中，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，会把此**对象复制一份**，放入串池，并返回串池中的对象地址

JDK1.7起，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，则会把**对象的引用地址复制一份**，放入串池，并返回串池中的引用地址
  

### 练习1

JDK6

```java
  String s = new String("a")+new String("b");//表示new String("ab")在堆区开辟了一个内存空间，并把其地址赋给了s
//在上一行代码执行之后，字符串常量池中并没有"ab"
  String s2 = s.intern();//在jdk6中：在字符串常量池中创建一个字符串"ab"，并把它的地址赋值给s2
  System.out.println(s2 == "ab"); // true "ab"就是常量池中的地址，所以与s2一致
  System.out.println(s == "ab"); // false s记录的是new的地址 在堆区的地址
```

JDK7/8

```java
  String s = new String("a")+new String("b");//表示new String("ab")在堆区开辟了一个内存空间，并把其地址赋给了s
//在上一行代码执行之后，字符串常量池中并没有"ab"
  String s2 = s.intern();//在jdk8中：在字符串常量池中没有创建一个字符串"ab"，而是创建一个引用，指向new String("ab") 并返回给s2 
  System.out.println(s2 == "ab"); // true 常量池中没有"ab"，所以"ab"就是堆空间的地址，所以与s2一致
  System.out.println(s == "ab"); // true s记录的是new的地址 在堆区的地址

```

### 练习2

```java
  String s1 = new String("a")+new String("b");//表示new String("ab")在堆区开辟了一个内存空间，并把其地址赋给了s1
//在上一行代码执行之后，字符串常量池中并没有"ab"
  s1.intern();
//在jdk8中：在字符串常量池中没有创建一个字符串"ab"，而是创建一个引用，指向new String("ab") 并返回给s2 
//在jdk6中：会在字符串常量池中创建一个字符串"ab"
  String s2="ab";//jdk8：因为常量池中没有"ab"所以把堆区的地址给了s2 jdk6：把常量池中的"ab"的地址给了s2
  System.out.println(s1 == s2); // jdk8:true jdk6:false

```



```java
 String s1 = new String("ab");//表示new String("ab")在堆区开辟了一个内存空间，并把其地址赋给了s1，并且执行完毕之后会在字符串常量池中生成"ab"
//在上一行代码执行之后，字符串常量池中已经有了"ab"
  s1.intern();//因为已经有了"ab"，所以 这步什么也没有做
  String s2="ab";//无论是jdk几，因为常量池中都有"ab"所以把常量池中的"ab"的地址给了s2
  System.out.println(s1 == s2); // jdk8:false jdk6:false
```

### intern的空间效率测试

```java
public class StringIntern2 {
    static final int MAX_COUNT = 1000 * 10000;
    static final String[] arr = new String[MAX_COUNT];

    public static void main(String[] args) {
        Integer [] data = new Integer[]{1,2,3,4,5,6,7,8,9,10};
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            arr[i] = new String(String.valueOf(data[i%data.length]));
            //arr[i] = new String(String.valueOf(data[i%data.length])).intern();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为：" + (end - start));

        try {
            Thread.sleep(1000000);
        } catch (Exception e) {
            e.getStackTrace();
        }
    }
}
```

**结论**：对于程序中大量使用存在的字符串时，尤其存在很多已经重复的字符串时，使用intern()方法能够节省内存空间。

## StringTable的垃圾回收

```java
public class StringGCTest {
    public static void main(String[] args) {
        for (int i = 0; i < 100000; i++) {
            String.valueOf(i).intern();
        }
    }
}

```

## G1中的String去重操作

注意这里说的重复，指的是在堆中的数据，而不是常量池中的，因为常量池中的本身就不会重复

**实现**

- 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的string对象。

- 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的string对象。

- 使用一个hashtab1e来记录所有的被string对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。

- 如果存在，string对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。

- 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组了。

  

  UsestringDeduplication（bool）：开启string去重，默认是不开启的，需要手动开启。
  Printstringbeduplicationstatistics（bool）：打印详细的去重统计信息
  stringpeduplicationAgeThreshold（uintx）：达到这个年龄的string对象被认为是去重的候选对象
  