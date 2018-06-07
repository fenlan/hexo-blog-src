---
title: Java volatitle
date: 2018-05-03 15:18:23
categories: Java
tags:
  - Java
  - 线程
  - Happens-Before
---

## Java Volatile Keyword
Java `volatitle`关键字用来标记变量"被存储在主存里"，更精确的说，`volatitle`变量的每一次读取都需要从计算机的主存读取而不是`CPU Cache`，而每一次写入都是直接写入主存，而不是`CPU Cache`。从Java 5开始`volatitle`关键字不仅仅是保证写入主存和从主存读取。

## 变量可见性问题
Java `volatitle`保证了变量跨线程的可见性，这么说可能有点抽象，so let me elaborate。

在线程操作`non-volatile`变量的多线程应用中，由于性能原因每个线程会从主存中拷贝变量到CPU Cache(each thread may copy variables from main memory into a CPU cache while working on them)。如果你的计算机是多CPU的，每个线程可能跑在不同的CPU上，这意味着每个线程会拷贝变量到不同CPU的Cache中，就像这样:
![](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-1.png)

<!--more-->

对于JVM来说，从主存读取值到CPU Cache和从CPU Cache写入数据到主存，`non-volatile variables`没有上述的保证，会引出一些问题。

设想一种情况，两个或更多线程可以访问包含一个计数器变量的共享对象，声明如下:
``` java
public class SharedObject {

    public int counter = 0;

}
```
再想象一下，只有Thread 1 increments the `counter` variable，但是Thread 1 和Thread 2都会一遍一遍的读取`counter` 变量。如果`counter`变量没有`volatitle`修饰，那么就不保证`counter`变量的值何时从CPU Cache写入主存。这就意味着在CPU Cache中`counter`的值与主存里的值可能不一样，就像这样:
![](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-2.png)

这个问题就是线程不能看到变量的最新值，因为在另一个线程中没有将变量最新值写入到主存中，这个问题叫做`可见性`问题。一个线程的更新对另一个线程不可见。

## Java volatitle可见性保证
Java关键字`volatitle`旨在解决变量可见性问题。定义了`counter`变量`volatitle`后线程的写入操作会立刻写入到主存，同时所有的读取都是直接从主存读取。下面就是`volatitle`定义`counter`变量:
``` java
public class SharedObject {

    public volatile int counter = 0;

}
```
定义了`volatitle`后就保证了变量的可见性，在上述的情况下，线程1修改`counter`，线程2读取`counter`，对于线程2来说它能够见到线程1的最新修改。但是当两个线程都要修改`counter`的时候，定义`volatitle`显然不够了。

## Full volatile Visibility Guarantee
Full保证如下:
- 如果线程A写入一个`volatitle`变量，并且线程B随后读取相同的`volatitle`变量，则在写入`volatitle`变量之前，线程A可见的所有变量在线程B读取`volatitle`变量后也将可见。
- 如果线程A读取一个`volatitle`变量，则读取`volatitle`变量时线程A可见的所有变量也将从主内存中重新读取。
> 可以看到，这个开销是比较恐怖的，首先直接读主存和写主存原本就很慢，慢到几十倍甚至几百倍，同时一个volatitle变量读取会带动其他所有volatitle重新读取，大大降低了性能。

来看一个例子：
``` java
public class MyClass {
    private int years;
    private int months
    private volatile int days;


    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```
`update()`方法会写入三个变量，其中只有`days`是`volatitle`，Full volatitle保证意味着，当写入`days`时，所有的变量都会写入主存。在这个例子中就是，当我们要写入`days`时，`years`和`months`也要写入主存。当要读取`years`、`months`和`days`时，你可以像这样做:
``` java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```
注意`totalDays()`方法在开始时读取`days`到`total`变量。当读取`days`值时，`years`和`months`也会从主存中读取，因此你可以通过上面的读取步骤读到所有变量的最新值。

## 指令重排
为了性能，Java VM和CPU是允许重排程序中的指令的，只要语义含义保持不变。举个例子，看看下面的指令：
``` java
int a = 1;
int b = 2;

a++;
b++;
```
在不丢失语义含义的情况下，上述指令可能被重排成下列指令序列：
``` java
int a = 1;
a++;

int b = 2;
b++;
```
但对于`volatitle`变量来说，指令重排是一个挑战，让我们来看一下上述MyClass的例子
``` java
public class MyClass {
    private int years;
    private int months
    private volatile int days;


    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```
当写入`days`时，也会同时写入`years`和`months`到主存。但是如果Java VM重排指令如下：
``` java
public void update(int years, int months, int days){
    this.days   = days;
    this.months = months;
    this.years  = years;
}
```
当`days`修改时，`months`和`years`也会写入主存，但是这次`days`写入发生在`years`和`months`之前，因此这个新值对其他线程不可见，这个语义含义已经被改变了。
> 特别注意，在写入时，把`volatitle`变量写入放在最后，这样当运行到`volatitle`变量写入时，会同时把其他非`volatitle`写入主存，但当`volatitle`写在最开始时，后面的非`volatitle`便不会写入最新值。读取相反，将`volatitle`读取放在最前，线程会读取所有值最新的主存值，如果放在最后面，那么前面读取的变量不是从主存直接读取的。

