## 场景分析
| 场景                 | 目标                                                                                  | 挑战                                                                                                     | **推荐工作模式**                                                                                                                                                                                                       | 效果                                                                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 大量存量老业务的应用         | - 突破单机资源瓶颈（主要是CPU/IO）。<br>    <br>- 优化SQL查询RT。                                      | - 历史存量业务的库表数目过多（库表数目可能达百级或千级），库表的JOIN关系错综复杂。<br>    <br>- 业务原来的SQL查询复杂多样，不允许修改，对分布式数据库的SQL兼容性高。        | 单表打散                                                                                                                                                                                                             | - 最大限度地保持对原有从多存量业务库表及其 SQL的兼容性与查询性能。<br>    <br>- 众多单表被打散到不同的DN节点，突破单机资源瓶颈，实现负载均衡与性能提升。                                        |
| 混合存量业务与新业务的应用      | - 突破单机资源瓶颈（主要是CPU/IO/DISK瓶颈 ）。<br>    <br>- 业务SQL查询性能尽量不回退。<br>    <br>- 大表的磁盘空间扩展。 | - 历史存量业务的库表很多，库表JOIN关系错综复杂。<br>    <br>- 存量业务原有SQL查询复杂多样，大部分不允许修改。<br>    <br>- 部分新业务库表，尤其是大表，数据量增长迅速。 | 单表打散+手动分区<br><br>**说明**<br><br>设置单表打散后，需要使用`ALTER TABLE <table_name> PARTITION BY KEY(<column_name>) locality='';`语句完成手动分区操作。更多信息，请参见[Locality](https://help.aliyun.com/zh/polardb/polardb-for-xscale/locality)。 | - 最大限度地保持对原有从多存量业务库表及SQL 的兼容性与查询性能。<br>    <br>- 众多单表被打散到不同的DN节点，突破单机资源瓶颈，实现负载均衡与性能提升。<br>    <br>- 业务大表手动分区，在解决扩展性的同时，保证读写性能。 |
| 基于单机 MySQL开发的新业务应用 | - 需要扩展性。<br>    <br>- 业务性能要求不高。                                                     | - 业务要尽量减少改造成本，需要快速上线。                                                                                  | 自动分区                                                                                                                                                                                                             | - 所有表自动分区，能突破单机资源瓶颈。<br>    <br>- 所有索引默认全局索引，保证非主键维度查询的基本性能。                                                                   |
| 高性能高吞吐的业务应用        | - 需要线性扩展。<br>    <br>- 高性能。                                                         | - 业务并发量大（几万或几十万QPS），并要求线性扩展。<br>    <br>- 业务对性能敏感，SQL查询要求快且稳定。                                         | 手动分区                                                                                                                                                                                                             | - 所有表均按业务场景，手动选择最合理的分区方案。<br>    <br>- 业务查询SQL能改造，满足线性扩展性。                                                                     |
## 单表打散
单表打散是将多个表分散在不同的DN来提升系统性能，比如 在此模式下 table1、table2 会被自动分布到 DN1、DN2 不同的数据节点；
#### 使用方式
创建数据库并标记 DEFAULT_SINGLE='on'
```sql
CREATE DATABASE autodb1 MODE='auto' DEFAULT_SINGLE='on';
```
创建表
```sql
CREATE TABLE sin_t1(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30),
 birthday datetime,
 primary key(id)
);

CREATE TABLE sin_t2(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30),
 birthday datetime,
 primary key(id)
);

CREATE TABLE sin_t3(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30),
 birthday datetime,
 primary key(id)
);

CREATE TABLE sin_t4(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30),
 birthday datetime,
 primary key(id)
);
```
查看表分布情况
```sql
SHOW TOPOLOGY FROM sin_t2
```
```sql
              ID: 0
       GROUP_NAME: AUTODB1_P00001_GROUP
       TABLE_NAME: sin_t2_IT7l
   PARTITION_NAME: p1
SUBPARTITION_NAME: 
      PHY_DB_NAME: autodb1_p00001
            DN_ID: polardbx-storage-1-master
STORAGE_POOL_NAME: _default
```
可以看到表被分布到了不同的 DN_ID
#### 局限
- Join性能变化：单表打散后，部分表的JOIN执行性能可能会出现变化，这是因为原本同一个DN的多个单表的`JOIN`由于落到不同的DN，导致`JOIN`算子无法继续下推至DN执行。
- 分布不均匀：若逻辑表没显式指定MySQL的分区定义，作为单表被随机分配到PolarDB-X的不同DN节点时，由于每张表的数据量不同，会导致不同节点空间大小不一样。

## 自动分区
![[IMG-20250119161816176.png]]
AUTO库默认是手动分区，需手动设置以下开关才能使用自动分区：

```sql
SET GLOBAL AUTO_PARTITION=true;
```