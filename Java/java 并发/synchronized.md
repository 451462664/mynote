# java 并发——synchronized

---

https://www.cnblogs.com/dennyzhangdd/p/6734638.html

## 介绍

在平常我们开发的过程中可能会遇到线程安全性的问题，为了保证线程之间操作数据的正确性，我们第一想到的可能就是使用 synchronized 并且 synchronized 使用的位置也是很有讲究的.首先我们来先看一下什么是 synchronized ？

1. 需要使得代码变为同步方法我们需要使用 synchronized 来修饰执行前会先去获取锁，执行完释放锁，执行期间其他线程会等待.
2. synchronized 有使用方法修饰在方法前面和修饰代码块.
3. synchronized 以性能为代价来保证数据的完整性，所以非必要勿用.
4. synchronized 只适用于单 jvm 虚拟机内，如果项目是集群的那么 synchronized 将不能保证数据是安全的.

### 使用场景说明

1. 如果 synchronized 修饰在实例方法上，那么当前的锁对象就是该类的实例对象.

    ```java
    public synchronized void test() {
        //...
    }
    ```

2. 如果 synchronized 修饰在静态方法上，那么当前的锁对象就是类对象.

    ```java
    public static synchronized void test() {
        //...
    }
    ```

3. 如果 synchronized 修饰同步代码块中是 this 那么当前锁对象还是类的实例对象.

    ```java
    public void test() {
        synchronized(this) {
            //...
        }
    }
    ```

4. 如果 synchronized 修饰同步代码块中是 Class 那么当前锁对象就是类对象.

    ```java
    public void test() {
        synchronized(synchronizedDemo.class) {
            //...
        }
    }
    ```

5. 如果 synchronized 修饰同步代码块中是任意一个对象那么当前锁对象就是这个对象实例.

    ```java
    private final Object lock = new Object();
    
    public void test() {
        synchronized(lock) {
            //...
        }
    }
    ```

## 是如何实现的呢？

我们先来先写一个例子

```java
public class SynchronizedTest {

    public synchronized void test01() {
        System.out.println("test01");
    }

    public void test02() {
        synchronized (this) {
            System.out.println("test02");
        }
    }
}
```

之后我们先通过 javac 来编译成 class 文件之后再 javap 查看 class 文件发现如下图

![1567666073594](D:\Typora\image\1567666073594.png)

我们通过上图发现 test01 这个方法给打上了一个标识 ACC_SYNCHRONIZED 来实现的。
而我们的 test02 同步代码块使用了 monitorenter 指令插入到了代码块开始的位置，monitorexit 指令插入到代码块结束的位置。必须是成对出现的.

在上面我们对 synchronized 已经有了一个大致的认识了，但是我们继续想学习锁的知识，就必须涉及到 cas 操作和 java 对象头了.

## cas 操作

cas: compare and swap.

百度百科: cas 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。

简单一点来说就是 cas 有 3 个操作数，内存值 V，旧的预期值 A，要修改的新值 B。如果 A = V，那么把 B 赋值给 V，返回 V；如果 A != V，直接返回 V。所以为了提高性能 jvm 很多操作都是依赖 cas 来实现的，cas 也就是乐观锁的实现.

## java 对象头

java 对象头包含两个部分

```
|--------------------------------------------------------------|
|                     Object Header (64 bits)                  |
|------------------------------------|-------------------------|
|        Mark Word (32 bits)         |    Klass Word (32 bits) |
|------------------------------------|-------------------------|
```

1. Mark World: 主要用于存储自身运行时的数据哈希码、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 id、偏向时间戳等等.
2. Klass Pointer: 类型指针.指向它的类元数据的指针，虚拟机通过这个指针来确定对象是哪个类的实例.

对象头中的 Mark Word，synchronized 实现就用了 Mark Word 来标识对象加锁状态.下面是 32 位虚拟机头的存储结构

![img](D:\Typora\image\584866-20170420091115212-1624858175.jpg)

看了上图我们发现有好几种锁偏向锁、轻量级锁、重量级锁(synchronized)其实是 jdk1.6 中对锁的实现引入了大量的优化来减少锁操作的开销：

**首先我们来看下锁的枚举**: 00 偏向锁、01 无锁、10 监视器锁，又叫重量级锁、11 GC标记、101 偏向锁.

1. 锁粗化: 将多个连续的锁扩展成一个大范围的锁，用来减少频繁的互斥获取锁释放锁导致的性能消耗.

2. 锁消除: jvm 通过检测判断一段代码中不可能存在共享数据竞争，jvm 会对这些同步锁进行锁消除。锁消除的依据是逃逸分析的数据支持。

    ```java
    @Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
    
    public void test() {
        StringBuffer sb = new StringBuffer();
        sb.append("");
    }
    ```

    我们都知道 StringBuffer 是线程安全的，我们虽然没有显示使用锁，但是我们在使用一些 jdk 的内置 API 时，如StringBuffer、Vector、HashTable 等，这个时候会存在隐形的加锁操作。

3. 轻量级锁: 在没有多线程竞争的情况下避免重量级互斥锁，只需要依靠一条 cas 原子指令就可以完成锁的获取及释放.
    ![img](D:\Typora\image\201812081005.png)

4. 偏向锁: 目的是消除数据再无竞争情况下的同步。使用 cas 记录获取它的线程。下一次同一个线程进入则偏向该线程，无需任何同步操作.
    ![img](D:\Typora\image\201812081006.png)

5. 自旋锁: 就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。怎么等待呢？执行一段无意义的循环即可(自旋).

6. 适应自旋锁: jdk1.6 引入了更加聪明的自旋锁，即自适应自旋锁。所谓自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。它怎么做呢？线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。

7. 重量级锁

经过 jdk 的优化所以加锁流程是：偏向锁——>轻量级锁——>重量级锁 并且锁只能升级膨胀并不能降级.

## 总结

这次主要说了 synchronized 原理以及 jdk 对 synchronized 的优化。简单来说解决三种场景：
1）只有一个线程进入临界区，偏向锁.
2）多个线程交替进入临界区，轻量级锁.
3）多线程同时进入临界区，重量级锁.

就好比是你在周末一个人在公司公司突然肚子痛要上厕所，这个时候不需要等待也随便你关不关厕所门就可以上厕所(偏向锁)哈哈哈.但是在工作日的话你去上厕所发现测试门被关闭了这个时候你选择站在门口等待一会(自旋锁).之后你等待了 5 分钟后发现没什么希望，你就回到座位去等了(锁膨胀重量级锁).例子可能举的不是那么明确，但是也只能想到这样了.感谢观看！