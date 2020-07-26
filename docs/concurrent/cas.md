# CAS
## 什么是CAS
cas是比较并交换 compare and swap的意思
通过比较**主物理内存**值是否是期望值，如果是，那么就进行交换

unsafe的操作，在系统底层，是cpu指令，是绝对原子性的。也就是绝对线程安全的

```java
 /**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }

```

很显然底层是使用了unsafe类

## 为什么使用CAS不适用synchronized

cas底层自旋锁，不上锁，效率肯定高

## 什么是unsafe类？
unsafe类使Java能够去操作实际的内存，通常认为Java不能直接操作实际的内存，但是unsafe其实一直都可以操作

在使用unsafe保证原子性的时候，都是要配合volatile的

源码：
```java
 private volatile int value;
```

## 原理

unsafe类，自旋锁

valueOffset：内存偏移量就是内存地址

unsafe类大部分都是native的方法，因为直接去操作操作系统的内存

在unsafe实现中，是使用了自旋锁

自旋锁其实就是do-while

```java

do{
    //获取
    this.getIntVolatile(xxx);
    //然后设置
}while(compareAndSet())
```

## 缺点

正因为unsafe类，使用了自旋锁，有可能多次获取不到期望值，因此可能导致系统cpu占用率突然升高

1. 不加锁，循环比较时间可能长，开销大
2. 只能保证一个共享变量的原子操作，如果是多个变量需要保证原子性那就只能上锁
3. 引入aba问题



## 什么是ABA问题

一个线程修改为A
突然另一个线程修改为B,又改为A

**ABA问题是CAS引入的最大问题**

如果要求严格，那么可以原子引用加版本号

使用类

```java
 AtomicStampedReference<T>
```



