---
title: MySQL数据库索引是个啥？
sidebar:
  - toc
date: 2022-02-05 11:10:04
tags: [MySQL]
categories: [MySQL]
---

MySQL数据库索引是一个开发中并不陌生的词汇，但是深究原理还是很有趣的。那么本post主要从，什么是数据库索引，为什么要使用数据库索引，如何使用数据库索引三个方面来研究，数据库索引是个什么东西。

<!-- more -->

## 数据库索引是个啥

> MySQL数据库索引是一种用于快速查询和检索数据的数据结构。常见的索引结构有: B 树， B+树和 Hash。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%20-%20%E7%B4%A2%E5%BC%95.png MySQL索引 download:true %}


那么到底什么是数据库索引，不如先讲一讲什么是数据库。就拿图书馆来举例子，图书馆就好比是一个数据库，图书馆的书籍就好比是数据库的记录。那么如何在数据库寻找某条记录，也就是寻找图书馆某本书。如果没有一个图书管理员寻找书籍是一件非常费事费力的事情，那么这个图书管理员就是DBMS（数据库管理系统）。

压力来到数据库管理员以后，数据库管理员又如何查找书籍具体在什么地方呢？最简单的办法，从图书馆一楼到图书馆顶楼一个一个找，显然，这样的查找方式，不仅效率低下。而且如果书本出现了丢失、或者图书馆根本没有这本书，那就必须把整个图书馆找一遍。想想看，这样的图书管理员得多辛苦，读者也是相当失望。

那么，怎么解决这个问题？

## 为什么要使用数据库索引

前面说到，图书管理员急需一种新的图书的组织方式，需要一种新的检索图书的方式，给图书管理员减轻工作量。怎么做呢？生活中图书管理员会把图书分类，然后分类里面又有子分类，子分类又有子分类... 然后每个分类下，都有图书，这样不就可以更加方便的寻找图书了吗？

这种组织就是索引的作用，索引用来更快的查询，检索数据。常见的数据结构主要有B树，B+树，Hash，但是MySQL数据库使用B+树来作为索引结构。

### 为什么是B+树，而是不是Hash？

* **使用Hash函数算出一个值散列**，这个散列值碰撞几率很低，从而能够通过一个Hash值，直接找到一个数据。

  既然如此，就会带来几个问题：数据库的数据量庞大，Hash值到底选择多少比较合适？如果发生了Hash值碰撞该怎么做？

  > 在Java中的HashMap就提供了解决方案，使用**链地址法**来解决上面这个问题。Java 8之前，如果发生了Hash值碰撞，那么就使用一个链表来存储发生碰撞的值，查询的时候也是按照链表一步一步查询。Java 8之后，为了提高查找的效率，如果碰撞现象很严重，链表太长，就会转换为红黑树，使用红黑树来存储。提高效率

如果数据库也要使用这种Hash表来存储，也会利用这种方式存储，但是**Hash碰撞**并不是一个致命的问题。

* **Hash表存储的数据不支持顺序和范围查询。**
  
  如果我们查找数据库，没有一个范围查询，这是一个很蛋疼的事情。例如，我们要查询图书馆从2022-01-02到2022-01-23日的图书数据。

  ```sql 查询一个区间
  SELECT * FROM library
  WHERE stroe_time between '2022-01-02' and '2022-01-23';
  ```

  如果使用Hash表来完成，就会穷举从2022-01-02到2022-01-23日的数据，直到找到所有的数据，这样的查询效率，显然不会让人满意。

### B树和B+树拯救数据库索引

> B 树也称 B-树,全称为 多路平衡查找树 ，B+ 树是 B 树的一种变体。B 树和 B+树中的 B 是 Balanced （平衡）的意思。

B+树和B-树其实数据结构很相似，都具有一下特征：

* B 树的所有节点既存放键(key) 也存放 数据(data)，而 B+树只有叶子节点存放 key 和 data，其他内节点只存放 key。
* B 树的叶子节点都是独立的;B+树的叶子节点有一条引用链指向与它相邻的叶子节点。 
* B 树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了。而 B+树的检索效率就很稳定了，任何查找都是从根节点到叶子节点的过程，叶子节点的顺序检索很明显。
{% grid %} 
参考 Guide哥，链接: https://javaguide.cn/database/mysql/mysql-index/#hash%E8%A1%A8-b-%E6%A0%91
{% endgrid %}

这就要求，作为索引的列，必须是有序的。这样可以通过B+树有效组织数据的同时，能够加快检索速度。

### 聚集索引和非聚集索引

> 聚集索引即**索引结构和数据一起存放**的索引。主键索引属于聚集索引。
> 
> 一般来说，下面这几个词是一个意思：聚簇索引 / 聚集索引 / 一级索引 / 主键

