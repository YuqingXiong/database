视频资料：[BiliBili 中字 SQL教程](https://www.bilibili.com/video/BV1UE41147KC/?p=9&share_source=copy_web&vd_source=a37109c4e6370e2d7ea80031c171da1aa)

文档资料：[MySQL 基础 (sjkjc.com)](https://www.sjkjc.com/mysql/data-query/)


1. [MySQL基础](https://github.com/YuqingXiong/database/blob/master/1_MySQL基础.md)
   - 查询，增删改
   - 数据类型
   - 事务
   - 范式
   - 索引
2. [MySQL原理]()
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
   - 事务
   - 锁
   - 日志
   - 性能调优
3. [MySQL高可用]()
   - 主从复制 
   - 分库分表 
   - 分布式ID