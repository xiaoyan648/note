# 一、结论

ZSTD压缩算法，强力推荐：**低成本**接入，**高****ROI**见效！

在 2.* 版本之后的 Kafka 实例 ，支持 ZSTD 压缩格式，它的压缩比、压缩效率、CPU 占用表现都是比较优秀的。所以我们强烈建议您使用 **ZSTD** 格式的压缩。

# 二、压缩算法分析

数据压缩可以减少网络 IO 传输量，减少磁盘存储空间。本文档为您介绍推荐的压缩算法及具体实践。

目前 Kafka 支持四种压缩算法： **ZSTD**、**GZIP**、**Snappy**、**LZ4**。

评估一个压缩算法的优劣，主要有两个指标：压缩比、压缩/解压缩吞吐量。在 Kafka 的实际使用中，四种算法的性能指标对比如下：

- 压缩比：ZSTD > LZ4 > GZIP > Snappy
    
- 吞吐量：ZSTD > LZ4 > Snappy > GZIP
    

因此，正常情况下四种压缩算法的推荐排序为： **ZSTD > LZ4 > GZIP > Snappy**。 经过长时间的运行观察，以上压缩算法推荐顺序适用于大部分场景。但是因为业务的数据内容不一样，会导致压缩算法的性能表现不一样。 故建议对 CPU 指标敏感的用户优先采用 ZSTD，然后根据实际表现调整压缩算法。

# 三、商业化实践

1. ## 成效
    

通过将大部分**Kafka**生产者压缩算法从**LZ4**切换至**ZSTD**（级别3），在**不损失吞吐性能**的前提下，成功降低**25%**的存储开销，每月节省**9000元**左右云成本。

2. ## 背景
    

许多业务数据通过kafka传递，避免出现问题时，数据找不到，需要将业务数据存在磁盘中（1-3天）。

3. ## 前置条件
    

- **librdkafka** 的版本 >= `v1.4.0`（Zstd 需要此版本及以上）
    
