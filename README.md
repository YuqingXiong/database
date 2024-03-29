视频资料：[BiliBili 中字 SQL教程](https://www.bilibili.com/video/BV1UE41147KC/?p=9&share_source=copy_web&vd_source=a37109c4e6370e2d7ea80031c171da1aa)

文档资料：[MySQL 基础 (sjkjc.com)](https://www.sjkjc.com/mysql/data-query/)


1. note link：[MySQL基础](https://github.com/YuqingXiong/database/blob/master/1_MySQL基础.md)
   - 查询，增删改
   - 视图、存储、触发器
   - 事务：ACID
   - 数据类型
   - 设计数据库：范式
   - 索引
2. note link：[MySQL原理](https://github.com/YuqingXiong/database/blob/master/2_MySQL原理.md)
   - 逻辑架构
     - 连接层、服务层、引擎层 
     - 连接器、查询缓存（8.0废除）、解析SQL、执行SQL
   - 存储引擎
     - 表结构、一行记录的存储格式、char与varchar、NULL
     - 数据页
     - B+ 树
     - 聚簇索引与二级索引
     - Buffer Pool (LRU优化：young 和 old 分区)
     - Change Buffer
   - 索引
     - 索引的数据结构、索引组织表
     - 联合索引对查询的优化：避免额外排序，索引覆盖避免回表
     - 索引失效的 6 个场景
     - 索引选择：基于成本的优化器有时判断回表的成本比全表扫描大，而不使用索引
     - 索引应用：正确使用索引的方式、MRR和跳跃式扫描优化机制
     - 面试题
   - 事务
     - ACID：原子性、一致性、隔离性、持久性
     - 隔离级别：读未提交、读已提交、可重复读、序列化
   - 锁
     - 锁的类型
     - 死锁分析：间隙锁互相兼容，但与插入意向锁冲突造成的死锁
   - 日志
     - un do log
     - re do log
     - bindo log
   - 性能调优
     - explain 执行计划：type、key、extra
     - 索引调优
     - SQL 优化
     - MySQL 性能优化
3. note link：[MySQL高可用](https://github.com/YuqingXiong/database/blob/master/3_MySQL高可用.md)
   - 主从复制 
   - 分库分表