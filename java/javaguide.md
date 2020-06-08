# JavaGuide

> git地址：https://snailclimb.gitee.io/javaguide
>
> github地址：https://github.com/Snailclimb/JavaGuide



### Java并发编程

#### 进程和线程的定义

进程是程序的一次执行过程，是系统运行程序的基本单元，系统运行一个程序就是进程的创建、运行和销毁的过程。

线程也被称为是轻量级进程，是比进程更小的执行单位，一个进程可以创建多个线程，多个线程共享堆和方法区资源，但又有各自的**程序计数器、虚拟机栈和本地方法栈**。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F.png)



