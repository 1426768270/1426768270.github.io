---
title: 多线程（4）
date: 2020-7-15 00:40:19
categories: 
- java基础
tag:
- java
- 多线程
---

## 集合类不安全

#### List 不安全

<!-- more -->

```java
// java.util.ConcurrentModificationException 并发修改异常！
public class ListTest {
    public static void main(String[] args) {
        // 并发下 ArrayList 不安全的吗，Synchronized；
        /*** 解决方案；
        * 1、List<String> list = new Vector<>();
        * 2、List<String> list = Collections.synchronizedList(new ArrayList<> ());
        * 3、List<String> list = new CopyOnWriteArrayList<>()；
  		*/
        // CopyOnWrite 写入时复制 COW 计算机程序设计领域的一种优化策略；
        // 多个线程调用的时候，list，读取的时候，固定的，写入（覆盖）
        // 在写入的时候避免覆盖，造成数据问题！
        // 读写分离
        // CopyOnWriteArrayList 比 Vector Nb 在哪里？
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list); 
                },String.valueOf(i)).start(); 
        }
    } 
}
```

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

lock 利用Arrays.copyof()来复制一份新数组，再进行设置

#### Set 不安全

```java
// Set<String> set = new HashSet<>(); 不安全
// Set<String> set = Collections.synchronizedSet(new HashSet<>());
Set<String> set = new CopyOnWriteArraySet<>();
```

hashSet 底层是什么？

```java
public HashSet() {
    map = new HashMap<>();
}
// add set 本质就是 map key是无法重复的！
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
private static final Object PRESENT = new Object();
```

#### Map 不安全

hashMap

加载因子 0.75

位运算  16

```java
// ConcurrentModificationException 
public class MapTest { 
    public static void main(String[] args) { 
        // map 是这样用的吗？ 不是，工作中不用 HashMap 
        // 默认等价于什么？ new HashMap<>(16,0.75); 
        // Map<String, String> map = new HashMap<>(); 
        // 唯一的一个家庭作业：研究ConcurrentHashMap的原理 
        Map<String, String> map = new ConcurrentHashMap<>(); 
        for (int i = 1; i <=30; i++) { 
            new Thread(()->{ 
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring( 0,5)); 
                System.out.println(map); },String.valueOf(i)).start(); 
        } 
    } 
}
```

##### ConcurrentHashMap

## 阻塞队列

FIFO 
写入 ：队列慢，就阻塞，
取：队列空，就必须等待生产

![Queue](java多线程(4)/Queue.png)![BlockingQueue](java多线程(4)/BlockingQueue.png)

#### BlockingQueue

![实现图](java多线程(4)/实现图.png)

 

阻塞队列：多线程并发处理，线程池！

**学会使用队列**

| **方式**     | **抛出异常** | **有返回值，不抛出异常** | **阻塞 等待** | **超时等待** |
| ------------ | ------------ | ------------------------ | ------------- | ------------ |
| 添加         | add          | offer()                  | put()         | offer(,,)    |
| 移除         | remove       | poll()                   | take()        | poll(,)      |
| 检测队首元素 | element      | peek                     |               |              |

添加、移除

#### SynchronousQueue 

同步队列

没有容量，进去一个元素，必须等待取出来之后，才能再往里面放一个元素！

```java
/**
 * 同步队列
 * 和其他的BlockingQueue 不一样， SynchronousQueue 不存储元素
 * put了一个元素，必须从里面先take取出来，否则不能在put进去值！
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>(); // 同步队列

        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+" put 1");
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+" put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+" put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();


        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}
```

## 线程池

线程池：三大方法、7大参数、4种拒绝策略

#### 池化技术

程序的运行，本质：占用系统的资源！ 优化资源的使用！=>池化技术

线程池、连接池、内存池、对象池///..... 创建、销毁。十分浪费资源

池化技术：事先准备好一些资源，有人要用，就来我这里拿，用完之后还给我。

**线程池的好处**

1、降低资源的消耗

2、提高响应的速度

3、方便管理。

**线程复用、可以控制最大并发数、管理线程**

#### 线程池：三大方法

