# jvm 面试题

## 双亲委派机制、沙箱机制

## 1. 你知道怎么计算对象大小吗？

Java 中是 8 字节对齐。

对象内存结构：对象头区域、实例数据区域、对齐填充区域

## 2. 空对象占用多少个字节？

空对象：没有普通属性的对象

开启指针压缩占用 16 字节

```java

-XX:+UseCompressedOops//开启指针压缩
```

未开启指针压缩暂用 16 个字节

## 2. 指针压缩？

64 位机器线性地址是 8 字节，实际使用的是 48 位，16 位保留。所以，最大支持 256T 内存。开启指针压缩以后使用

## 1. jvm 垃圾回收时如何确定垃圾？是否知道什么是 GC Roots?

对象没有被引用指向时换句话说就是内存中已经不再使用的空间就是垃圾。
确定垃圾可以有两种方法，引用计数法和可达性分析算法
引用计数法不能解决循环依赖的问题，所以实际上 jvm 垃圾回收是使用根搜索算法来确定垃圾。
通过 gc roots 对象作为起点，向下搜索，如果一个对象到 gc roots 没有任何引用链连接时，则说明对象不可用，可以进行垃圾回收。
gc roots 对象一共可以分为 4 种：

1. 虚拟机栈中引用的对象
2. 方法区中静态类属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中引用的对象

## 2. 你说你做过 jvm 调优，那么请问如何盘点查看 jvm 系统默认参数？

使用命令：-XX:+PrintCommandLineFlags

```java
-XX:InitialHeapSize=536223744 -XX:MaxHeapSize=8579579904 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
```

## 3. 你平时工作用的 jvm 常用基本配置参数有哪些？

最小堆大小: -Xms1024m
最大堆大小: -Xmx1024m
选择垃圾收集器:-XX:+UseG1GC

## 4. 四大引用分别是什么？

强软弱虚四大引用

- 强引用不管内存是否充足都不会进行垃圾回收
- 软引用内存充足不进行垃圾回收，内存不足会进行垃圾回收
  图片的缓存，内存不足时垃圾回收不会 OOM
- 弱引用不管内存都会被垃圾回收
  WeakHashMap,entry 对象继承了 WeakReference<Object>，当进行 gc 后会被清空
- 虚引用
  get 返回 null,必须配合引用队列，ReferenceQueue<>,被回收的时候放入引用队列保存一下

## 5. 谈谈对 oom 的认识？

## 6. gc 垃圾回收算法和垃圾收集器之间的关系？分别是什么请你谈谈？

四种主要的垃圾收集器为：

- serial 串行垃圾收集器，只有一个垃圾收集线程，stw 长，不使用
- parallel 并行垃圾收集器，多个垃圾收集线程一起工作，也存在 stw，适用于计算、大数据等弱交互
- cms 并发标记清除收集器，用户线程和垃圾收集线程同时执行
- G1 收集器

可配置的 GC 有:

- UseSerialGC
  开启以后新生代时候 serial,老年代使用 serial old
- UseParallelGC 默认
  新生代使用 parallel scavenge 吞吐量， 老年代使用 parallel old（1.6 之前使用 serial old），可以互相激活。-XX:ParallelGCThreads=n cpu>8 n=5/8,cpu<8 n=核心数
- UseConcMarkSweepGC
  使用来老年代。开启之后，新生代默认开启 UseParNewGC，serial old 作为后备。并发执行压力大，必须要在老年代堆内存用尽前完成回收，否则垃圾回收失败，会产生碎片，配置进行多少次 fullgc 后进行压缩。默认 0 每次都进行内存整理

  - 初始标记
    stw，标记 gc root 关联对象
  - 并发标记
  - 重新标记
    stw，修正用户线程导致标记变动，二次确认
  - 并发清除

- UseParNewGC
  新生代使用 parallel 老年代使用 serial old
- UseParallelOldGC
- UseG1GC

## 7. 怎么查看服务器默认的垃圾收集器是哪一个？生产上如何配置垃圾收集器？对垃圾收集器的理解？

使用 jinfo

## 8. 谈谈 g1 垃圾收集器？

## 9. 生产环境服务器变慢，谈谈诊断思路和性能评估？

## 10. 生产环境出现 cpu 占用过高，谈谈分析思路和定位？
