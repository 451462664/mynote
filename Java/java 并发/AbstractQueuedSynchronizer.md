# java 并发——AbstractQueuedSynchronizer

---

https://www.jianshu.com/p/282bdb57e343

## 简介

```java
abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements Serializable
```

提供一个基于先进先出(FIFO)等待队列，可以用于构建锁或者其他相关同步器(信号量、事件等)的基础框架。这些同步器都依赖于一个原子 int 值来标识其状态。使用该类的方法就是继承，因为 abstract class AbstractQueuedSynchronizer 是抽象类，子类继承该同步器后需要实现它的方法来修改管理状态。多线程环境中对状态的操作必须要确保原子性，因此需要使用同步器所提供的方法 getState() setState(int) 和 compareAndSetState(int, int) 来操作值。
推荐子类被定义为自定义同步器类的内部类。AbstractQueuedSynchronizer 类不实现任何同步接口。 相反，它定义了一些方法，如 acquireInterruptibly(int) ，可以通过具体的锁和相关同步器来调用适当履行其公共方法。 
该同步器即可以作为排他模式也可以作为共享模式，当它被定义为一个排他模式时，其他线程对其的获取就被阻止，而共享模式对于多个线程获取都可以成功。

## 内部队列结构

就像开始提到的先进先出(FIFO)等待队列，那么队列中的元素到底是什么呢？其实在内部维护了一个 Node 类.

```java
static final class Node {
    // 共享
    static final Node SHARED = new Node();
    // 独占
    static final Node EXCLUSIVE = null;
    // 因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态；
    static final int CANCELLED =  1;
    // 后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
    static final int SIGNAL    = -1;
    // 节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()后，改节点将会从等待队列中转移到同步队列中，加入到同步状态的获取中
    static final int CONDITION = -2;
    // 表示下一次共享式同步状态获取将会无条件地传播下去
    static final int PROPAGATE = -3;
    // 等待状态
    volatile int waitStatus;
    // 前驱节点
    volatile Node prev;
    // 后继节点
    volatile Node next;
    // 获取同步状态的线程
    volatile Thread thread;
    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
}
```

**waitStatus**: 表示节点的状态

	1. CANCELLED: 1 表示当前的线程被取消
 	2. SIGNAL: -1 表示当前节点的后继节点包含的线程需要运行，也就是 unpark
 	3. CONDITION: -2 表示当前节点在等待 condition，也就是在 condition 队列中
 	4. PROPAGATE: -3 表示当前场景下后续的 acquireShared 能够得以执行
 	5. 0 表示当前节点在 sync 队列中，等待着获取锁

**prev**: 前驱节点，比如当前节点被取消，那就需要前驱节点和后继节点来完成连接.
**next**: 后继节点.
**nextWaiter**: 存储 condition 队列中的后继节点.
**thread**: 入队列时的当前线程.
![1568713180811](D:\Typora\image\1568713180811.png)

## api 说明

**AbstractQueuedSynchronizer** 主要提供了以下方法

1. getState() 返回同步状态的当前值
2. setState(int newState) 设置当前同步状态
3. compareAndSetState(int expect, int update) 使用 CAS 设置当前状态，该方法能够保证状态设置的原子性
4. tryAcquire(int arg) 独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态
5. tryRelease(int arg)：独占式释放同步状态
6. tryAcquireShared(int arg) 共享式获取同步状态，返回值大于等于 0 则表示获取成功，否则获取失败
7. tryReleaseShared(int arg) 共享式释放同步状态
8. isHeldExclusively() 当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占
9. acquire(int arg) 独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待，该方法将会调用可重写的 tryAcquire(int arg) 方法
10. acquireInterruptibly(int arg) 与 acquire(int arg) 相同，但是该方法响应中断，当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出 InterruptedException 异常并返回
11. tryAcquireNanos(int arg,long nanos) 超时获取同步状态，如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回false，已经获取则返回 true
12. acquireShared(int arg) 共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态
13. acquireSharedInterruptibly(int arg) 共享式获取同步状态，响应中断
14. tryAcquireSharedNanos(int arg, long nanosTimeout) 共享式获取同步状态，增加超时限制
15. release(int arg) 独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒
16. releaseShared(int arg) 共享式释放同步状态

