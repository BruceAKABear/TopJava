# jvm Java 虚拟机

栈管运行，堆管存储。

## 类加载器

字节码加载以后放在方法区。类加载器只负责 class 文件的加载，是否可运行由执行引擎决定。

- 启动类加载器 Bootstrap 使用 c++语言写的。其他都是 Java 写的，bootstrap 负责加载 jre/lib/rt.jar,打印的话为 null.

- 扩展类加载器 ExtClassLoader,负责加载 jre/lib/ext/.jar.其父类加载器也是 null
- 应用加载器 AppClassLoader

```java

        ClassLoaderDemo object =  new ClassLoaderDemo();
        System.out.println(object.getClass().getClassLoader());
        System.out.println(object.getClass().getClassLoader().getParent());
        System.out.println(object.getClass().getClassLoader().getParent().getParent());
        //输出为
        sun.misc.Launcher$AppClassLoader@18b4aac2
        sun.misc.Launcher$ExtClassLoader@7c53a9eb
        null
```

## 双亲委派机制和沙箱安全

加载类的时候委托父类加载,如果父类已经加载那么就不在加载，保证不污染自带类的也叫沙箱机制。

- 如果自定义一个 java.lang.String 类的话运行会报安全性异常

```java

package java.lang;

public class String {

    public static void main(String[] args) {
        System.out.println("hello");
    }
}
//结果
错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application
Disconnected from the target VM, address: '127.0.0.1:8616', transport: 'socket'
[ERROR] Command execution failed.
org.apache.commons.exec.ExecuteException: Process exited with an error: 1 (Exit value: 1)


//如果不在自定义的类中调用，自己写的string类根本就不会加载
```

## 自定义类加载器

继承 ClassLoader 重写 findClass 方法。findClass 方法和 loadClass 方法之间有什么区别的？如果不想打破双亲委派机制，那么直接使用重写 findClass 方法，loadClass 方法可以不双亲委派直接加载类，使用 loadClass 方法的话可以实现热部署

## 执行引擎 Execution Engine

## native 方法

本地方法栈中是 native，没有垃圾回收，很小只有 10 来 KB，比如 thread 类中的 start0()方法

## 程序计数器

cpu 中的寄存器，是当前线程执行字节码的行号指示器。如果 native 方法，计数器是空的，因为 native 方法不是 Java 的。用来完成分支循环跳转异常处理等等。

## 方法区

很少垃圾回收，线程共享。是存放类的结构信息，也就是模板。如常量池字段方法数据构造函数普通方法等字节码内容，

方法区是规范，不同虚拟机实现不同。如永久代和原空间

存储：类结构信息、普通常量、静态常量、编译器编译后的代码等。也叫非堆内存，1.7 之前的永久代是方法区的一个实现而已。

## 虚拟机栈

没有 gc，栈溢出是错误不是异常

### 保存什么？

8 中基本类型，对象的引用，实例方法的都是在栈中分配。Java 中方法叫做栈帧

## 堆

规范上分为三代，实际上新生代和老年代是物理连续的，元空间不在堆上

- 新生代 1/3

* eden 区 8
* s0(from) 1
* s1(to) 1

- 老年代 2/3

  - 15 次垃圾回收以后进入

- 永久代

* 1.8 之后永久代不在 jvm 内存中，二是直接使用物理内存，默认大小为物理内存的 1/4

```java

Heap
 PSYoungGen      total 153088K, used 7895K [0x0000000715800000, 0x0000000720280000, 0x00000007c0000000)
  eden space 131584K, 6% used [0x0000000715800000,0x0000000715fb5dc8,0x000000071d880000)
  from space 21504K, 0% used [0x000000071ed80000,0x000000071ed80000,0x0000000720280000)
  to   space 21504K, 0% used [0x000000071d880000,0x000000071d880000,0x000000071ed80000)
 ParOldGen       total 349696K, used 0K [0x00000005c0800000, 0x00000005d5d80000, 0x0000000715800000)
  object space 349696K, 0% used [0x00000005c0800000,0x00000005c0800000,0x00000005d5d80000)
 Metaspace       used 3028K, capacity 4556K, committed 4864K, reserved 1056768K
  class space    used 319K, capacity 392K, committed 512K, reserved 1048576K
```

## JVM 内存划分

堆栈方法区程序计数器

栈、程序计数器线程私有
堆、方法区线程共享，存在垃圾回收

### 堆内存划分

新生代、老年代、持久代（1.8 没有老年代）
新生代：Eden 区和 servivor，servivor 区使用复制算法，每次只能使用一般的内存

## GC ROOTS 对象

1. 虚拟机栈中引用的对象
2. 方法区中静态类属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中引用的对象

## 4 大垃圾回收算法七大垃圾回收器

jvm 采用分代回收算法。1. 在次数上 young 区收集比较频繁，2.old 区收集比较少，3.基本上不动元空间

