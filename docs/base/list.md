# list 容器

## ArrayList 和 LinkedList

ArrayList 底层基于数组，初始化大小为 10，线程不安全。当需要的大小大于实际的大小的时候就会进行扩容，扩容大小为原大小的 1.5 倍，然后调用 Array.copyOf()方法将老数组中的数据复制到新数组。LinkedList 底层基于链表实现的，增删块，查找慢。
