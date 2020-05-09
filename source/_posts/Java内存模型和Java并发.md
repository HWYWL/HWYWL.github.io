---
title: Java内存模型和Java并发
date: 2018-08-24 16:11:35
tags: 
 - 高并发
 - 分布式
categories: 
 - 分布式
keywords: "分布式,高并发"
description: Java内存模型和Java并发。
---


Semaphore（信号量）
Semaphore（信号量）是一个线程同步结构，用于在线程间传递信号，以避免出现信号丢失，或者像锁一样用于保护一个关键区域。
<!--more-->
# Semaphore（信号量）
Semaphore（信号量）是一个线程同步结构，用于在线程间传递信号，以避免出现信号丢失，或者像锁一样用于保护一个关键区域。

### 前言

面试Java，必然要被问Java内存模型和Java并发开发。我被问到的时候，心里慌得一批，“额，是在《Thinking in Java》里面写的吗？果然每天增删改太low了”。

![](https://i.imgur.com/HU4Wb7j.jpg)
![](https://i.imgur.com/nekdQFv.jpg)
![](https://i.imgur.com/UKXLLTh.jpg)
![](https://i.imgur.com/ahXyP3Q.jpg)

我希望能解释的再简单一些，以上都不用

Java 并发代码
```
public class Example1 {
    public static int count = 0;
    public static int clientTotal = 5000;
    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < clientTotal ; i++) {
            executorService.execute(() -> {
                try {
                    add();
                } catch (Exception e) {
                    log.error("exception", e);
                }
            });
        }
    }
    private static void add() {
        count++;
    }
}
```

如果上面代码执行，count的值是多少？（为了说明重点问题，没有写最后打印的代码）5000？多次运行的结果，count的值是小于5000的。

解释一下上面的程序，首先定义了一个线程池，启动5000个线程执行add()操作，add函数处理静态成员变量count。

如果程序顺序调用，count的值应该是5000。

```
for(int i=0;i<5000;i++){
    add();
}
```

复制代码但是线程池启动多线程，是并发执行的。每个线程启动之后，不管是否运行结束，下一个线程会马上启动。

启动线程的过程，是一个异步过程，启动线程立即返回，启动下一个进程。

当多个线程对同一个变量add进行操作的时候，就会发生写写冲突。

线程1、线程2 同时对值为0的变量进行操作，结果返回1，而不是2。如果这个地方想不明白，就请留言，或者看看文章顶部那些原理图。

**要不简单点，记住“多线程对全局变量的写操作会发生冲突”。**
**答案，声明原子变量 AtomicInteger count**

```
public class ConcurrentSemaphore {
    // 请求总数
    public static int clientTotal = 5000;

    // 同时并发执行的线程数
    public static int threadTotal = 200;

    public static AtomicInteger count = new AtomicInteger(0);

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal ; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (Exception e) {
                    log.error("exception", e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count:{}", count.get());
    }
    private static void add() {
        count.incrementAndGet();
        log.info(" 线程:[" + Thread.currentThread().getName() + "] " + count.get());
    }
}
```

注，上面的代码用了生成者消费者模式，5000个生产者，200个消费者，对程序并发做一定限制，防止5000个线程卡死计算机。

内存模型，也说点简单的

**栈（heap），函数加载的时候，为函数内部变量分配的空间。和父函数的内部变量和运行指针共享同一块区域。**

**函数运行时，new的空间，都是放在堆中的。**

这个就是C的内存模型，做shellcode的基础知识。