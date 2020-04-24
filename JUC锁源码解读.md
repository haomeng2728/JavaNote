https://www.cnblogs.com/skywang12345/p/3496147.html

# 一、获取公平锁

ReentrantLock中lock的具体实现方式如下：

```java
public void lock() {
        sync.acquire(1);
    }
```

其中，sync是Sync的一个引用，acquire方法中的参数1指的是锁的状态设置参数，对于**独占锁**来说，锁在可获取的状态下状态值为0，当变成获取状态时状态值变为1。由于ReentrantLock（公平锁、非公平锁）是可重入锁，单个线程可以多次获取锁，没获取一次就将状态+1，也就是说，初次获取锁时，通过acquire(1)将锁的状态值设为1；再次获取锁时，将锁的状态值设为2；依次类推...这就是为什么获取锁时，传入的参数是1的原因了。

acquire的源码如下：

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

+ “当前线程”首先通过tryAcquire()尝试获取锁。
  + 获取成功的话，直接返回；
  + 尝试失败的话，进入到等待队列排序等待(前面还有可能有需要线程在等待该锁)。
+ “当前线程”尝试失败的情况下，先通过addWaiter(Node.EXCLUSIVE)来将“当前线程”加入到"CLH队列(非阻塞的FIFO队列)"末尾。CLH队列就是线程等待队列。
+ 执行完addWaiter(Node.EXCLUSIVE)之后，会调用acquireQueued()来获取锁。由于此时ReentrantLock是公平锁，它会**根据公平性原则**来获取锁。
+ “当前线程”在执行acquireQueued()时，会进入到CLH队列中休眠等待，直到获取锁了才返回！如果“当前线程”在休眠等待过程中被中断过，acquireQueued会返回true，此时"当前线程"会调用selfInterrupt()来自己给自己产生一个中断。至于为什么要自己给自己产生一个中断，后面再介绍。

## 1.1 tryAcquire()

### 1.1.1 tryAcquire()

tryAcquire()方法用来返回当前线程是否成功获取锁，FairSync中的tryAcquire()的实现方式如下：

```java
protected final boolean tryAcquire(int acquires) {
 						//获取当前线程
            final Thread current = Thread.currentThread();
  					//获取独占锁状态
            int c = getState();
  					//c=0意味着：锁没有被任何线程拥有
            if (c == 0) {
              //若锁没有被任何线程拥有
              //则hasQueuedPredecessors()判断当前线程是不是CLH队列中第一个线程
              //如果是，则compareAndSetState(0, acquires)设置锁的状态
              //setExclusiveOwnerThread(current)设置锁的拥有者为当前线程
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
  					//这个部分实现类独占锁的可重入性质
  					//如果独占锁的拥有者是当前线程，则更新锁的状态值，即不断加一
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

### 1.1.2 hasQueuedPredecessors()

hasQueuedPredecessors()在AQS中实现，源码如下：

```java
public final boolean hasQueuedPredecessors() {
        Node h, s;
        if ((h = head) != null) {
            if ((s = h.next) == null || s.waitStatus > 0) {
                s = null; // traverse in case of concurrent cancellation
                for (Node p = tail; p != h && p != null; p = p.prev) {
                    if (p.waitStatus <= 0)
                        s = p;
                }
            }
            if (s != null && s.thread != Thread.currentThread())
                return true;
        }
        return false;
    }
```

**说明**： 通过代码，能分析出，hasQueuedPredecessors() 是通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程。下面对head、tail和Node进行说明。

### 1.1.3 Node源码

```java
private transient volatile Node head;    // CLH队列的队首
private transient volatile Node tail;    // CLH队列的队尾

// CLH队列的节点
static final class Node {
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;

    // 线程已被取消，对应的waitStatus的值
    static final int CANCELLED =  1;
    // “当前线程的后继线程需要被unpark(唤醒)”，对应的waitStatus的值。
    // 一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
    static final int SIGNAL    = -1;
    // 线程(处在Condition休眠状态)在等待Condition唤醒，对应的waitStatus的值
    static final int CONDITION = -2;
    // (共享锁)其它线程获取到“共享锁”，对应的waitStatus的值
    static final int PROPAGATE = -3;

    // waitStatus为“CANCELLED, SIGNAL, CONDITION, PROPAGATE”时分别表示不同状态，
    // 若waitStatus=0，则意味着当前线程不属于上面的任何一种状态。
    volatile int waitStatus;