- 自 broker版本 Kafka 2.1.0 起，官方在 `kafka-clients` 库中内置了 纯 Java 实现的 Zstandard（Zstd）压缩器（通过 [Apache Commons Compress](https://commons.apache.org/proper/commons-compress/) 实现），否则需要在系统中安装`zstd`库
    
- 生产者配置（正常情况下唯一需要修改的配置）
    

```JSON
{
    "compression.type":              "zstd",   // 启用 Zstd 压缩
    // "compression.level":             "3",      // 可选：默认 3，范围 1-22
}
```

4. ## 验证阶段
    

5. ### 使用项目：`baize_adx_drs`
    

6. ### topic：`bz_adx_drs_report_pre`
    

7. ### 消息内容（含嵌套元数据与埋点信息）：
    

```json
{
    "environment": {
        "app_id": "92",
        "app_version": "7.72.70dev",
        "version_code": "77270",
        "os": "android",
        "os_version": "12",
        "ad_sdk_version": "1.0.0.7-********SHOT",
        "ad_sdk_code": "1000007",
        "package_name": "com.kmxs.reader",
        "ipv4": "180.169.86.54"
    },
    "identity": {
        "ad_source_uid": 1888174818053468160,
        "android_id": "029b509ebc4d4531",
        "uid": "56316E6BECF64A38B3******************4449513b96ecb678ee8",
        "oaid_no_cache": "56316E6BECF64A38B3******************e38ea84449513b96ecb678ee8",
        "oaid": "56316E6BECF64A38B3C20058CC******************ea84449513b96ecb678ee8",
        "country": "中国",
        "province": "上海",
        "city": "上海"
    },
    "common_params": {
        "request_id": "52C52E0A-******************256771E8D",
        "ad_unit_id": "3000115",
        "ad_format": "2",
        "flow_id": "93",
        "flow_group_id": "11111**********11111",
        "policy_id": "28,30,34"
    },
    "aggs": [
        {
            "event_id": "adex_#_#_adexpose",
            "params": {
                "partner_code": "79",
                "tag_id": "Strea*************om_applet",
                "cooperation_mode": "4",
                "create_id": "8f4bec*******2cbc",
                "price": "500",
                "settlement_price": "500",
                "bid_price": "500",
                "format_id": "2",
                "replace_package_name": "1234567"
            }
        }
    ],
    "drs_ts": 1747633618
}
```

4. ### 存储占用测试及基准测试
    

|   |   |   |   |   |
|---|---|---|---|---|
|压缩算法|压缩级别|1万条数据体积|吞吐量 (ns/op)|资源消耗趋势|
|LZ4|默认|8.8 MB|44,114,420|基准|
|ZSTD|3|6.6 MB|44,071,436|≈LZ4|
|ZSTD|6|6.6 MB|59,308,510|+35% CPU|

**数据解读**：

- **存储优化**：ZSTD（级别3）较LZ4减少**25%**存储占用（8.8MB → 6.6MB）。
    
- **性能表现**：压缩级别3时性能与LZ4持平，级别6时吞吐量下降但压缩率未提升，建议优先使用默认级别。
    

5. ### 消费验证
    

略

5. ## 上线阶段
    

本次上线不需要修改代码，仅需修改nacos中的配置即可！

我们首先上线了一些影响面较小的服务，并观察了数天，最终在6月19号完成了核心服务的上线；

#### 生产流量变化：

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDViOTAyZmEzZjkwOTM5NzBiM2UxNTUzZDAxYThhOTRfZk03NlVyT1NJWWozNnBkZ21TRkZMM0lDc00yVmJodlJfVG9rZW46Q3ZMWWJFdnhPb3hFM3N4M0dJaWM2Nlk3bjB0XzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2Q5M2Y1ZjhhY2YxMDMyYTQ5MjE5NWRkNzYxODVjOTdfc1EzQ2VnS0pJNmZ3bHZsR1R1WmlNc1Myd29uUkJ1M0RfVG9rZW46V3JyV2JFRDhCb1NEYTV4WTNYNWNiTE1ZbjJmXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=MmRkOTgxNTE0ZmU0YTc4NDUzYmI4OGE2ZDhkODQwZTNfbjVHYW9MUng5a0lBb3ZyQ0QyZ09pZUY2Sk51SzZmYmFfVG9rZW46VDlBeWJxajJ4b3BmdU14anRZN2NaUnZ1blhiXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

#### 磁盘占用变化(实际需按月考虑)：

1. 实例1，降低35%
    

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=ODU4ZmUzNDBkYzMwZWVhZDg4NWE1YTRlNzJhNWM4ZjlfSmZGMHVwMDZ2eXA3QW1mSjlNbWNDVXRNdU9YQ29SQWJfVG9rZW46VmVHamJoNDNWb1BEbzN4Q3k4MmNoTmt0bkxnXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=YzZiOWY4OGMyOWRhODU0ZTg1ZDI3NzRiNDY2MzJlZjJfNkVidTJqQWZKdzRCb244RDh3MlpldmZhcGZpR3V5ZE9fVG9rZW46SFZyY2JGNjE0b0tqTmt4Z3AxamNOTmhxbnRoXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

2. 实例2，降低37%
    

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=MTdiNDI5ZTk2NjMyZjZjOGM2MzAzNDQwZDg3ZjYzMjJfM09CQ3lOMVYwekh0YmpvTFhKOVJ5ZVNuUDNQQkFhMDdfVG9rZW46RDJ5c2JqNGRFb2J0Q3N4dFJCQmM5bXZKbmJkXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)

![](https://x0sgcptncj.feishu.cn/space/api/box/stream/download/asynccode/?code=NjM1ZThjY2FhYWQ3NzYzZGE4MWZhYmEwMWU0MjFlMDFfalJPMkRQdUs5djZpajJkRnR2M2xDTnRxVjViYnFkbnVfVG9rZW46TFY5d2JNV1NYb2JoWGN4Z3ZGNGNibUwwbkFiXzE3NTA4Mjg4MDg6MTc1MDgzMjQwOF9WNA)