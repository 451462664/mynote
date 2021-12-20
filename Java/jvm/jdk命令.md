# 常用命令

## jps

jps(JVM Process Status Tool)虚拟机进程状态工具

可以列出正在运行的虚拟机进程，并显示虚拟机执行主类(Main Class，main()函数所在的类)的名称，以及这些进程的本地虚拟机的唯一ID(**LVMID**,Local Vitual Machine Identifier)，它是使用频率最高的 JDK 命令行工具，因为其他 JDK 工具大多需要输入它查询到的 LVMID 来确定要监控的是哪一个虚拟机进程。

命令格式: jps [option] [hostid]

### option

| 选项 | 作用                                                   |
| :--- | :----------------------------------------------------- |
| -q   | 只输出 LVMID,省略主类的名称                            |
| -m   | 输出虚拟机进程启动时传递给 main() 函数的参数           |
| -l   | 输出主类的全名，如果进程执行的是 jar 包，输出 jar 路径 |
| -v   | 输出虚拟机进程启动时 JVM 参数                          |

## jstat

jstat(JVM Statistics Monitorning Tool)虚拟机统计信息监控工具

用于监控虚拟机各种运行状态信息的命令行工具。它可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT 编译等运行数据，它是运行期定位虚拟机性能问题的首选工具。

命令格式: jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

参数 interval 和 count 代表查询间隔和次数，如果省略这两个参数，说明只查询一次。建设需要每 250 毫秒查询一次进程 2764 垃圾收集的情况，一共查询 20 次，那么命令应该是：**jstat -gc 2764 250 20**

### option

| 选项              | 作用                                                         |
| :---------------- | :----------------------------------------------------------- |
| -class            | 监视类装载、卸载数量、中空间及类装载所耗费的时间             |
| -gc               | 监视 java堆状况，包括 Eden 区、2 个 Survivor 区、老年代、永久代等容量、已用空间、GC 合计时间等信息 |
| -gccapacity       | 监视内容与 -gc 基本相同，但输出主要关注 java 堆各区域使用到的最大和最小空间 |
| -gcutil           | 监控内容与 -gc 基本相同，但输出主要关注已使用空间占总空间的百分比 |
| -gccause          | 与 -gcutil 功能一样，但是会额外输出导致上一次 GC 产生的原因  |
| -gcnew            | 监视新生代 GC 的状况                                         |
| -gcnewcapacity    | 监视内容与 -gcnew 基本相同，输出主要关注使用到的最大和最小空间 |
| -gcold            | 监视老年代 GC 的状况                                         |
| -gcoldcapacity    | 监视内容与 -gcold 基本相同，输出主要关注使用到的最大和最小空间 |
| -gcpermcapacity   | 输出永久代使用到的最大和最小空间                             |
| -compiler         | 输出 JIT 编译器编译过的方法、耗时等信息                      |
| -printcompilation | 输出已被 JIT 编译的方法                                      |

### --gc 垃圾回收统计

| 列名 | 说明                                                  |
| :--- | :---------------------------------------------------- |
| S0C  | 年轻代中第一个survior（幸存区）的容量（kb）           |
| S1C  | 年轻代中第二个survior（幸存区）的容量（kb）           |
| S0U  | 年轻代中第一个survior（幸存区）目前已使用的容量（kb） |
| S1U  | 年轻代中第二个survior（幸存区）目前已使用的容量（kb） |
| EC   | eden区的容量（kb）                                    |
| EU   | eden区目前已使用的容量（kb）                          |
| OC   | 老年代的容量（kb）                                    |
| OU   | 老年代目前已使用的容量（kb）                          |
| PC   | perm永久代的容量（kb）                                |
| PU   | perm永久代目前已使用的容量（kb）                      |
| YGC  | 从应用程序启动到采集时年轻代中gc次数                  |
| YGCT | 从应用程序启动到采集时年轻代中gc所用时间（秒）        |
| FGC  | 从应用程序启动到采集时老年代中gc次数                  |
| FGCT | 从应用程序启动到采集时老年代gc所用的时间（秒）        |
| GCT  | 从应用程序启动到采集时gc所用的总时间（秒）            |

