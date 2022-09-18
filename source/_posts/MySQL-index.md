---
title: MySQL索引知识点总结
date: 2020-12-16 00:19:51
description: 本文将从一个案例开始，从索引的数据结构、分类、关键概念及如何使用索引提高查找效率等方面对索引知识进行总结
categories: [转发]
tags: [MySQL]
---

> 文章原出处为 <腾讯技术工程> 公众号，个人觉得写的非常好，故搬运过来，以便日后查阅。<br>作者：fanili，腾讯 WXG 后台开发工程师

<!-- More -->

# 什么是索引

在关系数据库中，索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。（百度百科）<br><br>
索引的目的是提高查找效率，对数据表的值集合进行了排序，并按照一定数据结构进行了存储。<br><br>
本文将从一个案例开始，从索引的数据结构、分类、关键概念及如何使用索引提高查找效率等方面对索引知识进行总结。

# 从一个案例开始

## 现象

业务中有个既存的历史 SQL 语句在运行时会导致 DB 服务器过载，进而导致相关服务阻塞无法及时完成。CPU 监控曲线如下：
![优化前CPU使用率](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%E4%BC%98%E5%8C%96%E5%89%8DCPU%E4%BD%BF%E7%94%A8%E7%8E%87.png)<br><br>
从 DB 的 CPU 使用率曲线可以看到业务运行一直处于“亚健康”状态（1），随着业务的增长随时都可能出现问题。这种问题（2）在 11 月 11 日凌晨出现，当时 DB CPU 一直处于 100%高负荷状态，且存在大量的慢查询语句。最终以杀死进程降低 DB 负载、减少业务进程（3）的方式恢复业务。<br><br>
在 11 月 11 日下午，对该业务的 SQL 语句进行了优化，优化的效果如下。业务运行时的 CPU 使用率峰值有很大的降低（对比图 2 的 1，2，3 可见）；慢查询语句几乎在监控曲线上也无法明显观察到（对比图 3 的 1，2，3 可见）。<br><br>
![优化前后的CUP使用率](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%E4%BC%98%E5%8C%96%E5%89%8D%E5%90%8E%E7%9A%84CPU%E4%BD%BF%E7%94%A8%E7%8E%87.png)<br><br>
![优化前后的慢查询数量](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%E4%BC%98%E5%8C%96%E5%89%8D%E5%90%8E%E7%9A%84%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%95%B0%E9%87%8F.png)<br><br>

## 分析

表结构

```sql
CREATE TABLE T_Mch******Stat ({% label primary@FStatDate %} int unsigned NOT NULL DEFAULT 19700101 COMMENT '统计日期',
{% label primary@FMerchantId %} bigint unsigned NOT NULL DEFAULT 0 COMMENT '商户ID',
{% label primary@FVersion %} int unsigned NOT NULL DEFAULT 0 COMMENT '数据版本号',
{% label primary@FBatch %} bigint unsigned NOT NULL DEFAULT 0 COMMENT '统计批次',
{% label primary@FTradeAmount %} bigint NOT NULL DEFAULT 0 COMMENT '交易金额'
PRIMARY KEY ({% label primary@FStatDate %},{% label primary@FMerchantId %},{% label primary@FVersion %}),
INDEX i_FStatDate_FVersion ({% label primary@FStatDate %},{% label primary@FVersion %}))
DEFAULT CHARSET = utf8 ENGINE = InnoDB;
```

从建表语句可以知道该表有两个索引：<br>

1. 主键索引，是一个组合索引，由字段 FStateDate、FMerchantId 和 FVersion 组成；
2. 普通索引，是一个组合索引，由字段 FStateDate 和 FVersion 组成；

优化前的 SQL 语句（做了部分裁剪）A：

```sql
SELECT SQL_CALC_FOUND_ROWS FStatDate,
    FMerchantId,
    FVersion,
    FBatch,
    FTradeAmount,
    FTradeCount
FROM T_Mch******Stat_1020
WHERE FStatDate = 20201020
    AND FVersion = 0
    AND FMerchantId > 0
ORDER BY FMerchantId ASC LIMIT 0, 8000
```

对该 SQL 进行 explain 得到如下结果，Extra 字段的值为 using where，说明并没有使用到索引。<br>
![sql1](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql1.png)<br><br>
优化后的 SQL 语句（做了部分裁剪）B：