    // 前一节点
    volatile Node prev;

    // 后一节点
    volatile Node next;

    // 节点所对应的线程
    volatile Thread thread;

    // nextWaiter是“区别当前CLH队列是 ‘独占锁’队列 还是 ‘共享锁’队列 的标记”
    // 若nextWaiter=SHARED，则CLH队列是“独占锁”队列；
    // 若nextWaiter=EXCLUSIVE，(即nextWaiter=null)，则CLH队列是“共享锁”队列。
    Node nextWaiter;

    // “共享锁”则返回true，“独占锁”则返回false。
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 返回前一节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    // 构造函数。thread是节点所对应的线程，mode是用来表示thread的锁是“独占锁”还是“共享锁”。
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 构造函数。thread是节点所对应的线程，waitStatus是线程的等待状态。
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```



## 1.2 addWaiter()

此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点。addWaiter源码如下：

```java
private Node addWaiter(Node mode) {
  			//新建一个Node节点，节点对应的线程是当前线程，当前线程的锁的模型是mode
        Node node = new Node(mode);

        for (;;) {
            Node oldTail = tail;
            if (oldTail != null) {
              //将node的prev节点设为oldTail
                node.setPrevRelaxed(oldTail);
                if (compareAndSetTail(oldTail, node)) {
                    oldTail.next = node;
                    return node;
                }
            } else {
                initializeSyncQueue();
            }
        }
    }
```

在java12中用initializeSyncQueue()来代替enq()进行CLH队列头结点的初始化，但enq()仍保留

```java
private final void initializeSyncQueue() {
        Node h;
        if (HEAD.compareAndSet(this, null, (h = new Node())))
            tail = h;
    }
```



## 1.3 addQueue()

将当前线程加入到CLH队尾后，acquireQueued()将逐步执行CLH里的线程，如果当前线程获得了锁则返回，否则线程进行休眠，直到唤醒并重新获取锁才返回。

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean interrupted = false;
        try {
            for (;;) {
              //获取上一个节点
                final Node p = node.predecessor();
              //当node对应的线程是头结点的下个节点的线程并且当前线程获取锁成功
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    return interrupted;
                }
              //如果自己可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            if (interrupted)
                selfInterrupt();
            throw t;
        }
    }
```

shouldParkAfterFailedAcquire()返回当前线程是够应该阻塞

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  			//前继节点的状态
        int ws = pred.waitStatus;
  			//singal说明需要唤醒后继节点，一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            pred.compareAndSetWaitStatus(ws, Node.SIGNAL);
        }
        return false;
    }
```



## 1.4 selfInterrupt()

```java
private static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

**说明**：selfInterrupt()的代码很简单，就是“当前线程”自己产生一个中断。但是，为什么需要这么做呢？
这必须结合acquireQueued()进行分析。如果在acquireQueued()中，当前线程被中断过，则执行selfInterrupt()；否则不会执行。

在acquireQueued()中，即使是线程在阻塞状态被中断唤醒而获取到cpu执行权利；但是，如果该线程的前面还有其它等待锁的线程，根据公平性原则，该线程依然无法获取到锁。它会再次阻塞！ 该线程再次阻塞，直到该线程被它的前面等待锁的线程锁唤醒；线程才会获取锁，然后“真正执行起来”！
也就是说，在该线程“成功获取锁并真正执行起来”之前，它的中断会被忽略并且中断标记会被清除！ 因为在parkAndCheckInterrupt()中，我们线程的中断状态时调用了Thread.interrupted()。该函数不同于Thread的isInterrupted()函数，isInterrupted()仅仅返回中断状态，而interrupted()在返回当前中断状态之后，还会清除中断状态。 正因为之前的中断状态被清除了，所以这里需要调用selfInterrupt()重新产生一个中断！



**小结**：selfInterrupt()的作用就是当前线程自己产生一个中断。

# 二、释放公平锁

## 2.1 unlock() 

```java
public void unlock() {
    sync.release(1);
}
```

unlock()是解锁函数，它是通过AQS的release()函数来实现的。
在这里，“1”的含义和“获取锁的函数acquire(1)的含义”一样，它是设置“释放锁的状态”的参数。由于“公平锁”是可重入的，所以对于同一个线程，每释放锁一次，锁的状态-1。

