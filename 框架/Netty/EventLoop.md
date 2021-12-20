# Netty 源码学习——EventLoop

---

>  在前面 Netty 源码学习——客户端流程分析中我们已经知道了一个 EventLoop 大概的流程，这一章我们来详细的看一看。

## NioEventLoopGroup 类层次结构

我们先来看下 NioEventLoopGroup 这个类。

```java
public class NioEventLoopGroup extends MultithreadEventLoopGroup { 
}
```

发现他的父类是 MultithreadEventLoopGroup。我们有必要来看一下继承结构图。
![1565059564235](D:\Typora\image\1565059564235.png)

### NioEventLoopGroup 实例化流程

我们再来回复习下 NioEventLoopGroup 的实例化流程。

```java
EventLoopGroup boss = new NioEventLoopGroup();
EventLoopGroup work = new NioEventLoopGroup();
```

在内部构造器调用到最后会出现在这一行。

```java
public NioEventLoopGroup(int nThreads, Executor executor, SelectorProvider selectorProvider, SelectStrategyFactory selectStrategyFactory) {
    super(nThreads, executor, new Object[]{selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject()});
}
```

**上面参数分别是**

1. nThreads 线程数量

2. exceutor 执行 runnable 的执行器实现了 Executor 接口，内部维护了一个线程工厂，在调用 public void execute(Runnable command); 的时候会新建一个线程。

3. selectorProvider

    1. SelectorProvider provider 属性: NioEventLoopGroup 构造器中通过 SelectorProvider.provider() 获取一个 SelectorProvider。
    2. Selector selector 属性: NioEventLoop 构造器中通过调用通过 selector = provider.openSelector() 获取一个 selector 对象。

4. selectStrategyFactory 主要用于选择策略。

    1. ```java
        public int calculateStrategy(IntSupplier selectSupplier, boolean hasTasks) throws Exception {
            return hasTasks ? selectSupplier.get() : -1;
        }
        ```

    2. 判断任务队列是否有任务，如果有，那么调用一次 selectNow 方法，并返回 selectNow 的结果，如果没有，返回 -1。

我们接着往下看发现最终会在 io.netty.util.concurrent.MultithreadEventExecutorGroup#MultithreadEventExecutorGroup(int, java.util.concurrent.Executor, io.netty.util.concurrent.EventExecutorChooserFactory, java.lang.Object...) 中完成初始化操作。后面的流程在之前客户端解析那一章节已经解释，再次不在说了。

```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                        EventExecutorChooserFactory chooserFactory, Object... args) {
    if (nThreads <= 0) {
        throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
    }

    if (executor == null) {
        executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
    }

    children = new EventExecutor[nThreads];

    for (int i = 0; i < nThreads; i ++) {
        try {
            children[i] = newChild(executor, args);
        } catch (Exception e) {
            // 省略不必要的代码只说明初始化操作
        } finally {
            // 省略不必要的代码只说明初始化操作
        }
    }

    chooser = chooserFactory.newChooser(children);
    // 省略不必要的代码只说明初始化操作
}
```

**总结**

1. 由此看来 EventLoopGroup 就是在父类中维护了一个 nThreads 长度的 EventExecutor 数组。
2. 在此方法内部会调用 newChild(executor, args); 来初始化 children 数组。
3. chooser 根据数组的长度来选择出 EventExecutorChooser。

## NioEventLoop

我们还是先来看下继承结构。

```java
public final class NioEventLoop extends SingleThreadEventLoop {
}
```

![1565061327740](D:\Typora\image\1565061327740.png)

我们可以看到几个熟悉的面孔 Executor、ExecutorService、ScheduledExecutorService 等等。那么我们可以认为 NioEventLoop 就是用来做线程调度的。因为他实现了 execute 方法来向任务队列中添加一个 task, 并由 NioEventLoop 进行调度执行。NioEventLoop 构造器的执行链 NioEventLoop -> SingleThreadEventLoop -> SingleThreadEventExecutor -> AbstractScheduledEventExecutor -> AbstractEventExecutor

### EventLoop 是在哪里和 Channel 的绑定的

还记得 io.netty.channel.AbstractChannel.AbstractUnsafe#register 这个方法吗？

```java
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    if (eventLoop == null) {
    } else if (AbstractChannel.this.isRegistered()) {
    } else if (!AbstractChannel.this.isCompatible(eventLoop)) {
    } else {
        // 绑定关系
        AbstractChannel.this.eventLoop = eventLoop;
        if (eventLoop.inEventLoop()) {
            this.register0(promise);
        } else {
            try {
                eventLoop.execute(new Runnable() {
                    public void run() {
                        AbstractUnsafe.this.register0(promise);
                    }
                });
            } catch (Throwable var4) {
            }
        }

    }
}
```