```sql
SELECT SQL_CALC_FOUND_ROWS a1.FStatDate,
    a1.FMerchantId,
    a1.FVersion,
    FBatch,
    FTradeAmount,
    FTradeCount
FROM T_Mch******Stat_1020 a1, (
    SELECT FStatDate, FMerchantId, FVersion
    FROM T_Mch******Stat_1020
    WHERE FStatDate = 20201020
        AND FVersion = 0
        AND FMerchantId > 0
        ORDER BY FMerchantId ASC LIMIT 0, 8000 ) a2
where a1.FStatDate = a2.FStatDate
    and a1.FVersion = a2.FVersion
    and a1.FMerchantId = a2.FMerchantId;
```

优化关键步骤为：

-   新增一个子查询，select 字段只有主键字段；

该 SQL 的 explain 结果如下，子查询语句使用了索引，而最终在线上运行结果也证明了优化效果显著。
![sql2](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql2.png)<br><br>

## 疑问

优化后的 SQL 语句 B 比原来的 SQL 语句 A 复杂的多（子查询，临时表关联等），怎么效率会提升，违反直觉？有三个疑问：

1. SQL 语句 A 的查询条件字段都在主键中，主键索引用到了没？
2. SQL 语句 B 的子查询为什么能够用到索引？
3. 前后两条语句执行流程的差异是什么？

# 索引的数据结构

在 MySQL 中，索引是在存储引擎层实现的，而不同的存储引擎根据其业务场景特点会有不同的实现方式。这里会先介绍我们常见的有序数组、Hash 和搜索树，最后看下 Innodb 的引擎支持的 B+树。

## 有序数组

数组是在任何一本数据结构和算法的书籍都会介绍到的一种重要的数据结构。有序数组如其字面意思，以 Key 的递增顺序保存数据在数组中。非常适合等值查询和范围查询。

| ID:1  | ID:2  | .... | ID:N  |
| :---: | :---: | :--: | :---: |
| name1 | name2 | .... | nameN |

在 ID 值没有重复的情况下，上述数组按照 ID 的递增顺序进行保存。这个时候如果需要查询特定 ID 值的 name，用二分法就可以快速得到，时间复杂度是 O(logn)。

```java
// 二分查找递归实现方式
int binary_search(const int arr[], int start, int end, int key)
{
    if (start > end)
        return -1;

    int mid = start + (end - start) / 2;
    if (arr[mid] > key)
        return binary_search(arr, start, mid - 1, key);
    else if (arr[mid] < key)
        return binary_search(arr, mid + 1, end, key);
    else
        return mid;
}
```

有序数组的优点很明显，同样其缺点也很明显。其只适合静态数据，如遇到有数据新增插入，则就会需要数据移动（新申请空间、拷贝数据和释放空间等动作），这将非常消耗资源。

## Hash

哈希表是一种以键-值（K-V）存储数据的结构，我们只需要输入键 K，就可以找到对应的值 V。哈希的思路是用特定的哈希函数将 K 换算到数组中的位置，然后将值 V 放到数组的这个位置。如果遇到不同的 K 计算出相同的位置，则在这个位置拉出一个链表依次存放。哈希表适用于等值查询的场景，对应范围查询则无能为力。<br><br>
![Hsah](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/Hash.png)

## 二叉搜索树

二叉搜索树，也称为二叉查找树、有序二叉树或排序二叉树，是指一颗空树或者具有以下性质的二叉树：

1. 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
2. 若任意节点的右子树不空，则右子树上所有节点的值均大于或等于它的根节点的值；
3. 任意节点的左、右子树也分别为二叉查找树；

![二叉搜索树](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%e4%bc%98%e5%8c%96%e5%89%8d%e5%90%8e%e7%9a%84%e6%85%a2%e6%9f%a5%e8%af%a2%e6%95%b0%e9%87%8f.png)
<br>二叉搜索树相比于其它数据结构的优势在于查找、插入的时间复杂度较低，为 O(logn)。为了维持 O(logn)的查询复杂度，需要保持这棵树是平衡二叉树。<br>
二叉搜索树的查找算法：

1. 若 b 是空树，则搜索失败，否则：
2. 若 x 等于 b 的根节点的值，则查找成功；否则：
3. 若 x 小于 b 的根节点的值，则搜索左子树；否则：
4. 查找右子树。

