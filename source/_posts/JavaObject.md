---
title: Java Object
date: 2018-04-28 15:18:23
categories: Java
tags:
  - Java
  - Object
  - 线程
---

## Obejct简介
Object类是Java中所有类的基类，在编译时会自动导入，位于java.lang包中，而Object中具有的属性和行为，是Java语言设计背后的思维体现。Object类中的大部分方法都是native方法，用此关键字修饰的方法是Java中的本地方法，一般是用C/C++语言来实现。

## native 关键字
native是与C\++联合开发的时候用的！java自己开发不用的！使用native关键字说明这个方法是原生函数，也就是这个方法是用C/C\++语言实现的，并且被编译成了DLL，由java去调用。 这些函数的实现体在DLL中，JDK的源代码中并不包含，你应该是看不到的。对于不同的平台它们也是不同的。这也是java的底层机制，实际上java就是在不同的平台上调用不同的native方法实现对操作系统的访问的。native 是用做java 和其他语言（如c++）进行协作时用的 也就是native 后的函数的实现不是用java写的。native的意思就是通知操作系统， 这个函数你必须给我实现，因为我要使用。 所以native关键字的函数都是操作系统实现的， java只能调用。java是跨平台的语言，既然是跨了平台，所付出的代价就是牺牲一些对底层的控制，而java要实现对底层的控制，就需要一些其他语言的帮助，这个就是native的作用了。
<!--more-->

## Object 构造函数
大部分情况下，Java中通过形如 new A(args..)形式创建一个属于该类型的对象。其中A即是类名，A(args..)即此类定义中相对应的构造函数。通过此种形式创建的对象都是通过类中的构造函数完成。为体现此特性，Java中规定：在类定义过程中，对于未定义构造函数的类，默认会有一个无参数的构造函数，作为所有类的基类，Object类自然要反映出此特性，在源码中，未给出Object类构造函数定义，但实际上，此构造函数是存在的。

当然，并不是所有的类都是通过此种方式去构建，也自然的，并不是所有的类构造函数都是**public**

## registerNatives()
``` java
private static native void registerNatives();
static {
    registerNatives();
}
```
该方法主要作用是将C/C\++中的方法映射到Java中的native方法，实现方法命名的解耦。registerNatives的修饰为private，因此该方法只在Object中使用，而接下来的static代码块在类加载的时候加载调用registerNatives方法。可以理解为这是Java在为使用底层做准备。

## getClass()
``` java
public final native Class<?> getClass();
```
该方法返回该对象的Class，同时可以直接通过`Object.class`来返回相同结果。例如：
``` java
package com.fenlan.base;
public class Test {
    public static void main(String[] args) {
        System.out.println(Test.class);
    }
}
```
该结果返回`class com.fenlan.base.Test`，这是这个对象运行时class

## hashCode()
``` java
public native int hashCode();
```
该方法返回对象的hash整型码值，关于hashCode()方法的介绍，在[这里有深入说明](http://fenlan.github.io/2018/03/01/Java%E5%B0%8F%E8%AE%B0/)，为对象设置hash值是为了支持hash tables 诸如HashMap、HashSet。hashCode()和equals()方法是经常被重写的方法，具体的重写准则如下:
- 在Java应用程序程序执行期间，对于同一对象多次调用hashCode()方法时，其返回的哈希码是相同的，前提是将对象进行equals比较时所用的标尺信息未做修改。在Java应用程序的一次执行到另外一次执行，同一对象的hashCode()返回的哈希码无须保持一致；
- 如果两个对象相等（依据：调用equals()方法），那么这两个对象调用hashCode()返回的哈希码也必须相等；
- 反之，两个对象调用hasCode()返回的哈希码相等，这两个对象不一定相等。程序员应该意识到，为不相等的对象生成不同的整数结果可能会提高散列表的性能。

## equals()
``` java
public boolean equals(Object obj) {
    return (this == obj);
}
```
`==`与`equals`在Java中经常被使用，大家也都知道==与equals的区别：==表示的是变量值完成相同(对于基础类型，地址中存储的是值，引用类型则存储指向实际对象的地址)；equals表示的是对象的内容完全相同，此处的内容多指对象的特征/属性。

那么问题来了，Object里面调用了`==`那为什么还需要`equals`，equlas()方法的正确理解应该是：判断两个对象是否相等。那么判断对象相等的标尺又是什么？Object判断相等是两个引用具有相同对象地址，但其他情况下我们不需要这么严格的要求，诸如两个类只要某些属性相同就可以认为是相等的，那么就需要重写`equals`。

## clone()
``` java
protected native Object clone() throws CloneNotSupportedException;
```
克隆方法是返回一个当前对象的拷贝，而这里的拷贝精确定义需要依赖这个对象的类定义。

## toString()
``` java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
返回一个表示当前对象的字符串，通过这个这个字符串可以简单的表示一些对象信息，为了更多的展示对象信息，可以选择重写该方法。默认情况下返回当前对象的类名和十六进制的hashCode。

## notify()
``` java
public final native void notify();
```
该方法唤醒一个等待在`this object's monitor`的单线程。如果有多个线程等待这个对象，那么唤醒其中一个。而`Object's monitor`中的线程是之前调用了`wait`方法。此外其他的说明参考源码注释。

> Wakes up a single thread that is waiting on this object's monitor. If any threads are waiting on this object, one of them is chosen to be awakened. The choice is arbitrary and occurs atthe discretion of the implementation. A thread waits on an object's monitor by calling one of the {@code wait} methods.

> This method should only be called by a thread that is the owner of this object's monitor. A thread becomes the owner of the object's monitor in one of three ways:
> 1. By executing a synchronized instance method of that object.
> 2. By executing the body of a {@code synchronized} statement that synchronizes on the object.
> 3. For objects of type {@code Class,} by executing a synchronized static method of that class.

> Only one thread at a time can own an object's monitor.

## notifyAll()
``` java
public final native void notifyAll();
```
该方法唤醒所有等待在`Object's monitor`的线程。具体的`notify`和`notifyAll`参考其他文章。

## wait()
``` java
public final native void wait(long timeout) throws InterruptedException;
```
让当前线程等待直到其他线程调用`notify()`并选择当前对象或者`notifyAll()`又或者指定等待时间过去。此外，一个线程在等待过程中被中断，会抛出中断异常。

## finalize()
``` java
protected void finalize() throws Throwable { }
```
在垃圾回收时确定没有引用在这个对象上时会调用这个方法。子类重写该方法决定如何去清理这个对象。

## 参考链接
- [Java-Object类源码解析](https://blog.csdn.net/benjaminlee1/article/details/72843713)
- [java native关键字](https://blog.csdn.net/youjianbo_han_87/article/details/2586375)
- [java object类详解](https://zhuanlan.zhihu.com/p/29511703)