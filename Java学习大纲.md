借鉴于[aka面团人博客](https://www.nowcoder.com/discuss/365165)

# 一、Java基础知识

## 1、String

+ String、StringBuffer、StringBuilder


|   String   | StringBuffer | StringBuilder |
| :--------: | :----------: | :-----------: |
| 线程不安全 |   线程安全   |  线程不安全   |
| 不可变对象 |   可变对象   |   可变对象    |
|   速度快   |    速度慢    |    速度快     |

``` java
String s1 = new String("aaa");
String s2 = "aaa";
String s3 = new String("aaa");
String s4 = "aaa";
```

其中，s2与s4的对象实例在堆中属于同一片内存，s1、s2、s3的对象实例在堆中不属于同一个内存，所以在用==判等时返回false，若用equals方法返回true，equals比较的是字面量，==比较的是内存地址。

String s1 = new String("aaa")的具体过程[参考这篇文章](https://www.zhihu.com/question/29884421/answer/113785601)

## 2、Java核心数据结构，Java容器

### 2.1 List

+ ArrayList、LinkedList、vector、CopyOnWriteArrayList
+ CopyOnWriteArrayList原理

### 2.2 Map

+ Map实现
+ HashMap、HashTable、LinkedHashMap、TreeMap、ConcurrentHashMap的区别及实现原理
+ HashMap的原理、底层、扩容机制 [参考文章](https://www.iteye.com/blog/zhangshixi-672697)
  + 在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失。
  + 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。
+ HashTable的原理、底层、扩容机制
+ ConcurrentHashMap的原理、底层

### 2.3 Set

+ Set的实现
+ HashSet、LinkedHashSet、TreeHashSet的区别及原理

### 2.4 容器中的设计模式

+ 迭代器模式：将元素的实现与遍历分开
+ 适配器模式：java.util.Arrays可以转换为List类型

## 3、零散知识点

+ 封装、继承、多态（面向对象编程的三大特性）
+ 重载与重写的区别
  + 重写是父类与子类之间多态性的表现，在运行时起作用（动态多态性，譬如实现动态绑定）
  + 重载是一个类中多态性的表现，在编译时起作用（静态多态性，譬如实现静态绑定）
+ final、finally、finalize
+ static关键字
+ i++与++i
+ equals()与==的区别
  + equals比较的是值valueOf，==是对引用的比较
+ 重写equals()方法为什么要重写hashCode()方法
  + 使用hashcode方法提前校验，可以避免每一次比对都调用equals方法，提高效率
  + 保证是同一个对象，如果重写了equals方法，而没有重写hashcode方法，会出现equals相等的对象，hashcode不相等的情况，重写hashcode方法就是为了避免这种情况的出现。
+ this和super的区别：
  + 代表事物不同：this表示所属函数的调用者对象，super表示父类空间的引用
  + 使用前提不同：this不需要继承关系，super需要继承关系才能使用
  + 调用的构造函数不同：super调用父类的构造函数，this调用所属类的构造函数
+ 自动装箱与自动拆箱
  + 装箱：将基本数据类型转为包装器类型(有一个IntegerCache，cache的界限可通过JVM参数设定)
  + 拆箱：将包装器类型转为基本数据类型
+ 异常
+ Java中throw与throws的区别

# 二、多线程

1. 线程与进程的区别 [参考文章](https://blog.csdn.net/qq_37791134/article/details/81516023)
2. 多线程的实现（Thread、Runnnable、callable+futuretask、线程池四种方法的区别、实现原理、适用场景）
3. 线程的启动（start()和run()的区别）
4. 线程传递参数（1、向run()方法传参；2、获取子线程的返回值）
5. Runnable和callable的区别
6. 线程状态（新建、执行、无限等待、限期等待、阻塞、终止6种状态变换过程）
7. sleep()和wait()区别
8. notify()与notifyAll()
9. yield()
10. 线程的停止
11. 线程之间的转换
12. 线程不安全的原因
13. synchronized关键字
14. synchronized和reentrantLock的对比、区别、适用场景
15. 锁的种类：公平锁与非公平锁、乐观锁与悲观锁、独享锁与共享锁、可重入锁、分段锁、锁的状态（无锁、偏向锁、轻量级锁、重量级锁、自旋锁、适应自旋锁）相关知识点和对比
16. CAS（compare and swap）原理
17. AQS（AbstractQueuedSynchronized）原理
18. 死锁含义、成立条件
19. 如果有效避免死锁
20. 银行家算法
21. volatile关键字
22. 线程之间的通信
23. Java线程池
24. 为什么使用Java线程池
25. 线程池的创建、核心参数
26. 线程池种类
27. 线程池任务执行方法（submit()、execute()两种的区别）
28. 线程池关闭
29. 线程池常用队列

# 三、Spring

1. Spring概述
2. AOP
3. AOP的实际例子
4. AOP的**模式
5. IOC
6. DI（依赖注入）
7. Bean的Scope作用域
8. Bean的生命周期
9. Spring的事务
10. Spring事务隔离级别
11. SpringMVC
12. Spring的过程

# 四、数据库

1. 范式（最好记住几个例子）
2. 索引（索引的含义、优缺点）
3. 索引的数据结构
4. 索引为什么使用B+树不用B树
5. 索引的分类
6. 聚簇索引与非聚簇索引
7. 建立索引的考虑因素
8. 使用索引的注意事项
9. 事务（ACID）
10. 多事务并发造成的问题（脏读、不可重复读、幻读）
11. 事务隔离性
12. 读锁与写锁
13. 高并发控制数据库
14. MySQL的工具分析
15. MyBatis一级缓存
16. MyBatis二级缓存
17. MyBatis中#与$的区别
18. 高并发数据库设计（百万级数据库设计）
19. 数据库优化
20. 主从复制
21. MySQL数据引擎的选用
22. 数据库连接池

# 五、计算机网络

[参考文章](https://blog.csdn.net/qq_39322743/article/details/79700863)

1. OSI七层模型、五层模型、TCP/IP四层模型
2. 三次握手（为什么不是两次、四次）
3. 四次挥手（为什么不是三次）
4. 保活计时器
5. 确认应答机制（ACK）
6. 滑动窗口
7. 超时重传机制（去重机制、快重传）
8. 流量控制
9. 拥塞控制（慢启动、拥塞窗口）
10. TCP可靠性保证的机制
11. TCP与UDP的区别
12. Post和get的区别
13. http与https
14. http常见返回码
15. http请求过程
16. 怎么防止http请求的攻击
17. 对称加密与非对称加密
18. cookie与session的区别

# 六、JVM

1. JVM组成（类加载器、运行时数据区、解释器、本地接口各自的含义与作用）
2. Java程序在JVM中的执行过程
3. 类加载的时机
4. 类加载器的种类（Bootstrap、Extension、APP、自定义四个类加载器的工作责任）
5. 类加载器的工作过程（双亲委派机制）
6. 类加载的详细过程（**加载、验证、准备、解析、初始化**、使用、卸载）
7. JVM内存模型（程序计数器、堆、虚拟机栈、本地方法栈、方法区）
8. GC（垃圾回收机制）
9. 判断对象存活的方法（引用计数法和可达性分析的区别）
10. 可达性分析中可以作为GC Roots的对象
11. 垃圾回收算法（标记-清除算法、复制算法、标记-整理算法、分代收集算法）
12. GC时间
13. 什么情况下出现内存溢出、内存泄漏
14. GC回收器
15. 堆中分配内存TLAB [参考文章](https://www.jianshu.com/p/8be816cbb5ed)

# 七、手撕代码

1. 线程安全的单例模式（多种写法）[参考代码](https://blog.csdn.net/absolute_chen/article/details/93380566)
   + 懒汉
   + 饿汉
   + 双检锁
   + 枚举
   + 静态内部类
2. 排序算法
   + 冒泡及优化
   + 快速排序（三种）及基准点选择优化
   + 堆排序（大顶堆升序，小顶堆降序）
   + 选择排序（选择极值）
   + 插入排序（认为开始的序列有序，后一个数字为插入数字并使得数组有序）
   + 归并排序（先分再和）
   + 桶排序
   + 希尔排序
3. 二叉树的前中后序遍历（递归与非递归）
   + 递归法
     + 前序：根左右
     + 中序：左根右
     + 后序：左右根
   + 非递归
     + 前序：用栈实现，push(root)->pop(root)->push(right)->push(left)
     + 中序：用栈实现，push(root)->push(left....left)->pop(root)->(cur = cur.right)
     + 后序：后序打印与逆后序打印的顺序相反，反转结果即可（双栈法或者Collections.reverse）
4. 多个线程交替打印（生产者消费者问题）[参考文章](https://blog.csdn.net/pcwl1206/article/details/89429548)
5. 两个栈实现队列（与后序非递归的双栈法类似）
6. 链表反转
   + 递归法（temp记录head.next，newNode记录返回节点，递归实际上是压栈和弹栈的过程）
   + 遍历法（pre记录每次反转的节点，next记录head.next，指导head != null）
7. 逆波兰表达式(LeetCode 150)
8. 斐波那契数列（青蛙跳台阶）：递归（设置停止条件发现规律）和非递归（使用数组记录fn的值，while实现）

# 八、参考网站

作者：垃圾程序员aka面团人
https://www.nowcoder.com/discuss/365165来源：牛客网

程序员必知的八大排序：https://www.cnblogs.com/crazylqy/p/7640829.html

java中的IO:https://www.cnblogs.com/ylspace/p/8128112.html

JVM面试知识点：https://www.cnblogs.com/lfs2640666960/p/9297176.html

JVM内存模型理解：https://www.jianshu.com/p/76959115d486

JAVA面试大纲：https://www.jianshu.com/p/a07d1d4004b0

java多线程学习：https://cloud.tencent.com/developer/article/1344265

Spring常见面试题：https://blog.csdn.net/a745233700/article/details/80959716

各大公司java面试题总结：https://www.cnblogs.com/itcx1213/p/10963809.html

2019java面试题：https://www.cnblogs.com/marsitman/p/9539369.html

剑指offer：https://www.nowcoder.com/ta/coding-interviews

leetcode100题：https://leetcode-cn.com/problemset/hot-100/

常见算法面试题：https://blog.csdn.net/qiaoer2017/article/details/82715028

计算机网络面试题总结：https://blog.csdn.net/qq_39322743/article/details/79700863

# 九、高级知识储备

+ Redis
+ docker
+ SpringCloud
+ Dubbo
+ SpringBoot
+ 分布式原理
+ 设计模式：
  + 适配器模式体现在java容器、SpringMVC、SpringAOP