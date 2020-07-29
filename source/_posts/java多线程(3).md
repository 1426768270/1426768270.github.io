---
title: 多线程（3）
date: 2020-7-15 00:40:06
categories: 
- java基础
tag:
- java
- 多线程
---

## Lock

#### reentrantlock

使用reentrantlock可以完成sync的功能

CAS锁

需要注意的是，必须要必须要必须要**手动释放锁**（重要的事情说三遍）

使用syn锁定的话如果遇到异常，jvm会自动释放锁，但是lock必须手动释放锁，因此经常在finally中进行锁的释放

<!-- more -->

```java
public class T02_ReentrantLock2 {
   Lock lock = new ReentrantLock();

   void m1() {
      try {
         lock.lock(); //synchronized(this)
         for (int i = 0; i < 10; i++) {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(i);
         }
      } catch (InterruptedException e) {
         e.printStackTrace();
      } finally {
         lock.unlock();
      }
   }
   void m2() {
      try {
         lock.lock();
         System.out.println("m2 ...");
      } finally {
         lock.unlock();
      }
   }
   public static void main(String[] args) {
      T02_ReentrantLock2 rl = new T02_ReentrantLock2();
      new Thread(rl::m1).start();
      try {
         TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
      new Thread(rl::m2).start();
   }
}
```

使用tryLock进行尝试锁定，不管锁定与否，方法都将继续执行

可以根据tryLock的返回值来判定是否锁定

 也可以指定tryLock的时间，由于tryLock(time)抛出异常，所以要注意unclock的处理，必须放到finally中

```java
void m2() {
		boolean locked = false;
		try {
			locked = lock.tryLock(5, TimeUnit.SECONDS);
			System.out.println("m2 ..." + locked);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			if(locked) lock.unlock();
		}
		
	}
```

使用ReentrantLock还可以调用lockInterruptibly方法，可以对线程interrupt方法做出响应，

 在一个线程等待锁的过程中，**可以被打断**

打断了之前加的所

```java 
Thread t2 = new Thread(()->{
   try {
      //lock.lock();
      lock.lockInterruptibly(); //可以对interrupt()方法做出响应
      System.out.println("t2 start");
      TimeUnit.SECONDS.sleep(5);
      System.out.println("t2 end");
   } catch (InterruptedException e) {
      System.out.println("interrupted!");
   } finally {
      lock.unlock();
   }
});
t2.start();
```

```java
ReentrantLock还可以指定为公平锁
private static ReentrantLock lock=new ReentrantLock(true);
```



#### synchronized与Lock的区别 

两者区别：
1.首先synchronized是java内置关键字，在jvm层面，Lock是个java类；
2.synchronized无法判断是否获取锁的状态，Lock可以判断是否获取到锁；
3.synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在finally中手工释放锁（unlock()方法释放锁），否则容易造成线程死锁；
4.用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了；
5.synchronized的锁**可重入、不可中断、非公平**，而Lock锁**可重入、可判断、可公平**（两者皆可）
6.Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题。

#### Condition 

精准的通知和唤醒线程

```java
class ShareData{

    private int number = 1;
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void print5(){
        lock.lock();
        try {
            //1.判断
            while (number!=1){
                condition1.await();
            }s
            //2.干活
            for (int i = 1;i<=5;i++){
                System.out.println(Thread.currentThread().getName()+"\t"+i);
            }
            //3.通知
            number = 2;
            condition2.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void print10(){
        lock.lock();
        try {
            //1.判断
            while (number!=2){
                condition2.await();
            }
            //2.干活
            for (int i = 1;i<=10;i++){
                System.out.println(Thread.currentThread().getName()+"\t"+i);
            }
            //3.通知
            number = 3;
            condition3.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void print15(){
        lock.lock();
        try {
            //1.判断
            while (number!=3){
                condition3.await();
            }
            //2.干活
            for (int i = 1;i<=15;i++){
                System.out.println(Thread.currentThread().getName()+"\t"+i);
            }
            //3.通知
            number = 1;
            condition1.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

}
public class ConditionDemo {

    public static void main(String[] args) {
        ShareData shareData = new ShareData();

        new Thread(() -> {
            for (int i = 1;i<10;i++){
                shareData.print5();
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 1;i<10;i++){
                shareData.print10();
            }
        },"B").start();

        new Thread(() -> {
            for (int i = 1;i<10;i++){
                shareData.print15();
            }
        },"C").start();
    }

}
```