### --gcutil 统计 gc 信息

| 列名 | 说明                                             |
| :--- | :----------------------------------------------- |
| S0   | 年轻代中第一个（survisor）幸存区已使用的容量占比 |
| S1   | 年轻代中第二个（survisor）幸存区已使用的容量占比 |
| E    | 伊旬园（eden）区已使用的容量占比                 |
| O    | 老年代区已使用的容量占比                         |
| P    | 永久代（perm）已使用的容量占比                   |
| YGC  | 年轻代到目前gc次数                               |
| YGCT | 年轻代到目前gc耗费的总时间（秒）                 |
| FGC  | 老年代目前gc次数                                 |
| FGCT | 老年代目前gc耗费的总时间（秒）                   |
| GC   | 从应用程序到目前gc总耗时（秒）                   |

### -gccapacity 堆内存统计

| 列名  | 说明                                         |
| :---- | :------------------------------------------- |
| NGCMN | 年轻代（young）中初始化（最小）的大小（kb）  |
| NGCMX | 年轻代（young）中初始化（最大）的大小（kb）  |
| NGC   | 年轻代（young）中当前的容量（kb）            |
| S0C   | 年轻代中第一个（survisor）幸存区的容量（kb） |
| S1C   | 年轻代中第二个（survisor）幸存区的容量（kb） |
| EC    | 年轻代中（Eden)伊旬园的容量（kb）            |
| OGCMN | 老年代（old）中初始化（最小）的容量（kb）    |
| OGCMX | 老年代（old）中初始化（最大）的容量（kb）    |
| OGC   | 当前老年代的大小（kb）                       |
| OC    | 当前老年代的大小（kb）                       |
| PGCMN | 永久代（perm）中初始化（最小）的大小（kb）   |
| PGCMX | 永久代（perm）中初始化（最大）的大小（kb）   |
| PGC   | 永久代当前的大小（kb）                       |
| PC    | 永久代当前的大小（kb）                       |
| YGC   | 从应用程序启动到采集时年轻代gc的次数         |
| FGC   | 从应用程序启动带采集时老年代gc的次数         |

### -gcnew 新生代垃圾回收统计

| 列名 | 说明                                                   |
| :--- | :----------------------------------------------------- |
| S0C  | 年轻代中第一个（survisor）幸存区的容量（kb）           |
| S1C  | 年轻代中第二个（survisor）幸存区的容量（kb）           |
| S0U  | 年轻代中第一个（survisor）幸存区目前已使用的容量（kb） |
| S1U  | 年轻代中第二个（survisor）幸存区目前已使用的容量（kb） |
| TT   | 对象在新生代中存活的次数                               |
| MTT  | 对象在新生代中存活的最大次数                           |
| DSS  | 当前需要survivor(幸存区)的容量 (kb)                    |
| EC   | 伊旬园（eden）区的大小（kb）                           |
| EU   | 伊旬园（eden）区已使用的大小（kb）                     |
| YGC  | 到目前年轻代gc的次数                                   |
| YGCT | 到目前年轻代gc所耗费的时间（秒）                       |

### -gcnewcapacity 新生代内存统计

| 列名  | 说明                                         |
| :---- | :------------------------------------------- |
| MGCMN | 年轻代中初始化最小容量（kb）                 |
| MGCMX | 年轻代中初始化最大容量（kb）                 |
| NGC   | 年轻代当前容量（kb）                         |
| S0CMX | 年轻代第一个幸存区（survisor）最大容量（kb） |
| S0C   | 年轻代第一个幸存区（survisor）当前容量（kb） |
| S1CMX | 年轻代第二个幸存区（survisor）最大容量（kb） |
| S1C   | 年轻代第二个幸存区（survisor）当前容量（kb） |
| ECMX  | 年轻代伊旬园区（Eden）最大容量（kb）         |
| EC    | 年轻代伊旬园区（Eden）当前容量（kb）         |
| YGC   | 截止到目前年轻代gc次数                       |
| FGC   | 截止到目前老年代gc次数                       |

