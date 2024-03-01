# 逻辑架构

## MySQL 架构

![image-20240223161340352](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037504.png)

MySQL使用 C/S 架构。分为三个层次：连接层、服务层和引擎层

- 连接层：客户端与MySQL服务器建立TCP链接，对传输过来的账号密码做身份认证和权限获取

![image-20240223161812048](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037587.png)

- 服务层：
  - SQL 接口：接收用户的SQL命令，返回用户需要的查询结果
  - 解析器：对SQL语句进行语法分析、语义分析后构建语法树
  - 优化器：生成执行计划，选择索引
  - 查询缓存：缓存SELECT执行结果，避免重复查询

- 引擎层：真正负责数据的存储和提取

## 一条SQL的执行过程

博客：[执行一条 select 语句，期间发生了什么？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/base/how_select.html)

B站视频：https://www.bilibili.com/video/BV1iq4y1u7vj/?p=110

- 连接器：
  - 客户端通过TCP三次握手与MySQL服务器建立连接
  - 校验客户端发来的用户名和密码是否正确
  - 读取用户权限
- 查询缓存（8.0废除）
  - 如果是查询语句，先去查询缓存里查找之前是否执行过这个查询语句，如果命中了就直接将结果返回给客户端
- 解析SQL
  - 词法分析：识别关键字
  - 语法分析：根据语法规则判断是否满足MySQL语法后构建语法树
- 执行SQL
  - 预处理阶段：检查表和字段是否存在；将 `SELECT *` 中的 `*` 扩展为所有列
  - 优化阶段：选择成本最小的执行计划，例如选择查询所需要用的索引
  - 执行阶段：根据执行计划执行SQL查询语句，从存储引擎中读取记录

## 引擎分类

### InnoDB 引擎：外键支持功能的事务存储引擎

- MySQL5.5以后默认采用 InnoDB 引擎
- InnoDB是MySQL**默认事务型引擎**，被设计用于处理大量短期(short-lived)事务，确保了事务的完整提交(Commit)和回滚(Rollback)
- 除了增加和查询，还需要更新和删除操作，那么优先选择InnoDB存储引擎
- 数据文件结构：
  - `表名.frm` ：存储表结构(MySQL 8.0时，合并在 `表名.ibd` 中)
  - `表名.ibd` ：存储数据和索引
- InnoBD适用于**巨大数据量，高并发**的场景（其行锁的特性）
- 对于 MyISAM 存储引擎，**InnoDB的写处理效率差一些**，并且占用更多的磁盘空间以保存数据和索引
- MyISAM只混村索引，不缓存真实数据；InnDB不仅缓存索引还缓存真实数据，**对内存要求较高** ，而且内存大小对性能有决定性影响。

### MyISAM 引擎：主要的非事务处理存储引擎

- MyISAM 不支持事务，行级锁，外键，崩溃后无法安全恢复
- 5.5前的默认存储引擎
- 优点是访问速度快，适用于对事务完整性没有要求或者主要是 SELECT 和 INSERT为主的应用
- 针对数据统计有额外的常数存储，所以 count(*)的查询效率很高
- 数据文件结构：
  - `表名.frm`：存储表的结构
  - `表名.MYD`：存储数据
  - `表名.MYI`：存储索引
- 应用场景：适用于查询和插入为主的应用

![image-20240226155612542](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037567.png)

### 其他存储引擎

- Archive 引擎：只支持 INSERT 和 SELECT ...
- Blackhole
- CSV
- Federated
- Memory
- Merge
- NDB 集群引擎
- 第三方存储引擎
  - OLTP类引擎：XtraDB, PBXT
  - ...

# InnoDB存储引擎

## 一行记录的存储格式

[MySQL 一行记录是怎么存储的？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/base/row_format.html)

- InnoDB表结构，存放位置
- varchar, Null 只的保存方式

## char 和 varchar

- 区别和性能差异

- 占用空间的计算：字符长度+占用字符空间

- 占用文件和内存大小的区别：

  [MySql中varchar(10)和varchar(100)的区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/130731372)

  - 占用磁盘的存储空间是一样的
  - 但是消耗的内存不一样，更长的列消耗的内存会更多：
  - ![image-20240227151830806](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037450.png)