## Happens-Before
为了解决指令重组问题，Java `volatitle`关键字除了可见性保证，还给了一个`happens-before`保证，`happens-before`保证如下：
- 如果读写发生在写入`volatitle`变量之前，那么读写其他变量的指令不能被重组到写入`volatitle`变量之后。写入`volatitle`变量之前的读写被保证`happens before`写`volatitle`变量。需要注意，写入`volatitle`变量之后的读写其他变量仍然有可能被重组到写入`volatitle`变量之前。因此**From after to before is allowed, but from before to after is not allowed**（`volatitle`写操作之后的指令被重组到写之前是允许的，而写操作之前的指令不允许被重组到写之后）
- 读`volatitle`变量之后的读写操作不能被重组到`volatitle`之前，但同样注意发生在读`volatitle`之前的指令可以重组到读`volatitle`之后。因此**From before to after is allowed, but from after to before is not allowed**(`volatitle`读之前的指令被重组到读之后值允许的，而读操作之后的指令不允许被重组到读值前)

> Reads from and writes to other variables cannot be reordered to occur after a write to a volatile variable, if the reads / writes originally occurred before the write to the volatile variable.
The reads / writes before a write to a volatile variable are guaranteed to "happen before" the write to the volatile variable. Notice that it is still possible for e.g. reads / writes of other variables located after a write to a volatile to be reordered to occur before that write to the volatile. Just not the other way around. From after to before is allowed, but from before to after is not allowed

> Reads from and writes to other variables cannot be reordered to occur before a read of a volatile variable, if the reads / writes originally occurred after the read of the volatile variable. Notice that it is possible for reads of other variables that occur before the read of a volatile variable can be reordered to occur after the read of the volatile. Just not the other way around. From before to after is allowed, but from after to before is not allowed

重点解释一下这两个保证：
对于`volatitle`写来说，如果写之前的指令被重组到写之后，那么当执行到`volatitle`时，最新值就不能被立刻写入主存；但写之后的指令被重组到写之前是没有影响的，因为写之后的指令最终还是要被写到主存，只是重组之后提前将新值写入主存。
对于`volatitle`读来说，如果读之后的指令被重组到读之前，那么指令没有获取最新值，因为只有到`volatitle`读执行时才重读所有变量的主存值，比如当`years`读取发生`volatitle days`之后，那么执行到`volatitle days`会刷新`years`的值，但`years`被重组到`volatitle days`之前，`years`的值没有刷新；相反，如果读取`years`在`volatitle days`之前，被重组到之后，它会读到最新值，这正是我们希望的结果。

## volatile is Not Always Enough
当多个线程都需要对一个`volatitle`变量进行写入，且写入之前需要依赖`volatitle`之前的值，这时候就会出现竞争条件：

想象一下，如果线程1读取一个`shared  counter`值到自己CPU Cache中，开始值为0，然后`increment`该值为1，但没有把改变的值写入主存；然后线程2也从主存中读取`counter`，由于线程1没有将改变的值写入主存，此时线程2读到的值为0，它也对值进行`increment`后值变为1，此时线程2也没有将值写入主存，就像这样:
![](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-3.png)

线程1和线程2几乎不同步，而我们希望的是进行两次加操作后值为2，但线程1和线程2的CPU Cache中`counter`都是1，即使我们将它们的值写入到主存，结果仍然不对，这都是**volatile is Not Always Enough**

## When is volatile Enough?
既然出现了上诉情况，那么如果解决这个问题？之前`volatitle`已经明显不够，那么这时候可以使用Java另外一个关键字`synchronized`去保证读取变量和写入变量是原子性的操作。上诉的`volatitle`读写操作并不会产生线程阻塞，使用`synchronized`保证临界区读写不出现上述问题，但这样会导致线程阻塞。另外也可以使用`java.util.concurrent package`里面的`AtomicLong` or `AtomicReference`或者其他。

在这样的情况下，只有一个线程允许对`volatitle`变量进行读写操作，其他线程只允许读取，这样即保证读取最新的值，也保证直接写入主存。但是光有`synchronized`并不能保证可见性。

## 性能问题
之前已经说过，`volatitle`的读写都要直接接触主存，那么与读写CPU Cache相比，开销增大许多倍，放一张计算机各级缓存以及主存的读取速度比较:
![](http://www.qdpma.com/SystemArchitecture_files/CPU_4c.png)

## 参考链接
- [Java Volatile Keyword](http://tutorials.jenkov.com/java-concurrency/volatile.html)