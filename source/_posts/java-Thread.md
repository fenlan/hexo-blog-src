---
title: java线程学习记录
tags: 线程
date: 2016-06-18 15:18:23
categories: Java
---

**在接触Android时候发现java基础没打好，所以回来重新打基础**

### java线程基础

<!--more-->

**java线程是在一个程序代码中CPU同步运行多个程序片段，而不是一行代码一行代码地执行到底，这样做是为了在运行大程序时提高程序的运行效率。虽然说的是同步运行，其实是“微观串行，宏观并行”，只是线程之间的切换时间太短以至于我们看上去是并行的。另外线程的调度模式有    （1）分时模型   （2）抢占模型**
**java用的是抢占模型，其他知识以后再补充，现在放代码**

> 重要提示：以下的代码的运行结果可能并不能代表多线程真正的运行顺序，因为本身线程运行时输出信号到显示器这段时间也是不可预计的，所以显示器上的显示顺序不完全代表运行代码时间先后。这个问题叫做`显示调度器不可预测`

**1. 继承`Thread`类，重写`run`函数**
``` java

class ThreadTest {
	public static void main (String[] args){
		MyThread thread1 = new MyThread("thread1");
		MyThread thread2 = new MyThread("thread2");
		thread1.start();
		thread2.start();
		System.out.println("The main runnnig is stopped");
	}
}

class MyThread extends Thread {
	public MyThread(String str){
		super(str);
	}
	public void run(){
		for (int i=0;i<3;i++){
			System.out.println(getName() + "is running");
			try {
				sleep(100);
			} catch (InterruptedException e) {}
		}
		System.out.println(getName() + "stopped");
	}
}

```

**运行结果如下**
``` bash
The main runnnig is stopped
thread2is running
thread1is running
thread1is running
thread2is running
thread1is running
thread2is running
thread2stopped
thread1stopped
```

**2. 实现`Runable`接口**

``` java
class TestSync implements Runnable {

    private int balance;

    @Override
    public void run() {
        for(int i = 0; i < 50; i++) {
            increament();
            System.out.println("balance is " + balance);
        }
    }

    private void increament() {
        int i = balance;
        balance = i + 1;
    }
}

public class TestSyncTest {

    public static void main(String[] args) {
        TestSync job = new TestSync();
        Thread a = new Thread(job);
        Thread b = new Thread(job);
        a.start();
        b.start();
    }
}
```

### 线程调度函数及代码实例
- `sleep(long millis)` : 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）
- `join()` : 在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

``` java
class MyThread extends Thread {
    public MyThread(String name) {
        super(name);
    }

    public void run() {
        int result = 0;
        System.out.println(this.getName() + "thread is started!");
        for (int i = 0; i < 50; i++ ) {
            result += i;
        }
        System.out.println(this.getName() + "thread is stopped!");
    }
}

public class ThreadTest {

    public static void main(String[] args) throws InterruptedException {

        System.out.println("Main thread is started!");
        MyThread a = new MyThread("A");
        a.start();
        a.join();
        System.out.println("Main thread is stopped!");
    }

}
```
输出结果:
``` bash
Main thread is started!
Athread is started!
Athread is stopped!
Main thread is stopped!
```

- `yield()` : `运行`-->`可运行`,让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

``` java
class MyThread extends Thread
{
    public void run()
    {
        for (int i=0; i<5 ; i++)
            System.out.println(Thread.currentThread().getName()
                    + " in control");
    }
}

// Driver Class
public class ThreadTest
{
    public static void main(String[]args)
    {
        MyThread t = new MyThread();
        t.start();

        for (int i=0; i<5; i++)
        {
            // Control passes to child thread
            Thread.yield();

            // After execution of child Thread
            // main thread takes over
            System.out.println(Thread.currentThread().getName()
                    + " in control");
        }
    }
}
```
可能的情况：
- main运行时让掉CPU，过后main抢到CPU继续执行
- main运行时让掉CPU，过后子线程抢到CPU继续执行

所以输出可能是
``` bash
Thread-0 in control
main in control
main in control
main in control
main in control
main in control
Thread-0 in control
Thread-0 in control
Thread-0 in control
Thread-0 in control
```
也可能是
``` bash
Thread-0 in control
Thread-0 in control
Thread-0 in control
Thread-0 in control
Thread-0 in control
main in control
main in control
main in control
main in control
main in control
```