相对于有序数组和 Hash，二叉搜索树在查找和插入两端的表现都非常不错。后续基于此不断的优化，发展出 N 叉树等。

## B+树

Innodb 存储引擎支持 B+树索引、全文索引和哈希索引。其中 Innodb 存储引擎支持的哈希索引是自适应的，Innodb 存储引擎会根据表的使用情况自动为表生成哈希索引，不能人为干预。B+树索引是关系型数据库中最常见的一种索引，也将是本文的主角。

### 数据结构

在前文简单介绍了有序数组和二叉搜索树，对二分查找法和二叉树有了基本了解。B+树的定义相对复杂，在理解索引工作机制上无须深入、只需理解数据组织形式和查找算法即可。我们可以简单的认为 B+树是一种 N 叉树和有序数组的结合体。<br>
<br>例如:<br>
![B+树数据结构](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/B%2b%e6%a0%91%e6%95%b0%e6%8d%ae%e7%bb%93%e6%9e%84.png)
B+树的 3 个优点:

1. 层级更低，IO 次数更少
2. 每次都需要查询到叶子节点，查询性能稳定
3. 叶子节点形成有序链表，范围查询方便

### 操作算法

-   查找

由根节点自顶向下遍历树，根据分离值在要查找的一边的指针；在节点内使用二分查找来确定位置。

-   插入

![插入](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%e6%8f%92%e5%85%a5.jpg)

-   删除

![删除](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%e5%88%a0%e9%99%a4.jpg)

填充因子（innodb_fill_factor）：索引构建期间填充的每个 B-tree 页面上的空间百分比，其余空间保留给未来索引增长。从插入和删除操作中可以看到填充因子的值会影响到数据页的 split 和 merge 的频率。将值设置小些，可以减少 split 和 merge 的频率，但是索引相对会占用更多的磁盘空间；反之，则会增加 split 和 merge 的频率，但是可以减少占用磁盘空间。Innodb 对于聚集索引默认会预留 1/16 的空间保证后续的插入和升级索引。

# Innodb B+树索引

前文介绍了索引的基本数据结构，现在开始我们从 Innodb 的角度了解如何使用 B+树构建索引，索引如何工作和如何使用索引提升查找效率。

## 聚集索引和非聚集索引

数据库中的 B+树索引可以分为聚集索引和非聚集索引。聚集索引和非聚集索引的不同点在于叶子节点是否是完整行数据。<br><br>
Innodb 存储引擎表是索引组织表，即表中的数据按照主键顺序存放。聚集索引就是按照每张表的主键构造一棵 B+树，叶子节点存放的是表的完整行记录。非聚集索引的叶子节点不包含行记录的全部数据。Innodb 存储引擎的非聚集索引的叶子节点的内容为主键索引值。<br><br>
若数据表没有主键聚集索引是怎么建立的？在没有主键时 Innodb 会给数据表的每条记录生成一个 6 个字节长度的 RowId 字段，会以此建立聚集索引。

## Select 语句查找记录的过程

下面例子将展示索引数据的组织形式及 Select 语句查询数据的过程。

-   建表语句：

```sql
create table T (
    ID int primary key,
    k int NOT NULL DEFAULT 0,
    s varchar(16) NOT NULL DEFAULT '',
    index k(k)
) engine=InnoDB DEFAULT CHARSET=utf8;

insert into T values(100, 1, 'aa'),(200, 2, 'bb'),(300, 3, 'cc'),(500, 5, 'ee'),(600,6,'ff'),(700,7,'gg');
```

-   索引结构示意

![索引结构示意](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%e7%b4%a2%e5%bc%95%e7%bb%93%e6%9e%84%e7%a4%ba%e6%84%8f.png)
左边是以主键 ID 建立起的聚集索引，其叶子节点存储了完整的表记录信息；右边是以普通字段 K 建立的普通索引，其叶子节点的值是主键 ID。

-   Select 语句执行过程

```sql
select * from T where k between 3 and 5;
```

执行流程如下:

