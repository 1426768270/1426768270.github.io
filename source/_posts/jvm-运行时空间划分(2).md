---
title: jvm(3)--堆
date: 2020-7-16 11:53:13
categories: 
- java基础
tag:
- java
- jvm
---

## 本地方法接口

<!--more -->

**一个Native Method就是一个java调用非java代码的接口**，一个Native Method 是这样一个java方法：该方法的底层实现由非Java语言实现，比如C。这个特征并非java特有，很多其他的编程语言都有这一机制，比如在C++ 中，你可以用extern “C” 告知C++ 编译器去调用一个C的函数。
在定义一个native method时，并不提供实现体（有些像定义一个Java interface），因为其实现体是由非java语言在外面实现的。
本地接口的作用是融合不同的编程语言为java所用，它的初衷是融合C/C++程序。
标识符native可以与其他所有的java标识符连用，但是abstract除外。

- 与java环境外交互：
- 与操作系统交互（比如线程最后要回归于操作系统线程）
- Sun’s Java
  Sun的解释器是用C实现的，这使得它能像一些普通的C一样与外部交互。

## 本地方法栈

1.Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法（一般非Java实现的方法）的调用

2.本地方法栈，也是线程私有的。

3.允许被实现成固定或者是可动态拓展的内存大小。（和Java虚拟机栈在内存溢出方面情况是相同的）

如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个StackOverFlowError异常。

如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么java虚拟机将会抛出一个OutOfMemoryError异常。

4.本地方法是使用C语言实现的

5.它的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载本地方法库。

**6.当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限**

本地方法可以通过本地方法接口来 **访问虚拟机内部的运行时数据区**

它甚至可以直接使用本地处理器中的寄存器

直接从本地内存的堆中分配任意数量的内存

7.并不是所有的JVM都支持本地方法。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。如果JVM产品不打算支持native方法，也可以无需实现本地方法栈。

## 堆

所有的线程共享java堆

数组或对象永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。

在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除

堆，是GC(Garbage Collection，垃圾收集器)执行垃圾回收的重点区域



jdk目录，jdk1.8.0_171.jdk/Contents/Home/bin下找到jvisualvm 运行（或者直接终端运行jvisualvm），查看进程，可以看到我们设置的配置信息： 



JDK 8以后： 逻辑上分为新生区+养老区+元空间（即Xms/Xmx分配的内存物理上没有涉及元空间）

元空间在方法区

- Young Generation Space：又被分为Eden区和Survior区 
- Tenure generation Space： Old/Tenure
- Meta Space： Meta

#### 设置堆内存大小与OOM

1.Java堆区用于存储java对象实例，堆的大小在jvm启动时就已经设定好了，可以通过 "-Xmx"和 "-Xms"来进行设置

- -Xms 用来设置堆空间（年轻代+老年代）的初始内存大小，等价于 -XX:InitialHeapSize
  - -X 是jvm的运行参数
  - ms 是memory start
- -Xmx 用于设置堆的最大内存，等价于 -XX:MaxHeapSize

2.一旦堆区中的内存大小超过 -Xmx所指定的最大内存时，将会抛出OOM异常。

- 默认情况下，初始内存大小：物理内存大小/64;最大内存大小：物理内存大小/4。
- 手动设置：-Xms600m -Xmx600m

3.通常会将-Xms和-Xmx两个参数配置相同的值，其目的就是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。

- 比如说：默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。
  因此服务器一般设置-Xms、-Xmx相等以避免在每次GC 后调整堆的大小

4.查看设置的堆内存参数：

- 方式一： ==终端输入jps== ， 然后 ==jstat -gc 进程id==
- 方式二：（控制台打印）Edit Configurations->VM Options 添加 ==-XX:+PrintGCDetails==

**form 和to区只有一个能放**

```java
/**
 * -Xms600m -Xmx600m
 */
public class OOMTest {
    public static void main(String[] args) {
        ArrayList<Picture> list = new ArrayList<>();
        while(true){
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024 * 1024)));
        }
    }
}

class Picture{
    private byte[] pixels;

    public Picture(int length) {
        this.pixels = new byte[length];
    }
}
```

#### 年轻代与老年代

![堆空间](jvm-运行时空间划分(2)/堆空间.png)