## 2.2 release()

```java
public final boolean release(int arg) {
        if (tryRelease(arg)) {
          //找到头结点
            Node h = head;
            if (h != null && h.waitStatus != 0)
              //唤醒等待队列的下一个线程
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

release()会先调用tryRelease()来尝试释放当前线程锁持有的锁。成功的话，则唤醒后继等待线程，并返回true。否则，直接返回false。

## 2.3 tryRelease()

```java
protected final boolean tryRelease(int releases) {
  					//c是本次释放锁之后的状态
            int c = getState() - releases;
  					//如果当前线程不是锁的持有者，则抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
  					//如果锁彻底被当前线程释放，则设置锁的状态为null，即可获取状态
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
  					//设置当前线程锁的状态
            setState(c);
            return free;
        }
```

tryRelease()在ReentrantLock.java的Sync类中实现，

## 2.4 unparkSuccessor()

```java
private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            node.compareAndSetWaitStatus(ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node p = tail; p != node && p != null; p = p.prev)
                if (p.waitStatus <= 0)
                    s = p;
        }
  			//唤醒后续节点对应的线程
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```



# 三、获取非公平锁

NonFairSync中的tryAcquire()的实现方式如下：

```java
protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }

final boolean nonfairTryAcquire(int acquires) {
  					//获取当前线程
            final Thread current = Thread.currentThread();
  					//获取锁的状态
            int c = getState();
  					//c=0表示锁没有被任何线程所拥有的，可以通过CAS函数设置锁的状态为acquires
  					//同时设置当前线程为锁的持有者
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
  					//可重入锁的设置
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

**公平锁和非公平锁在tryAcquire上的区别**

公平锁和非公平锁，它们尝试获取锁的方式不同：

+ 公平锁在尝试获取锁时，即使“锁”没有被任何线程锁持有，它也会判断自己是不是CLH等待队列的表头；是的话，才获取锁
+ 非公平锁在尝试获取锁时，如果“锁”没有被任何线程持有，则不管它在CLH队列的何处，它都直接获取锁。

# 四、获取共享锁

获取共享锁的思想(即lock函数的步骤)，是先通过tryAcquireShared()尝试获取共享锁。尝试成功的话，则直接返回；尝试失败的话，则通过doAcquireShared()不断的循环并尝试获取锁，若有需要，则阻塞等待。doAcquireShared()在循环中每次尝试获取锁时，都是通过tryAcquireShared()来进行尝试的

## 4.1 lock()

```java
public void lock() {
            sync.acquireShared(1);
        }
```

## 4.2 acquireShared()

```java
public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

这里tryAcquireShared()依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里acquireShared()的流程就是：

1. tryAcquireShared()尝试获取资源，成功则直接返回；
2. 失败则通过doAcquireShared()进入等待队列，直到获取到资源为止才返回。

## 4.3 tryAcquireShared() ？？？

```java
protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
  					//获取当前线程
            Thread current = Thread.currentThread();
  					//获取锁的状态
            int c = getState();
  					//如果锁是互斥锁（写锁），并且获取锁的线程不是current线程；则返回-1。
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
  					//获取“读取锁”的共享计数
            int r = sharedCount(c);
  					// 如果“不需要阻塞等待”，并且“读取锁”的共享计数小于MAX_COUNT；
    				// 则通过CAS函数更新“锁的状态”，将“读取锁”的共享计数+1。
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                  	// 第1次获取“读取锁”
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                  // 如果想要获取锁的线程(current)是第1个获取锁(firstReader)的线程
                    firstReaderHoldCount++;
                } else {
                  // HoldCounter是用来统计该线程获取“读取锁”的次数
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null ||
                        rh.tid != LockSupport.getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
```



## 4.4 fullTryAcquireShared() ？？？

```java
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        // 获取“锁”的状态
        int c = getState();
        // 如果“锁”是“互斥锁”，并且获取锁的线程不是current线程；则返回-1。
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        // 如果“需要阻塞等待”。
        // (01) 当“需要阻塞等待”的线程是第1个获取锁的线程的话，则继续往下执行。
        // (02) 当“需要阻塞等待”的线程获取锁的次数=0时，则返回-1。
        } else if (readerShouldBlock()) {
            // 如果想要获取锁的线程(current)是第1个获取锁(firstReader)的线程
            if (firstReader == current) {
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != current.getId()) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                // 如果当前线程获取锁的计数=0,则返回-1。
                if (rh.count == 0)
                    return -1;
            }
        }
        // 如果“不需要阻塞等待”，则获取“读取锁”的共享统计数；
        // 如果共享统计数超过MAX_COUNT，则抛出异常。
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // 将线程获取“读取锁”的次数+1。
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            // 如果是第1次获取“读取锁”，则更新firstReader和firstReaderHoldCount。
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            // 如果想要获取锁的线程(current)是第1个获取锁(firstReader)的线程，
            // 则将firstReaderHoldCount+1。
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != current.getId())
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                // 更新线程的获取“读取锁”的共享计数
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

