# 广告数据查询SQL优化方案

  

## 问题分析

  

当前 `ad_data_rt_inc_view` 相关查询存在性能问题：

- 汇总查询耗时：1274ms

- 项目维度分组查询：耗时较长

- 主要瓶颈：多表JOIN + 大量聚合运算 + 缺少索引

  

## 优化方案

  

### 1. 数据库索引优化（优先级：最高）

  

```sql

-- 1. 主视图表索引（按过滤条件顺序建立联合索引）

ALTER TABLE dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

ADD INDEX idx_query_main (product, data_date, account_id, project_id);

  

-- 如果经常按项目ID查询，额外添加：

ALTER TABLE dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

ADD INDEX idx_project_query (project_id, data_date, product);

  

-- 2. 账户表索引

ALTER TABLE media_ad_account

ADD INDEX idx_account_business (account_id, business_id, admin_id);

  

-- 3. 权限表索引（非常重要，影响INNER JOIN性能）

ALTER TABLE media_ad_account_user

ADD INDEX idx_permission_check (admin_id, media_ad_account_id, permission);

  

-- 4. 用户表索引

ALTER TABLE user

ADD INDEX idx_user_id (id);

  

-- 5. 如果经常按推广ID查询，添加：

ALTER TABLE dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

ADD INDEX idx_promotion_query (promotion_id, data_date);

```

  

### 2. 查询改写优化

  

#### 2.1 使用 EXISTS 替代 INNER JOIN（已实现）

  

**优化前：**

```sql

INNER JOIN media_ad_account_user au

ON acc.account_id = v.account_id

AND au.media_ad_account_id = acc.id

AND au.admin_id = ?

AND (au.permission & 1) != 0

```

  

**优化后：**

```sql

LEFT JOIN media_ad_account acc ON v.account_id = acc.account_id

AND acc.business_id = ?

AND EXISTS (

SELECT 1 FROM media_ad_account_user au

WHERE au.media_ad_account_id = acc.id

AND au.admin_id = ?

AND (au.permission & 1) != 0

)

```

  

**优势：** EXISTS 子查询在找到第一条匹配记录后立即返回，避免了完整的JOIN操作。

  

#### 2.2 分离汇总查询和列表查询

  

对于分页列表查询，考虑分两步：

1. 先查询符合条件的ID列表（轻量级）

2. 再根据ID获取详细数据

  

```go

// 伪代码示例

// Step 1: 获取符合条件的project_id列表

projectIDs := getFilteredProjectIDs(query)

  

// Step 2: 根据project_id获取聚合数据

query.ProjectIDs = projectIDs

results := getAggregatedData(query)

```

  

### 3. 应用层优化

  

#### 3.1 添加查询缓存

  

对于汇总数据，可以添加Redis缓存：

  

```go

// 缓存key示例：ad_data_summary:{business_id}:{product}:{date_range}

cacheKey := fmt.Sprintf("ad_data_summary:%d:%s:%s-%s",

query.BusinessID, query.Product, query.StartDataDate, query.EndDataDate)

  

// 尝试从缓存获取

if cached, err := redis.Get(cacheKey); err == nil {

return cached, nil

}

  

// 查询数据库

result := repo.GetSummary(ctx, query)

  

// 写入缓存，设置5分钟过期

redis.Set(cacheKey, result, 5*time.Minute)

```

  

#### 3.2 异步查询优化

  

对于Count + List的分页查询，可以并发执行：

  

```go

var (

total int64

records []*entity.AdDataRtIncView

errG errgroup.Group

)

  

// 并发执行count和list查询

errG.Go(func() error {

var err error

total, err = r.GetCount(ctx, query)

return err

})

  

errG.Go(func() error {

var err error

records, err = r.GetList(ctx, query)

return err

})

  

if err := errG.Wait(); err != nil {

return nil, nil, err

}

```

  

#### 3.3 合理的分页限制

  

```go

const (

MaxPageSize = 100 // 最大分页数

DefaultPageSize = 20 // 默认分页数

)

  

// 限制查询数量

if query.Limit > MaxPageSize {

query.Limit = MaxPageSize

}

```

  

### 4. 数据库配置优化

  

```sql

-- 检查MySQL配置参数

SHOW VARIABLES LIKE 'innodb_buffer_pool_size'; -- 建议设置为物理内存的70-80%

SHOW VARIABLES LIKE 'tmp_table_size'; -- 临时表大小，建议 256M+

SHOW VARIABLES LIKE 'max_heap_table_size'; -- 内存表大小，建议 256M+

SHOW VARIABLES LIKE 'join_buffer_size'; -- JOIN缓冲区，建议 8M+

```

  

