
PolarDB-X支持集中式和分布式一体化形态，具备金融级数据高可用、分布式水平扩展、HTAP混合负载、低成本存储和极致弹性等能力。

PolarDB-X坚定以兼容MySQL开源生态，从SQL语法、事务行为、生态工具等多个维度都做了深度兼容，应用无需或者修改少量代码即可从MySQL迁移到PolarDB-X。
![[IMG-20250119145958711.png]]

## 核心特性
### 金融级高可用
PolarDB-X采用数据多副本架构，为了保证副本间的强一致性（RPO=0），采用Paxos的多数派复制协议，每次写入都要获得超过半数节点的确认，即便其中1个节点宕机，集群也仍然能正常提供服务。

Paxos算法能够保证副本间的强一致性，彻底解决副本不一致问题，满足金融行业的容灾要求。

[参考文档](https://help.aliyun.com/zh/polardb/polardb-for-xscale/high-availability-rpo/?spm=a2c4g.11186623.0.0.2a0964ca1zitbL)

### 透明分布式
PolarDB-X聚焦分布式的线性扩展能力，给用户提供类单机MySQL数据库使用体验。 透明分布式，需要基于分布式事务一致性、数据分布和调度、全局二级索引、跨分片查询等技术，减少用户对于分布式的使用门槛。

面向分布式的运维，通过全局Binlog和全局一致性备份，分别解决分布式数据库各节点向下游流转难题以及各节点备份时间差造成的恢复一致性问题。

**数据分布**
- 通过分区表的分区规则:将数据均匀分布到多个节点。|[分区表概述](https://help.aliyun.com/zh/polardb/polardb-for-xscale/overview-partition-table)|
- 全局二级索引: 跨分区的索引，提供分布式下多维度的查询。|[全局二级索引（GSI）](https://help.aliyun.com/zh/polardb/polardb-for-xscale/global-secondary-index-gsi)|
- 手动分区/自动分区: 提供更灵活的数据分区策略，满足存量业务一键迁移。|[透明分布式概述](https://help.aliyun.com/zh/polardb/polardb-for-xscale/overview-transparent-distributed-database)|
**运维兼容**
- 全局binlog: 兼容MySQL的binlog协议，提供CDC增量日志订阅。|[日志服务概述](https://help.aliyun.com/zh/polardb/polardb-for-xscale/overview-binlog)|
- 分布式备份恢复: 支持分布式一致性的备份恢复。|[备份与恢复](https://help.aliyun.com/zh/polardb/polardb-for-xscale/backup-and-restoration-3/)|

### 集中式和分布式一体化
PolarDB-X全面构建集中式和分布式一体化的架构能力（简称“集分一体”），提供标准版（集中式架构）和企业版（分布式架构），支持从标准版原地升级为企业版。

### HTAP一体化
PolarDB-X提供列存索引的形态（Clustered Columnar Index，CCI），行存表默认有主键索引和二级索引，列存索引是一份额外基于列式结构的二级索引（覆盖行存所有列），一张表可以同时具备行存和列存的数据。同时，全面构建面向行列混合场景的代价优化器、以及向量化执行算子，通过一套SQL引擎支持行列混合查询。

|   |   |   |   |
|---|---|---|---|
|**列式数据**|列存索引|行存表默认有主键索引和二级索引，列存索引是一份额外基于列式结构的二级索引。|[列存索引概述](https://help.aliyun.com/zh/polardb/polardb-for-xscale/overview)|
|冷数据归档|基于时间分区滚动，将历史数据定期归档到OSS对象存储上。|[冷数据归档概述](https://help.aliyun.com/zh/polardb/polardb-for-xscale/overview-cold-data)|
|**只读实例**|列存只读实例|列存只读实例，提供链路隔离的能力，基于向量化引擎提供分析查询。|[列存只读实例](https://help.aliyun.com/zh/polardb/polardb-for-xscale/add-a-column-store-read-only-instance)|

### 开源与多云

PolarDB-X在2021年11月份正式全内核开源，通过定期同步商业版本到开源版本（大约3~6个月），从而持续保持开源版本的迭代和功能对齐。

在开源生态中，PolarDB-X提供了配套的轻量化管控、生态工具的适配，可以基于开源完成业务的生产部署，满足开源自建、多云容灾等自主可控的需求。

|   |   |   |   |
|---|---|---|---|
|**开源**|全内核开源|包含多个分布式组件，CN、DN、CDC、GMS、Columnar等。|[PolarDB-X开源项目](https://github.com/polardb/polardbx-sql)|
|轻量化管控|基于K8s Operator提供生产部署的能力。|[PolarDB-X Operator](https://github.com/polardb/polardbx-operator)|
|**多云**|ECS自建|基于IDC物理机或者云ECS，部署PolarDB-X开源版。|[快速部署](https://openpolardb.com/document?type=PolarDB-X)|
|MyBase多云|MyBase基于ECS托管PolarDB-X开源，支持多云ECS统一管理。||