AbstractChannel.this.eventLoop = eventLoop; 将 eventLoop 维护到 AbstractChannel 的 private volatile EventLoop eventLoop; 成员变量中。

我们在上面代码还能发现 eventLoop.execute(); 的执行，因为执行到这里是 ServerBootStrap 主线程执行的，所以不会进去 if (eventLoop.inEventLoop()); 分支会走 else 的 eventLoop.execute(); 点进去我们看代码发现是 SingleThreadEventExecutor 里的。

```java
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    } else {
        boolean inEventLoop = this.inEventLoop();
        this.addTask(task);
        if (!inEventLoop) {
            // 重点看这行
            this.startThread();
            if (this.isShutdown()) {
                boolean reject = false;
                try {
                    if (this.removeTask(task)) {
                        reject = true;
                    }
                } catch (UnsupportedOperationException var5) {
                }
                if (reject) {
                    reject();
                }
            }
        }
        if (!this.addTaskWakesUp && this.wakesUpForTask(task)) {
            this.wakeup(inEventLoop);
        }
    }
}
```

看到上面代码 this.startThread(); 直接启动一个线程。

```java
private void startThread() {
    if (this.state == 1 && STATE_UPDATER.compareAndSet(this, 1, 2)) {
        try {
            this.doStartThread();
        } catch (Throwable var2) {
            STATE_UPDATER.set(this, 1);
            PlatformDependent.throwException(var2);
        }
    }
}
```

我们会看到当 state 等于 1 的时候会去执行里面的代码，在 SingleThreadEventExecutor 初始化也就是 NioEventLoop 初始化的时候已经把 state 赋值成了 1，也就是说只有在第一次调用的 时候才会是 1。

```java
protected SingleThreadEventExecutor(EventExecutorGroup parent, Executor executor, boolean addTaskWakesUp, int maxPendingTasks, RejectedExecutionHandler rejectedHandler) {
    super(parent);
    this.threadLock = new Semaphore(0);
    this.shutdownHooks = new LinkedHashSet();
    // 将状态赋值为 1
    this.state = 1;
    this.terminationFuture = new DefaultPromise(GlobalEventExecutor.INSTANCE);
    this.addTaskWakesUp = addTaskWakesUp;
    this.maxPendingTasks = Math.max(16, maxPendingTasks);
    this.executor = ThreadExecutorMap.apply(executor, this);
    this.taskQueue = this.newTaskQueue(this.maxPendingTasks);
    this.rejectedExecutionHandler = (RejectedExecutionHandler)ObjectUtil.checkNotNull(rejectedHandler, "rejectedHandler");
}
```

我们继续回到 startThread(); 方法中，发现里面调用了 this.doStartThread(); 因为代码只保留了关键代码。

```java
private void doStartThread() {
    // 断言
    assert this.thread == null;
    this.executor.execute(new Runnable() {
        public void run() {
            // 给成员属性 thread 赋值
            SingleThreadEventExecutor.this.thread = Thread.currentThread();
            if (SingleThreadEventExecutor.this.interrupted) {
                SingleThreadEventExecutor.this.thread.interrupt();
            }

            boolean success = false;
            SingleThreadEventExecutor.this.updateLastExecutionTime();
            boolean var112 = false;

            int oldState;
            label1907: {
                try {
                    var112 = true;
                    // 调用 NioEventLoop run();
                    SingleThreadEventExecutor.this.run();
                    success = true;
                    var112 = false;
                    break label1907;
                } catch (Throwable var119) {

                } finally {

                }
            }
        });
    }
```

**上面代码主要做了三件事情**

1. 判断 thread 是否为 null
2. 给 thread 赋值
3. 调用 run() 方法

```java
protected void run() {
	for (;;) {
		try {
			try {
				switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
				case SelectStrategy.CONTINUE:
					continue;
				case SelectStrategy.BUSY_WAIT:
					// fall-through to SELECT since the busy-wait is not supported with NIO
				case SelectStrategy.SELECT:
					select(wakenUp.getAndSet(false));
					if (wakenUp.get()) {
						selector.wakeup();
					}
					// fall through
				default:
				}
			} catch (IOException e) {

			}

			cancelledKeys = 0;
			needsToSelectAgain = false;
			final int ioRatio = this.ioRatio;
			if (ioRatio == 100) {
				try {
					processSelectedKeys();
				} finally {
					// Ensure we always run tasks.
					runAllTasks();
				}
			} else {
				final long ioStartTime = System.nanoTime();
				try {
					processSelectedKeys();
				} finally {
					// Ensure we always run tasks.
					final long ioTime = System.nanoTime() - ioStartTime;
					runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
				}
			}
		} catch (Throwable t) {
			handleLoopException(t);
		}
		// Always handle shutdown even if the loop processing threw an exception.
		try {
			if (isShuttingDown()) {
				closeAll();
				if (confirmShutdown()) {
					return;
				}
			}
		} catch (Throwable t) {
			handleLoopException(t);
		}
	}
}
```

