# AQS 

AQS即是AbstractQueuedSynchronizer，一个用来构建锁和同步工具的框架，包括常用的ReentrantLock、CountDownLatch、Semaphore等。

AQS没有锁之类的概念，它有个state变量，是个int类型，在不同场合有着不同含义。本文研究的是锁，为了好理解，姑且先把state当成锁。

AQS围绕state提供两种基本操作“获取”和“释放”，有条双向队列存放阻塞的等待线程，并提供一系列判断和处理方法，简单说几点：

- state是独占的，还是共享的；
- state被获取后，其他线程需要等待；
- state被释放后，唤醒等待线程；
- 线程等不及时，如何退出等待。

至于线程是否可以获得state，如何释放state，就不是AQS关心的了，要由子类具体实现。

直接分析AQS的代码会比较难明白，所以结合子类ReentrantLock来分析。AQS的功能可以分为独占和共享，ReentrantLock实现了独占功能

简单来说就是AQS维护一个state变量和一个队列（队列节点为线程相关信息），当state为0是也就是没有任何线程持有锁，当大于0时，队列中的头节点持有锁，ReentrantLock是可重入锁，利用state变量维护重入次数，当释放锁时，依次把state数值减1，直到减为0表示释放锁。

```
static final class Node {

        /**
         * The thread that enqueued this node.  Initialized on
         * construction and nulled out after use.
         */
         //线程对象
        volatile Thread thread;
        
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null；
        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;
        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        Node nextWaiter;
```



# ReentrantLock

ReentrantLock的锁对象为ReentrantLock自身，不同的ReentrantLock对象为不同的锁，synchronized锁对象为类或类对象

### ReentrantLock对比synchronized

```csharp
Lock lock = new ReentranLock();
lock.lock();
try{
    //do something
}finally{
    lock.unlock();
}
```

ReentrantLock实现了Lock接口，加锁和解锁都需要显式写出，注意一定要在适当时候unlock。

和synchronized相比，ReentrantLock用起来会复杂一些。在基本的加锁和解锁上，两者是一样的，所以无特殊情况下，推荐使用synchronized。ReentrantLock的优势在于它更灵活、更强大，除了常规的lock()、unlock()之外，还有lockInterruptibly()、tryLock()方法，支持中断、超时。

### 公平锁和非公平锁

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

ReentrantLock的内部类Sync继承了AQS，分为公平锁FairSync和非公平锁NonfairSync。

- 公平锁：线程获取锁的顺序和调用lock的顺序一样，FIFO；
- 非公平锁：线程获取锁的顺序和调用lock的顺序无关，全凭运气。

ReentrantLock默认使用非公平锁是基于性能考虑，公平锁为了保证线程规规矩矩地排队，需要增加阻塞和唤醒的时间开销。如果直接插队获取非公平锁，跳过了对队列的处理，速度会更快。

### 尝试获取锁