## 数据页与B+树

[从数据页的角度看 B+ 树 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/index/page.html)

### 单数据页内以页目录为索引，用二分查询

InnoDB给行记录创建的页目录：

- 将记录分为几个组，包含最小记录和最大记录
- 每个记录组内的最后一条记录的头信息中存储该组中一共有多少条记录，作为 `n_owned` 字段（粉色字段）
- 页目录中的槽（slot）存储每组最后一条记录的地址偏移量

![image-20240227155312990](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037528.png)

通过槽查找记录时，可以使用**二分法**快速定位查询的记录在哪个槽（分组），再遍历组内所有记录，找到对应记录

### 多数据页用B+树建立索引

![image-20240227161102222](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037483.png)

- 只有叶子节点才存放数据，其他上层节点仅存放目录项的索引
- 所有节点按照索引键大小排序，构成一个双向链表，便于范围查询

当查找某个主键的记录时，首先通过B+树二分的查找该记录所在的数据页，然后在数据页内再次通过页目录二分查找该记录

### 聚簇索引和二级索引

聚簇索引是一种数据存储方式，聚簇索引默认是主键（没有主键，InnoDB会选择一个唯一的非空索引或者隐式定义一个主键）

聚簇索引按照表的主键构造一棵B+树，树的叶子节点存储数据页。这个特性决定了索引组织表中数据是索引的一部分，数据只有一份，所以聚簇索引只有一个

非聚簇索引（二级索引/辅助索引）的叶子节点存储不再是数据页而是主键值，查找时，需要先找到主键值，再二次查找根据主键值找到行数据。

## Buffer Pool

[揭开 Buffer Pool 的面纱 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/buffer_pool/buffer_pool.html)

为了提升MySQL的查询性能，当数据从磁盘取出后，InnoDB设计了一个缓冲池（Buffer Pool），将数据缓存到内存中，便于下次查询同样数据的时候，直接从内存中读取。

Buffer Pool 缓存的数据：

InnoDB 是以数据页为单位在磁盘和内存之间进行数据交换的，所以 InnoDB为Buffer Pool申请一块连续的内存空间，然后按照默认的 16kb 大小划分为一个个页，这个Buffer Pool 中的页称为缓存页。此时缓存页是空闲的，随着程序的运行，才会有来自磁盘中的页被缓存。

Buffer Pool 除了缓存「索引页」和「数据页」，还包括了 undo 页，插入缓存、自适应哈希索引、锁信息等等。

InnoDB为每个缓存页创建一个控制块，放在 Buffer Pool 的最前面

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037168.png)

### 管理 Buffer Pool

#### 空闲页

从磁盘缓存数据到内存需要找到一个空闲页，为了能快速找到空闲的缓存页，使用了一个 Free链表 （空闲链表）的结构，将空闲的缓存页的控制块作为链表的节点

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037243.png)

这样就可以根据 Free 链表 找到空闲缓存页，然后再控制块填上对应的缓存信息，再把该控制块对应的节点从Free链表中移除

#### 脏页

更新数据时，会先更新缓存中的数据，再更新到磁盘中，这些被更新的数据页为脏页。

为了知道哪些数据页是脏的，设计了 Flush 链表，链表节点指向脏页的控制块

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037206.png)

后台线程可以遍历 Flush 链表将脏页写入磁盘。

#### 提高缓存命中率（LRU的优化）

Buffer Pool 维护一个LRU(least recently used)链表，当访问的页不在 Buffer Pool 中时，将该数据页插入链表头部，同时淘汰末尾的最久未使用节点。

**预读失效问题：**

由于程序有空间局部性的，所以MySQL设置了预读机制。加载数据页的时候，会把它相邻的数据页一起加载进来，减少磁盘IO

但是当这些提前加载来的数据页没有被访问的时候，就是预读失效。

这时简单的 LRU 算法就会把预读页放到 LRU 链表头部，如果这些预读页一直不被访问，就会出现不被访问的数据页占据LRU链表前排的位置，而末尾淘汰的页可能是频繁访问的页，大大降低了缓存命中率

