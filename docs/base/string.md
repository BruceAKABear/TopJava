# String

String 是 final 修饰的，所以不可以被继承。string 是常量，当创建以后就不能修改

## StringBuilder 和 StringBuffer

StringBuffer 是线程安全的，效率低。StringBuilder 是线程非安全的。

源码分析：

```java
//1. StringBuffer append方法
    @Override
    public synchronized StringBuffer append(Object obj) {
        toStringCache = null;
        super.append(String.valueOf(obj));
        return this;
    }

//2. StrngBuilder 的append方法
    @Override
    public StringBuilder append(Object obj) {
        return append(String.valueOf(obj));
    }


```
