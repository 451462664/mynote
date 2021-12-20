# java 并发——ReentrantLock

---

## 简介

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    // 继承了 AbstractQueuedSynchronizer 具体操作的执行者
    private final Sync sync;
    
    abstract static class Sync extends AbstractQueuedSynchronizer {
        // ...
    }
}
```

重入锁: 一种可重入互斥锁具有与使用 synchronized 方法和语句访问的隐式监视锁相同的基本行为和语义，但是具有扩展功能。
ReentrantLock 类的构造方法可以接收一个布尔值，当设置为 true 的情况下就是公平锁模式，在竞争的情况下有利于授予等待最长时间的线程。否则 false 是非公平锁该锁不保证任何特定的访问顺序。使用多线程访问的情况下非公平锁比公平锁具有更快的吞吐量。但是请注意，锁的公平性不能保证线程调度的公平性。 因此，使用公平锁的许多线程之一可以连续获得多次，而其他活动线程不进行而不是当前持有锁。 另请注意， 未定义的 tryLock() 方法不符合公平性设置。 如果锁可用，即使其他线程正在等待，它也会成功。 

建议使用方法是 lock 始终与 try 块成对出现。

![1568863183339](D:\Typora\image\1568863183339.png)

## 方法分析

首先我们先来看看构造器

```java
public ReentrantLock() {
    // 默认使用非公平锁
    sync = new NonfairSync();
}

// 传递 boolean 值来选择锁的类型
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### lock 非公平加锁操作

```java
public void lock() {
    sync.lock();
}

final void lock() {
    // 首先进行 cas 操作修改 state 状态
    if (compareAndSetState(0, 1))
        // 如果状态修改成功则设置为当前线程所有
        setExclusiveOwnerThread(Thread.currentThread());
    else
        // 没有修改成功则调用 AQS 的 acquire(int arg) 方法
        acquire(1);
}
```

上面代码主要做了: 

1. 将 state 状态值设置为 1.
2. 如果设置成功则将锁设置为当前线程所有.
3. 如果 state 状态已经被其他线程设置了则会失败则调用 AQS 的 acquire(int arg) 方法.

```java
public final void acquire(int arg) {
    // 调用子类重写的 tryAcquire 方法
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

上面我们可以看到在 AQS 抽象类中我们发现了提供给子类重写 tryAcquire 的方法，那么我们就去 NonfairSync 类中看下其中实现代码

```java
protected final boolean tryAcquire(int acquires) {
    // 调用父类 Sync.nonfairTryAcquire
    return nonfairTryAcquire(acquires);
}

final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取状态
    int c = getState();
    // 如果发现状态等于 0 说明没有锁处理空闲状态
    if (c == 0) {
        // 再次尝试修改 state 如果获取成功则设置当前线程所有
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 如果 state 不是 0 并且锁持有者的线程就是当前线程判定为重入
    else if (current == getExclusiveOwnerThread()) {
        // 将 state 的值 + 1
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        // 为 state 赋予新值
        setState(nextc);
        return true;
    }
    return false;
}
```

上面代码主要做了: 

1. 首先判断同步状态 state == 0
2. 如果是表示该锁还没有被线程持有，直接通过 cas 获取同步状态，如果成功返回 true
3. 如果 state != 0，则判断当前线程是否为获取锁的线程，如果是则获取锁，成功返回 true

### unlock 释放锁操作

```java
public void unlock() {
    // 调用 AQS release
    sync.release(1);
}

public final boolean release(int arg) {
    // 调用子类重写的 Sync.tryRelease
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

protected final boolean tryRelease(int releases) {
    // 获取状态减去 releases 如果没有重入则 c = 0
    int c = getState() - releases;
    // 如果锁持有者线程不是该线程则抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 如果 c=state==0 表示已经释放
    if (c == 0) {
        free = true;
        // 设置锁持有者线程为 null
        setExclusiveOwnerThread(null);
    }
    // 重新设置 state
    setState(c);
    return free;
}
```

上面代码可以看到只有同步状态 state == 0 时才算时真正的彻底释放锁，会将锁持有者线程设置为 null 表示释放成功。

### lock 公平锁加锁操作

公平锁与非公平锁的区别在于获取锁的时候是否按照FIFO的顺序来。释放锁不存在公平性和非公平性。我们来看加锁操作。

```java
final void lock() {
    // 调用 AQS acquire
    acquire(1);
}

public final void acquire(int arg) {
    // 调用子类公平锁的实现 tryAcquire
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 我们发现多了一行 !hasQueuedPredecessors()
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

public final boolean hasQueuedPredecessors() {
    // 尾部节点
    Node t = tail;
    // 头部节点
    Node h = head;
    Node s;
    // 头部节点不等于尾部节点并且(头部节点没有后继节点或者头部节点线程不等于当前线程)
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

我们发现公平锁和非公平锁的代码很大一部分都是一模一样的，只是多了一行 !hasQueuedPredecessors() 判断.
hasQueuedPredecessors 主要是判断同步队列中是否还有等待的节点线程，如果有则返回 true 没有返回 false.

## 总结

ReentrantLock 与 synchronized 比较

1. ReentrantLock 提供了更多，更加全面的功能，具备更强的扩展性。
2. ReentrantLock 还提供了条件 Condition，对线程的等待、唤醒操作更加详细和灵活。
3. ReentrantLock 提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized 则一旦进入锁请求要么成功要么阻塞，所以相比 synchronized 而言，ReentrantLock 会不容易产生死锁些。
4. ReentrantLock 支持更加灵活的同步代码块，但是使用 synchronized 时，只能在同一个 synchronized 块结构中获取和释放。
5. ReentrantLock 支持中断处理，且性能较 synchronized 会好些。