### 5. 慢查询分析

  

#### 5.1 使用 EXPLAIN 分析

  

```sql

EXPLAIN SELECT ... FROM dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view AS v ...

```

  

关注：

- `type`: 应该是 `ref`、`range` 或 `index`，避免 `ALL` 全表扫描

- `rows`: 扫描行数，越少越好

- `Extra`: 避免出现 `Using temporary` 和 `Using filesort`

  

#### 5.2 监控慢查询

  

```sql

-- 开启慢查询日志

SET GLOBAL slow_query_log = 'ON';

SET GLOBAL long_query_time = 1; -- 记录超过1秒的查询

  

-- 查看慢查询

SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;

```

  

### 6. 考虑物化视图或汇总表

  

如果查询仍然较慢，考虑：

  

#### 6.1 创建项目维度汇总表（定时刷新）

  

```sql

CREATE TABLE dm_mktp_project_data_summary (

project_id BIGINT,

data_date DATE,

product VARCHAR(50),

account_id BIGINT,

-- 预聚合的指标字段

ad_cost DECIMAL(20,2),

promotion_expenses DECIMAL(20,2),

show_cnt BIGINT,

click_cnt BIGINT,

-- ... 其他聚合字段

PRIMARY KEY (project_id, data_date, product),

INDEX idx_date_product (data_date, product)

) ENGINE=InnoDB;

  

-- 定时任务每天凌晨刷新

INSERT INTO dm_mktp_project_data_summary

SELECT

project_id,

data_date,

product,

MAX(account_id) as account_id,

SUM(ad_cost) as ad_cost,

-- ... 其他聚合

FROM dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

WHERE data_date = CURDATE() - INTERVAL 1 DAY

GROUP BY project_id, data_date, product;

```

  

#### 6.2 使用增量更新策略

  

对于实时数据，可以采用：

- T-1天及之前：查询汇总表（已固化）

- T日：查询明细表（实时聚合）

- 最后合并结果

  

### 7. 预期优化效果

  

实施上述优化后，预期：

- **索引优化**：查询时间减少 50-70%（从1274ms -> 400-600ms）

- **查询改写**：再减少 20-30%（-> 280-480ms）

- **缓存策略**：重复查询几乎为0延迟（<10ms）

- **汇总表**：项目维度查询减少 80%+（-> <200ms）

  

### 8. 实施顺序

  

1. **立即执行**：添加数据库索引（无风险，立竿见影）

2. **代码优化**：EXISTS替代INNER JOIN（已完成）

3. **监控评估**：观察1-2天，分析效果

4. **按需实施**：

- 如果仍不满意，添加缓存

- 如果查询量大，考虑汇总表

- 如果业务允许，适当增加分页限制

  

### 9. 监控指标

  

优化后持续监控：

```go

// 添加查询耗时日志

start := time.Now()

result, err := repo.GetList(ctx, query)

duration := time.Since(start)

  

if duration > 500*time.Millisecond {

log.Warn("慢查询告警",

"duration", duration,

"query", query,

)

}

```

  

## 附录：索引创建脚本

  

```bash

# 执行索引创建

mysql -u your_user -p your_database < create_indexes.sql

```

  

**create_indexes.sql:**

```sql

-- 开始事务

START TRANSACTION;

  

-- 1. 主表索引

ALTER TABLE dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

ADD INDEX idx_query_main (product, data_date, account_id, project_id);

  

ALTER TABLE dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view

ADD INDEX idx_project_query (project_id, data_date, product);

  

-- 2. 账户表索引

ALTER TABLE media_ad_account

ADD INDEX idx_account_business (account_id, business_id, admin_id);

  

-- 3. 权限表索引

ALTER TABLE media_ad_account_user

ADD INDEX idx_permission_check (admin_id, media_ad_account_id, permission);

  

-- 4. 用户表索引（如果不存在主键）

ALTER TABLE user ADD INDEX idx_user_id (id);

  

-- 提交事务

COMMIT;

  

-- 查看索引创建结果

SHOW INDEX FROM dm_mktp_newmedia_promotion_all_ad_id_data_rt_inc_d_view;

SHOW INDEX FROM media_ad_account;

SHOW INDEX FROM media_ad_account_user;

```