---
title: Java 线程池
date: 2018-03-17 15:18:23
categories: Java
tags:
  - Thread
  - 线程池
---

## 概念
`Java Thread pool` represents a group of worker threads that are waiting for the job and reuse many times.
Java中创建一个线程是一个相对耗时的操作，当程序中频繁的创建和使用线程时，会产生严重的内存管理开销(significant memory management overhead)。基于这个原因，Java有了线程池的概念，在使用线程之前，创建一个线程池。当一个任务需要一个线程去运行时，程序去线程池中选择一个空闲线程去运行。当任务结束后，线程又重新放进线程池中等待下一个任务，这样就避免了频繁的创建线程，大大节省了内存管理开销。

## 线程池分类
`fixed thread pool` : 固型线程池----创建时指定创建的线程数，当任务使用完线程池中的空闲线程，则新任务将等待被占用的线程执行完任务。

固定长度线程池的优点 : 用Web服务器举例说明，Web服务器需要单独的线程去处理一个HTTP请求，当出现大量的HTTP请求，超过了系统能够承受的范围，那么这个Web服务器就会停止响应所有的请求。而如果使用固定长度线程池，虽然不能立刻服务请求，但系统会尽最大能力去处理。

<!-- more -->

``` java
class WorkerThread implements Runnable {
    private String message;
    public WorkerThread(String s){
        this.message=s;
    }
    public void run() {
        System.out.println(Thread.currentThread().getName()+" (Start) message = "+message);
        processmessage();//call processmessage method that sleeps the thread for 2 seconds
        System.out.println(Thread.currentThread().getName()+" (End)");//prints thread name
    }
    private void processmessage() {
        try {  Thread.sleep(2000);  } catch (InterruptedException e) { e.printStackTrace(); }
    }
}

public class TestThreadPool {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);//creating a pool of 5 threads
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);//calling execute method of ExecutorService
        }
        executor.shutdown();
        while (!executor.isTerminated()) {   }

        System.out.println("Finished all threads");
    }
}
```

`CachedThreadPool` : 缓存线程池，这样的线程池适合于有许多`short-lived`任务的程序。任务执行前先查看线程池中是否有当前执行线程的缓存，如果有就resue(复用),如果没有,那么需要创建一个线程来完成当前的调用。并且这类线程池内部规定能resue(复用)的线程，空闲的时间不能超过60s,一旦超过了60s,就会被移出线程池。
`SingleThreadExecutor` : 单例执行器，这样的`exector`保证了一个时刻只有一个线程执行。
`ScheduledThreadPool` : 调度型线程池，调度型线程池会根据Scheduled(任务列表)进行延迟执行，或者是进行周期性的执行.适用于一些周期性的工作。

## 创建线程池
- 创建固定型线程
``` java
ExecutorService service3 = Executors.newFixedThreadPool(10);
```
- 创建缓存型线程池
``` java
ExecutorService service2 = Executors.newCacheThreadPool();
```
- 创建调度型线程池
``` java
ExecutorService service4 = Executors.newScheduledThreadPool(10);
```

## Future 介绍
Future表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。Future的cancel方法可以取消任务的执行，它有一布尔参数，参数为 true 表示立即中断任务的执行，参数为 false 表示允许正在运行的任务运行完成。Future的 get 方法等待计算完成，获取计算结果。
``` java
package com.reapal.brave.main;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableAndFuture {

    public static class  MyCallable  implements Callable{
        private int flag = 0;
        public MyCallable(int flag){
            this.flag = flag;
        }
        public String call() throws Exception{
            if (this.flag == 0){
                return "flag = 0";
            }
            if (this.flag == 1){
                try {
                    while (true) {
                        System.out.println("looping.");
                        Thread.sleep(2000);
                    }
                } catch (InterruptedException e) {
                    System.out.println("Interrupted");
                }
                return "false";
            } else {
                throw new Exception("Bad flag value!");
            }
        }
    }

    public static void main(String[] args) {
        MyCallable task1 = new MyCallable(0);
        MyCallable task2 = new MyCallable(1);
        MyCallable task3 = new MyCallable(2);
        ExecutorService es = Executors.newFixedThreadPool(3);
        try {
            // 提交并执行任务，任务启动时返回了一个Future对象，
            // 如果想得到任务执行的结果或者是异常可对这个Future对象进行操作
            Future future1 = es.submit(task1);
            // 获得第一个任务的结果，如果调用get方法，当前线程会等待任务执行完毕后才往下执行
            System.out.println("task1: " + future1.get());
            Future future2 = es.submit(task2);
            // 等待5秒后，再停止第二个任务。因为第二个任务进行的是无限循环
            Thread.sleep(5000);
            System.out.println("task2 cancel: " + future2.cancel(true));
            // 获取第三个任务的输出，因为执行第三个任务会引起异常
            // 所以下面的语句将引起异常的抛出
            Future future3 = es.submit(task3);
            System.out.println("task3: " + future3.get());
        } catch (Exception e){
            System.out.println(e.toString());
        }
        // 停止任务执行服务
        es.shutdownNow();
    }
}
```

## execute与submit区别
- 接收的参数不一样
- submit有返回值，而execute没有
- submit方便Exception处理
- execute是Executor接口中唯一定义的方法；submit是ExecutorService（该接口继承Executor）中定义的方法

## 关闭线程池
- `shutdown()` : 不会立刻停止，只是表示停止接收任务，而等待线程池中正在执行的线程结束后，才关闭线程池
- `shutdownNow()` : 立刻关闭线程池，包括不接受任务，线程池中正在执行的任务也立刻停止。

## 参考链接
- [https://www.jianshu.com/p/edd7cb4eafa0](https://www.jianshu.com/p/edd7cb4eafa0)
- [https://docs.oracle.com/javase/tutorial/essential/concurrency/pools.html](https://docs.oracle.com/javase/tutorial/essential/concurrency/pools.html)
