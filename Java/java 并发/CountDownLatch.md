# java 并发——CountDownLatch

---

## 简介

```java
public class CountDownLatch {
    private final Sync sync;
    
    private static final class Sync extends AbstractQueuedSynchronizer {
        Sync(int count) {
            setState(count);
        }
        // ...
    }
}
```

允许一个或多个线程等待直到其他线程中执行的一组操作完成的同步辅助工具。
例如一个项目中有十个模块功能分别给十个人开发，工期是 10 天后发布上线。知道到 10 天后所有人都开发完毕后才可以上线少一个都不可以。

构造函数

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

可以看到构造函数中我们传入了一个 count 值，由上面的代码我们可以发现其实就是 AQS 中 state 的值。是倒数计数，所以 CountDownLatch 的用法通常是设定一个大于 0 的 state 值，该值即代表需要等待的总任务数，每完成一个任务后，将总任务数减一，直到最后该值为0，说明所有等待的任务都执行完了，后面的任务可以继续执行。

## 方法分析

public void countDown();

```java
public void countDown() {
    // 调用 AQS 的 releaseShared 方法传入 1
    sync.releaseShared(1);
}

// AbstractQueuedSynchronizer 类中的方法
public final boolean releaseShared(int arg) {
    // tryReleaseShared 方法在 CountDownLatch 中被重写
    if (tryReleaseShared(arg)) {
        // 唤醒等待的线程
        doReleaseShared();
        return true;
    }
    return false;
}

// CountDownLatch 重写的方法
protected boolean tryReleaseShared(int releases) {
    // 自旋循环
    for (;;) {
        // 先获取出 state
        int c = getState();
        // 如果 state 是 0
        if (c == 0)
            // 直接返回 false
            return false;
        // 否则进行减 1
        int nextc = c-1;
        // 将新的值通过 cas 更新
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

上面方法我们可以看到获取当前的 state 值，如果已经为 0 了，直接返回 false.由此可见，该方法只有在 count 值原来不为 0，但是调用后变为 0 时，才会返回 true，否则就会返回 false.也就是说只有所有子任务都完成减 1 后，在最后一个任务减完 state 就是 0 了就会返回 true，然后在调用 AQS 中的 doReleaseShared() 方法来唤醒主线程来继续执行后面的任务.

public final void await()

```java
public void await() throws InterruptedException {
    // 调用 AQS 的 acquireSharedInterruptibly 方法传入 1
    sync.acquireSharedInterruptibly(1);
}

// AbstractQueuedSynchronizer 类中的方法
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // tryAcquireShared 方法在 CountDownLatch 中被重写
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

// CountDownLatch 重写的方法
protected int tryAcquireShared(int acquires) {
    // 如果 state 的值为 0 则返回 1 否则返回 -1
    return (getState() == 0) ? 1 : -1;
}
```

上面方法我们看到只有在 state 值 > 0 的时候才会直接退出说明所有等待的子任务都已经完成了，否则会进入 doAcquireSharedInterruptibly() 方法中。

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    // 创建一个 node 并放入队列中
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            // 如果自己上一个节点是 head
            final Node p = node.predecessor();
            if (p == head) {
                // 则再次尝试获取 return (getState() == 0) ? 1 : -1; 看是否成功
                // tryAcquireShared 方法会被各种重写，这里的只代码 CountDownLatch 中的方法
                int r = tryAcquireShared(arg);
                // 如果返回 1
                if (r >= 0) {
                    // 则将自己设置为 head 并返回
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            // 如果上面条件没有满足则进入阻塞状态
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

上面的代码我们可以知道主线程在调用 await 方法后会去查看所有子任务是否都已经完成，如果没有完成则进入到阻塞当中，直到最后一个子线程将 state 变为 0 时调用 doReleaseShared() 函数来将它给唤醒。

### 示例

我们用 jdk 官方的示例看一看

```java
class Driver { // ...
    void main() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);

        for (int i = 0; i < N; ++i) // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();

        doSomethingElse();            // don't let run yet
        startSignal.countDown();      // let all threads proceed
        doSomethingElse();
        doneSignal.await();           // wait for all to finish
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    public void run() {
        try {
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```

上面的例子我们可以看到一共有两个 CountDownLatch 一个是 startSignal(1) 一个是 doneSignal(N).在 Driver 中创建了 N 个线程 Worker 在 Worker 中 调用了 startSignal.await(); 会进入阻塞，只有 Driver 主线程执行到第 10 行 startSignal.countDown(); 的时候所有的 Worker 线程才可以继续往下去执行，并且在 doWork() 执行完后会在第 29 行调用 doneSignal.countDown(); 只有在最后一个 Worker 执行完才能唤醒 Driver 主线程，因为主线程在第 12 行执行了 doneSignal.await() 来等待所有 Worker.代码本身的逻辑非常简单好懂，这里不继续赘述了。