#### Callable

和runable

1、可以有返回值

2、可以抛出异常

3、方法不同，run()/ call()

![Callable实现](java多线程(3)\Callable实现.png)

FutureTask是一个Runnable的实现类

他的构造方法可以和Callable有关

```java
class  MyThread implements Runnable{
    @Override
    public void run() {

    }
}
class  MyThread2 implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        System.out.println("*****come in call method ");
        return 1024;
    }
}
public class CallableDemo {
    public static void main(String[] args) {
        FutureTask futureTask = new FutureTask(new MyThread2());
        new Thread(futureTask,"A").start(); // 结果会被缓存，效率高
        Integer o = (Integer) futureTask.get();//这个get 方法可能会产生阻塞！把他放到 最后
        // 或者使用异步通信来处理！
        System.out.println(o);
    }
}
```

1. 有缓存
2. 会阻塞



## AQS(AbstractQueuedSynchronizer)

````text
AQS是Doug Lea大师为JDK编写的一套基于API层面的抽象队列同步器.
AbstractQueuedSynchronizer,抽象队列同步器.
Lock,CountDownLatch等等这些并发工具都是基于AQS来实现的。
由此可以看出Doug Lea大师的功力已经臻至化境
````

#### AQS概述

**AQS的核心思想是如果被请求的资源空闲，那么就将当前请求资源的线程设置为有效的工作线程;
如果请求的资源被其他线程所占有， 那么就使用CLH线程阻塞队列来提供阻塞线程并唤线程分配资源的机制。
在CLH队列中，每个请求资源的线程都会被封装成队列中的一个节点。**

**在AQS内部有一个int类型的state表示线程同步状态，
当线程lock获取到锁后，该state计数就加1,unlock就减1，
这就是为什么解锁次数要对应加锁次数的原因。**

**AQS主要实现技术为:CLH队列(Craig,Landin and Hagersten)，
自旋CAS，park(阻塞线程)以及unparkSuccessor(唤醒阻塞队列中的后继线程)。**
     

#### AQS的两种共享资源的访问方式

>AQS定义了两种共享资源方式.

1. 独占式(Exclusive): **同一时间只有一个线程可以访问共享资源,也就是独占锁。**
   如:Synchronized,ReentrantLock。
   **对于独占式锁的实现,在AQS中对应tryAcquire获取锁和tryRelease释放锁。**
         

* 共享式(Share): **同一时间允许多个线程同时访问共享资源,也就是共享锁。**
  CountDownLatch,Semaphore,ReentrantReadWriteLock的ReadLock都是共享锁。
  **对于共享式锁的实现,在AQS中对应tryAcquireShare获取锁和tryReleaseShare释放锁。**

#### lock,tryLock和lockInterruptibly区别

**PS: AQS中的锁计数指的是 state 变量。**

- lock: 如果线程获取到了锁或线程已经拥有了锁就更改锁计数，
  否则线程就加入阻塞队列并一直CAS自旋获取。

- tryLock: 线程尝试获取锁，如果线程获取到了锁或线程已经拥有了锁就更改锁计数，否则返回false。

- lockInterruptibly: 如果线程在获取锁之前被设置了中断状态，那么当线程获取锁时就会响应中断状态，
  抛出InterruptedException异常。如果获取不到就加入阻塞队列并自旋获取，并且阻塞自旋期间还会响应中断，
  也就是说在阻塞自旋期间可能抛出InterruptedException异常。
  **所以lockInterruptibly优先响应中断，而不是优先获取锁。** 
  如果线程获取到了锁或线程已经拥有了锁才更改锁计数。

#### CountDownLatch

倒计时 不一定在一个线程只减一下，可以多下

>CountDownLatch允许count个线程阻塞在一个地方，直至所有线程的任务都执行完毕。

CountDownLatch是共享锁的一种实现,**它默认构造AQS的state为count。
当线程使用countDown方法时,其实使用了tryReleaseShared方法以CAS的操作来减少state,
直至state为0就代表所有的线程都调用了countDown方法。**
假如某线程A调用await方法时，如果state不为0，就代表还有线程未执行countDown方法，
那么就把线程A放入阻塞队列Park，并自旋CAS判断state == 0。
直至最后一个线程调用了countDown，使得state == 0，
于是阻塞的线程判断成功，并被唤醒，就继续往下执行。

