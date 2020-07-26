# zookeeper

可以理解成数据库、也可以理解成文件系统。可以统一命名服务

- 集群启动时会同步数据
- 和 leader 同步，少的同步，多的回滚

## datatree 内存树

zk 将所有数据放在内存中，处理的时候首先去生成事务日志，然后更新内存。

非事务请求是直接获取 datatree 中数据

## 请求类型

- 事务性请求
  create、set、delete。先生成事务 id:zxid,判断数据新旧程度就是比较最大事务 id。
  如果一开始启动，没有 zxid，name 去判断 myid.

  所有的写请求先由 leader 处理，

- 非事务性请求

## cap,zk只保证cp

强一致性、可用性、分区容错性

## 2cp 阶段提交

预提交、收到 ack、提交

- 预提交

1. leader 生成事务日志
2. follower 也生成事务日志

- ack
  follower 生成事务日志成功后，响应给 leader 一个 ack。等待提交更新 datatree

- 提交
  leader 提交，follower 将数据更新到 datatree。leader 提交是异步的，提交以后直接就返回了，不用等 follower

**客户端连接 follower 操作时，follower 将请求转发到 leader**

## 领导选举机制

- 集群启动时需要选举

- leader 挂了要选举

- follower 挂掉了

## ZAB 协议

规定领导选举、过半机制、2cp 提交、数据同步
