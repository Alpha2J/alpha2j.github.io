---
title: 一致性哈希算法详解（原理与Java实现）
date: 2022-08-14
tags: 
  - tutorial
  - consistence hash
categories: technology
keywords: 'system-design java'
---

# 1. 是什么
哈希算法的一种，由麻省理工学院提出，目的是解决：传统哈希算法中，哈希表内槽数量变更时**导致的大量键重映射问题**。

如下内容引用自[维基百科](https://en.wikipedia.org/wiki/Consistent_hashing)：
> In computer science, consistent hashing is a special kind of hashing technique such that when a hash table is resized, only n/m keys need to be remapped on average where n is the number of keys and m is the number of slots. In contrast, in most traditional hash tables, a change in the number of array slots causes nearly all keys to be remapped because the mapping between the keys and the slots is defined by a modular operation.</br></br>
在计算机科学中，一致哈希是一种特殊的哈希技术，当一个哈希表被调整大小时，平均只有n/m个键需要重新映射，其中n是键的数量，m是槽的数量。相比之下，在大多数传统哈希表中，数组槽数量的变化会导致几乎所有键都要重新映射，因为键和槽之间的映射是由模块化操作定义的。

## 1.1 如何理解键的重映射
背景：分库分表的场景下，有一张逻辑表`example_table`，在物理上被拆分为四张表`example_table_0`、`example_table_1`、`example_table_2`、`example_table_3`，路由逻辑为对`user_id`计算哈希值并余上物理表的数量。

假设使用如下代码来计算四个userId的哈希值及所属节点（物理表）：
```java
public static void main(String[] args) {
    List<Integer> userIds = new ArrayList<>();
    userIds.add(89999442);
    userIds.add(987446272);
    userIds.add(763889234);
    userIds.add(847462221);

    userIds.forEach(userId -> {
        int hash = userId.hashCode();
        hash = Optional.of(hash)
                .filter(h -> h > 0)
                .orElse(-hash);
        System.out.println("userId: " + userId + " , hash is: " + hash + " , routing to physical table example_table_" + hash % 4);
    });
}
```
能得出如下键值关系：

| 键 | 哈希值 | 节点编号 | 命中的物理表
| --- | --- | --- | --- |
| 89999442 | 89999442  | 2 | example_table_2
| 987446272 | 987446272  | 0 | example_table_0
| 763889234 | 763889234  | 2 | example_table_2
| 847462221 | 847462221  | 1 | example_table_1

这是四个节点的场景，而如果由于业务原因，四张表要扩为6张，键与节点的映射关系要重新计算，`%4`变成`%6`，得到以下结果：
| 键 | 哈希值 | 节点编号 | 命中的物理表
| --- | --- | --- | --- |
| 89999442 | 89999442 | 2 -> 0 | example_table_0
| 987446272 | 987446272 | 0 -> 4 | example_table_4
| 763889234 | 763889234 | 2 -> 2 | example_table_2
| 847462221 | 847462221 | 1 -> 3 | example_table_3

可以看到，同一个键在节点数发生改变时，映射关系也随着改变，而这就是“键的重映射”。这种问题在不同场景下会产生不同的影响：
1. 分库分表：需要扫描所有物理表，计算数据重映射后的所属节点，并迁移数据。影响范围为：**所有表下的所有数据**。
2. 分布式负载均衡：

## 1.2 应用场景
一致性哈希算法在很多领域都有应用，如分布式缓存领域的Redis，各种RPC框架的客户端负载均衡等。

# 2. 原理
一致性哈希算法通过一致性哈希环实现，环的起点是0，终点是2^32 - 1，并且起点和终点相连，因此环的分布范围是`[0, 2^32 - 1]`，如图所示：

![哈希环.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3ba51accd1649fea38764038d56171b~tplv-k3u1fbpfcp-watermark.image?)

将节点放到环上：

![映射到环上的节点.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7be9b1f60e3a48f8a02f39e28f708efc~tplv-k3u1fbpfcp-watermark.image?)



将键放到环上：

![映射到环上的键.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73b2eba9dc5042fb8f0206d197bcbebb~tplv-k3u1fbpfcp-watermark.image?)

在寻找键所属的节点时，往顺时针方向找第一个最近的节点即可：

![键的查找结果.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7679f116bbf144108718c26c08d2a967~tplv-k3u1fbpfcp-watermark.image?)

新增节点newNode时，只有key4的数据会移动到newNode，**其他键的数据不需要变化**

![新增节点.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc3b3e020c6c45c1a3c0669e41044600~tplv-k3u1fbpfcp-watermark.image?)

删除节点example_table_2后，只有key3的数据会移动到example_table_3，**其他键的数据不需要变化**

![删除节点.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3e7c1fb5f7243fe84e952c64c11f373~tplv-k3u1fbpfcp-watermark.image?)

## 2.1 存在的问题
当节点数量过少时，可能会出现数据倾斜（某些节点没有负载）。一致性哈希通过引入虚拟节点来解决数据倾斜问题，即：把一个Node扩展为一组虚拟Node，这些虚拟Node分布在一致性环的不同位置，但指向同一个实际存在的Node。如下图

虚拟节点解决的问题是：尽可能让节点在一致性哈希环上均匀分布，避免数据倾斜，当然，这是不可能完全规避掉，通常虚拟节点数量越多越均匀，因此实际使用中虚拟节点数量一半会大于32个。

## 2.2 哈希算法的选择
一致性哈希在将节点以及键映射到哈希环时，需要使用到哈希算法，不同场景下的可能会对应不同选择，如负载均衡场景下需要考虑安全性和性能，而安全性则没那么重要。通常，会有如下可选择的哈希算法：sha-1，md5，crc，lookup3，murmurhash，cityhash，spookyhash。在众多哈希算法中，我做项目比较倾向于使用murmurhash（虽然应该根据具体场景来选择，但够用）：高运算性能，低碰撞率（分布均匀），由Austin Appleby创建于2008年，现已应用到Hadoop、libstdc++、nginx等开源系统中。在Java的实现，Guava的Hashing类里面有，Jedis，Cassandra里都有相关的Utils类。
第一代：

# 4. Java语言实现

# 5. 常见的开源实现
# 6. 参考资料
https://www.cnkirito.moe/consistent-hash-lb/

