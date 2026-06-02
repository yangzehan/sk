---
title: ClickHouse TOO_MANY_PARTS 故障排查
type: concept
created: 2026-05-25
tags:
  - ClickHouse
  - troubleshooting
  - partition
related:
  - [[Codex Schema Index]]
source: portrait_2 集群 (6 节点，3 分片 x 2 副本，22.2.2.1)
---

# ClickHouse TOO_MANY_PARTS 故障排查

## 错误信息

```
Code: 252. DB::Exception: Too many partitions for single INSERT block (more than 100).
```
每 30 秒出现一次，属于持续活跃的写入失败。

## 问题表

- 表：`portrait_2.portrait_tag_0_backup_local`
- 分区键：`(base_day, tag_code)` 复合字段
- 表引擎：分布式表，本地表 + ReplicatedMergeTree

## 根因分析

分区键为双字段组合 `(base_day, tag_code)`。每次 INSERT 的 VALUES 批量中，这两个字段的笛卡尔积即为本次 INSERT 的分区数。

**示例**：一次性写入 5 天数据，`tag_code` 有 30 种组合，则单次 INSERT 涉及 `5 × 30 = 150` 个分区，超过默认值 100，触发错误。

**结论**：应用层写入逻辑以较宽的分区粒度批量写入，单次 INSERT 跨分区数超过 ClickHouse 安全阈值。

## 排查步骤

### 1. 查询 ClickHouse 错误日志

```sql
SELECT event_time, query, exception_code, exception
FROM system.query_log
WHERE event_time > now() - INTERVAL 60 MINUTE
  AND type = 'ExceptionWhileProcessing'
  AND query LIKE '%portrait_tag_0_backup_local%'
ORDER BY event_time DESC
LIMIT 50
```

### 2. 查看表结构

```sql
SHOW CREATE TABLE portrait_2.portrait_tag_0_backup_local
```

### 3. 查看当前分区数量

```sql
SELECT partition, count() AS parts, sum(rows) AS rows
FROM system.parts
WHERE database = 'portrait_2' AND table = 'portrait_tag_0_backup_local' AND active
GROUP BY partition ORDER BY partition DESC
```

## 解决方案

### 方案一：扩大 max_partitions_per_insert_block（快速止血）

在 6 台 ClickHouse 节点创建配置覆盖：

```xml
<clickhouse>
    <profiles>
        <default>
            <max_partitions_per_insert_block>1000</max_partitions_per_insert_block>
        </default>
    </profiles>
</clickhouse>
```

配置路径：`/etc/clickhouse-server/users.d/max_partitions.xml`
> 注意：`users.d/` 配置修改后需要重启 ClickHouse 才能生效

批量执行：
```bash
for h in 15.28.140.41 15.28.140.42 15.28.140.43 15.28.140.44 15.28.140.45 15.28.140.46; do
  ssh "$h" "mkdir -p /etc/clickhouse-server/users.d && cat > /etc/clickhouse-server/users.d/max_partitions.xml <<'EOF'
<clickhouse>
    <profiles>
        <default>
            <max_partitions_per_insert_block>1000</max_partitions_per_insert_block>
        </default>
    </profiles>
</clickhouse>
EOF
systemctl restart clickhouse-server"
done
```

### 方案二：优化应用写入逻辑（治本）

将大批量跨分区 INSERT 改为小批次写入，避免单次 INSERT 跨过多分区。

### 方案三：修改分区键（彻底解决）

考虑改为单字段 `base_day` 按月/周分区，减少分区总数。需要重建表并迁移历史数据。

## 关键参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `max_partitions_per_insert_block` | 100 | 单次 INSERT 块中允许的最大分区数。0 表示无限制。值过大会影响写入性能和后台 merge 效率。 |

> ClickHouse 官方建议：表的合理总分区的数量在 **1000-10000** 之间。设置过高的分区上限是安全阈值被提高，不等于分区数量合理。

## 验证命令

```sql
-- 检查配置是否生效
SELECT name, value, changed FROM system.settings
WHERE name = 'max_partitions_per_insert_block'
-- 预期：value=1000, changed=1

-- 检查是否还有 TOO_MANY_PARTS 错误
SELECT event_time, query, exception_code
FROM system.query_log
WHERE event_time > now() - INTERVAL 5 MINUTE
  AND type = 'ExceptionWhileProcessing'
  AND query LIKE '%portrait_tag_0_backup_local%'

-- 检查 INSERT 是否成功
SELECT event_time, query_kind, exception_code,
       if(exception = '', 'OK', exception) AS result
FROM system.query_log
WHERE event_time > now() - INTERVAL 5 MINUTE
  AND type = 'QueryFinish'
  AND query LIKE '%INSERT INTO portrait_2.portrait_tag_0_backup_local%'
```