## 方法分析

首先我们肯定是先看加锁操作，加锁操作入口是 acquire(int arg) 函数我们来看一看.

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

我们可以看到 acquire 函数中有 tryAcquire、addWaiter、acquireQueued、selfInterrupt 这几个方法都是十分重要的函数，我们接下来依次学习一下这些函数，理解它们的作用。
因为 AbstractQueuedSynchronizer 是抽象类所以我们看到了 tryAcquire 是一个抽象方法是留着给子类去实现的，这不就是模板方法模式吗，所以我们跳过该方法先看下面的两个，这个方法以后会说.

```java
/** 标记表示节点正在独占模式下等待 */
static final Node EXCLUSIVE = null;

addWaiter(Node.EXCLUSIVE), arg);
    
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // 首先获取到队尾的元素
    Node pred = tail;
    // 如果不是为 null
    if (pred != null) {
        // 该节点的前趋指针指向 tail
        node.prev = pred;
        // cas 将尾指针指向该节点
        if (compareAndSetTail(pred, node)) {
            // 让旧列尾节点的 next 指针指向该节点
            pred.next = node;
            return node;
        }
    }
    // 在 pred == null 或者 cas 操作失败的情况下调用 enq
    enq(node);
    return node;
}

private Node enq(final Node node) {
    // 开启死循环自旋
    for (;;) {
        Node t = tail;
        if (t == null) {
            // 进行初始化
            if (compareAndSetHead(new Node()))
                // 需要注意的是 head 是一个哨兵的作用,并不代表某个要获取锁的线程节点
                tail = head;
        } else {
            // 和 addWaiter 中操作一致,不过有了外侧的无限循环,不停的尝试,自旋锁
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

通过调用 addWaiter 方法就已经将当前线程加入到了等待队列中，但是还没有阻塞当前线程的执行，接下来我们就来分析一下acquireQueued 函数。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // 中断状态标志
        boolean interrupted = false;
        // 开启死循环
        for (;;) {
            // 获取当前线程的前驱节点
            final Node p = node.predecessor();
            // 如果前驱节点是头节点就说明 node 是将要获取锁的下一个节点.并且同步状态成功(所以再次尝试获取独占性变量)
            if (p == head && tryAcquire(arg)) {
                // 设置头节点为当前线程节点
                setHead(node);
                // help GC
                p.next = null;
                failed = false;
                return interrupted;
            }
            // 获取失败判断是否要进入阻塞状态
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    // SIGNAL 后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行
    if (ws == Node.SIGNAL)
        return true;
    // 如果大于 0
    if (ws > 0) {
        do {
            // 前一个节点处于取消获取独占性变量的状态,所以,可以跳过去
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 将上一个节点的状态设置为 signal,返回 false,
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

下面是整个流程图
![1568720703335](D:\Typora\image\1568720703335.png)

上面我们看了如果 shouldParkAfterFailedAcquire 函数返回 true 说明需要进入阻塞状态，并且会进入到 parkAndCheckInterrupt 方法中.方法主要是把当前线程挂起，从而阻塞住线程的调用栈，同时返回当前线程的中断状态.

```java
private final boolean parkAndCheckInterrupt() {
    // 将自己传入进去，LockSupport 工具类的 park() 方法来阻塞该方法
    LockSupport.park(this);
    return Thread.interrupted();
}
```

获取独占锁的代码我们已经看完了，下面我们来看下释放独占锁 release 代码的分析.

```java
public final boolean release(int arg) {
    // 同样是需要留给子类去实现我们暂且跳过
    if (tryRelease(arg)) {
        // 获取头节点
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 唤醒后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    // 获取当前线程节点的状态
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    // 获取后继节点
    Node s = node.next;
    // 如果为 null 或者已经是取消的状态就继续往后遍历找到不为 null 状态不为取消的后继节点
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 调用 LockSupport 工具类的 unpark 方法进行唤醒
        LockSupport.unpark(s.thread);
}
```

以上就是对 AQS AbstractOwnableSynchronizer 的介绍。