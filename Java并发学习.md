[参考学习1](https://blog.csdn.net/weixin_41754415/article/details/89811919?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

[参考学习2](https://www.cnblogs.com/xll1025/p/11334921.html)

[参考文章3](https://blog.csdn.net/GavinZhera/article/details/86471828)

https://blog.csdn.net/weixin_41922289/article/details/103474635

https://blog.csdn.net/bskfnvjtlyzmv867/article/details/83005685



# 一、并发基础和多线程

## 1.1 线程与多线程

**1、线程是什么？**
线程（Thread）是一个对象（Object）。用来干什么？Java 线程（也称 JVM 线程）是 Java 进程内允许多个同时进行的任务。该进程内并发的任务成为线程（Thread），一个进程里至少一个线程。

Java 程序采用多线程方式来支持大量的并发请求处理，程序如果在多线程方式执行下，其复杂度远高于单线程串行执行。那么多线程：指的是这个程序（一个进程）运行时产生了不止一个线程。

**2、为啥使用多线程？**

- 适合多核处理器。一个线程运行在一个处理器核心上，那么多线程可以分配到多个处理器核心上，更好地利用多核处理器。
- 防止阻塞。将数据一致性不强的操作使用多线程技术（或者消息队列）加快代码逻辑处理，缩短响应时间。

3、聊到多线程，多半会聊**并发与并行**，咋理解并区分这两个的区别呢？

- 类似单个 CPU ，通过 CPU 调度算法等，处理多个任务的能力，叫并发
- 类似多个 CPU ，同时并且处理相同多个任务的能力，叫做并行

## 1.2 线程的创建与运行

的

# 二、JMM内存模型

# 三、synchronized volatile final等关键字

# 四、JUC包

# 五、实践

# 六、补充