### -gcold 老年代垃圾回收统计

| 列名 | 说明                           |
| :--- | :----------------------------- |
| PC   | 永久区（perm）容量（kb）       |
| PU   | 永久区（perm）已使用容量（kb） |
| OC   | 老年代容量（kb）               |
| OU   | 老年代已使用容量（kb）         |
| YGC  | 截止到目前年轻代gc次数         |
| FGC  | 截止到目前老年代gc次数         |
| GCT  | 截止到目前gc耗费的总时间（秒） |

### -gcoldcapacity 老年代内存统计

| 列名  | 说明                                 |
| :---- | :----------------------------------- |
| OGCMN | 老年代最小容量（kb）                 |
| OGCMX | 老年代最大容量（kb）                 |
| OGC   | 老年代目前生成的容量（kb）           |
| OC    | 老年代目前容量（kb）                 |
| YGC   | 截止到目前年轻代gc次数               |
| FGC   | 截止到目前老年代gc次数               |
| FGCT  | 截止到目前老年代gc耗费的总时间（秒） |
| GCT   | 截止到目前gc耗费的总时间（秒）       |

### -gcpermcapacity 永久代内存统计

| 列名  | 说明                               |
| :---- | :--------------------------------- |
| PGCMN | 永久代最小容量（kb）               |
| PGCMX | 永久代最大容量（kb）               |
| PGC   | 永久代当前生成的容量（kb）         |
| PC    | 永久代当前容量（kb）               |
| YGC   | 截止目前年轻代gc次数               |
| FGC   | 截止目前老年代gc次数               |
| FGCT  | 截止目前年轻代gc耗费的总时间（秒） |
| GCT   | 截止目前老年代gc耗费的总时间（秒） |

### -gccause 最近两次 gc 统计

| 列名 | 说明               |
| :--- | :----------------- |
| LGCC | 最近垃圾回收的原因 |
| GCC  | 当前垃圾回收的原因 |

### -printcompilation jvm 编译方法统计

| 列名     | 说明                     |
| :------- | :----------------------- |
| Compiled | 最近编译方法的数量       |
| Size     | 最近编译方法的字节码数量 |
| Type     | 最近编译方法的编译类型   |
| Method   | 方法名标识               |

## jmap

jmap(Memory Map For Java)内存映射工具

主要用于打印指定 Java 进程(或核心文件、远程调试服务器)的共享对象内存映射或堆内存细节。jmap 命令可以获得运行中的 jvm 的堆的快照，从而可以离线分析堆，以检查内存泄漏，检查一些严重影响性能的大对象的创建，检查系统中什么对象最多，各种对象所占内存的大小等等。可以使用 jmap 生成 Heap Dump。

如果不想使用 jmap 命令，要想获取 java 堆转储快照还有一些比较“暴力”的手段：譬如在前面用过的 -XX:+HeapDumpOnOutOfMemoryError 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，通过 -XX:+HeapDumpOnCtrlBreak 参数可以使用 [ctrl]+[Break] 键让虚拟机生成 dump 文件，又或者在 Linux 系统下通过 Kill -3 命令发送进程退出信息“恐吓”一下虚拟机，也能拿到 dump 文件。

jmap 的作用并不仅仅是为了获取 dump 文件，他还可以查询 finalize 执行队列，java 堆和永久代的详细信息，如空间使用率、当前用的是哪种收集器等。

命令格式: jmap [option] <pid> or jmap [option] [server_id@]<remote server IP or hostname>

### option

