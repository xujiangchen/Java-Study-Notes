- [1、serial垃圾收集器](#1-serial-----)
- [2、serial old 垃圾收集器](#2-serial-old------)
- [3、PerNew垃圾收集器](#3-pernew-----)
- [4、Parallel Scavenge收集器](#4-parallel-scavenge---)
    + [4.1 Parallel Scavenge 是什么？](#41-parallel-scavenge-----)
    + [4.2 Parallel Scavenge 应用场景](#42-parallel-scavenge-----)
    + [4.3 Parallel Scavenge 的参数设置](#43-parallel-scavenge------)
- [5、Parallel Old收集器](#5-parallel-old---)
- [6、CMS收集器](#6-cms---)

## 1、serial垃圾收集器
- serial 回收器是最基础，历史最悠久的收集器，用于**新生代**
- Serial是一个**单线程**的垃圾收集器

**特点**

1. “Stop The World”，它进行垃圾收集时，必须暂停其他所有的线作线程，直到它收集结束。
2. 多用于桌面应用，**Client端**的垃圾回收器。桌面应用（Client端应用）内存小，进行垃圾回收的时间比较短，只要不频繁发生停顿就可以接受
3. 使用复制算法完成垃圾清理工作。

**缺点**

1. 垃圾回收速度较慢且回收能力有限

![serial垃圾收集器](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/serial%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.jpg)

## 2、serial old 垃圾收集器
- Serial Old是 Serial收集器的老年代版本
- 针对老年代
- 采用"标记-整理"算法
- 单线程收集
- 主要用于Client模式

## 3、PerNew垃圾收集器
-  ParNew垃圾收集器是Serial收集器的**多线程**版本。**除了多线程外，其余的行为、特点和Serial收集器一样**。
- 可以**使用-XX: ParallelGCThreads参数**来限制垃圾收集的线程数。一般设置和CPU的核数相等即可，但也不用太多，会产生线程上下文切换的时耗。

![PerNew垃圾收集器](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/parnew%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.jpg)

## 4、Parallel Scavenge收集器

#### 4.1 Parallel Scavenge 是什么？
Parallel Scavenge收集器是一个**新生代收集器**，它也是使用**复制算法**的收集器，也是**并行的多线程**收集器。

由于与吞吐量关系密切，Parallel Scavenge收集器也经常称为“吞吐量优先”收集器

> 什么是吞吐量：吞吐量是什么？CPU用于运行用户代码的时间与CPU总时间的比值，99%时间执行用户线程，1%时间回收垃圾 ，这时候吞吐量就是99%。

#### 4.2 Parallel Scavenge 应用场景
- 场景一： 高吞吐量为目标，即减少垃圾收集时间（就是每次垃圾收集时间短，但是收集次数多），让用户代码获得更长的运行时间。
- 场景二：当应用程序运行在具有多个CPU上，对暂停时间没有特别高的要求时，即程序主要在后台进行计算，而不需要与用户进行太多交互；就是说可以计算完后进行一次长时间的GC。

#### 4.3 Parallel Scavenge 的参数设置

（A）、"-XX:MaxGCPauseMillis"

控制最大垃圾收集停顿时间，大于0的毫秒数；MaxGCPauseMillis设置得稍小，停顿时间可能会缩短，但也可能会使得吞吐量下降；因为可能导致垃圾收集发生得更频繁。

（B）、"-XX:GCTimeRatio"

设置垃圾收集时间占总时间的比率，0＜n＜100的整数；GCTimeRatio相当于设置吞吐量大小； 垃圾收集执行时间占应用程序执行时间的比例的计算方法是： 

```
1 / (1 + n)

例如，选项-XX:GCTimeRatio=19，设置了垃圾收集时间占总时间的5%--1/(1+19)

默认值是1%--1/(1+99)，即n=99；
```

## 5、Parallel Old收集器
- Parallel Old垃圾收集器是Parallel Scavenge收集器的老年代版本
- JDK1.6中才开始提供，JDK1.6及之后用来代替老年代的Serial Old收集器
- 采用"标记-整理"算法
- 多线程收集
- "-XX:+UseParallelOldGC"：指定使用Parallel Old收集器

## 6、CMS收集器
> 集中在互联网站或者B/S系统的服务端上

CMS收集器：Mostly-Concurrent收集器，也称并发标记清除收集器（Concurrent Mark-Sweep GC，CMS收集器），**它管理新生代的方式与Parallel收集器和Serial收集器相同，而在老年代则是尽可能得并发执行**，每个垃圾收集器周期只有==2次==短停顿。

**步骤**
1. 初始标记（CMS initial mark)-----标记⼀下GC Roots 能直接关联到的对象，速度很快 
2. 并发标记（CMS concurrent mark --------并发标记阶段就是进行 GC RootsTracing（跟搜索算法） 的过程
3. 重新标记（CMS remark) -----------为了修正并发标记期间因⽤户程序导致标记产生变动的标记记录
4. 并发清除（CMS concurrent sweep)

**缺点**
- 对CPU资源非常敏感
- 无法处理浮动垃圾，程序在进行并发清除阶段⽤户线程所产生的新垃圾叫做浮动垃圾
- 标记-清除算法会产生空间碎片

![CMS收集器](https://github.com/xujiangchen/Java-Study-Notes/blob/main/JVM/asset/cms%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.jpg)