1. 在 K 索引树上找到 k=3 的记录，取得 ID=300；
2. 再到 ID 索引树上查找 ID=300 对应的 R3；
3. 在 k 索引树取下一个值 k=5，取得 ID=500；
4. 再回到 ID 索引树查到 ID=500 对应的 R4；
5. 在 k 索引树取下一个值 k=6，不满足条件，循环结束。
   上述查找记录的过程中引入了一个重要的概念：<b>回表</b>，即回到主键索引树搜索的过程。避免回表操作是提升 SQL 查询效率的常规思路及重要方法。那么如何避免回表？

## 覆盖索引

MySQL 5.7,建表语句：

```sql
CREATE TABLE {% label primary@employees %} (
  {% label primary@emp_no %} int(11) NOT NULL,
  {% label primary@birth_date %} date NOT NULL,
  {% label primary@first_name %} varchar(14) NOT NULL,
  {% label primary@last_name %} varchar(16) NOT NULL,
  {% label primary@gender %} enum('M','F') NOT NULL,
  {% label primary@hire_date %} date NOT NULL,
  PRIMARY KEY ({% label primary@emp_no %}),
  KEY {% label primary@i_first_name %} ({% label primary@first_name %}),
  KEY {% label primary@i_hire_date %} ({% label primary@hire_date %})
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

-   SQL 语句 A

```sql
explain select * from employees where hire_date > '1990-01-14';
```

explain 结果：
![sql3](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql3.png)

-   SQL 语句 B

```sql
explain select emp_no from employees where hire_date > '1990-01-14';
```

explain 结果：
![sql4](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql4.png)

-   分析

从前后两次 explain 的结果可以看到 SQL 语句 A 的 extra 为 using where，SQL 语句 B 的 extra 为 using where;using index。这说明 A 没有使用索引，而 B 使用了索引。<br><br>
索引 K 中包含了查询语句所需要的字段 ID 的值，无需再次回到主键索引树查找，也就是“覆盖”了我们的查询需求，我们称之为覆盖索引。覆盖索引可以减少树的搜索次数，显著提升查询性能。

## 最左匹配

-   SQL 语句 A

```sql
explain select * from employees where hire_date > '1990-01-14' and first_name like '%Hi%';
```

explain 结果：
![sql5](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql5.png)

-   SQL 语句 B

```sql
explain select * from employees where hire_date > '1990-01-14' and first_name like 'Hi%';
```

explain 结果：
![sql6](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql6.png)

-   分析

在上述测试的 SQL 语句 A 使用了极端方式: first_name like '%Hi%'，前后都增加模糊匹配使得 SQL 语句无法使用到索引；当去掉最左边的‘%’后，SQL 语句 B 就使用了索引。最左匹配可以是字符串索引的最左 N 个字符，也可以是联合索引的最左 M 的字段。合理规划、使用最左匹配可以减少索引，从而节约磁盘空间。

## 索引下推

何为索引下推？我们先从下面这组对比测试开始，将在 MySQL5.5 版本和 MySQL5.7 版本中执行同一条 SQL 语句：

```sql
select * from employees where hire_date > '1990-01-14' and first_name like 'Hi%';
```

-   在 MySQL 5.5 执行 explain，extra 字段的值显示没有使用索引

![sql7](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql7.png)
执行查询花费时间为 0.12s
![sql8](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql8.png)

-   在 MySQL 5.7 执行 explain，extra 字段的值显示使用了索引下推

![sql9](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql9.png)
执行查询花费时间为 0.02s
![sql10](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql10.png)

-   索引下推

explain 结果中的 extra 字段值包含 using index condition，则说明使用了索引下推。索引下推功能是从 5.6 版本开始支持的。在 5.6 版本之前，i_first_name 索引是没有使用上的，需要每次去主键索引表取完整的记录值进行比较。从 5.6 版本开始，由于索引 i_first_name 的存在，可以直接取索引的 first_name 值进行过滤，这样不符合"first_name like 'Hi%'"条件的记录就不再需要回表操作。

## MRR 优化

MySQL 5.6 版本开始支持 Multi-Range Read(MRR)优化，MRR 优化的目的是为减少磁盘的随机访问，并且将随机访问转化为较为顺序的数据访问，对于 IO-bound 类型的 SQL 查询语句可带来性能极大提升。我们先看下对比测试，以下测试语句在同一个 MySQL 实例下执行，执行前均进行 mysql 服务重启，以保证缓存此没被预热。

-   关闭 MRR

```sql
SET @@optimizer_switch='mrr=off';
select * from employees where hire_date > '1990-01-14' and first_name like 'Hi%';
```

执行耗时为 0.90s
![sql11](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql11.png)

-   开启 MRR

```sql
SET @@optimizer_switch='mrr=off';
SET @@optimizer_switch='mrr=on,mrr_cost_based=off';
 select * from employees where hire_date > '1990-01-14' and first_name like 'Hi%';