```java
private static void usingCountDownLatch() {
    Thread[] threads = new Thread[100];
    CountDownLatch latch = new CountDownLatch(threads.length);

    for(int i=0; i<threads.length; i++) {
        threads[i] = new Thread(()->{
            int result = 0;
            for(int j=0; j<10000; j++) result += j;
            latch.countDown();
        });
    }

    for (int i = 0; i < threads.length; i++) {
        threads[i].start();
    }

    try {
        latch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    System.out.println("end latch");
}
```



#### CyclicBarrier

栅栏 达到一定条件栅栏开放  

条件 动作 限流

CycliBarrier的功能与CountDownLatch相似，但是**CountDownLatch的实现是基于AQS的，
而CycliBarrier是基于ReentrantLock(ReentrantLock也属于AQS同步器)和Condition的。**

CountDownLatch虽然可以令多个线程阻塞在同一代码处，但只能await一次就不能使用了。
而CycliBarrier有Generation代的概念，一个代，就代表CycliBarrier的一个循环，

```
public static void main(String[] args) {
    //CyclicBarrier barrier = new CyclicBarrier(20);

    CyclicBarrier barrier = new CyclicBarrier(20, () -> System.out.println("满人"));
    for(int i=0; i<100; i++) {

            new Thread(()->{
                try {
                    barrier.await();

                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        
    }
}
```

#### Phaser

让线程阶段性的同步执行 

```java
static class MarriagePhaser extends Phaser {
    @Override
    protected boolean onAdvance(int phase, int registeredParties) {

        switch (phase) {
            case 0:
                System.out.println("所有人到齐了！" + registeredParties);
                System.out.println();
                return false;
            case 1:
                System.out.println("所有人吃完了！" + registeredParties);
                System.out.println();
                return false;
            case 2:
                System.out.println("所有人离开了！" + registeredParties);
                System.out.println();
                return false;
            case 3:
                System.out.println("婚礼结束！新郎新娘抱抱！" + registeredParties);
                return true;
            default:
                return true;
        }
    }
}
phaser.bulkRegister(7);//注册线程数
 phaser.arriveAndDeregister();//结束
phaser.arriveAndAwaitAdvance();//到达
```

#### ReadWriteLock

读写锁 ：共享锁、排他锁

```java
public class T10_TestReadWriteLock {
    static Lock lock = new ReentrantLock();
    private static int value;

    static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    static Lock readLock = readWriteLock.readLock();
    static Lock writeLock = readWriteLock.writeLock();

    public static void read(Lock lock) {
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println("read over!");
            //模拟读取操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public static void write(Lock lock, int v) {
        try {
            lock.lock();
            Thread.sleep(1000);
            value = v;
            System.out.println("write over!");
            //模拟写操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) {
        //Runnable readR = ()-> read(lock);
        Runnable readR = ()-> read(readLock);

        //Runnable writeR = ()->write(lock, new Random().nextInt());
        Runnable writeR = ()->write(writeLock, new Random().nextInt());

        for(int i=0; i<18; i++) new Thread(readR).start();
        for(int i=0; i<2; i++) new Thread(writeR).start();
    }
}
```

#### Semaphore

信号量  公平锁

>Semaphore允许一次性最多(不是同时)permits个线程执行任务。

Semaphore与CountDownLatch一样，也是共享锁的一种实现。
**它默认构造AQS的state为permits。
当执行任务的线程数量超出permits,那么多余的线程将会被放入阻塞队列Park,并自旋判断state是否大于0。
只有当state大于0的时候，阻塞的线程才有机会继续执行,此时先前执行任务的线程继续执行release方法，
release方法使得state的变量会加1，那么自旋的线程便会判断成功。**

如此，**每次只有不超过permits个的线程能自旋成功，便限制了执行任务线程的数量。**
所以这也是我为什么说它可能不是permits个线程同时执行，
因为只要state>0,线程就有机会执行.

```java
public class T11_TestSemaphore {
    public static void main(String[] args) {
        //Semaphore s = new Semaphore(2);
        Semaphore s = new Semaphore(2, true);
        //允许一个线程同时执行
        //Semaphore s = new Semaphore(1);

        new Thread(()->{
            try {
                s.acquire();

                System.out.println("T1 running...");
                Thread.sleep(200);
                System.out.println("T1 running...");

            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                s.release();
            }
        }).start();

        new Thread(()->{
            try {
                s.acquire();

                System.out.println("T2 running...");
                Thread.sleep(200);
                System.out.println("T2 running...");

                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```