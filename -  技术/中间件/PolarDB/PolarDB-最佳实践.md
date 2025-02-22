## 场景分析
| 场景                 | 目标                                                                                  | 挑战                                                                                                     | **推荐工作模式**                                                                                                                                                                                                       | 效果                                                                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 大量存量老业务的应用         | - 突破单机资源瓶颈（主要是CPU/IO）。<br>    <br>- 优化SQL查询RT。                                      | - 历史存量业务的库表数目过多（库表数目可能达百级或千级），库表的JOIN关系错综复杂。<br>    <br>- 业务原来的SQL查询复杂多样，不允许修改，对分布式数据库的SQL兼容性高。        | 单表打散                                                                                                                                                                                                             | - 最大限度地保持对原有从多存量业务库表及其 SQL的兼容性与查询性能。<br>    <br>- 众多单表被打散到不同的DN节点，突破单机资源瓶颈，实现负载均衡与性能提升。                                        |
| 混合存量业务与新业务的应用      | - 突破单机资源瓶颈（主要是CPU/IO/DISK瓶颈 ）。<br>    <br>- 业务SQL查询性能尽量不回退。<br>    <br>- 大表的磁盘空间扩展。 | - 历史存量业务的库表很多，库表JOIN关系错综复杂。<br>    <br>- 存量业务原有SQL查询复杂多样，大部分不允许修改。<br>    <br>- 部分新业务库表，尤其是大表，数据量增长迅速。 | 单表打散+手动分区<br><br>**说明**<br><br>设置单表打散后，需要使用`ALTER TABLE <table_name> PARTITION BY KEY(<column_name>) locality='';`语句完成手动分区操作。更多信息，请参见[Locality](https://help.aliyun.com/zh/polardb/polardb-for-xscale/locality)。 | - 最大限度地保持对原有从多存量业务库表及SQL 的兼容性与查询性能。<br>    <br>- 众多单表被打散到不同的DN节点，突破单机资源瓶颈，实现负载均衡与性能提升。<br>    <br>- 业务大表手动分区，在解决扩展性的同时，保证读写性能。 |
| 基于单机 MySQL开发的新业务应用 | - 需要扩展性。<br>    <br>- 业务性能要求不高。                                                     | - 业务要尽量减少改造成本，需要快速上线。                                                                                  | 自动分区                                                                                                                                                                                                             | - 所有表自动分区，能突破单机资源瓶颈。<br>    <br>- 所有索引默认全局索引，保证非主键维度查询的基本性能。                                                                   |
| 高性能高吞吐的业务应用        | - 需要线性扩展。<br>    <br>- 高性能。                                                         | - 业务并发量大（几万或几十万QPS），并要求线性扩展。<br>    <br>- 业务对性能敏感，SQL查询要求快且稳定。                                         | 手动分区                                                                                                                                                                                                             | - 所有表均按业务场景，手动选择最合理的分区方案。<br>    <br>- 业务查询SQL能改造，满足线性扩展性。                                                                     |
## 单表打散
单表打散是将多个表分散在不同的DN来提升系统性能，比如 在此模式下 table1、table2 会被自动分布到 DN1、DN2 不同的数据节点；
![[IMG-20250120161518180.png]]
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
自动分区的工作模工下，建表SQL在不指定分区键的情况下，PolarDB-X会按主键（如果表没有指定主键，则使用隐式主键）并使用**KEY分区**策略进行默认分区，默认分区的分区数目 = 实例创建时逻辑节点数 × 8。比如，PolarDB-X实例创建时，逻辑节点是2，那默认分区数目就是16。
![[IMG-20250119161816176.png]]
AUTO库默认是手动分区，需手动设置以下开关才能使用自动分区：

```sql
SET GLOBAL AUTO_PARTITION=true;
```

创建表后，会自动加上分区语句
```sql
mysql> CREATE TABLE auto_t1(
    ->  id bigint not null auto_increment, 
    ->  bid int, 
    ->  name varchar(30),
    ->  birthday datetime,
    ->  primary key(id),
    ->  index idx_name(name)
    -> );

Query OK, 0 rows affected (2.44 sec)

mysql> 
mysql> show full create table auto_t1\G
*************************** 1. row ***************************
       Table: auto_t1
Create Table: CREATE PARTITION TABLE `auto_t1` (
	`id` bigint(20) NOT NULL AUTO_INCREMENT,
	`bid` int(11) DEFAULT NULL,
	`name` varchar(30) DEFAULT NULL,
	`birthday` datetime DEFAULT NULL,
	PRIMARY KEY (`id`),
	GLOBAL INDEX /* idx_name_$6425 */ `idx_name` (`name`) 
		PARTITION BY KEY(`name`,`id`)
		PARTITIONS 3,
	LOCAL KEY `_local_idx_name` (`name`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4
PARTITION BY KEY(`id`)
PARTITIONS 3
/* tablegroup = `tg3741` */
1 row in set (0.03 sec)
```

## 手动分区
PolarDB-X手动分区适合对业务性能有要求的应用，尤其是高并发高吞吐的核心应用。 因此，手动分区要求使用者需要对分布式数据库原理有所了解。但是，手动分区可以让用户选择最适合应用场景的维度进行水平切分，因此，手动分区最能发挥出分布式数据库的扩展性与高性能。

同样需要 Auto mode
```sql
CREATE DATABASE autodb1 MODE='auto'
```

### 表类型
PolarDB-X中的手动分区，允许用户手动创建三种不同类型的逻辑表，它们分别是：**单表、广播表与分区表。**

| **逻辑表类型** | **物理表拓扑**                        | **适用场景**               | **读写负载分析**                                        |
| --------- | -------------------------------- | ---------------------- | ------------------------------------------------- |
| 单表        | 一个单表对应一张物理表。                     | 数据量较小、并发访问的小表。         | 读写集中在一个DN节点                                       |
| 广播表       | 广播表在每个DN节点都有一个镜像，它们之间的数据总是完全一样。  | 适合于读多写少、数据量不大的表，比如配置表。 | 读均衡：可以随机地分摊到不同DN节点<br><br>写放大：需要同时写所有 DN的镜像，保持一致。 |
| 分区表       | 分区表有多个分区并分布到多个DN节点，且每个分区对应一个物理表。 | 适用于高并发高吞吐的数据大的表。       | 读写压力能按分区列条件自动路由到不同DN节点，实现负载均衡。                    |
#### 创建单表
```sql
CREATE TABLE xxx (...)
SINGLE
```
#### 创建广播表
```sql
CREATE TABLE xxx (...)
BROADCAST
```
#### 创建分区表
以订单表为例, 将订单表按照 trade_no 订单号进行分区（大部分请求都是通过订单号查询），同时满足用户和分销商查询订单，建立了2个 GSI（全局二级索引）并覆盖了常用字段、排序字段减少回主表次数；
```sql
CREATE TABLE `pay_order` (
	`id` bigint UNSIGNED NOT NULL AUTO_INCREMENT,
	`uid` bigint UNSIGNED NOT NULL COMMENT '用户ID',
	`open_id` varchar(64) NOT NULL DEFAULT '' COMMENT '用户OpenID',
	`app_id` varchar(64) NOT NULL COMMENT '小程序id',
	`project` varchar(64) DEFAULT NULL COMMENT '项目 pay_play_dyminiapp；pay_play_wxminiapp',
	`platform` varchar(32) NOT NULL DEFAULT '' COMMENT '平台：ios, android',
	`trade_no` varchar(64) NOT NULL COMMENT '订单编号, 年月日时分秒+6位随机数',
	`mid` varchar(16) NOT NULL COMMENT '交易中心的mid: PMMPVIP->会员;PMMPCB->金币',
	`pay_type` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '支付方式:  wxpay->微信; douyin->抖音',
	`goods_id` tinyint UNSIGNED NOT NULL COMMENT '商品id',
	`goods_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '商品名',
	`goods_type` tinyint UNSIGNED NOT NULL COMMENT '产品类型: 1->vip订单; 2->金币订单；3-解锁全集',
	`pay_amount` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '订单金额, 单位为分',
	`refund_amount` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '退款金额, 单位为分',
	`refund_time` datetime NOT NULL COMMENT '退款时间, 完成时间',
	`status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '订单状态: 1->待支付; 2->支付成功 3->支付失败;',
	`remark` varchar(64) NOT NULL COMMENT '备注',
	`app_version` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '小程序版本',
	`playlet_id` int UNSIGNED DEFAULT NULL COMMENT '短剧id',
	`link_id` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '推广链接id',
	`distributor_id` int UNSIGNED DEFAULT NULL COMMENT '分销商id',
	`promoter_id` int UNSIGNED DEFAULT NULL COMMENT '分销商id',
	`order_time` datetime DEFAULT NULL COMMENT '下单时间',
	`pay_time` datetime DEFAULT NULL COMMENT '支付时间, 完成时间',
	`pay_date` date DEFAULT NULL COMMENT '日期',
	`create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
	`update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
	`extra` json NOT NULL COMMENT '额外信息',
	PRIMARY KEY (`id`),
	GLOBAL INDEX `g_i_distributor_id` (`distributor_id`) COVERING (`trade_no`, `status`, `promoter_id`, `order_time`, `pay_time`) 
		PARTITION BY KEY(`distributor_id`)
		PARTITIONS 256,
	GLOBAL INDEX `g_i_uid` (`uid`) COVERING (`trade_no`, `pay_amount`, `status`, `order_time`, `pay_time`) 
		PARTITION BY KEY(`uid`)
		PARTITIONS 256,
	UNIQUE KEY `trade_no` (`trade_no`),
	KEY `uid` (`uid`, `status`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 DEFAULT COLLATE = utf8mb4_0900_ai_ci AUTO_INCREMENT = 296 COMMENT '订单表'
PARTITION BY KEY(`trade_no`)
PARTITIONS 256
```
更加详细请参考 https://help.aliyun.com/zh/polardb/polardb-for-xscale/partition-table-1/?spm=a2c4g.11186623.help-menu-2249963.d_4_10.2fab3ddddcDBgJ&scm=20140722.H_2771551._.OR_help-T_cn~zh-V_1

### 核心建表语句
PolarDB 分区表建表语句和普通建表语句的差异主要在两个部分
#### [分区函数](https://help.aliyun.com/zh/polardb/polardb-for-xscale/partition-table-type-and-usage-description/?spm=a2c4g.11186623.help-menu-2249963.d_4_10_1.27d67188TRnZtS&scm=20140722.H_2773431._.OR_help-T_cn~zh-V_1)
可以通过 HASH、KEY、RANGE、LIST、CO_HASH 等多种分区策略将数据打散到不同数据节点.
#### [GSI 全局二级索引](https://help.aliyun.com/zh/polardb/polardb-for-xscale/global-secondary-index-gsi?spm=a2c4g.11186623.help-menu-2249963.d_4_10_4.326f8c5eF1itLN)
通过全局二级索引，用户能够按需增加分区维度、提供全局唯一约束等
![[Pasted image 20250122211446.png]]


##### 全局二级索引（Global Secondary Index 简称GSI）

全局二级索引可以提供和主表不同的分区方式，当查询SQL的条件中未包含主表的分区键但包含了GSI的分区键时，仍可以避免全分区扫描。

比如对于下面的用户表user_tbl， 如果既希望按照user_id查询，又希望按照用户名name查询，就可以建立全局二级索引 g_i_name，在按照用户名name查询的时候避免全分区扫描。

```sql
CREATE TABLE user(
 user_id bigint,
 name varchar(10),
 addr varchar(30),
 GLOBAL INDEX `g_i_name` (name) PARTITION BY HASH(name),
 PRIMARY KEY(user_id)
) PARTITION BY KEY(user_id);
```

##### 全局唯一索引（Unique Global Secondary Index 简称UGSI）

全局唯一索引是特殊的GSI，它不仅有普通GSI的性质，还能实现全局唯一约束。

比如对于下面的用户表user2，如果要求用户手机号全局唯一，那么可以建立一个phone字段为索引键的UGSI。

```sql
CREATE TABLE user2(
 user_id bigint,
 phone varchar(20),
 addr varchar(30),
 UNIQUE GLOBAL INDEX `g_i_phone`(phone)  PARTITION BY HASH(phone), 
 PRIMARY KEY(user_id)
) PARTITION BY KEY(user_id);
```

##### 全局聚簇索引 （Clustered Global Secondary Index 简称Clustered GSI）

全局聚簇索引是特殊的GSI，它默认冗余了主表的全部列（该索引所占磁盘空间等于主表所占磁盘空间）。如果既希望避免全分区扫描，又希望避免回表开销，可以使用全局聚簇索引。

比如对于订单表order_tbl，希望支持按照user_id或order_id来查询，且希望避免用user_id查询订单时回表，就可以创建一个以user_id为索引键的全局聚簇索引cg_i_user。以user_id为条件查询订单信息时，PolarDB-X会将查询路由到cg_i_user上的一个特定分区，又因为cg_i_user上有主表的所有数据，因此无需回表。

```sql
CREATE TABLE order_tbl(
 order_id bigint,
 user_id bigint,
 addr varchar(30),
 info text,
 create_time datetime,
 CLUSTERED INDEX `cg_i_user`(user_id) PARTITION BY HASH(user_id), 
 PRIMARY KEY(order_id)
) PARTITION BY KEY(order_id);
```

## 列索引 CCI

## 表组
在PolarDB-X中，为加速SQL的执行效率，当分区表间通过各自的分区键进行等值关联的JOIN操作时，优化器会倾向于将这些操作转化为Partition-Wise Join来实现计算下推。这种方法可以显著提升查询的执行效率。然而，在表A发生诸如分区分裂、迁移等变更后，其分区规则与表B不再对齐，原本A表和B表支持下推的JOIN查询条件将会不复存在，由此导致计算下推变得无效，并对业务的执行效率造成不利影响。
为了保障表之间的JOIN操作能够稳定地实现计算下推，在PolarDB-X中引入了表组的概念。
同一个表组的表分区策略必须一致，相同的分片就会落在同一个DN：
![[Pasted image 20250120191652.png]]

[操作文档](https://help.aliyun.com/zh/polardb/polardb-for-xscale/table-group/?spm=a2c4g.11186623.help-menu-2249963.d_5_1_8.47615587FNw6Qs)

## 分区情况
### 分区情况查询
若需要简单查询分区表的整体拓扑以及各个分区的物理位置（物理库表所在的DN），也可以采用`SHOW TOPOLOGY FROM #table_name`命令快速查看（如下所示）：

### 分区详细情况查询
PolarDB-X兼容MySQL的INFORMATION_SCHEMA.PARTITIONS的视图查询，支持通过PARTITIONS视图查询各个一级分区及其二级分区的相关元信息，例如：
```sql
select * from information_schema.partitions where table_schema='autodb2' and table_name='test_tbl_part_name2' order by partition_name, subpartition_name;
```

### 查看数据分布
查看不同分区的插入、查询情况
```sql
select table_name, partition_name, subpartition_name, percent, 
     rows_read, rows_inserted, rows_updated, rows_deleted from information_schema.table_detail  
     where table_schema='autodb2' and table_name='test_tbl_part_name2' 
     order by partition_name, subpartition_name;
```
![[Pasted image 20250120200158.png]]

## 性能优化
### 索引优化
**强制指定分区**，规则如下：
```sql
SELECT/UPDATE/DELETE ...  tbl_name [PARTITION ( part_name[, part_name, ...] )]
```
案例如下：
```sql
SELECT * FROM tb_k PARTITION( p1,p2 );
```


**索引分析**
和常规的sql分析一样，我们可以通过 `explain` 关键字分析 sql 语句执行情况
比如一个二级索引的请求流程如下：
![[Pasted image 20250121102027.png]]

  可以看出先通过 `g_i_uid`索引筛选后，最终是回主表获取信息，可以考虑在 `g_i_uid` 中覆盖更堵字段减少回主表次数。

**索引诊断**
通过如下语句可以查询索引存在的问题和优化建议，
```sql
-- 索引静态诊断
INSPECT FULL INDEX FROM pay_order

-- 索引动态诊断
-- DYNAMIC模式的诊断需要依据索引的使用数据，建议收集一个完整的业务流量周期的数据（如1天、1周，这取决于您的业务特点）后再进行诊断。
set global GSI_STATISTICS_COLLECTION=true
INSPECT FULL INDEX FROM pay_order mode=dynamic
```

**索引使用情况**
```sql
select * from information_schema.global_indexes
select * from information_schema.global_indexes where `table` = 'pay_order'
```

返回信息说明：
- SIZE_IN_MB：索引占用的空间。
- USE_COUNT：自您开启GSI_STATISTICS_COLLECTION后，索引被使用的次数。
- LAST_ACCESS_TIME：自您开启GSI_STATISTICS_COLLECTION后，索引最后一次被使用的时间。
- CARDINALITY：索引的基数。
- ROW_COUNT：索引的行数。



### 如何解决数据热点
以订单表举例，当我们按照 卖家id 分区，如果出现大卖家就会出现数据倾斜的情况，如下：
![[Pasted image 20250120202150.png]]
在 PolarDB 中如何解决呢？
我们可以通过如下两步将热点数据打散
1. 首先将 `partition by key(seller_id) ` 改为 `partition by key(seller_id，id) ` ，此时 `id` 列实际上并没有参与具体的路由计算，仅仅是一个分区键的"占位符"
```sql
alter table orders partition by key(seller_id,id) partitions 5
```
2. 通过如下 split 将热点数据打散，下面语句的含义是将 `seller_id = 88` 的记录通过第二个分区键 `id` 进行再次分区，分区数为 2
```sql
alter table orders split into H88_ partitions 2 by hot value(88)
```
打散后的效果如下：
分裂前后非热点数据（seller_id不等于88）并没有发生变化，原来在P1的还在P1，在P2的还在P2，仅仅是影响到了热点数据的，分裂前seller_id=88会自动路由到P5分区，分裂后路由到H88_1和H88_2。
![[Pasted image 20250120203438.png]]
## 读写分离
在Mysql中，读写分离代理会伪造成Mysql和应用程序建立好链接，解析发送进来的每一条sql,如果是
update、delete、insert、create等写操作则直接发往主库，如果是select则发送到备库。
![[Pasted image 20250121103825.png]]
带来的问题：延迟导致的查询不一致，使用mysqlE时不可避免地会遇到备库select查询数据不一致的现象（因为主备有延迟)。负载低的时候延迟可以控制在秒级别，但当负载很高时，尤其是对大表做DDL

在 POLARDB 的链路中间层做读写分离的同时，中间层会 track 各个节点已经 apply 了的 redolog 位点即 LSN,同时每次更新时会记录此次更新的位点为 Session LSN,当有新请求到来时我们会比较 Session LSN 和当前各个节点的 LSN ,仅将请求发往 LSN>=Session LSN 的节点，从而保证了会话一致性；表面上看该方案可能导致主库压力大，但是因为 POLARDB 是物理复制，速度极快，在上述场景中，当更新完成后，返回客户端结果时复制就同步在进行，而当下一个读请求到来时主从极有可能已经完成，然后大多数应用场景都是读多写少，所以经验证在该机制下即保证了会话一致性，也保证了读写分离负载均衡的效果
![[Pasted image 20250121104042.png]]