### 生产者消费者
``` java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;

public class ProducerCustomer {

    public static void main(String args[]) {
        System.out.println("Solving Producer Consumper Problem");
        Queue<Integer> buffer = new LinkedList<>();
        int maxSize = 10;
        Thread producer = new Producer(buffer, maxSize, "PRODUCER");
        Thread consumer = new Consumer(buffer, maxSize, "CONSUMER");
        producer.start(); consumer.start();
    }
}

class Producer extends Thread {
    private Queue<Integer> queue;
    private int maxSize;
    public Producer(Queue<Integer> queue, int maxSize, String name) {
        super(name);
        this.queue = queue;
        this.maxSize = maxSize;
    }
    @Override public void run()
    {
        while (true)
        {
            synchronized (queue) {
                while (queue.size() == maxSize) {
                    try {
                        System.out .println("Queue is full, " + "Producer thread waiting for " + "consumer to take something from queue");
                        queue.wait();
                    } catch (Exception ex) {
                        ex.printStackTrace(); }
                }
                Random random = new Random();
                int i = random.nextInt();
                System.out.println("Producing value : " + i);
                queue.add(i);
                queue.notifyAll();
            }
        }
    }
}

class Consumer extends Thread {
    private Queue<Integer> queue;
    private int maxSize;
    public Consumer(Queue<Integer> queue, int maxSize, String name){
        super(name);
        this.queue = queue;
        this.maxSize = maxSize;
    }
    @Override public void run() {
        while (true) {
            synchronized (queue) {
                while (queue.isEmpty()) {
                    System.out.println("Queue is empty," + "Consumer thread is waiting" + " for producer thread to put something in queue");
                    try {
                        queue.wait();
                    } catch (Exception ex) {
                        ex.printStackTrace();
                    }
                }
                System.out.println("Consuming value : " + queue.remove());
                queue.notifyAll();
            }
        }
    }
}
```

### ABC交替打印
#### synchronized方式
``` java
public class ABC_Synch {
    public static class ThreadPrinter implements Runnable {
        private String name;
        private Object prev;
        private Object self;

        private ThreadPrinter(String name, Object prev, Object self) {
            this.name = name;
            this.prev = prev;
            this.self = self;
        }

        @Override
        public void run() {
            int count = 10;
            while (count > 0) {// 多线程并发，不能用if，必须使用whil循环
                synchronized (prev) { // 先获取 prev 锁
                    synchronized (self) {// 再获取 self 锁
                        System.out.print(name);// 打印
                        count--;

                        self.notifyAll();// 唤醒其他线程竞争self锁，注意此时self锁并未立即释放。
                    }
                    // 此时执行完self的同步块，这时self锁才释放。
                    try {
                        if (count == 0) {// 如果count==0,表示这是最后一次打印操作，通过notifyAll操作释放对象锁。
                            prev.notifyAll();
                        } else {
                            prev.wait(); // 立即释放 prev锁，当前线程休眠，等待唤醒
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        Object a = new Object();
        Object b = new Object();
        Object c = new Object();
        ThreadPrinter pa = new ThreadPrinter("A", c, a);
        ThreadPrinter pb = new ThreadPrinter("B", a, b);
        ThreadPrinter pc = new ThreadPrinter("C", b, c);

        new Thread(pa).start();
        Thread.sleep(10);// 保证初始ABC的启动顺序
        new Thread(pb).start();
        Thread.sleep(10);
        new Thread(pc).start();
        Thread.sleep(10);
    }
}
```

由交替打印代码可以得出`wait()`和`notify()`的区别
- wait() 与 notify/notifyAll() 是Object类的方法，在执行两个方法时，要先获得锁。
- 当线程执行wait()时，会把当前的锁释放，然后让出CPU，进入等待状态。
- 当执行notify/notifyAll方法时，会唤醒一个处于等待该 对象锁 的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域（synchronized修饰的代码块）后再释放锁。

**`notify/notifyAll()`执行后，并不立即释放锁，而是要等到执行完临界区中代码后，再释放。**

#### lock方式
``` java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ABC_Lock {
    private static Lock lock = new ReentrantLock();
    private static int state = 0;

    static class ThreadA extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10; ) {
                try {
                    lock.lock();
                    while (state % 3 == 0) {
                        System.out.print("A");
                        state++;
                        i++;
                    }
                } finally {
                    lock.unlock();	// 必须放在finally中
                }
            }
        }
    }

    static class ThreadB extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10; ) {
                try {
                    lock.lock();
                    while (state % 3 == 1) {
                        System.out.print("B");
                        state++;
                        i++;
                    }
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    static class ThreadC extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10; ) {
                try {
                    lock.lock();
                    while (state % 3 == 2) {
                        System.out.print("C");
                        state++;
                        i++;
                    }
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    public static void main(String[] args) {
        new ThreadA().start();
        new ThreadB().start();
        new ThreadC().start();
    }
}
```

#### synchronized 和 lock的区别
- `synchronized` : 在资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好，不管用没用过5.0多线程包的程序员都能理解。
- `lock(ReentrantLock)` : 提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。

我们写同步的时候，优先考虑synchronized，如果有特殊需要，再进一步优化。ReentrantLock如果用的不好，不仅不能提高性能，还可能带来灾难。