看了上面代码，我们大致可以猜测到在一个无限循环里面 NioEventLoop 事件循环的核心就是这里!

### I/O 事件处理

首先我们看到这一行代码 switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())); 

```java
@Override
public int calculateStrategy(IntSupplier selectSupplier, boolean hasTasks) throws Exception {
    return hasTasks ? selectSupplier.get() : SelectStrategy.SELECT;
}
```

主要是查看任务队列里有没有任务，如果有的话直接 selectSupplier.get(); 返回，没有返回 SelectStrategy.SELECT 值为 -1 需要阻塞等待。那么 selectSupplier.get(); 返回的是什么呢，我们发现 NioEventLoop 有一个属性传入到了这个方法中。

```java
private final IntSupplier selectNowSupplier = new IntSupplier() {
    @Override
    public int get() throws Exception {
        return selectNow();
    }
};

int selectNow() throws IOException {
    try {
        return selector.selectNow();
    } finally {
        // restore wakeup state if needed
        if (wakenUp.get()) {
            selector.wakeup();
        }
    }
}
```

 selectNow() 是直接使用了原生 NIO Selector.selectNow() 方法会检查当前是否有就绪的 IO 事件, 如果有, 则返回就绪 IO 事件的个数; 如果没有, 则返回0. `注意, selectNow() 是立即返回的, 不会阻塞当前线程。

需要需要执行 I/O 事件我们就继续 run(); 往下看。

```java
if (ioRatio == 100) {
    try {
        processSelectedKeys();
    } finally {
        // Ensure we always run tasks.
        runAllTasks();
    }
} else {
    final long ioStartTime = System.nanoTime();
    try {
        processSelectedKeys();
    } finally {
        // Ensure we always run tasks.
        final long ioTime = System.nanoTime() - ioStartTime;
        runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
    }
}
```

我们看到会分别执行 processSelectedKeys(); runAllTasks(); 方法我们先看第一个方法。

```java
private void processSelectedKeys() {
    if (selectedKeys != null) {
        processSelectedKeysOptimized();
    } else {
        processSelectedKeysPlain(selector.selectedKeys());
    }
}
```

上面代码首先会判断 selectedKeys 如果不为 null 的话就执行 processSelectedKeysOptimized(); 否则执行 processSelectedKeysPlain(selector.selectedKeys()); 一般情况下是不会为 null 的，因为里面保存了 I/O 事件。所以我们直接跟进 processSelectedKeysOptimized();

```java
private void processSelectedKeysOptimized() {
    for (int i = 0; i < selectedKeys.size; ++i) {
        final SelectionKey k = selectedKeys.keys[i];
        selectedKeys.keys[i] = null;
        final Object a = k.attachment();
        if (a instanceof AbstractNioChannel) {
            processSelectedKey(k, (AbstractNioChannel) a);
        } else {
            @SuppressWarnings("unchecked")
            NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
            processSelectedKey(k, task);
        }
        if (needsToSelectAgain) {
            selectedKeys.reset(i + 1);
            selectAgain();
            i = -1;
        }
    }
}
```

上面代码遍历每一个 SelectionKey 并执行 processSelectedKey(); 方法。其中有一行 final Object a = k.attachment(); 这个 a 是从哪里放进去的呢？就是当服务启动注册 channel 的时候 AbstractNioChannel.doRegister。

```java
@Override
protected void doRegister() throws Exception {
    // 传递了 this 这个 this 就是 channel
    selectionKey = javaChannel().register(eventLoop().selector, 0, this);
}
```

那么我们继续看下 processSelectedKey(); 方法。

```java
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    if (!k.isValid()) {
        final EventLoop eventLoop;
        try {
            eventLoop = ch.eventLoop();
        } catch (Throwable ignored) {
            return;
        }
        if (eventLoop != this || eventLoop == null) {
            return;
        }
        unsafe.close(unsafe.voidPromise());
        return;
    }

    try {
        int readyOps = k.readyOps();
        // 是否是连接事件
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            int ops = k.interestOps();
            // 将连接事件数据位置清 0
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);

            unsafe.finishConnect();
        }
        // 是否是可写事件
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            ch.unsafe().forceFlush();
        }
        // 是否是可读事件
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

看了上面代码主要判断是否是建立连接事件、可写事件、可读事件。