```java
final void lock() { acquire(1);}

public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

先来看公平锁的实现，lock方法很简单的一句话调用AQS的acquire方法：

```java
protected boolean tryAcquire(int arg) {    
        throw new UnsupportedOperationException();
}
```

噢，AQS的tryAcquire不能直接调用，因为是否获取锁成功是由子类决定的，直接看ReentrantLock的tryAcquire的实现。

```java
protected final boolean tryAcquire(int acquires) {
   final Thread current = Thread.currentThread();
   int c = getState();
   if (c == 0) {
       if (!hasQueuedPredecessors() &&
           compareAndSetState(0, acquires)) {
           setExclusiveOwnerThread(current);
           return true;
       }
   }
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

获取锁成功分为两种情况，第一个if判断AQS的state是否等于0，表示锁没有人占有。接着，hasQueuedPredecessors判断队列是否有排在前面的线程在等待锁，没有的话调用compareAndSetState使用cas的方式修改state，传入的acquires写死是1。最后线程获取锁成功，setExclusiveOwnerThread将线程记录为独占锁的线程。

第二个if判断当前线程是否为独占锁的线程，因为**ReentrantLock是可重入的**，线程可以不停地lock来增加state的值，对应地需要unlock来解锁，直到state为零。

如果最后获取锁失败，下一步需要将线程加入到等待队列。

### 线程进入等待队列

AQS内部有一条双向的队列存放等待线程，节点是Node对象。每个Node维护了线程、前后Node的指针和等待状态等参数。

线程在加入队列之前，需要包装进Node，调用方法是addWaiter：

```csharp
private Node addWaiter(Node mode) {
   Node node = new Node(Thread.currentThread(), mode);
   // Try the fast path of enq; backup to full enq on failure
   Node pred = tail;
   if (pred != null) {
       node.prev = pred;
       if (compareAndSetTail(pred, node)) {
           pred.next = node;
           return node;
       }
   }
   enq(node);
   return node;
}
```

每个Node需要标记是独占的还是共享的，由传入的mode决定，ReentrantLock自然是使用独占模式Node.EXCLUSIVE。

创建好Node后，如果队列不为空，使用cas的方式将Node加入到队列尾。注意，这里只执行了一次修改操作，并且可能因为并发的原因失败。因此修改失败的情况和队列为空的情况，需要进入enq。

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

enq是个死循环，保证Node一定能插入队列。注意到，当队列为空时，会先为头节点创建一个空的Node，因为**头节点代表获取了锁的线程**，现在还没有，所以先空着。

### 阻塞等待线程

线程加入队列后，下一步是调用acquireQueued阻塞线程。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //1
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //2
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

**标记1**是线程唤醒后尝试获取锁的过程。如果前一个节点正好是head，表示自己排在第一位，可以马上调用tryAcquire尝试。如果获取成功就简单了，直接修改自己为head。这步是实现公平锁的核心，保证释放锁时，由下个排队线程获取锁。（看到线程解锁时，再看回这里啦）

**标记2**是线程获取锁失败的处理。这个时候，线程可能等着下一次获取，也可能不想要了，Node变量waitState描述了线程的等待状态，一共四种情况：

```java
 bstatic final int CANCELLED =  1;   //取消
static final int SIGNAL    = -1;     //下个节点需要被唤醒
static final int CONDITION = -2;  //线程在等待条件触发
static final int PROPAGATE = -3; //（共享锁）状态需要向后传播
```

shouldParkAfterFailedAcquire传入当前节点和前节点，根据前节点的状态，判断线程是否需要阻塞。

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  int ws = pred.waitStatus;
  if (ws == Node.SIGNAL)
      return true;
  if (ws > 0) {
      do {
          node.prev = pred = pred.prev;
      } while (pred.waitStatus > 0);
      pred.next = node;
  } else {
      compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
  }
  return false;
}
```

- 前节点状态是SIGNAL时，当前线程需要阻塞；
- 前节点状态是CANCELLED时，通过循环将当前节点之前所有取消状态的节点移出队列；
- 前节点状态是其他状态时，需要设置前节点为SIGNAL。

如果线程需要阻塞，由parkAndCheckInterrupt方法进行操作。

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

parkAndCheckInterrupt使用了LockSupport，和cas一样，最终使用UNSAFE调用Native方法实现线程阻塞（以后有机会就分析下LockSupport的原理，park和unpark方法作用类似于wait和notify）。最后返回线程唤醒后的中断状态，关于中断，后文会分析。

到这里总结一下获取锁的过程：线程去竞争一个锁，可能成功也可能失败。成功就直接持有资源，不需要进入队列；失败的话进入队列阻塞，等待唤醒后再尝试竞争锁。

### 释放锁

通过上面详细的获取锁过程分析，释放锁过程大概可以猜到：头节点是获取锁的线程，先移出队列，再通知后面的节点获取锁。

```cpp
public void unlock() {
    sync.release(1);
}
```

ReentrantLock的unlock方法很简单地调用了AQS的release：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

和lock的tryAcquire一样，unlock的tryRelease同样由ReentrantLock实现：

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

因为锁是可以重入的，所以每次lock会让state加1，对应地每次unlock要让state减1，直到为0时将独占线程变量设置为空，返回标记是否彻底释放锁。

最后，调用unparkSuccessor将头节点的下个节点唤醒：

```csharp
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