```

执行耗时为 0.03s
![sql12](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql12.png)

-   分析

从测试结果可以发现在 mrr 从关闭到开启，耗时从 0.90s 减少到 0.03s，查询速率达到 30 倍的提升。

# 常见的索引失效场景

在 MySQL 表中建立了索引，SQL 查询语句就会一定使用到索引么？不一定，存在着索引失效的场景。我们给 employees 表增一个组合索引，后续例子均基于此表进行分析、测试。

```sql
alter table employees add index i_b_f_l(birth_date, first_name, last_name)
alter table employees add index i_h(hire_date);
```

![sql13](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql13.png)

## 失效场景

-   范围查询（>,<,<>）

```sql
explain select * from employees where hire_date > '1989-06-02';
```

![sql14](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql14.png)

-   查询条件类型不一致

```sql
alter table employees add index i_first_name (first_name);
explain select * from employees where first_name = 1;
```

![sql15](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql15.png)

-   查询条件使用了函数

```sql
explain select * from employees where CHAR_LENGTH(hire_date) = 10;
```

![sql16](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql16.png)

-   模糊查询

```sql
explain select * from employees where hire_date  like  '%1995';
```

![sql17](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql17.png)

-   不使用组合索引的首个字段当条件

```sql
explain select * from employees where last_name = 'Kalloufi' and first_name = 'Saniya';
```

![sql18](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql18.png)

## 为什么会失效

-   顺序读比离散读性能要好<br>
    范围查询一定会导致索引失效么？<br>
    并不会！稍微更改下查询条件看下 explain 的对比结果，可以看到新语句用到索引下推，说明索引并未失效。为什么？<br>
    在不使用覆盖索引的情况下，优化器只有在数据量小的时候才会选择使用非聚集索引。受制于传统的机械磁盘特性，通过聚集索引顺序读数据行的性能会比通过非聚集索引离散读数据行要好。所以，优化器在即使有非聚集索引、但是访问数据量可能达到送记录数的 20%时会选择聚集索引。当然也可以用 Force index 强制使用索引。

```sql
explain select * from employees where hire_date > '1999-06-02';
```

![sql19](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql19.png)

-   无法使用 B+索引快速查找<br>
    B+树索引支持快速查询的基本要素是因为其索引键值是有序存储的，从左到右由小到大，这样就可以在每个层级的节点中快速查并进入下一层级，最终在叶子节点找到对应的值。<br>
    使用函数会使得 MySQL 无法使用索引进行快速查询，因为对索引字段做函数操作会破坏索引值的有序性，所以优化器选择不使用索引。而查询条件类型不一致其实也是同样的情况，因为其使用了隐式类型转换\*。

模糊匹配和不使用组合索引的首字段作为查询条件均是无法快速定位索引位置从而导致无法使用索引。模糊匹配当查询条件是 lwhere A ike 'a%'，a 是 A 的最左前缀时是可能用上索引的（最左匹配），是否用上最终还是依赖优化器对查询数据量的评估。

# 回到初始的案例

让我们回到文章初的案例，尝试回答下当时提出的 3 个问题。

```sql
-- A语句
SELECT FStatDate, FMerchantId, FVersion, FBatch, FTradeAmount, FTradeCount FROM T_Mch******Stat_1020 WHERE FStatDate = 20201020     AND FVersion = 0     AND FMerchantId > 0 ORDER BY FMerchantId ASC LIMIT 0, 8000;

-- B语句
SELECT SQL_CALC_FOUND_ROWS a1.FStatDate,
    a1.FMerchantId,
    a1.FVersion,
    FBatch,
    FTradeAmount,
    FTradeCount