默认 15 次 GC 存在老年代，jdk8 定死了 15。gc 不是三块区域一起回收，主要发生在 young 区

### 垃圾回收算法

1. 引用计数算法
2. 标记清除算法：老年代使用，最大的问题会产生内存碎片，两次扫描耗时，有 stw ，标记要清除的对象，然后统一进行回收
3. 标记整理算法：老年代使用，和标记清除混合使用。两次扫描，移动对象需要耗时，效率低于复制算法。工程上是使用标记清除，到达一定条件后进行整理也叫标记清除整理算法
4. 复制算法：young 区使用复制算法，minorgc 会把 eden 区中对象移到 survivor 区中，如果 survivor 区放不下，那么剩下的活的对象就会被异动到 old 区，一旦收集完成 eden 区就会变成空。优点不会产生内存碎片， 缺点耗空间

### 垃圾收集器

## jvm 调优

### 前提知识

Java 将内存抽象为 Runtime 对象,通过 Runtime.getRuntime()获取

java 虚拟机参数分为

- 标准参数如(-) -version
- 非标准参数(-XX)

* 布尔型，用加号标识开启减号标识关闭：

- -XX:+PrintGCDetails 开启打印 gc 日志
- -XX:+PrintCommandLineFlags 打印启动配置信息
- -XX:NewRatio=2 设置新生代和老年代的比例，值直接代表老年代占比，剩下的 1 就是新生代的。
- -XX:ServivorRatio=8 设置 endn 和 servivor 的比例，8:1:1
- -XX:MaxTenuringThreshold=15 设置垃圾新生代到老年代要经过的 gc 次数，默认是 15 次。如果设置 0 则不经过 servivor 直接进入老年代。必须设置在 0-15 之间

* key-vlue 键值型-XX:InnitialHeapSize=1m
  - -Xms 堆最小值
  - -Xmx 堆最大值
  - -Xss 单个线程栈大小，一般 512k-1024k。初始值查询出来的话是 0，所以如果查询出来 0 就代表是初始值。linux 默认是 1024k
  - -Xmn 设置新生代大小，默认不用调试用默认值
  - -XX:MetaspaceSize=1024m,元空间试用操作系统物理内存，默认情况 jdk 只分配了 21MB

```java

    public static void main(String[] args) {
        //剩余堆内存
        System.out.println(Runtime.getRuntime().freeMemory());
        //处理器个数
        System.out.println(Runtime.getRuntime().availableProcessors());
        //最大内存，默认是系统的1/4
        System.out.println(Runtime.getRuntime().maxMemory());
        //最小堆内存，默认是系统内存的1/64
        System.out.println(Runtime.getRuntime().totalMemory());
    }
```

**实际线上必须将最小内存和最大内存设置为一样,避免 gc 和应用争抢内存浪费性能**

### 常用参数

- -XX:+PrintCommandLineFlags
  打印 jvm 的启动参数
- -Xms 等价于-XX:InitialHeadSize
  设置堆最小内存，默认情况下为物理内存的 1/64
- -Xmx 等价于-XX:MaxHeapSize
  设置堆内存的最大值。默认情况下为物理内存的 1/4
- -XX:+PrintGcDetails
  打印 gc 日志

- jinfo 查询正在运行的服务参数
  可以使用 jinfo pid 也可以使用 jinfo flag(s) pid

### gc 日志

eden 区 minor gc old 区发生 major gc 也叫 full gc，fullgc 慢 8-10 倍，为什么慢？因为 full gc 扫描空间比较大

```java

[GC (Allocation Failure) [PSYoungGen: 1000K->488K(1024K)] 1016K->576K(1536K), 0.0006887 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

[Full GC (Ergonomics) [PSYoungGen: 649K->137K(1024K)] [ParOldGen: 408K->408K(512K)] 1058K->545K(1536K), [Metaspace: 3020K->3020K(1056768K)], 0.0041367 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
```

### 死锁定位

java 模拟

```java
public class DeadLockDemo {

    public static void main(String[] args) {
        String lockA = "lockA";
        String lockB = "lockB";
        new Thread(new res(lockA, lockB), "aaaaaaaaaaaa").start();
        new Thread(new res(lockB, lockA), "bbbbbbbbbbbb").start();
    }
}

class res extends Thread {
    private String lockA;
    private String lockB;
    public res(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }
    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println("获得锁" + lockA + "尝试获取" + lockB);
            try {
                TimeUnit.SECONDS.sleep(3);
                synchronized (lockB) {
                    System.out.println("获得锁" + lockB);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

```

实操过程：

- 1. 使用 jps 查看进程编号(linux 和 windows 通用)

```java
jps -l

//打印信息
PS C:\Users\bear\IdeaProjects\jvmdemo> jps -l
11920 org.jetbrains.jps.cmdline.Launcher
12448 org.jetbrains.idea.maven.server.RemoteMavenServer36
9424
11044
15736 org.codehaus.classworlds.Launcher
5788 pro.dengyi.DeadLockDemo
6940 sun.tools.jps.Jps
7548 org/netbeans/Main

```