- 配置新生代与老年代在堆结构的占比
  - 默认-XX:NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3
  - 可以修改-XX:NewRatio=4，表示新生代占1，老年代占4，新生代占整个堆的1/5
- 在hotSpot中，Eden空间和另外两个Survivor空间缺省所占的比例是8：1：1（测试的时候是6：1：1），开发人员可以通过选项 -XX:SurvivorRatio 调整空间比例，如-XX:SurvivorRatio=8
- 可以使用选项-Xmn设置新生代最大内存大小（这个参数一般使用默认值就好了）

**过程**

1.new的对象先放伊甸园区。此区有大小限制。

2.当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC),将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区

3.然后将伊甸园中的剩余的幸存对象移动到幸存者0区。

4.如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。

5.如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。

6.啥时候能去养老区呢？可以设置次数。默认是15次。·可以设置参数：-XX:MaxTenuringThreshold=进行设置。

7.在养老区，相对悠闲。当老年区内存不足时，再次触发GC：Major GC，进行养老区的内存清理。

8.若养老区执行了Major GC之后发现依然无法进行对象的保存，就会产生OOM异常。

总结：针对幸存者s0,s1区：复制之后有交换，谁空谁是to。
      关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不再永久区/元空间收集。

注意：只有伊甸园满了才会触minorGC/youngGC,而幸存者区满了是绝对不会触发minorGC的

#### 常用调优工具

1.JDK命令行

2.Eclipse：Memory Analyzer Tool

3.Jconsole

4.VisualVM

5.Jprofiler

6.Java Flight Recorder

7.GCViewer

8.GC Easy

针对hotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）

#### Minor GC、Major GC、Full GC

**1.部分收集**：不是完整收集整个Java堆的垃圾收集。其中又分为：

- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
- 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集
  - 目前，只有CMS GC会有单独收集老年代的行为
  - 注意，很多时候Major GC 会和 Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收
- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
  - 混合收集不涉及方法区回收，只是新生代，老年代的混合收集。
  - 目前，之后G1 GC会有这种行为。

**2.整堆收集**（Full GC）：收集整个java堆和方法区的垃圾收集。

**年轻代GC（Minor GC）触发机制**：

- 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC.(每次Minor GC会清理年轻代的内存，Survivor是被动GC，不会主动GC)

**老年代GC(Major GC/Full GC)触发机制**

- 出现了Major GC，经常会伴随至少一次的Minor GC（不是绝对的，在Parallel Scavenge 收集器的收集策略里就有直接进行Major GC的策略选择过程）
  - 也就是老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC
- Major GC速度一般会比Minor GC慢10倍以上，STW时间更长
- 如果Major GC后，内存还不足，就报OOM了

**Full GC触发机制**

- 触发Full GC执行的情况有以下五种
  - ①调用System.gc()时，系统建议执行Full GC，但是**不必然执行**
  - ②老年代空间不足
  - ③方法区空间不足
  - ④通过Minor GC后进入老年代的平均大小小于老年代的可用内存
  - ⑤由Eden区，Survivor S0（from）区向S1（to）区复制时，对象大小由于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
- 说明：Full GC 是开发或调优中尽量要避免的，这样暂停时间会短一些

#### 内存分配策略总结

- 如果对象在Eden出生并经过第一次Minor GC后依然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，把那个将对象年龄设为1.对象

  在Survivor区中每熬过一次MinorGC

  ，年龄就增加一岁，当它的年龄增加到一定程度（默认15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代中

  - 对象晋升老年代的年龄阈值，可以通过选项 -XX：MaxTenuringThreshold来设置

- 针对不同年龄段的对象分配原则如下：

  - 优先分配到Eden
  - 大对象直接分配到老年代
    - 我们要尽量避免程序中出现过多的大对象
  - 长期存活的对象分配到老年代
  - 动态对象年龄判断
    - 如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入到老年代。无需等到MaxTenuringThreshold中要求的年龄

## TLAB

#### **（Thread Local Allocation Buffer）**

- 众所周知堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据。由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的。
- 一般为了避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。
- 为了解决这一问题，TLAB应运而生。

#### **什么是TLAB**

- 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它包含在Eden空间内
- 多线程同时分配内

![TLAB](jvm-运行时空间划分(2)/TLAB.png)