```java
// Executors 工具类、3大方法
public class Demo01 {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线程
        // ExecutorService threadPool = Executors.newFixedThreadPool(5);// 创建一 个固定的线程池的大小
        // ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的，遇强则强，遇弱则弱
        try {
            for (int i = 0; i < 100; i++) {
            // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

#### 7大参数

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory){
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
//本质
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

1. corePoolSize: 线程池的核心线程数(常驻线程数),也就是线程池的最小线程数,这部分线程不会被回收.

2. maximumPoolSize: 线程池最大线程数,线程池中允许同时执行的最大线程数量

3. keepAliveTime: 当线程池中的线程数量超过corePoolSize，但此时没有任务执行，
   那么空闲的线程会保持keepAliveTime才会被回收，corePoolSize的线程不会被回收。

4. unit: KeepAliveTime的时间单位

5. workQueue: 当线程池中的线程达到了corePoolSize的线程数量，
   并仍然有新任务，那么新任务就会被放入workQueue。          

6. threadFactory: 创建工作线程的工厂,也就是如何创建线程的,一般采用默认的

7. handler: 拒绝策略，当线程池中的工作线程达到了最大数量，
   并且阻塞队列也已经满了，那么拒绝策略会决定如何处理新的任务。ThreadPoolExecutor 提供了四种策略:

   - AbortPolicy(是线程池的默认拒绝策略): 如果使用此拒绝策略，那么将对新的任务抛出RejectedExecutionException异常，来拒绝任务。
   - DiscardPolicy: 如果使用此策略，那么会拒绝执行新的任务，但不会抛出异常。
   - DiscardOldestPolicy: 如果使用此策略，那么不会拒绝新的任务，但会抛弃阻塞队列中等待最久的那个线程。     
   - CallerRunsPolicy: 如果使用此策略，不会拒绝新的任务，但会让调用者执行线程。
     也就是说哪个线程发出的任务，哪个线程执行。

#### 四种拒绝策略

```java
new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人的，抛出异常
new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的去哪里！ main线程执行
new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！
new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和最早的竞争，也不会抛出异常！
```



#### 阿里巴巴开发者手册

不建议开发者使用Executors创建线程池

**newFixedThreadPool和newSingleThreadPoolExecutor都是创建固定线程的线程池,
尽管它们的线程数是固定的，但是它们的阻塞队列的长度却是Integer.MAX_VALUE的,所以，
队列的任务很可能过多，导致OOM。**

**newCacheThreadPool和newScheduledThreadPool创建出来的线程池的线程数量却是Integer.MAX_VALUE的，
如果任务数量过多,也很可能发生OOM。**          

![线程池规范](java多线程(4)/线程池规范.png)

#### 手动创建一个线程池

```java
public class ThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy() //拒绝策略
        );
        try {
            // 最大承载：Deque + max
            // 超过 RejectedExecutionException
            for (int i = 1; i <= 9; i++) {
                // 使用了线程池之后，使用线程池来创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" ok");
                });
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }

}
```

#### 拓展

```test
最大线程到底该如何定义
1、CPU 密集型，几核，就是几，可以保持CPu的效率最高！
Runtime.getRuntime().availableProcessors(),
2、IO  密集型   > 判断你程序中十分耗IO的线程，
程序   15个大型任务  io十分占用资源！
```

## 四大函数式接口（必需掌握）

lambda表达式、链式编程、函数式接口、Stream流式计算

#### 函数式接口： 只有一个方法的接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
// 泛型、枚举、反射 
// lambda表达式、链式编程、函数式接口、Stream流式计算 
// 超级多FunctionalInterface 
// 简化编程模型，在新版本的框架底层大量应用！ 
// foreach(消费者类的函数式接口)
```

四大函数式接口

Consumer、Function 、Predicate、Supplier

#### Function 

![Function](java多线程(4)/Function.png)

```java
public static void main(String[] args) {
    //Function 函数型接口, 有一个输入参数，有一个输出
    //只要是 函数型接口 可以 用 lambda表达式简化
    //
    //        Function<String,String> function = new Function<String,String>() {
    //            @Override
    //            public String apply(String str) {
    //                return str;
    //            }
    //        };

    Function<String,String> function = str->{return str;};
    System.out.println(function.apply("asd"));
}
```

#### Predicate

断定型接口

![Predicate](java多线程(4)/Predicate.png)

```java
public static void main(String[] args) {
    //断定型接口：有一个输入参数，返回值只能是 布尔值
    //判断字符串是否为空
    // Predicate<String> predicate = new Predicate<String>() {
    //     @Override
    //     public boolean test(String str) {
    //         return str.isEmpty();
    //     }
    // };
    Predicate<String> predicate = (str)->{
        return str.isEmpty();
    };
    System.out.println(predicate.test(""));
}
```

#### Consumer

消费者接口

![Consumer](java多线程(4)/Consumer.png)

```java
public static void main(String[] args) {
	//只有输入，没有返回值
    // Consumer<String> consumer = new Consumer<String>() {
    //     @Override
    //     public void accept(String s) {
    //         System.out.println(s);
    //     }
    // };
    Consumer<String> consumer = str ->{
        System.out.println(str);
    };
    consumer.accept("aaaa");
}
```

#### Supplier

供给型接口

![Supplier](java多线程(4)/Supplier.png)