- 根据自己项目的包名使用 jstack 命令查看栈信息

```java
PS C:\Users\bear\IdeaProjects\jvmdemo> jstack 5788
//打印信息
2020-06-25 22:49:12
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.241-b07 mixed mode):

"DestroyJavaVM" #17 prio=5 os_prio=0 tid=0x0000000003652800 nid=0x838 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"bbbbbbbbbbbb" #16 prio=5 os_prio=0 tid=0x00000000277ae800 nid=0x1de4 waiting for monitor entry [0x000000002866e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at pro.dengyi.res.run(DeadLockDemo.java:29)
        - waiting to lock <0x00000007159a77d0> (a java.lang.String)
        - locked <0x00000007159a7808> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)

"aaaaaaaaaaaa" #14 prio=5 os_prio=0 tid=0x00000000277ab000 nid=0xf60 waiting for monitor entry [0x000000002856e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at pro.dengyi.res.run(DeadLockDemo.java:29)
        - waiting to lock <0x00000007159a7808> (a java.lang.String)
        - locked <0x00000007159a77d0> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)

"Service Thread" #12 daemon prio=9 os_prio=0 tid=0x0000000027779800 nid=0x33cc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #11 daemon prio=9 os_prio=2 tid=0x00000000276f3000 nid=0xc8c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #10 daemon prio=9 os_prio=2 tid=0x00000000276ea000 nid=0x2cf8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #9 daemon prio=9 os_prio=2 tid=0x00000000276e9000 nid=0x90c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Command Reader" #8 daemon prio=10 os_prio=0 tid=0x0000000027690800 nid=0x6ec runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Event Helper Thread" #7 daemon prio=10 os_prio=0 tid=0x000000002768d000 nid=0x14f4 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Transport Listener: dt_socket" #6 daemon prio=10 os_prio=0 tid=0x000000002767e800 nid=0x2a60 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x0000000027678800 nid=0x1dac waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x0000000026330000 nid=0x2444 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000000374c800 nid=0x2920 in Object.wait() [0x000000002766e000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000715808ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x0000000715808ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000002630a800 nid=0x10d8 in Object.wait() [0x000000002756f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000715806c00> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x0000000715806c00> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x00000000262e8000 nid=0x14e8 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000000003668000 nid=0xf30 runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x0000000003669800 nid=0x85c runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x000000000366b800 nid=0xd84 runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x000000000366e000 nid=0x3280 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x00000000277a5000 nid=0x3734 waiting on condition

JNI global references: 1499


Found one Java-level deadlock:
=============================
"bbbbbbbbbbbb":
  waiting to lock monitor 0x00000000037494f8 (object 0x00000007159a77d0, a java.lang.String),
  which is held by "aaaaaaaaaaaa"
"aaaaaaaaaaaa":
  waiting to lock monitor 0x000000000374bd88 (object 0x00000007159a7808, a java.lang.String),
  which is held by "bbbbbbbbbbbb"

Java stack information for the threads listed above:
===================================================
"bbbbbbbbbbbb":
        at pro.dengyi.res.run(DeadLockDemo.java:29)
        - waiting to lock <0x00000007159a77d0> (a java.lang.String)
        - locked <0x00000007159a7808> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)
"aaaaaaaaaaaa":
        at pro.dengyi.res.run(DeadLockDemo.java:29)
        - waiting to lock <0x00000007159a7808> (a java.lang.String)
        - locked <0x00000007159a77d0> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

## JMM java 内存模型

## Java 加载执行顺序

new 对象时的加载顺序

1. 静态代码块优先执行，加载一次
2. 普通代码块
3. 构造方法
4. 其他

**父类构造方法，子类会在构造方法中调用**

代码研究：

```java

public class LoadFatcher {
    static {
        System.out.println("父类中的静态代码块");
    }
    //普通代码块又叫做构造块
    {
        System.out.println("父类中的普通代码块执行");
    }
    public LoadFatcher() {
        System.out.println("父类构造方法执行");
    }
}
class LoadSon extends LoadFatcher {
    static {
        System.out.println("子类中的静态代码块");
    }
    {
        System.out.println("子类中的普通代码块执行");
    }
    public LoadSon() {
        System.out.println("子类构造方法执行");
    }


    public static void main(String[] args) {
        LoadSon loadSon = new LoadSon();
        System.out.println("子类中的普通方法执行---------------");
        LoadSon loadSon1 = new LoadSon();
    }
}
//输出
父类中的静态代码块
子类中的静态代码块
父类中的普通代码块执行
父类构造方法执行
子类中的普通代码块执行
子类构造方法执行
子类中的普通方法执行---------------
父类中的普通代码块执行
父类构造方法执行
子类中的普通代码块执行
子类构造方法执行

```