聚簇索引使用B+树的结构存储，节点存储数据。一般来说一个表只有一个聚簇索引，这个聚簇索引为主键，如果没有主键，那么第一个 `unique索引` 作为聚簇索引。

优点：**聚集索引的查询速度非常的快**，因为整个 B+树本身就是一颗多叉平衡树，叶子节点也都是有序的，定位到索引的节点，就相当于定位到了数据。

缺点：依赖于有序数据、更新代价大

> 非聚集索引即索引**结构和数据分开存放**的索引。
> 
> 一般来说，下面这几个词是一个意思：非聚簇索引 / 非聚集索引 / 二级索引 / 辅助索引

优点：**更新代价比聚集索引要小**。非聚集索引的更新代价就没有聚集索引那么大了，非聚集索引的叶子节点是不存放数据的。

缺点：依赖于有序数据、可能会发生二次查询（**回表**）

## 如何使用数据库索引

1. 添加 PRIMARY KEY（主键索引）
  ```sql 主键索引
  ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
  ```
2. 添加 UNIQUE(唯一索引)
  ```sql 唯一索引
  ALTER TABLE `table_name` ADD UNIQUE ( `column` )
  ```
3. 添加 INDEX(普通索引) 
  ```sql 普通索引
  ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
  ```
4. 添加 FULLTEXT(全文索引)
  ```sql 全文索引
  ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
  ```
5. 添加多列索引
  ```sql 多列索引
  ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
  ```
  **Note:** 多列索引遵循**最左匹配原则**。也就是最左优先，以最左边的为起点任何连续的索引都能匹配上。例如，有一个多列索引（age, grade），如果使用 `where age > 15 and grede >= 60`就可以利用这个多列索引，但是如果查询条件是 `where grade >= 60 and age > 50` 虽然两个条件最后结果都是一样的，但是后者的查询效率远低于前者。

### 创建索引的注意事项

1. **选择合适的字段创建索引**
  * **不为 NULL 的字段** ：索引字段的数据应该尽量不为 NULL，因为对于数据为 NULL 的字段，数据库较难优化。如果字段频繁被查询，但又避免不了为 NULL，建议使用 0,1,true,false 这样语义较为清晰的短值或短字符作为替代。 
  * **被频繁查询的字段** ：我们创建索引的字段应该是查询操作非常频繁的字段。 
  * **被作为条件查询的字段** ：被作为 WHERE 条件查询的字段，应该被考虑建立索引。 
  * **频繁需要排序的字段** ：索引已经排序，这样查询可以利用索引的排序，加快排序查询时间。 
  * **被经常频繁用于连接的字段** ：经常用于连接的字段可能是一些外键列，对于外键列并不一定要建立外键，只是说该列涉及到表与表的关系。对于频繁被连接查询的字段，可以考虑建立索引，提高多表连接查询的效率。 
2. **被频繁更新的字段应该慎重建立索引** 
  虽然索引能带来查询上的效率，但是维护索引的成本也是不小的。如果一个字段不被经常查询，反而被经常修改，那么就更不应该在这种字段上建立索引了。 
3. **尽可能的考虑建立联合索引而不是单列索引**
  因为索引是需要占用磁盘空间的，可以简单理解为每个索引都对应着一颗 B+树。如果一个表的字段过多，索引过多，那么当这个表的数据达到一个体量后，索引占用的空间也是很多的，且修改索引时，耗费的时间也是较多的。如果是联合索引，多个字段在一个索引上，那么将会节约很大磁盘空间，且修改数据的操作效率也会提升。
4. **注意避免冗余索引**
  冗余索引指的是索引的功能相同，能够命中索引(a, b)就肯定能命中索引(a) ，那么索引(a)就是冗余索引。如（name,city ）和（name ）这两个索引就是冗余索引，能够命中前者的查询肯定是能够命中后者的。在大多数情况下，都应该尽量扩展已有的索引而不是创建新索引。
5. **考虑在字符串类型的字段上使用前缀索引代替普通索引**
  前缀索引仅限于字符串类型，较普通索引会占用更小的空间，所以可以考虑使用前缀索引带替普通索引。

### 使用索引的注意事项

* 对于中到大型表索引都是非常有效的，但是特大型表的话维护开销会很大，不适合建索引 
* 避免 where 子句中对字段施加函数，这会造成无法命中索引。 
* 在使用 InnoDB 时使用与业务无关的自增主键作为主键，即使用逻辑主键，而不要使用业务主键。 
* 删除长期未使用的索引，不用的索引的存在会造成不必要的性能损耗 
* MySQL 5.7 可以通过查询 sys 库的 schema_unused_indexes 视图来查询哪些索引从未被使用 在使用 
* limit offset 查询缓慢时，可以借助索引来提高性能
{% grid %} 
参考 Guide哥，链接: https://javaguide.cn/database/mysql/mysql-index/
{% endgrid %}
