# 1、JVM基础知识点

##  1、JVM内存模型的概述

![JVM内存模型的概述](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/JVM%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E7%9A%84%E6%A6%82%E8%BF%B0.jpg)

**线程共享数据区：方法区、堆**

**线程隔离数据区：虚拟机栈、本地方法栈、程序计数器**

## 2、程序计数器

#### 2.1 什么是程序计数器?

程序计数器是当前线程正在执行的字节码的地址。程序计数器是线程隔离的，每一个线程在工作的时候都有一个独立的计数器。**它可以看作是当前线程所执行的字节码的序号号指示器**。

#### 2.2 字节码的执行原理

编译后的字节码在没有经过JIT(实时编译器)编译前，是通过字节码解释器进行解释执行。其执行原理为：字节码解释器读取内存中的字节码，按照顺序读取字节码指令，读取一个指令就将其翻译成固定的操作，根据这些操作进行分支，循环，跳转等动作。

#### 2.3 程序计数器的作用?

从字节码的执行原理来看，**单线程的情况下程序计数器是可有可无的**。因为即使没有程序计数器的情况下，程序会按照指令顺序执行下去，即使遇到了分支跳转这样的流程也会按照跳转到指定的指令处继续顺序执行下去，是完全能够保证执行顺序的。

**但是现实中程序往往是多线程协作完成任务的**。JVM的多线程是通过CPU时间片轮转来实现的，某个线程在执行的过程中可能会因为时间片耗尽而挂起。当它再次获取时间片时，需要从挂起的地方继续执行。在JVM中，通过程序计数器来记录程序的字节码执行位置。**程序计数器具有线程隔离性，每个线程拥有自己的程序计数器**

#### 2.4 程序计数器的特点

(1)程序计数器具有线程隔离性

(2)程序计数器占用的内存空间非常小，可以忽略不计

(3)程序计数器是java虚拟机规范中唯一一个没有规定任何OutofMemeryError(内存溢出)的区域，原因jvm自行维护，程序员不允许操作，所以不会出现om的情况

(4)程序执行的时候，程序计数器是有值的，其记录的是程序正在执行的字节码的地址

(5)执行native本地方法时，程序计数器的值为空。原因是native方法是java通过jni调用本地C/C++库来实现，非java字节码实现，所以无法统计

**实战**
```java
public class User{
	
	String name;
	
	public String getName() { 
		return name;
	}
	public void setName(String name){
		this.name = name;
	}
}
```

![程序计数器](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/%E7%A8%8B%E5%BA%8F%E8%AE%A1%E6%95%B0%E5%99%A8.jpg)

## 3、JAVA虚拟机栈

- 栈是一个先进后出，后进先出的数据结构

#### 3.1 什么是JAVA虚拟机栈?

作用于方法执行的Java内存区域。它的生命周期与线程相同（**随线程而生，随线程而灭**）

#### 3.2 虚拟机栈原理

每个方法在执行的同时都会创建多个栈帧（StackFramel）用于存储局部变量表、操作数、栈、动态链接等信息。**每一个方法从调用直到执行完成的过程，就对应着一个栈帧在虚拟机栈中进栈到出栈的过程。**

#### 3.3 易出现的问题

- 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出**StackOverflowError异常**
- 如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够的内存，就会抛出**OutOfMemoryError异常**；当前大部分JVM都可以动态扩展，只不过JVM规范也允许固定长度的虚拟机栈

**实战**

```
public class A {
    public native static void c();
    
    public static void a(){
        System.out.println("enter method a");
    }
    
    public static void b(){
        a();
        System.out.println("enter method b");
    }
    
    public static void main(String[] args) {
        b();
        System.out.println("enter method main");
    }
}
```

![虚拟机栈](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88.jpg)

## 4、本地方法栈

#### 4.1 什么是本地方法栈?

本地方法栈（Native MethodStacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。

#### 4.2 什么是Native方法

Native关键字修饰的方法叫做本地方法，本地方法和其它方法不一样，本地方法意味着==和平台有关，因此使用了native的程序可移植性都不太高。被Native关键字声明的方法说明该方法不是以Java语言实现==的，而是以本地语言实现的，Java可以直接拿来用，例如，某些dll动态链接库文件。

#### 4.3 本地方法栈和Java虚拟机栈的异同

其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。**与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常**。

#### 4.4 注意点

虽然jvm规定本地方法栈和Java虚拟机栈是分开的，但是本地方法栈和Java虚拟机栈结构和功能相似，Hotshot将Java虚拟机栈和本地方法栈合二为一。

> HotSpot虚拟机是SunJDK和OpenJDK中所带的虚拟机，也是目前使用范围最广的Java虚拟机。HotSpot只是jvm实现的一种方式而已

![Hotshot](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/Hotshot.jpg)

## 5、Java堆

#### 5.1 什么是Java堆

是Java内存区域中一块用来存放对象实例的区域，**几乎所有的数组和对象实例都在这里分配内存**。

#### 5.2 Java堆的特点

(1)Java 中的堆是 JVM 所管理的最大的一块内存空间

(2)Java 堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC 堆”(Garbage Collection)

(3)Java堆可以分成新生代和老年代，新生代又可分为To Space、From Space、Eden,这样划分的目的是为了使JVM能够更好的管理堆内存中的对象，包括内存的分配以及回收。
> 老年代 ： 三分之二的堆空间，年轻代 ： 三分之一的堆空间 eden区： 8/10 的年轻代空间 survivor0:1/10的年轻代空间survivor1:1/10的年轻代空间

>在JDK1.8版本废弃了永久代，替代的是元空间（MetaSpace），元空间与永久代上类似，都是方法区的实现，他们最大区别是：元空间并不在JVM中，而是使用本地内存。

堆的内存模型大致为(以jdk1.8为例)：

![java1.8堆内存模型](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/1.8%E5%A0%86%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.jpg)
**实战**
![java堆](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/java%E5%A0%86.jpg)
