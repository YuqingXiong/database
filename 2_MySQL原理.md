# 逻辑架构

## MySQL 架构

![image-20240223161340352](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231613417.png)

MySQL使用 C/S 架构。分为三个层次：连接层、服务层和引擎层

- 连接层：客户端与MySQL服务器建立TCP链接，对传输过来的账号密码做身份认证和权限获取

![image-20240223161812048](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231618159.png)

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

![image-20240226155612542](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402261556941.png)

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
  - ![image-20240227151830806](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402271518904.png)

## 数据页与B+树

[从数据页的角度看 B+ 树 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/mysql/index/page.html)

### 单数据页内以页目录为索引，用二分查询

InnoDB给行记录创建的页目录：

- 将记录分为几个组，包含最小记录和最大记录
- 每个记录组内的最后一条记录的头信息中存储该组中一共有多少条记录，作为 `n_owned` 字段（粉色字段）
- 页目录中的槽（slot）存储每组最后一条记录的地址偏移量

![image-20240227155312990](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402271553199.png)

通过槽查找记录时，可以使用**二分法**快速定位查询的记录在哪个槽（分组），再遍历组内所有记录，找到对应记录

### 多数据页用B+树建立索引

![image-20240227161102222](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402271611291.png)

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

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281450662.png)

### 管理 Buffer Pool

#### 空闲页

从磁盘缓存数据到内存需要找到一个空闲页，为了能快速找到空闲的缓存页，使用了一个 Free链表 （空闲链表）的结构，将空闲的缓存页的控制块作为链表的节点

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281456386.png)

这样就可以根据 Free 链表 找到空闲缓存页，然后再控制块填上对应的缓存信息，再把该控制块对应的节点从Free链表中移除

#### 脏页

更新数据时，会先更新缓存中的数据，再更新到磁盘中，这些被更新的数据页为脏页。

为了知道哪些数据页是脏的，设计了 Flush 链表，链表节点指向脏页的控制块

![img](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281500701.png)

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

![image-20240228194608570](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281946762.png)

**索引组织表：数据根据主键排序存放在索引中。** 此时主键索引也叫聚集索引。在索引组织表中，数据即索引，索引即数据。

![image-20240228194622726](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281946868.png)

二级索引的叶子节点存放的不是数据，而是索引键值（主键值）

**二级索引根据主键值进行再一次查询的操作称为“回表”**

![image-20240228194800830](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402281948903.png)

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

### 对索引使用左或者左右模糊匹配

`like %xx` 或者 `like %xx%` 会造成索引失效

**索引 B+ 树是按照 索引值 有序排列存储的，只能根据前缀进行比较**