```java
public static void main(String[] args) {
   /* Supplier<Integer> supplier = new Supplier<>() {
        @Override
        public Object get() {
            System.out.println("get");
            return 1024;
        }
    };*/
    Supplier supplier = ()->{
        System.out.println("get");
        return 1024;
    };
    System.out.println(supplier.get());
}
```

## Stream流式计算

大数据：存储 + 计算

集合、MySQL 本质就是存储东西的；

计算都应该交给流来操作！

```java
/**
 * 题目要求：一分钟内完成此题，只能用一行代码实现！
 * 现在有5个用户！筛选：
 * 1、ID 必须是偶数
 * 2、年龄必须大于23岁
 * 3、用户名转为大写字母
 * 4、用户名字母倒着排序
 * 5、只输出一个用户！
 */
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1,"a",21);
        User u2 = new User(2,"b",22);
        User u3 = new User(3,"c",23);
        User u4 = new User(4,"d",24);
        User u5 = new User(6,"e",25);
        // 集合就是存储
        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);

        // 计算交给Stream流
        // lambda表达式、链式编程、函数式接口、Stream流式计算
        list.stream()
                .filter(u->{return u.getId()%2==0;})
                .filter(u->{return u.getAge()>23;})
                .map(u-> {return u.getName().toUpperCase();})
                .sorted( (uu1,uu2)->{return uu2.compareTo(uu1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```

## ForkJoin

ForkJoin 在 JDK 1.7 ， 并行执行任务！提高效率。大数据量！

大数据：Map Reduce （把大任务拆分为小任务）

#### ForkJoin 特点：工作窃取

这个里面维护的都是双端队列

![工作窃取](java多线程(4)/工作窃取.png)

![ForkJoin](java多线程(4)/ForkJoin.png)

```java
public class ForkJoinDemo  extends RecursiveTask<Long> {
    //3000   6000（ForkJoin）  9000（Stream并行流）
    //1、forkjoinPool 通过它来执行
    // 2、计算任务 forkjoinPool.execute(ForkJoinTask task)
    // 3. 计算类要继承 ForkJoinTask
    private Long start;  // 1
    private Long end;    // 1990900000

    // 临界值
    private Long temp = 10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute(){
        if ( (end-start)<temp){
            Long sum = 0L;
            for (Long i=start;i<end;i++){
                sum += i;
            }
            return sum;
        }else {
            //分支合并计算  递归
            long middle = (start + end) / 2;
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();// 拆分任务，把任务压入线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle+1, end);
            task2.fork(); // 拆分任务，把任务压入线程队列
            return task1.join() + task2.join();
        }
    }
}

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //test1(); //8151
        //test2(); //4333
        test3();//359
    }

    // 普通程序员
    public static void test1(){
        Long sum = 0L;
        long start = System.currentTimeMillis();
        for (Long i = 1L; i <= 10_0000_0000; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum="+sum+" 时间："+(end-start));
    }

    // 会使用ForkJoin
    public static void test2() throws ExecutionException, InterruptedException {
        Long start = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);// 提交任务
        Long sum = submit.get();

        long end = System.currentTimeMillis();

        System.out.println("sum="+sum+" 时间："+(end-start));
    }

    public static void test3(){
        long start = System.currentTimeMillis();
        // Stream并行流 ()  (]
        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum="+sum+"时间："+(end-start));
    }
}
```

## Future

 设计的初衷： 对将来的某个事件的结果进行建模

```java
/**
 * 异步调用： CompletableFuture
 * // 异步执行
 * // 成功回调
 * // 失败回调
 */
public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        /*//无返回值
        CompletableFuture<Void> com = CompletableFuture.runAsync( ()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"runAsync=>Void");
        });
        System.out.println("111111111");
        com.get();  //获取阻塞执行结果*/
        //有返回值
        // ajax，成功和失败的回调
        // 返回的是错误信息；
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync( ()->{
            System.out.println(Thread.currentThread().getName()+"supplyAsync=>supply");
            return 1024;
        });
        System.out.println(completableFuture.whenComplete((t, u) -> {
            System.out.println("t:" + t); // 正常的返回结果
            System.out.println("u:" + u); // 错误信息：java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
        }).exceptionally((e) -> {
            System.out.println(e.getMessage());
            return 233;// 可以获取到错误的返回结果
        }).get());

    }
}
```

## 单例模式

#### 饿汉式

```java
public class Hungry {
    //可能浪费空间
    private byte[] data1 =new byte[1024*1024];
    private byte[] data2 =new byte[1024*1024];
    private byte[] data3 =new byte[1024*1024];
    private byte[] data4 =new byte[1024*1024];

    private Hungry(){

    }

    private final static Hungry HUNGRY = new Hungry();

    public static Hungry getInstance(){
        return HUNGRY;
        }
}
   
```

#### 懒汉式

