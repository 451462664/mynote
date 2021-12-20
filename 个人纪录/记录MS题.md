## JVM

程序计数器、虚拟机栈、本地方法栈、java堆、方法区

JVM堆内存被分为两部分：年轻代（Young Generation）和⽼年代（Old Generation）。
年轻代是所有新对象产⽣的地⽅。当年轻代内存空间被⽤完时，就会触发垃圾回收。这个垃圾回收叫做
Minor GC。年轻代被分为3个部分——Enden区和两个Survivor区。

年轻代空间的要点：
1. ⼤多数新建的对象都位于Eden区。
2. 当Eden区被对象填满时，就会执⾏Minor GC，并把所有存活下来的对象转移到其中⼀个survivor
区。
3. Minor GC同样会检查存活下来的对象，并把它们转移到另⼀个survivor区。这样在⼀段时间内，总
会有⼀个空的survivor区。
4. 经过多次GC周期后，仍然存活下来的对象会被转移到年⽼代内存空间，通常这是在年轻代有资格提
升到年⽼代前通过设定年龄阈值来完成的。

年⽼代
年⽼代内存⾥包含了⻓期存活的对象和经过多次Minor GC后依然存活下来的对象，通常会在⽼年代内存
被占满时进⾏垃圾回收。

## 线程池

**ExecutorService**。

java中的有哪些线程池？newCachedThreadPool创建一个可缓存线程池程、newFixedThreadPool 创建一个定长线程池、newScheduledThreadPool 创建一个定长线程池、newSingleThreadExecutor 创建一个单线程化的线程池。
corePoolSize 线程池核心线程大小、maximumPoolSize 线程池最大线程数量、keepAliveTime 空闲线程存活时间、workQueue 工作队列、threadFactory 线程工厂、handler 拒绝策略。
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务。

## Redis

### 数据结构

字符串String、字典Hash、列表List、集合Set、有序集合SortedSet。

### 分布式锁

先拿setnx来争抢锁，抢到之后，再⽤expire给锁加⼀个过期时间防⽌锁忘记了释放。我记得set指令有⾮常复杂的参
数，这个应该是可以同时把setnx和expire合成⼀条指令来⽤的！

### 淘汰策略

⼀般的剔除策略有 FIFO 淘汰最早数据、LRU 剔除最近最少使⽤、和 LFU 剔除最近使⽤频率最低的数
据⼏种策略。
noeviction:返回错误当内存限制达到并且客户端尝试执⾏会让更多内存被使⽤的命令（⼤部分的写
⼊指令，但DEL和⼏个例外）
allkeys-lru: 尝试回收最少使⽤的键（LRU），使得新添加的数据有空间存放。
volatile-lru: 尝试回收最少使⽤的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间
存放。
allkeys-random: 回收随机的键使得新添加的数据有空间存放。
volatile-random: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。
volatile-ttl: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空
间存放。
如果没有键满⾜回收的前提条件的话，策略volatile-lru, volatile-random以及volatile-ttl就和
noeviction 差不多了。

### 先操作数据库 VS 先操作缓存

解决这个问题的方向是：**如果出现不一致，谁先做对业务的影响较小，就谁先执行。**
假设先写数据库再写缓存第一步写数据库操作成功，第二步淘汰缓存失败，则会出现DB中是新数据，Cache中是旧数据，数据不一致。
假设先淘汰缓存，再写数据库第一步淘汰缓存成功，第二步写数据库失败，则只会引发一次Cache miss。
**结论：数据和缓存的操作时序，结论是清楚的：先淘汰缓存，再写数据库。**

## MQ

### 为什么使用 MQ

我们公司本身的业务体量很⼩，所以直接单机⼀把梭啥都能搞定了，但是后⾯业务体量不
断扩⼤，采⽤微服务的设计思想，分布式的部署⽅式，所以拆分了很多的服务，随着体量的增加以及业
务场景越来越复杂了，很多场景单机的技术栈和中间件以及不够⽤了，⽽且对系统的友好性也下降了，
最后做了很多技术选型的⼯作，我们决定引⼊消息队列中间件。

这三个场景也是消息队列的经典场景，⼤家基本上要烂熟于⼼那种，就是⼀说到消息队列你脑⼦
就要想到异步、削峰、解耦，条件反射那种。

### 区别

ActiveMQ和RabbitMQ这两着因为吞吐量还有GitHub的社区活跃度的原因，在
各⼤互联⽹公司都已经基本上绝迹了，业务体量⼀般的公司会是有在⽤的，但是越来越多的公司更⻘
睐RocketMQ这样的消息中间件了。

就拿吞吐量来说，早期⽐较活跃的ActiveMQ 和RabbitMQ基本上不
是后两者的对⼿了，在现在这样⼤数据的年代吞吐量是真的很重要。

![1597445289376](D:\Typora\image\1597445289376.png)

```shell
1、如何给很长的网址映射成6位短地址（64进制）
2、kafka为什么这么快
3、使用过什么设计模式
4、nio了解吗
5、netty用过没有
6、项目中使用的技术栈
7、分布式锁的实现
8、zk有什么特点，怎么实现分布式锁（强一致性）
9、分布式的三个原则（？ap。。忘了）
10、数据库的隔离级别
11、sync、cas的实现原理，是否需要底层指令的支持
12、sync锁升级的概念
13、java栈中除了基本信息还保存了什么？
14、redis怎么实现分布式锁
```

