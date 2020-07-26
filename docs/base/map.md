# map 集合

map 是以 key-value 的形式存储数组的数据结构

## hashmap

1. 默认大小 16，2 的 4 次方。最大大小 2 的 30 次方，大概在 10 亿 7 千万多
2. 负载因子 0.75

存储数据时，将 hash 值，key，value 等封装成一个对象

### 初始化大小为什么时 2 的次幂？

因为这样的话能加快 hash 值得计算

### 为什么加载因子要设置成 0.75？

过小浪费空间，过大哈希碰撞概率大

### 什么是 hash 冲突，hashmap 是怎么处理的？

当，我们对存入的 key 进行 hash 计算以后，得到的 hash 桶的位置和其他的 key 计算出来的 hash 桶的位置相同的时候，这就是 hash 冲突（哈希碰撞）。存在链表上

### hash 表采用什么算法计算出 hash 值?还有什么算法可以计算出 hash 值？

采用 key 的 hashcode 值，按位异或 key 的哈希值结合数组的长度进行无符号的右移(>>>)，也就是 key 的哈希值无符号右移 16 位。
可以计算哈希值的算法有：

1. 取余
2. 平方取中
3. 伪随机数法

其他计算方式，效率比较低，尤其是取余底层是一直减。采用异或速度是非常快的。

### 1.7 中的 HashMap

底层原理树数组加链表，数组为 entry 数组，实现了 mep.entry

#### hashmap 的死锁问题？

### 1.8 中的 HashMap

3. 链表转树的阈值为 8
4. 树转链表的阈值为 6
5. 树转换数组大小阈值 64

底层实现原理是数组加链表加红黑树,数组为 node 数组，也实现了 map.entry

#### 为什么链表长度大于等于 7 时，就要转成红黑树？

首先，不是说链表的长度大于 8 就进行树的转换。源码里面写了只有当，数组的长度大于 64，同时，链表的长度大于 8，那么才进行树的转换。
**为什么呢？**红黑树在插入数据得时候会进行左旋右旋得操作，导致插入比较慢，虽然查询快，但是其实性能是有缺陷的。所以没有上来就使用红黑树，所以源码里面在数组没有大于 64 的时候，如果链表长度大于 8 其实是进行了数组的扩容。只有当数组长度大于 64 且链表长度大于 8 以后，才进行链表转红黑树。

## 1.7 源码分析

### 1. 1.7 中 put 方法

```java
    public V put(K key, V value) {
        //如果哈希表为空，则初始化entry数组
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        //如果key是null值，直接调用putnull方法
        if (key == null)
        {
            return putForNullKey(value);
        }
        //计算key的哈希值
        int hash = hash(key);
        //计算key在hash桶中的位置
        int i = indexFor(hash, table.length);
        //根据桶的位置，获取原有值。如果不为空，则遍历所有的链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            //如果链表中hash相同同时key相同，则替换原有值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        //如果原有值为空，那么直接添加
        addEntry(hash, key, value, i);
        return null;
    }
```

### 2. 1.7 源码中，并不仅仅是数组达到阈值 12 就进行扩容，还要判断要存储的位置是否为 null

源码：

```java
//添加entry对象方法
void addEntry(int hash, K key, V value, int bucketIndex) {
    //判断阈值是否大于等于，同时哈希表上不为null
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }

```

### 3. 1.7 中使用头插法，拉链法进行存储

```java
  //添加对象方法
  void createEntry(int hash, K key, V value, int bucketIndex) {
        //首先获取哈希表中的对象
        Entry<K,V> e = table[bucketIndex];
        //然后添加对象，添加对象在头部
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }

```

### 4. 1.7 扩容方法

```java
//resize方法。1.产生新的数组，2.将老数组中的对象重新进行hash放置于芯数组中
  void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        //第二个参数控制是否需要rehash
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```

transfer 方法将老数组中的值重新计算 hash 赋值到新数组中

```java
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        //遍历老数组
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

### 5. 1.7 中的 get 方法

```java
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
```

```java
    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : hash(key);
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }


```

## 1.8 源码分析

### 1. 1.8 中的 put 方法

```java

   public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //判断有无初始化，然后初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //计算index，如果获得index的数组为null那么直接将node对象放在数组上
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //如果不为null,判断hash是否相同，判断key如果相同然后将新值替换旧值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
                //如果是树类型，那么就调用树添加方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
                //遍历链表，将新node对象放置在末尾
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果长度大于等于8那么就进行树转换
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```

## ConcurrentHashMap

### 1.7 中的 ConcurrentHashMap

分段锁加 cas

当我们 put 的时候，根据 key 的 hash 值，先去获取 segment 数组下标，获取到 segment 对象,如果segment对象为空，那么就new一个segement对象，相关的数据从0相关的属性，segment 对象内部有一个小的 hashentry 数组，也就是一个小的 hahsmap.然后根据 hash 计算出哈希桶的位置，去 put 数据，这个过程中呢会使用非公平锁 unfairLock.

### 1.8 中的 ConcurrentHashMap

底层使用 synchronized 和 cas

和hashmap很像，在获取时使用了cas,在添加时使用了sync