fullTryAcquireShared()会根据“是否需要阻塞等待”，“读取锁的共享计数是否超过限制”等等进行处理。如果不需要阻塞等待，并且锁的共享计数没有超过限制，则通过CAS尝试获取锁，并返回1。

## 4.5 doAcquireShared()

此方法用于将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。下面是doAcquireShared()的源码：

```java
private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);//加入队列尾部
        boolean interrupted = false;//等待过程中是否被中断过的标志
        try {
            for (;;) {
                final Node p = node.predecessor();
              //如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                      //将head指向自己，还有剩余资源可以再唤醒之后的线程
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        return;
                    }
                }
              //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            throw t;
        } finally {
            if (interrupted)
                selfInterrupt();
        }
    }

```

# 五、释放共享锁

释放共享锁的思想，是先通过tryReleaseShared()尝试释放共享锁。尝试成功的话，则通过doReleaseShared()唤醒“其他等待获取共享锁的线程”，并返回true；否则的话，返回flase。

## 5.1 unlock()

```java
public void unlock() {
            sync.releaseShared(1);
        }
```

**说明**：该函数实际上调用releaseShared(1)释放共享锁。

## 5.2 releaseShared()

```java
public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

releaseShared()的目的是让当前线程释放它所持有的共享锁。
它首先会通过tryReleaseShared()去尝试释放共享锁。尝试成功，则直接返回；尝试失败，则通过doReleaseShared()去释放共享锁。

## 5.3 tryReleaseShared() ？？？

```java
protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
  				// 如果想要释放锁的线程(current)是第1个获取锁(firstReader)的线程，
          // 并且“第1个获取锁的线程获取锁的次数”=1，则设置firstReader为null；
          // 否则，将“第1个获取锁的线程的获取次数”-1。
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            } else {
              // 获取rh对象，并更新“当前线程获取锁的信息”。
                HoldCounter rh = cachedHoldCounter;
                if (rh == null ||
                    rh.tid != LockSupport.getThreadId(current))
                    rh = readHolds.get();
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
            for (;;) {
                int c = getState();
                int nextc = c - SHARED_UNIT;
                if (compareAndSetState(c, nextc))
                    // Releasing the read lock has no effect on readers,
                    // but it may allow waiting writers to proceed if
                    // both read and write locks are now free.
                    return nextc == 0;
            }
        }
```

**HoldCounter的作用是什么？**没搞懂

## 5.4 doReleaseShared()

```java
private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!h.compareAndSetWaitStatus(Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !h.compareAndSetWaitStatus(0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }

```

doReleaseShared()会释放“共享锁”。它会从前往后的遍历CLH队列，依次“唤醒”然后“执行”队列中每个节点对应的线程；最终的目的是让这些线程释放它们所持有的锁。

# 六、公平共享锁和非公平共享锁

和互斥锁ReentrantLock一样，ReadLock也分为公平锁和非公平锁。

公平锁和非公平锁的区别，体现在判断是否需要阻塞的函数readerShouldBlock()是不同的。
公平锁的readerShouldBlock()的源码如下：

```java
final boolean readerShouldBlock() {
    return hasQueuedPredecessors();
}
```

在公平共享锁中，如果在当前线程的前面有其他线程在等待获取共享锁，则返回true；否则，返回false。
非公平锁的readerShouldBlock()的源码如下：

```java
final boolean readerShouldBlock() {
    return apparentlyFirstQueuedIsExclusive();
}
```

在非公平共享锁中，它会无视当前线程的前面是否有其他线程在等待获取共享锁。只要该非公平共享锁对应的线程不为null，则返回true。