| 选项           | 作用                                                         |
| :------------- | :----------------------------------------------------------- |
| -dump          | 生成 java 堆转储快照，格式为：-dump:[live,]format=b,file=<filename>,其中 live 子参数说明是否只 dump 出存活对象 |
| -finalizerinfo | 显示在 F-Queue 中等待 Finalizer 线程执行 finalize 方法的对象，只在 linux/solaris平台下有效 |
| -heap          | 显示堆详细信息，如使用哪种回收期、参数配置、分带状况等，只在 linux/solaris 平台下有效 |
| -histo         | 显示堆中对象统计信息，包括类、实例数量和合计容量             |
| -permstat      | 以 ClassLoader 为统计口径显示永久代内存状况，只在 linux/solaris 平台下有效 |
| -F             | 当虚拟机进程对 -dump 选项没有响应时，可以使用这个选项强制生成 dump 快照，只在linux/solaris 平台下有效 |

### -dump

生成 java 对转存快照 jmap -dump:[live,]format=b,file=文件名 <pid>
可以使用 jdk 提供的 jvisualvm.exe 查看 hprof 文件

### -heap

显示堆详细信息 jmap -heap <pid>

### -histo

显示堆中对象统计信息，包括类、实例数量和合计容量 jmap -histo[:live] <pid>

## jstack

jstack(Stack Trace For Java)是 java 虚拟机自带的一种堆栈跟踪工具。jstack 用于打印出给定的 java 进程 ID或 core file 或远程调试服务的 Java 堆栈信息，如果是在 64 位机器上，需要指定选项 "-J-d64"，Windows 的jstack 使用方式只支持以下的这种方式

jstack [-l] pid

1. 针对活着的进程做本地的或远程的线程 dump
2. 针对 core 文件做线程 dump

jstack 用于生成 java 虚拟机当前时刻的线程快照。
线程快照是当前 java 虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。
线程出现停顿的时候通过 jstack 来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。
如果 java 程序崩溃生成 core 文件，jstack 工具可以用来获得 core 文件的 java stack 和 native stack 的信息，从而可以轻松地知道 java 程序是如何崩溃和在程序何处发生问题。
另外，jstack 工具还可以附属到正在运行的 java 程序中，看到当时运行的 java 程序的 java stack 和 native stack 的信息, 如果现在运行的 java 程序呈现 hung 的状态，jstack 是非常有用的。

### 线程状态

1. NEW：未启动的。不会出现在 dump 中。
2. RUNNABLE：在虚拟机内执行的。运行中状态，可能里面还能看到 locked 字样，表明它获得了某把锁。
3. BLOCKED：受阻塞并等待监视器锁。被某个锁(synchronizers)給 block 住了。
4. WATING：无限期等待另一个线程执行特定操作。等待某个 condition 或 monitor 发生，一般停留在 park(), wait(),sleep(),join() 等语句里。
5. TIMED_WATING：有时限的等待另一个线程的特定操作。和 WAITING 的区别是 wait() 等语句加上了时间限制 wait(timeout)。
6. TERMINATED：已退出的。

命令格式: jstack 5661
jstack [ option ] pid
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP

### options

| 列名                                                         | 说明                                      |
| :----------------------------------------------------------- | :---------------------------------------- |
| executable Java executable from which the core dump was produced | 可能是产生 core dump 的 java 可执行程序   |
| core                                                         | 将被打印信息的 core dump 文件             |
| remote-hostname-or-IP                                        | 远程 debug 服务的主机名或 ip              |
| server-id                                                    | 唯一 id,假如一台主机上多个远程 debug 服务 |

### 基本参数

| 列名        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| -F          | 当 jstack [-l] pid 没有响应的时候，强制打印线程堆栈信息，一般情况不需要使用 |
| -l          | 长列表. 打印关于锁的附加信息，例如属于 java.util.concurrent 的 ownable synchronizers 列表，会使得 jvm 停顿得长久得多（可能会差很多倍，比如普通的 jstack 可能几毫秒和一次 GC 没区别，加了 -l 就是近一秒的时间），-l 建议不要用，一般情况不需要使用 |
| -m          | 打印 java 和 native c/c++ 框架的所有栈信息.可以打印 JVM 的堆栈，显示上 Native 的栈帧，一般应用排查不需要使用 |
| -h \| -help | 打印帮助信息                                                 |
| pid         | 需要被打印配置信息的 java 进程 id，可以用 jps 查询           |