```java
public class LazyMan {

    private static boolean liuqi = false;

    private LazyMan(){
        synchronized (LazyMan.class){
            if (liuqi==false){
                liuqi=true;
            }else {
                throw new RuntimeException("不要使用反射破坏异常");
            }
        }
        System.out.println(Thread.currentThread().getName()+"ok");
    }

    private volatile  static LazyMan lazyMan;

    //双重检查锁 DCL懒汉式
    public static LazyMan getInstance(){
        if (lazyMan==null){
            synchronized (LazyMan.class){
                if (lazyMan == null) {
                    lazyMan=new LazyMan(); //不是一个原子操作
                    /**
                     * new分为3步
                     * 1.分配内存空间
                     * 2.执行构造方法，初始化
                     * 3.把这个对象指向这个空间
                     * 可能重排序
                     */
                }
            }
        }
        return lazyMan;
    }

    public static void main(String[] args) throws Exception {

        Field liuqi = LazyMan.class.getDeclaredField("liuqi");
        liuqi.setAccessible(true);
        Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        LazyMan instance = declaredConstructor.newInstance();
        liuqi.set(instance,false);
        LazyMan instance2 = declaredConstructor.newInstance();

        System.out.println(instance);
        System.out.println(instance2);
    }

}
```

#### 静态内部类

```java
public class Holder {
    private Holder(){

    }

    public static Holder getInstance(){
        return InnerClass.HOLDER;
    }

    public static class InnerClass{
        private static  final Holder HOLDER = new Holder();
    }

}
```

#### 枚举

```java
//本身也是一个class类
public enum  EnumSingle {

    INSTANCE;
    private EnumSingle(){

    }
    public EnumSingle getInstance(){
        return INSTANCE;
    }

}
class Test{
    public static void main(String[] args) throws Exception {
        EnumSingle instance1 = EnumSingle.INSTANCE;
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        EnumSingle instance2 = declaredConstructor.newInstance(String.class,int.class);
        // NoSuchMethodException: com.kuang.single.EnumSingle.<init>()
        System.out.println(instance1);
        System.out.println(instance2);
    }
}
```

![反编译枚举](java多线程(4)/反编译枚举.png)

## 原子引用



```java
public class CASDemo {

    //AtomicStampedReference 注意，如果泛型是一个包装类，注意对象的引用问题

    // 正常在业务操作，这里面比较的都是一个个对象
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1,1);

    // CAS  compareAndSet : 比较并交换！
    public static void main(String[] args) {

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp(); // 获得版本号
            System.out.println("a1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            Lock lock = new ReentrantLock(true);

            atomicStampedReference.compareAndSet(1, 2,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);

            System.out.println("a2=>"+atomicStampedReference.getStamp());


            System.out.println(atomicStampedReference.compareAndSet(2, 1,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));

            System.out.println("a3=>"+atomicStampedReference.getStamp());

        },"a").start();


        // 乐观锁的原理相同！
        new Thread(()->{
            int stamp = atomicStampedReference.getStamp(); // 获得版本号
            System.out.println("b1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(atomicStampedReference.compareAndSet(1, 6,
                    stamp, stamp + 1));

            System.out.println("b2=>"+atomicStampedReference.getStamp());

        },"b").start();

    }
}
```

**注意：**
**Integer 使用了对象缓存机制，默认范围是 -128 ~ 127 ，推荐使用静态工厂方法 valueOf 获取对象实例，而不是 new，因为 valueOf 使用缓存，而 new 一定会创建新的对象分配新的内存空间；**

![Integer](java多线程(4)/Integer.png)

## 锁的理解

#### 1.公平锁，非公平锁

公平锁： 非常公平， 不能够插队，必须先来后到！

非公平锁：非常不公平，可以插队 （默认都是非公平）

```java
public ReentrantLock() { 
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync(); 
}
```

#### 2.可重入锁

递归锁

##### synchronize

```java
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();

        new Thread(()->{
            phone.sms();
        },"A").start();


        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}

class Phone{

    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName() + "sms");
        call(); // 这里也有锁
    }

    public synchronized void call(){
        System.out.println(Thread.currentThread().getName() + "call");
    }
}
```

#### 3.自旋锁

![spinlock](java多线程(4)/spinlock.png)

```java
public class SpinlockDemo {

    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    //加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>mylock");
        //自旋锁
        while (!atomicReference.compareAndSet(null,thread)){

        }
    }

    //解锁
    public void myUnLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>mylock");
        atomicReference.compareAndSet(thread,null);
    }
}
```

#### 4.死锁

1、使用 jps -l 定位进程号

![定位进程号](java多线程(4)/定位进程号.png)

2、使用 jstack 进程号 找到死锁问题

![排查死锁](java多线程(4)/排查死锁.png)

排查问题

1.日志

2.堆栈信息