1. OP_CONNECT, 连接建立事件, 即 TCP 连接已经建立, Channel 处于 active 状态.
2. OP_WRITE, 可写事件, 即上层可以向 Channel 写入数据.
3. OP_READ, 可读事件, 即 Channel 中收到了新数据可供上层读取.

#### 连接事件

```java
// 是否是连接事件
if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
    int ops = k.interestOps();
    // 将连接事件数据位置清 0
    ops &= ~SelectionKey.OP_CONNECT;
    k.interestOps(ops);

    unsafe.finishConnect();
}
```

在处理连接事件中只做了两件事情

1. 将连接事件取消掉也就是将连接标识清空为 0。
2. 调用 unsafe.finishConnect() 通知上层连接已建立。

#### 可写事件

调用 forceFlush，一旦没有什么可写的，它也将清除 OP_WRITE。

```java
 if ((readyOps & SelectionKey.OP_WRITE) != 0) {
     ch.unsafe().forceFlush();
 }
```

#### 可读事件

```java
if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
    unsafe.read();
}
```

我们继续跟进 read(); 方法，发现在 AbstractNioByteChannel 中实现。

```java
@Override
public final void read() {
    final ChannelConfig config = config();
    if (shouldBreakReadReady(config)) {
        clearReadPending();
        return;
    }
    final ChannelPipeline pipeline = pipeline();
    final ByteBufAllocator allocator = config.getAllocator();
    final RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
    allocHandle.reset(config);

    ByteBuf byteBuf = null;
    boolean close = false;
    try {
        do {
            byteBuf = allocHandle.allocate(allocator);
            allocHandle.lastBytesRead(doReadBytes(byteBuf));
            if (allocHandle.lastBytesRead() <= 0) {
                byteBuf.release();
                byteBuf = null;
                close = allocHandle.lastBytesRead() < 0;
                if (close) {
                    readPending = false;
                }
                break;
            }
            allocHandle.incMessagesRead(1);
            readPending = false;
            pipeline.fireChannelRead(byteBuf);
            byteBuf = null;
        } while (allocHandle.continueReading());

        allocHandle.readComplete();
        pipeline.fireChannelReadComplete();

        if (close) {
            closeOnRead(pipeline);
        }
    } catch (Throwable t) {
        handleReadException(pipeline, byteBuf, t, close, allocHandle);
    } finally {
        if (!readPending && !config.isAutoRead()) {
            removeReadOp();
        }
    }
}
```


上面 read() 做了如下工作:

1. byteBuf = allocHandle.allocate(allocator); 分配 ByteBuf
2. allocHandle.lastBytesRead(doReadBytes(byteBuf)); 从 SocketChannel 中读取数据
3. 调用 pipeline.fireChannelRead 发送一个 inbound 事件.

ChannelPipeline 已经在前面介绍过了，此章就不继续说了。至此 I/O 事件处理就说完了。

### 任务处理

现在让我们回到之前的代码。

```java
if (ioRatio == 100) {
    try {
        processSelectedKeys();
    } finally {
        // Ensure we always run tasks.
        runAllTasks();
    }
}
```

processSelectedKeys(); 我们已经看完了，现在将看下 runAllTasks();

```java
protected boolean runAllTasks() {
    assert inEventLoop();
    boolean fetchedAll;
    boolean ranAtLeastOne = false;

    do {
        fetchedAll = fetchFromScheduledTaskQueue();
        if (runAllTasksFrom(taskQueue)) {
            ranAtLeastOne = true;
        }
    } while (!fetchedAll);

    if (ranAtLeastOne) {
        lastExecutionTime = ScheduledFutureTask.nanoTime();
    }
    afterRunningAllTasks();
    return ranAtLeastOne;
}
```

上面代码 fetchedAll = fetchFromScheduledTaskQueue(); 将 ScheduledTaskQueue 中可以执行的任务放入到 taskQueue 中。if (runAllTasksFrom(taskQueue)) {} 执行任务队列中的任务。

```java
protected final boolean runAllTasksFrom(Queue<Runnable> taskQueue) {
    Runnable task = pollTaskFrom(taskQueue);
    if (task == null) {
        return false;
    }
    for (;;) {
        safeExecute(task);
        task = pollTaskFrom(taskQueue);
        if (task == null) {
            return true;
        }
    }
}

protected static void safeExecute(Runnable task) {
    try {
        task.run();
    } catch (Throwable t) {
    }
}

protected static Runnable pollTaskFrom(Queue<Runnable> taskQueue) {
    for (;;) {
        Runnable task = taskQueue.poll();
        if (task == WAKEUP_TASK) {
            continue;
        }
        return task;
    }
}
```

就是从 taskQueue 中不断的 poll 出一个可执行的 task 进行 run 执行。

事件有限至此 EventLoop 匆匆看完，如有不对的地方，请大家指出，感谢观看。