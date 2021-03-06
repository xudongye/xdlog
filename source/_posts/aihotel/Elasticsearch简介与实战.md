---
title: Elasticsearch简介与实战
date: 2019-11-7 09:25:20
categories: 
- 程序设计
tags:
- learn
---

### 应用场景

*   因为做到了商城系统，商城数据商品表的复杂性，mysql中进行模糊查询，会放弃索引，导致商品表全表查询，
    ，在PB级数据库中，效率非常低下。而引入es做一个全文索引，我们将经常查询的商品的某些字段，商品名称，
    描述，价格，id等fields放入索引库，提高查询速度。
    
### 为什么ElasticSearch/Lucene检索比Mysql快？
    
*   什么是索引
    * 我个人理解，索引就好比图书目录，查内容可以通过目录提供的页码快速找到。在关系型数据库中，
索引是一个单独的、物理的对数据库表中的一列或多列的值进行排序的一种存储结构，是某一列或多列值的集合和相应的指向
表中物理标识这些值的数据页的逻辑指针清单

*   索引的作用
    * 保证数据的准确性，加快检索速度，唯一索引对应唯一值，提高系统性能

*   mysql索引如何实现？
    * 在mysql中，索引属于引擎级别的概念，不同的存储引擎对索引的实现方式都是不同的。
    * MyISAM索引实现
        * MyISAM表的索引和数据是分离的，索引保存在”表名.MYI”文件内，而数据保存在“表名.MYD”文件内
        * 为了和InnoDB的聚集索引区分，MyISAM的索引方式叫做非聚集索引
    * InnoDB索引实现
        * InnoDB使用B+Tree作为索引结构，实现方式与MyISAM截然不同
        * 主键聚集方式，InnoDB要求表必须有主键，如果没有显示指定，则mysql系统会自动选择一个可以唯一标识数据记录的列作为
        主键，如果不存在，则mysql自动为InnoDB表生成一个隐性的类型为长整型长度为6字节的字段作为主键。
        * 聚集索引的实现方式使得按主键的搜索十分高效，但是辅助索引是需要检索两边索引的：首先检索辅助索引获得主键，然后通过
        主键的主索引检索获得记录
    * 了解了不同引擎的索引实现方式对于正确使用和优化索引都是有帮助的，比如InnoDB索引不建议使用过长的字段作为索引，因为辅助
    索引都是引用主索引，过长的主索引会令辅助索引变得过大。还有就是用非单调的字段作为主键在InnoDB中不是一个好主意，因为InnoDB
    数据文件本身是一棵B+Tree,非单调的主键会造成插入新纪录时数据文件为了维护B+Tree的特性而频繁的分裂调整，十分低效，而使用自增
    字段作为主键是一个很好的选择。
    * 传统的关系型数据库大部分采用的B-Tree和B+Tree这样的数据结构。二叉树的查找效率是logN，同时写入更新新节点不必移动全部节点，
    所以树型结构存储索引，能同时兼顾写入和查询性能。当我们不需要支持快速的更新的时候，则可以用预先排序等方式换取更小的存储空间，
    更快的检索速度等好处，其代价就是更新慢。下面我们进一步深入了解Lucene索引是如何实现的。
    * Lucene 索引实现

*   lucene索引是如何实现(包含压缩技术)？


### 总结
*   Lucene/Elasticsearch就是尽量将磁盘里的东西搬进内存，减少磁盘随机读取次数，同时利用磁盘顺序读取特性，结合压缩算法，
高效使用内存，从而达到高速搜索的特性。
