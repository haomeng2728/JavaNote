# Java集合

## 一、接口继承关系和实现



Java集合主要在java.util包中，主要包含set、list（包含queue）、map

1、Collections主要是集合List、Set、Queue的基本接口

2、Iterator：迭代器，可以通过迭代器遍历集合中的数据

3、Map：是映射表的基础接口

## 二、List

list是有序的Collection，Java List有是三个实现类：ArrayList、Vector和LinkedList

### 1、ArrayList 数组

内部使用数组实现，允许对元素的快速访问，缺点是不允许元素之间有间隔。

当数组大小不满足时需要增加存储能力，就要将已有的数组的数据复制到新的空间中，当从ArrayList中间位置插入或者删除数据时，需要对数组进行复制、移动，代价比较高。因此，它适合随机查找和遍历，不适合插入和删除

### 2、Vector 数组实现，线程同步

Vector也是通过数组实现，但是它支持线程同步，即某一时刻只能有一个线程写Vector，避免多线程同时写造成不一致性，但是同步需要很高的花费，因为访问它比访问ArrayList慢

### 3、LinkedList 链表

使用链表结构存储数据，适合数据的动态插入和删除，随机访问和遍历速度慢。另外，他还提供给了List接口中没有定义的方法，用于操作表头和表尾数剧，可以当做堆栈、队列和双向队列使用。

## 三、Set

## 四、Map

