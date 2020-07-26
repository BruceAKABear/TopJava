# 集合

## List

非线程安全arraylist并发修改异常concurrentmodificationexception产生的原理

arraylist的add方法中使用size++

size++本身是线程不安全的

在演示时，其实是数据不一致异常，因为输出语句会去遍历，而新增可能还在新增

## set

## map