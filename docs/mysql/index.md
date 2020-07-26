# 关系型数据库 MySQL

## 数据库设计三范式

1. 有主键
2. 列最小不可分
3. 消除依赖传递

## 存储引擎选择

| 引擎类型 | 特点                                                       |
| -------- | ---------------------------------------------------------- |
| innodb   | 支持事务，支持行锁，innodb 中的页大小为 16K 大概能存 80 亿 |
| myisam   | 不支持事务                                                 |

## 索引

索引是一种数据结构，可以加快查询。

索引有多种，有 hash 索引没办法范围查找。二叉树，如果数据特殊树会呈线性。平衡二叉树，如果数据量很大，那么树的高度会很高，磁盘 io 次数会特别多。b 树不只有一个叉，但是扫库扫表能力不强。b+树数据存储在叶子节点，范围查询能力强，io 次数少，查询效率稳定 io 次数稳定

### 1. 创建索引原则

列的离散度越高越好

## MySQL 实际调优