**LRU划分两个区域解决预读失效问题：**

将 LRU 划分为两个区域：old 区域 和 young 区域

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/mysql/innodb/young%2Bold.png)

当有预读页的时候首先加入到 old 区域，只有当这个页被真正访问的时候才被会被插入 young 区域的头部

**Buffer Pool 污染：**

当一个 SQL 扫描了大量数据时候，Buffer Pool 中的页可能都会被替换出去，导致大量的热数据被淘汰了，这些热数据再次被访问的时候又会导致大量的磁盘 IO

**增加停留 old 区域时间判断解决 Buffer Pool 污染问题：**

像全表扫描的这种查询，很多缓冲页其实只会被访问一次，但是却进入了 young 区域，所以需要提高进入 young 区域的门槛。

于是进入 young 区域条件增加了一个停留在 old 区域的时间判断：

- 首先在控制块中记录 old 区域缓冲页的第一次访问时间
- 然后如果后续的访问时间与第一次访问的时间差值在某个时间间隔内，那么这个缓冲页就不会被从 old 移动到 young 头部
- 反之如果差值大于间隔，那么就把缓冲页从 old 移动到 young 头部

这个间隔时间是由 `innodb_old_blocks_time` 控制的，默认是 1000 ms。

#### 脏页刷盘的时机

修改后的缓冲页需要被刷入磁盘保持数据一致性，每次修改就刷盘的性能很差，所以往往会在一定的时机批量刷盘

- 首先 redo log 日志满了，触发刷盘
- Buffer Pool 空间不足，需要将部分缓冲页淘汰，如果该数据页是脏页，就需要先将脏页同步到磁盘
- MySQL认为空闲，后台线程会定期刷入适量的脏页到磁盘
- MySQL 正常关闭前，把所有脏页刷入磁盘

MySQL宕机时，日志会让MySQL拥有崩溃恢复的能力，所以数据不会丢失。

## Change Buffer