FROM T_Mch******Stat_1020 a1, (
    SELECT FStatDate, FMerchantId, FVersion
    FROM T_Mch******Stat_1020
    WHERE FStatDate = 20201020
        AND FVersion = 0
        AND FMerchantId > 0
        ORDER BY FMerchantId ASC LIMIT 0, 8000 ) a2
where a1.FStatDate = a2.FStatDate
    and a1.FVersion = a2.FVersion
    and a1.FMerchantId = a2.FMerchantId;
```

<b>SQL 语句 A 的查询条件字段都在主键中，主键索引用到了没？</b><br>

-   主键索引其实是有被使用的：索引的范围查询，只是其需要逐条读取和解析所有记录才导致慢查询。<br>

<b>SQL 语句 B 的子查询为什么能够用到索引？</b><br>

1. 前文中我们介绍了聚集索引，其索引键值就是主键。
2. 两条 SQL 语句的不同之处在于 B 语句的子查询语句的 Select 字段都包含在主键字段中，而 A 语句还有其它字段（例如 FBatch 和 FTradeAmount 等）。这种情况下只凭主键索引的键值就能满足 B 语句的字段要求；A 语句则需要逐条取整行记录进行解析。<br>

<b>前后两条语句执行流程的差异是什么？</b><br>

-   SQL 语句 A 的执行过程：
    1. 逐条扫描索引表并比较查询条件
    2. 遇到符合查询条件的则读取整行数据返回
    3. 回到 a 步骤，直至完成所有索引记录的比较
    4. 对返回的所有符合条件的记录（完整的记录）进行排序
    5. 选取前 8000 条数据返回
-   SQL 语句 B 的执行过程：
    1. 逐条扫描索引表并比较查询条件
    2. 遇到符合查询条件的则从索引键中取相关字段值返回
    3. 回到 a 步骤，直至完成所有索引记录的比较
    4. 对返回的所有符合条件的记录（每条记录只有 3 个主键）进行排序
    5. 选取前 8000 条数据返回形成临时表
    6. 关联临时表与主表，使用主键相等比较查询 8000 条数据
-   对比两个 SQL 语句的执行过程，可以发现差异点集中在步骤 2 和步骤 4。在步骤 2 中 SQL 语句 A 需要随机读取整行数据并解析非常耗资源；步骤 4 涉及 MySQL 的排序算法，这里也会对执行效率有影响，排序效果上看 SQL 语句 B 比 SQL 语句 A 好。

![sql20](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/sql20.png)

# 名词解释

-   主键索引
-   顾名思义该类索引由表的主键组成，从左到右由小到大排序。一个 Innodb 存储表只有一张主键索引表（聚集索引）。
-   普通索引
-   最为平常的一种索引，没有特别限制。
-   唯一索引
-   该索引的字段不能有相同值，但允许有空值。
-   组合索引
-   由多列字段组合而成的索引，往往是为了提升查询效率而设置。

# 总结

在文章开始时介绍了常见的几种索引数据结构，适合静态数据的有序数组、适合 KV 结构的哈希索引及兼顾查询及插入性能的搜索二叉树；然后介绍了 Innodb 的常见索引实现方式 B+树及 Select 语句使用 B+树索引查找记录的执行过程，在这个部分我们了解了几个关键的概念，回表、覆盖索引、最左匹配、索引下推和 MMR；之后还总结了索引的失效场景及背后的原因。最后，我们回到最初的案例，分析出优化前后 SQL 语句在使用索引的差异，进而导致执行效率的差异。<br><br>
本文介绍了索引的一些粗浅知识，希望能够对读者有些许帮助。作为阶段性学习的一个总结，文章对 MySQL 索引的相关知识基本上是浅藏辄止，日后还需多多使用和深入学习。<br><br>
何以解忧？唯有学习。<br><br>
![思维导图](https://cdn.jsdelivr.net/gh/Kinsiy/cdn/KINSIY-PIC/%e6%80%9d%e7%bb%b4%e5%af%bc%e5%9b%be.png)

# 餐卡书目和资料

-   《MySQL 技术内幕-InnoDB 存储引擎》第二版，作者：姜承尧
-   《MySQL 实战 45 讲》，作者：林晓斌
-   https://dev.mysql.com/doc/refman/8.0/en/
-   https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9
-   重温数据结构：理解 B 树、B+ 树特点及使用场景 - Android
-   https://github.com/zhangyachen/zhangyachen.github.io/issues/117