“-XX:UseTLAB“ 设置是够开启TLAB空间

”-XX:TLABWasteTargetPercent“ 设置TLAB空间所占用Eden空间的百分比大小

一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden空间中分配了内存

## 堆空间的参数设置

- -XX:PrintFlagsInitial: 查看所有参数的默认初始值

- -XX:PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）

  - 具体查看某个参数的指令：
    - jps：查看当前运行中的进程
    - jinfo -flag SurvivorRatio 进程id： 查看新生代中Eden和S0/S1空间的比例

- -Xms: 初始堆空间内存（默认为物理内存的1/64）

- -Xmx: 最大堆空间内存（默认为物理内存的1/4）

- -Xmn: 设置新生代大小（初始值及最大值）

- -XX:NewRatio: 配置新生代与老年代在堆结构的占比

- -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例

- -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄(默认15)

- -XX:+PrintGCDetails：输出详细的GC处理日志

  - 打印gc简要信息：① -XX:+PrintGC ② -verbose:gc

- -XX:HandlePromotionFailure：是否设置空间分配担保

  在发生Minor Gc之前，虚拟机会检查老年代**最大可用的连续空间是否大于新生代所有对象的总空间**。

  - 如果大于，则此次Minor GC是安全的
  - 如果小于，则虚拟机会查看-XX:HandlePromotionFailure设置值是否允许担保失败。（==JDK 7以后的规则HandlePromotionFailure可以认为就是true）
    - 如果HandlePromotionFailure=true,那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小。
      - √如果大于，则尝试进行一次Minor GC,只是尝试看能否触发分配担保（我们肯定希望的是分配担保成功Eden-->old/tentired），但这次Minor GC依然是有风险的；
      - √如果小于，则改为进行一次Full GC。
    - √如果HandlePromotionFailure=false,则改为进行一次Full GC。

## 堆是分配对象的唯一选择么?

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过逃逸分析（Escape Analysis)后发现，一**个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。**这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。

- 逃逸分析的基本行为就是分析对象动态作用域：
  - 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
  - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。
- 如何快速的判断是否发生了逃逸分析，就看new的对象实体是否有可能在方法外被调用

```java
public void method(){
    V v = new V();
    //use V
    //......
    v = null;
}
```

没有发生逃逸的对象，则可以分配到栈上：原因如下：

1.随着方法执行的结束，栈空间就被移除。

2.虚拟机栈空间是线程私有的，不会被共享。

```java
public static String createStringBuffer(String s1,String s2){
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    //return sb;发生了逃逸
    return sb.toString();//没有发生
}
```

```java
/**
 * 逃逸分析
 *
 *  如何快速的判断是否发生了逃逸分析，就看“new的对象实体”是否有可能在方法外被调用。
 */
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /*
    方法返回EscapeAnalysis对象，发生逃逸
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /*
    为成员属性赋值，发生逃逸
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    //思考：如果当前的obj引用声明为static的？仍然会发生逃逸。

    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
        //getInstance().xxx()同样会发生逃逸
    }
}
```

## 代码优化理论

#### 栈上分配

将堆分配转化为栈分配。如果一个对象在子线程中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配

常见的发生逃逸的场景：给成员变量赋值、方法返回值、实例引用传递

```java
/**
 * 栈上分配测试
 * -Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }
    private static void alloc() {
        User user = new User();//未发生逃逸
    }
    static class User {

    }
}
```

#### 同步省略

在动态编译同步块的时候，JIT编译器可以借助逃逸分析来**判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程**。如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫 **锁消除**

#### 分离对象或标量替换

- 标量Scalar：是指一个无法在分解成更小的数据的数据。Java中的原始数据类型就是标量。
- 相对的，那些还可以分解的数据叫做 聚合量(Aggregate)，Java中对象就是聚合量，因为它可以分解成其他聚合量和标量。
- 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来替代。这个过程就是标量替换

```java
public class ScalarTest {
    public static void main(String[] args) {
        alloc();   
    }
    public static void alloc(){
        Point point = new Point(1,2);
    }
}
class Point{
    private int x;
    private int y;
    public Point(int x,int y){
        this.x = x;
        this.y = y;
    }
}
public static void alloc(){
    int x = 1;
    int y = 2;
}
```

配置参数：-XX:+EliminateAllocations，开启标量替换。