寻找下个待唤醒的线程是从队列尾向前查询的，找到线程后调用LockSupport的unpark方法唤醒线程。被唤醒的线程重新执行acquireQueued里的循环，就是上文关于acquireQueued标记1部分，线程重新尝试获取锁。

### 中断锁

```cpp
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

在acquire里还有最后一句代码调用了selfInterrupt，功能很简单，对当前线程产生一个中断请求。

为什么要这样操作呢？因为LockSupport.park阻塞线程后，有两种可能被唤醒。

第一种情况，前节点是头节点，释放锁后，会调用LockSupport.unpark唤醒当前线程。整个过程没有涉及到中断，最终acquireQueued返回false时，不需要调用selfInterrupt。

第二种情况，LockSupport.park支持响应中断请求，能够被其他线程通过interrupt()唤醒。但这种唤醒并没有用，因为线程前面可能还有等待线程，在acquireQueued的循环里，线程会再次被阻塞。parkAndCheckInterrupt返回的是Thread.interrupted()，不仅返回中断状态，还会清除中断状态，保证阻塞线程忽略中断。最终acquireQueued返回true时，真正的中断状态已经被清除，需要调用selfInterrupt维持中断状态。

因此普通的lock方法并不能被其他线程中断，ReentrantLock是可以支持中断，需要使用lockInterruptibly。

两者的逻辑基本一样，不同之处是parkAndCheckInterrupt返回true时，lockInterruptibly直接throw new InterruptedException()。

### 非公平锁

分析完公平锁的实现，还剩下非公平锁，主要区别是获取锁的过程不同。

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

在NonfairSync的lock方法里，第一步直接尝试将state修改为1，很明显，这是抢先获取锁的过程。如果修改state失败，则和公平锁一样，调用acquire。

```java
final boolean nonfairTryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  if (c == 0) {
      if (compareAndSetState(0, acquires)) {
          setExclusiveOwnerThread(current);
          return true;
      }
  }
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

nonfairTryAcquire和tryAcquire乍一看几乎一样，差异只是缺少调用hasQueuedPredecessors。这点体验出公平锁和非公平锁的不同，公平锁会关注队列里排队的情况，老老实实按照FIFO的次序；非公平锁只要有机会就抢占，才不管排队的事。

# ArrayBlockingQueue

put方法

```
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
        //如果数组满了，生产者等待
            notFull.await();  
        //如果不满调用入队方法
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

enqueue方法

```
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    //唤醒消费者
    notEmpty.signal();
}
```

take方法

```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
        //如果队列为空。消费者等待
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

dequeue方法

```
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
        //唤醒生产者
    notFull.signal();
    return x;
}
```

# Condition

在进行线程间的通信时，当我们使用synchronized时，可以用基于Object对象的wait和notify方法实现等待/通知机制，但是在AQS相关类中怎么实现这种等待/通知机制呢？答案是Condition，Condition是一个接口，AbstractQueuedSynchronizer中有一个内部类实现了这个接口

condition可以用于生产者消费者，condition依赖ReentrantLock，ReentrantLock依赖于AOS的同步队列，condition内部维持一个队列，队列中的每个节点维护线程信息，当调用condition.await方法会将当前线程（ReentrantLock队列中的节点，也就是AQS队列）移到condition的队列并释放锁，当调用condition.signal方法会把队列中的头节点放到ReentrantLock中的队列中，也就是AQS的队列。一个ReentrantLock队列可以对应多个Condition。

# LockSupport

park方法与unpark方法与object类的wait方法和notify方法类似都是阻塞线程与唤醒线程，但是object类的方法需要先获取对象的锁，而LockSupport直接传入Thread对象利用UNSAFE操作阻塞与唤醒线程

park方法

```
 public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        //调用unsafe
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```

unpark方法

```
  public static void unpark(Thread thread) {
      //调用unsafe
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```

