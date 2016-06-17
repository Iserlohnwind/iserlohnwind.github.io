---
layout: post
title: "MySQL的自适应hash索引"
date: 2016-06-16 18:00:02 +0800
categories: jekyll update
---

​	B-Tree索引是MySQL中最常见的索引结构，它查询过程是从根节点开始进行二分法查找，所以它的时间复杂度是O(log(n))，这种查询特别适合范围查询。但是有时候，我们期望可以获得更好的性能，尤其是我们在进行等值查询时，肯定希望一次就命中所需要的结果比如说如下语句：

​			`SELECT * FROM user WHERE name=‘小明’;`

​	对于这样的场景，我们很容易就能想到基于hash的查询能够给效率带来显示的提升。不幸的是，hash索引的建立只在MEMORY存储引擎中得到支持。

​	然而，在InnoDB引擎中引入了自适应哈希索引（[adaptive hash index](http://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_adaptive_hash_index)）。InnoDB存储引擎会监控对表上索引的查找，如果观察到建立哈希索引可以带来速度的提升，则建立哈希索引，所以称之为自适应（adaptive） 的。自适应哈希索引通过缓冲池的B+树构造而来，因此建立的速度很快。而且不需要将整个表都建哈希索引，InnoDB存储引擎会自动根据访问的频率和模式来为某些页建立哈希索引。在MySQL 5.7.8之前的版本，因为自适应哈希索引由单一的锁存器保护，这在高负载下会成为一个竞争点。MySQL 5.7.8中自适应哈希索引做了分区，每个索引被分配到一个特定的一个分区，每个分区被一个独立的锁存器保护，缓解了之前版本的问题。

​	因为自适应哈希索引可以极大地提升效率，因此默认状态下它是开启的，可以通过*innodb_adaptive_hash_index*参数来关闭它，也可以在启动server的时候添加*--skip-innodb_adaptive_hash_index*来禁用。