[09 普通索引和唯一索引，应该怎么选择？ (lianglianglee.com)](https://learn.lianglianglee.com/专栏/MySQL实战45讲/09  普通索引和唯一索引，应该怎么选择？.md)

查询：

普通索引比唯一索引多一次查询，由于读一次数据是以数据页为单位的，所以多一次查询的代价并不大

更新：

对于普通索引，当需要更新一个数据页且不在缓存中时，InnoDB会将更新操作缓存到 change Buffer 中。这样就不需要从磁盘中读入这个数据页了，下次查询到这个数据页的时候，再执行 change buffer 中与这个页有关的操作，得到最新结果的过程称为 merge

对于唯一索引，无法使用 change buffer，因为更新数据时，需要判断插入的数据是否是唯一的，就需要将数据页读入内存才能判断。

# 索引

## 索引数据结构

[08 索引：排序的艺术 (lianglianglee.com)](https://learn.lianglianglee.com/专栏/MySQL实战宝典/08  索引：排序的艺术.md)

**什么是索引？**

索引是提升查询速度的一种**数据结构**

因为索引在建立时对数据进行了排序，才会提升查询速度

**B+树与其他数据结构相比的特点：**

基于磁盘的数据结构，树很矮，通常3-4层，可以存放千万到上亿的排序数据。树的高度低意味着访问效率高，只用3-4次IO

[10｜数据库索引：为什么MySQL用B+树而不用B树？ | JUST DO IT (leeshengis.com)](https://leeshengis.com/archives/672553)

## 索引存储

[09 索引组织表：万物皆索引 (lianglianglee.com)](https://learn.lianglianglee.com/专栏/MySQL实战宝典/09  索引组织表：万物皆索引.md)

堆表中的数据无序存放，数据的排序完全依赖于索引

![image-20240228194608570](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037329.png)

**索引组织表：数据根据主键排序存放在索引中。** 此时主键索引也叫聚集索引。在索引组织表中，数据即索引，索引即数据。

![image-20240228194622726](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037477.png)

二级索引的叶子节点存放的不是数据，而是索引键值（主键值）

**二级索引根据主键值进行再一次查询的操作称为“回表”**

![image-20240228194800830](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402282037761.png)

二级索引的设计可以使得记录发生修改时，其他索引无需进行维护，除非记录的主键发生修改。所以，与堆表相比，索引组织表在存在大量变更的场景下，性能优势明显。

二级索引的性能开销：

二级索引的插入随机时，性能开销较大

由于二级索引需要再次查询主键值进行回表，所以主键值尽可能紧凑，对二级索引的性能和存储友好

主键值比较顺序时，对聚集索引性能友好

## 联合索引

[10 组合索引：用好，性能提升 10 倍！ (lianglianglee.com)](https://learn.lianglianglee.com/专栏/MySQL实战宝典/10  组合索引：用好，性能提升 10 倍！.md)

组合索引/联合索引（Compound Index）指多个列组合而成的 B+ 树索引，所以是对多个列排序

组合索引可以是主键索引，或者二级索引。

（a,b) 和 (b,a) 的排序结果并不同，所以组合索引 (a, b) 可以对下面的索引优化：

```mysql
SELECT * FROM my_table WHERE a = ?
SELECT * FROM my_table WHERE a = ? AND b = ?
SELECT * FROM my_table WHERE b = ? AND a = ?

SELECT * FROM my_table WHERE a = ? ORDER BY b
```

但是 (a, b) 排序并不能推出 (b, a) 排序，所以下面的无法优化：

```mysql
SELECT * FROM my_table WHERE b = ?

SELECT * FROM my_table WHERE b = ? ORDER BY a
```

### 索引设计

#### 避免额外排序

有时遇到根据某个列 my_col 进行查询，然后按照时间 date 排序的方式逆序展示

如果仅仅为该列设置索引，取出数据后还需要进行额外的排序

**最好的方法是：取出结果时就已经排好序了**

创建组合索引：idx_col_date 对字段 （my_col, date)进行索引

```mysql
ALTER TABLE my_table ADD INDEX
idx_col_date(my_col, date);
```

#### 避免回表

二级索引查到主键值后还需要回表定位到完整的数据。

但是如果查询的字段都在二级索引的叶子节点中，就可以直接返回结果，无需回表，因为数据已经都拿到了。这种**通过组合索引避免回表的优化技术也成为索引覆盖**

## 索引失效

[索引失效有哪些？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/index/index_lose.html)

并不是所有查询都能用索引，有时候会发生索引失效，从而进行全表扫描，使得查询性能大大降低。

这里有6中索引失效的场景

### 1 对索引使用左或者左右模糊匹配

`like %xx` 或者 `like %xx%` 会造成索引失效

**因为索引 B+ 树是按照「索引值」有序排列存储的，只能根据前缀进行比较。**

### 2 对索引使用函数

索引保存的是索引字段的原始值，而不是经过函数计算后的值，所以就无法应用索引。

例如 :

```mysql
select * from t_user where length(name) = 6;
select * from t_user where data_format(register_date, '%Y-%m') = '2024-01';
-- 或许开发同学认为在 register_date 创建了索引，所以所有的 SQL 都可以使用该索引。但索引的本质是排序， 索引 idx_register_date 只对 register_date 的数据排序，又没有对DATE_FORMAT(register_date) 排序，因此上述 SQL 无法使用二级索引idx_register_dat
```

函数索引可以解决问题：

```mysql
alter table t_user add key idx_name_length ((length(name)));
```

### 3 对索引进行表达式计算

```mysql
select * from t_user where id + 1 = 10; -- ALL
select * from t_user where id = 10 - 1; -- 索引
```

因为索引保存的是索引字段的原始值，而不是 id + 1 表达式计算后的值，所以无法走索引，只能通过把索引字段的取值都取出来，然后依次进行表达式的计算来进行条件判断，因此采用的就是全表扫描的方式。

### 4 对索引隐式类型转换

字符串类型字段，查询参数是整型，造成索引失效

```mysql
-- phone 为 varchar
select * from t_user where phone = 1300000001;
```

整数类型字段，查询参数即使是字符串，也不会索引失效

```mysql
 explain select * from t_user where id = '1';
```

这是因为 MySQL 是将字符串转换为数字进行处理的。当**MySQL 在遇到字符串和数字比较的时候，会自动把字符串转为数字，然后再进行比较**。

### 5 联合索引不是最左匹配

联合索引的正确使用要遵循最左匹配原则

联合索引 (a, b, c) 可匹配的情况：

```mysql
where a = 1;
where a = 1 and b = 2 and c = 3;
where a = 1 and b = 2;
-- a 字段的顺序不重要，mysql 有查询优化器
```

但是下面不包含 a 的查询就不符合最左匹配原则：

```mysql
where b = 2;
where c = 3;
where b = 2 and c = 3;
```

特殊的索引截断:

```mysql
where a = 1 and c = 3;
-- MySQL 5.5 前是 a 走索引后开始回表，交给 Server 层进行 c 字段的比较
-- MySQL 5.6 后使用索引下推，先在引擎层过滤出复合条件的数据再返回给 Server 层，减少回表次数
```

### 6  WHERE 子句中的 OR

OR 的含义就是两个只要满足一个即可，因此只有一个条件列是索引列是没有意义的。

只要有条件列不是索引列，就会进行全表扫描。

例如:

```mysql
select * from t_user where id = 1 or age = 18;
```

避免全表扫描，把 age 设置为索引后，MySQL 就会 Index merge，将根据 id 和 age 索引的搜索结果集进行合并

## 索引选择

[11 索引出错：请理解 CBO 的工作原理 (lianglianglee.com)](https://learn.lianglianglee.com/专栏/MySQL实战宝典/11  索引出错：请理解 CBO 的工作原理.md)

MySQL 的优化器如何执行？选择索引的依据？

SQL 优化器会选择成本最低的执行计划执行，称为基于成本的优化器 CBO (Cost-based Optimizer)

一条 SQL 的计算成本：

```
Cost = Server Cost + Engine Cost
     = CPU cost + IO cost
```

CPU cost ：计算开销，比如索引键值变焦、记录值比较，结果集排序

IO cost ： IO 开销，区分表数据是否在内存，计算读内存的IO开销以及读磁盘的IO开销

### 索引出错案例

#### 1 未能使用创建的索引

查询范围不同导致优化器选择不同

使用二级索引需要回表，查询范围大时，大量回表操作的成本可能比全表扫描的成本大。

#### 2 索引创建在有限状态上

创建索引的列上的数据存在数据倾斜，通过索引查询少量数据时，优化器认为数据是平均分布的，为了避免回表，而选择全表扫描。

这时可以用 8.0 的功能创建直方图，让 MySQL 知道数据分布

## 索引应用

[(五)MySQL索引应用篇：建立索引的正确姿势与使用索引的最佳指南！ - 掘金 (juejin.cn)](https://juejin.cn/post/7149074488649318431)

建立索引的优点和缺点？正确使用索引？判断场景是否适合建立索引？

#### MRR(Multi-Range Read)机制

针对辅助索引的回表查询，减少离散 IO ，将随机 IO 转换为顺序 IO ，从而提高查询效率

`MRR`机制中，对于辅助索引中查询出的`ID`，会将其放到缓冲区的`read_rnd_buffer`中，然后等全部的索引检索工作完成后，或者缓冲区中的数据达到`read_rnd_buffer_size`大小时，此时`MySQL`会对缓冲区中的数据排序，从而得到一个有序的`ID`集合：`rest_sort`，最终再根据顺序`IO`去聚簇/主键索引中回表查询数据。

#### Index Skip Scan索引跳跃式扫描

MySQL 8.x 加入了索引跳跃式扫描的优化机制，使得查询条件不是必须满足最左匹配原则。

实现方式：重构 SQL，AND 增加一个第一个字段的查询条件，把第一个字段的所有不重复值都作为索引连接到原来的查询上：

```mysql
SELECT * FROM `tb_xx` WHERE B = `xxx` AND C = `xxx`;

-- 重构：
SELECT * FROM `tb_xx` WHERE B = `xxx` AND C = `xxx`
UNION ALL
SELECT * FROM `tb_xx` WHERE B = `xxx` AND C = `xxx` AND A = "yyy"
......
SELECT * FROM `tb_xx` WHERE B = `xxx` AND C = `xxx` AND A = "zzz";

```

## 索引常见面试题

[索引常见面试题 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/index/index_interview.html)

# 事务

## 事务 ACID

- 原子性（Atomicity）：操作不可分割
- 一致性（Consistency）：数据满足现实约束
- 隔离性（Isolation)：不同操作互不影响
- 持久性（Durability）：状态的转换结果永久保存

## 事务的隔离级别

[事务隔离级别是怎么实现的？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/transaction/mvcc.html)

并行事务时可能会遇到的问题：

- **脏读：**读到了另一个未提交事务修改后过的数据（提前读了）
- **不可重复读：**同一个事务多次读取同一数据，但是前后读到的数据不同。（前后读的间隔，数据被其他事务修改了）
- **幻读：**同一事务前后两次查询到的**记录数量**不同。（前后读间隔数据被其他事务修改了）

简单的说：

- 脏读：读到其他事务未提交的数据；
- 不可重复读：前后读取的数据不一致；
- 幻读：前后读取的记录数量不一致

SQL 提出四个隔离级别来避免上述三个现象，隔离级别越高，性能越差：

- 读未提交：事务未提交时，所作的变更可以被其他事务看到
- 读已提交：事务提交后，所作的变更才可以被其他事务看到
- 可重复读：InnoDB 默认隔离级别。同一事务执行过程中看到的数据，与启动事务时看到的数据一致。
- 序列化（串行化）：对读取的记录加读写锁，多个事务发生读写冲突时，后访问的事务必须等待前面的事务完成，才能执行

不同隔离级别可以发生的现象：

![image-20240229150955166](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402291509400.png)

- 在「读未提交」隔离级别下，可能发生脏读、不可重复读和幻读现象；
- 在「读提交」隔离级别下，可能发生不可重复读和幻读现象，但是不可能发生脏读现象；
- 在「可重复读」隔离级别下，可能发生幻读现象，但是不可能脏读和不可重复读现象；
- 在「串行化」隔离级别下，脏读、不可重复读和幻读现象都不可能会发生。

**MySQL InnoDB 引擎的默认隔离级别虽然是「可重复读」，但是它很大程度上避免幻读现象**

- 针对**快照读**（普通 select 语句），是**通过 MVCC 方式解决了幻读**
- 针对**当前读**（select ... for update 等语句），是**通过 next-key lock（记录锁+间隙锁）方式解决了幻读**

**四种隔离级别具体是如何实现？**

- 对于「读未提交」隔离级别的事务来说，因为可以读到未提交事务修改的数据，所以直接读取最新的数据就好了；
- 对于「串行化」隔离级别的事务来说，通过加读写锁的方式来避免并行访问；
- 对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 **Read View **来实现的，它们的区别在于创建 Read View 的时机不同，大家可以把 Read View 理解成一个数据快照。**「读提交」隔离级别是在「每个语句执行前」都会重新生成一个 Read View，而「可重复读」隔离级别是「启动事务时」生成一个 Read View，然后整个事务期间都在用这个 Read View**。

### Read View 在 MVCC 里如何工作的？

创建事务时，会创建 **Read View，包含四个字段**:

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402291543776.png)

- 记录了事务的id，
- 活跃且未提交的事务 id 列表，
- 列表中最小事务的id，
- 给下一个事务的id(最大事务id+1)

聚簇索引包含的两个与事务有关的隐藏列：

![图片](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402291549709.png)

- trx_id：当一个事务对某个聚簇索引记录进行改动，就会记录该事务的 id 到 trx_id
- roll_pointer：每次对某个聚簇索引记录进行改动时，会把旧版本的记录写入 undo 日志中。**这个隐藏列是一个指针，指向每个旧版本的记录**。便于找到未修改前的记录

每一行记录的 trx_id 在遇到一个事务的 Read View 时，可以划分为三种情况：

- trx_id < min_trx_id 该版本记录对当前事务可见
- trx_id > max_trx_id 该版本记录对当前事务不可见
- min_trx_id < trx_id  < max_trx_id :
  - trx_id 在 m_ids 列表中，该事务未提交，该版本记录对当前事务不可见
  - trx_id 不在 m_ids 列表中，该事务已提交，该版本记录对当前事务可见

**这种通过 版本链 来控制并发事务访问同一条记录的行为叫做 MVCC （多版本并发控制）**

#### 可重复读是如何工作的？

启动事务的时候，生成一个 Read View

整个事务期间都用这个 Read View，所以这个事务后的修改对当前事务屏蔽了，修改就不可见了，读到的都是同一个数据

#### 读提交是如何工作的？

每次读取数据的时候，生成一个Read View

每次读数据就创建新的 Read View 那么前面所有已提交的事务就都对这次读可见了。

# 锁

[MySQL 有哪些锁？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/lock/mysql_lock.html)

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402291616161.png)

元数据锁（MDL）：对数据库表进行操作时，会自动为这个表加上MDL，防止执行CURD时，其他线程对这个表结构做变更

意向锁：

- 在使用 InnoDB 引擎的表里对某些记录加上「共享锁」之前，需要先在表级别加上一个「意向共享锁」；
- 在使用 InnoDB 引擎的表里对某些纪录加上「独占锁」之前，需要先在表级别加上一个「意向独占锁」；

**意向锁的目的是为了快速判断表里是否有记录被加锁**。

行级锁：

共享锁（S锁）满足读读共享，读写互斥。独占锁（X锁）满足写写互斥、读写互斥。

行级锁的类型主要有三类：

- Record Lock，记录锁，也就是仅仅把一条记录锁上；这样其他事务就无法对这条记录进行修改了。
- Gap Lock，间隙锁，锁定一个范围，但是不包含记录本身；只存在于可重复读隔离级别，目的是为了解决可重复读隔离级别下幻读的现象。
- Next-Key Lock：Record Lock + Gap Lock 的组合，锁定一个范围，并且锁定记录本身。

## MDL 锁

[加索引可能引发的事故，我们要心中有数 - 掘金 (juejin.cn)](https://juejin.cn/post/6844904193531052040)

## 死锁

[MySQL 死锁了，怎么办？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/lock/deadlock.html)

[字节面试：加了什么锁，导致死锁的？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/lock/show_lock.html)

更新数据时，可能会对一个范围的数据加上X类型的间隙锁，然而两个事务的间隙锁是相互兼容的

当插入数据时，事务需要获取一个行的插入意向锁（行级锁），然而一个事务的插入意向锁和另一个事务的间隙锁是冲突的。

# 日志

[MySQL 日志：undo log、redo log、binlog 有什么用？ | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/log/how_update.html)

更新语句的流程涉及三种日志：

- undo log(回滚日志)：实现事务的原子性，用于事务回滚和 MVCC
- redo log(重做日志)：实现事务的持久性，用于掉电等故障恢复
- binlog(归档日志)：Server层日志，用于数据备份和主从复制

> 具体更新一条记录 `UPDATE t_user SET name = 'xiaolin' WHERE id = 1;` 的流程如下:
>
> 1. 执行器负责具体执行，会调用存储引擎的接口，通过主键索引树搜索获取 id = 1 这一行记录：
>    - 如果 id=1 这一行所在的数据页本来就在 buffer pool 中，就直接返回给执行器更新；
>    - 如果记录不在 buffer pool，将数据页从磁盘读入到 buffer pool，返回记录给执行器。
> 2. 执行器得到聚簇索引记录后，会看一下更新前的记录和更新后的记录是否一样：
>    - 如果一样的话就不进行后续更新流程；
>    - 如果不一样的话就把更新前的记录和更新后的记录都当作参数传给 InnoDB 层，让 InnoDB 真正的执行更新记录的操作；
> 3. 开启事务， InnoDB 层更新记录前，首先要记录相应的 undo log，因为这是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面，不过在内存修改该 Undo 页面后，需要记录对应的 redo log。
> 4. InnoDB 层开始更新记录，会先更新内存（同时标记为脏页），然后将记录写到 redo log 里面，这个时候更新就算完成了。为了减少磁盘I/O，不会立即将脏页写入磁盘，后续由后台线程选择一个合适的时机将脏页写入到磁盘。这就是 **WAL 技术**，MySQL 的写操作并不是立刻写到磁盘上，而是先写 redo 日志，然后在合适的时间再将修改的行数据写到磁盘上。
> 5. 至此，一条记录更新完了。
> 6. 在一条更新语句执行完成后，然后开始记录该语句对应的 binlog，此时记录的 binlog 会被保存到 binlog cache，并没有刷新到硬盘上的 binlog 文件，在事务提交时才会统一将该事务运行过程中的所有 binlog 刷新到硬盘。
> 7. 事务提交，剩下的就是「两阶段提交」的事情了

# 性能调优

## Benchmark

> 高性能 MySQL 第三版 第二章

## Explain 执行计划

### table

Explanin 语句输出的每条记录对应着某个**单表**的访问方法，table列表示该表表名

### id

查询语句的**每一个 SELECT 关键字，MySQL 会给它分配唯一的 id 值**

对于连接查询，一个SELECT 语句跟随多个表，上文说过，每条记录对应单个表，所以**在连接查询的执行计划中，每个表都会对应一条记录，这些记录的 id 值相同；**

> 查询优化器可能对涉及子查询的查询语句进行重写，从而转换为连接查询

### select_type

[搞清楚 MySQL 派生表、物化表、临时表-CSDN博客](https://blog.csdn.net/cschmin/article/details/123148143)

为每个SELECT关键字代表的小查询定义了一个称之为 select_type 的属性。可取的值：

- SIMPLE ：查询语句不包含 UNION 或者子查询的查询算作 simple 类型
- PRIMARY：包含 UNION、UNION ALL 或者子查询的大查询来说，它由几个小查询组成，最左边的那个子查询的 select_type 就职 primary
- UNION：包含 UNION、UNION ALL 或者子查询的大查询来说，它由几个小查询组成，除了最左边的那个子查询外的 select_type 就是 union
- UNION RESULT：MySQL 选择临时表完成 UNION 查询的去重工作，该临时表的 select_type 就是 union result
- SUBQUERY：**没懂**
- DEPENDENT SUBQUERY
- DEPENDENT UNION
- DERIVED：派生表对应的子查询
- MATERIALIZED：where 下的子查询结果被物化后与外层查询连接，该物化表为 materialized
- UNCACHEABLE SUBQUERY：不常用
- UNCACHEABLE UNION ：不常用

### partitions

分区这个列一般为 NULL

### type

type 表示对某个表执行查询的访问方法

- system；该表使用的存储引擎的统计数据是精确的，比如 MyISAM，Memory
- const：根据主键或者唯一二级索引列与常数进行等值匹配时，对表的访问方法就是 const
- index：使用索引覆盖，但需要扫描全部索引记录，该表的访问方法为 index
- ALL：全表扫描
- ...

### possible_keys

可能用到的索引

### key

经过查询优化器计算使用不同索引的成本后，最后决定使用的 索引

### key_len

索引的最大长度

### ref

索引列使用等值匹配的条件去执行查询时，即type为const、eq_ref、ref...其中之一时。ref 展示与索引列作等值匹配的东西具体是啥，比如一个常数或某个具体的列

### rows

语句扫描的行数

### filtered

满足搜索条件的记录数量

### extra

输出额外信息

- No tables used ：没有FROM子句
- impossible WHERE：where 子句永远为 false时提示
- No matching min/max row
- Using index：使用索引覆盖
- Using index condition：使用了索引下推
- Using filesort：在内存或者磁盘中进行排序
- ...

## 索引调优

> 高性能 MySQL 第三版 第五章

## SQL 优化

> 高性能 MySQL 第三版 第六章

## 连接池

- 短链接
- 长连接
- 连接池中请求处理的策略

## MySQL 性能优化

优化思路
