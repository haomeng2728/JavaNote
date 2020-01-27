# 多线程通信

## 一、synchronized加锁的线程的**Object类**的wait()/notify()/notifyAll()

```java
/**
 * Wakes up a single thread that is waiting on this object's
 * monitor. If any threads are waiting on this object, one of them
 * is chosen to be awakened. The choice is arbitrary and occurs at
 * the discretion of the implementation. A thread waits on an object's
 * monitor by calling one of the wait methods
 */
public final native void notify();
 
/**
 * Wakes up all threads that are waiting on this object's monitor. A
 * thread waits on an object's monitor by calling one of the
 * wait methods.
 */
public final native void notifyAll();
 
/**
 * Causes the current thread to wait until either another thread invokes the
 * {@link java.lang.Object#notify()} method or the
 * {@link java.lang.Object#notifyAll()} method for this object, or a
 * specified amount of time has elapsed.
 * <p>
 * The current thread must own this object's monitor.
 */
public final native void wait(long timeout) throws InterruptedException;
```

从以上三个方法的可得知以下几点信息：

1. wait()、notify()、notifyAll()方法是本地方法，并且为final方法，无法被重写
2. 调用某个对象的wait()方法能够让当前线程阻塞，并且当前线程必须拥有此对象的monitor（锁），执行wait()完成后，这个线程会释放锁，进行对象的等待池，但是一下三种方法不会释放锁：
   + sleep()会使当前线程放弃CPU，开始睡眠，在睡眠中不会释放锁
   + yield()方法当前线程放弃CPU，但不会释放锁
   + 其他线程执行了当前对象的suspend()方法，当前线程被暂停，但不会释放锁。但Thread类的suspend()方法已经被废弃
3. 调用**某个对象的notify()方法**能够**唤醒一个正在等待这个对象的monitor的线程**，如果有多个线程都在等待这个对象的monitor，则**只能唤醒其中一个线程**
4. 调用notifyAll()方法能够**唤醒所有正在等待这个对象的monitor的线程**

