---
title: ClickHouse TOO_MANY_SIMULTANEOUS_QUERIES 排查手册
tags:
  - clickhouse
  - troubleshooting
  - database
  - ck
created: 2026-06-24
source: 实战排查
---

# ClickHouse TOO_MANY_SIMULTANEOUS_QUERIES 排查手册

> **适用场景**：CK 抛出 `Code: 202. DB::Exception: Too many simultaneous queries. Maximum: 100. (TOO_MANY_SIMULTANEOUS_QUERIES)`

## 0. 核心思路

错误只是**症状**，要查的是**为什么并发数瞬间打满**。六个步骤：

1. 定位异常节点 → 2. 看当前进程 → 3. 确认 query_log 可用 → 4. 找 202 异常明细 → 5. 画雪崩曲线 → 6. 抓慢查询元凶

---

## 1. 排查步骤（含可直接复用的 SQL）

### Step 1：确认错误节点

报错堆栈里 `Received from <ip>:9000` 是关键。`system.processes` 和 `system.query_log` 都只看**本地节点**——必须先连到正确节点。

```sql
SELECT cluster, shard_num, replica_num, host_name, port
FROM system.clusters
ORDER BY cluster, shard_num, replica_num
```

### Step 2：查看当前查询分布

```sql
SELECT
    user,
    multiIf(
        startsWith(lower(query), 'select '),  'SELECT',
        startsWith(lower(query), 'insert '),  'INSERT',
        startsWith(lower(query), 'alter '),   'ALTER',
        startsWith(lower(query), 'create '),  'CREATE',
        startsWith(lower(query), 'optimize'), 'OPTIMIZE',
        startsWith(lower(query), 'system '),  'SYSTEM',
        'OTHER'
    ) AS qkind,
    count() AS qty,
    max(elapsed) AS max_elapsed_s,
    sum(read_rows) AS total_read_rows,
    formatReadableSize(sum(memory_usage)) AS total_memory
FROM system.processes
GROUP BY user, qkind
ORDER BY qty DESC
LIMIT 30
```

- `qty ≈ 100` → 当前正在堆积（故障中）
- `qty = 0` → 问题已过或在其他节点，需要查 query_log

### Step 3：确认 query_log 落库情况

```sql
SELECT type, count() AS cnt
FROM system.query_log
WHERE event_time > now() - INTERVAL 1 HOUR
GROUP BY type
ORDER BY cnt DESC
```

🔑 **关键认知**：202 异常记录在 `ExceptionBeforeStart`，**不是** `ExceptionWhileProcessing`——CK 在查询启动前就拒绝了，所以是 BeforeStart。

### Step 4：捞 202 异常明细

```sql
SELECT
    event_time,
    user,
    query_duration_ms,
    exception_code,
    substring(query, 1, 300) AS query_preview
FROM system.query_log
WHERE type = 'ExceptionBeforeStart'
  AND exception_code = 202
  AND event_time > now() - INTERVAL 24 HOUR
ORDER BY event_time DESC
LIMIT 50
```

### Step 5：画雪崩曲线（按秒分桶）

```sql
SELECT
    formatDateTime(event_time, '%Y-%m-%d %H:%M:%S') AS sec,
    countIf(type = 'QueryFinish')          AS finished,
    countIf(type = 'ExceptionBeforeStart') AS rejected_202,
    maxIf(query_duration_ms, type = 'QueryFinish') AS max_dur_ms
FROM system.query_log
WHERE event_time BETWEEN '2026-06-24 08:00:00' AND '2026-06-24 08:30:00'
GROUP BY sec
ORDER BY sec
```

看三个信号：
- `finished` 飙升的秒 → 应用突发
- `rejected_202 > 0` 的秒 → 已被 CK 拒绝
- `max_dur_ms` 大（如 100000+）→ 慢查询霸占槽位

### Step 6：抓慢查询元凶

```sql
SELECT
    formatDateTime(event_time, '%Y-%m-%d %H:%M:%S') AS event_time,
    user,
    query_duration_ms / 1000.0 AS dur_s,
    read_rows,
    formatReadableSize(memory_usage) AS memory,
    substring(query, 1, 250) AS query_preview
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_time BETWEEN '2026-06-24 08:02:00' AND '2026-06-24 08:06:00'
  AND query_duration_ms > 30000
ORDER BY query_duration_ms DESC
LIMIT 30
```

最常见的元凶：
- `ALTER TABLE ... DROP PARTITION`（典型 100-300s）
- `INSERT INTO ... SELECT` 大批量写入
- 全表 `SELECT *` 没带 WHERE

---

## 2. 进阶排查（按需使用）

### 看活跃的 mutations（正在跑的 ALTER）

```sql
SELECT database, table, mutation_id, command, create_time,
       is_done, latest_fail_reason, parts_to_do
FROM system.mutations
WHERE is_done = 0
ORDER BY create_time
```

### 看活跃的 merges（也可能霸占资源）

```sql
SELECT database, table, elapsed, progress, num_parts,
       total_size_bytes_compressed, is_mutation
FROM system.merges
ORDER BY elapsed DESC
```

### 看当前并发阈值配置

```sql
SELECT name, value, changed
FROM system.settings
WHERE name IN (
    'max_concurrent_queries',
    'max_concurrent_insert_queries',
    'max_concurrent_select_queries',
    'max_concurrent_queries_for_user'
)
```

### 看某个时刻的瞬时并发数（用 query_log 反推）

按毫秒桶统计 QueryStart 和 QueryFinish 的差值，可粗略推算并发量：

```sql
SELECT
    formatDateTime(event_time, '%H:%M:%S') AS sec,
    countIf(type = 'QueryStart')  AS started,
    countIf(type = 'QueryFinish') AS finished,
    countIf(type = 'ExceptionBeforeStart') AS rejected
FROM system.query_log
WHERE event_time > now() - INTERVAL 5 MINUTE
GROUP BY sec
ORDER BY sec
```

---

## 3. ⚠️ ClickHouse 22.2 特别注意事项

| 现象 | 原因 | 替代方案 |
|------|------|---------|
| `query_kind` 列不存在 | 22.3+ 才有 | 用 `multiIf + startsWith` 分类（见 Step 2） |
| `exception_text` 列不存在 | 22.3+ 才有 | 删掉这个字段，只看 `exception_code` |
| `toStartOfSecond(DateTime64)` 报 ILLEGAL_TYPE_OF_ARGUMENT | 22.2 不支持 | 用 `formatDateTime(x, '%Y-%m-%d %H:%M:%S')` |
| `toDateTime(DateTime64)` 仍返回 DateTime64 | 老版本行为 | 同上，用 `formatDateTime` |
| `system.query_log.event_time` 是 `DateTime64(3)` | — | 上述 formatDateTime 套路必学 |

**通用规则**：v22.2 上处理 `DateTime64` 时间，**优先用 `formatDateTime`**，比 `toStartOfSecond` 稳得多。

---

## 4. 典型案例：portrait 标签清理任务导致 202

**故障现场**（2026-06-24）：
- 报错：CK 拒绝标签更新请求，code 202
- 雪崩窗口：08:05:29 – 08:05:53（约 25 秒）
- 业务影响：标签服务报错
- 集群拓扑：3 分片 × 2 副本 = 6 节点，错误节点 `10.128.55.16`（Shard 3 / Replica 2）

**根因**：
- `portrait_2` 标签清理任务在 08:04:44 同时发起 **30+ 条 `ALTER TABLE portrait_tag_0_backup_local DROP PARTITION`**
- 每条 ALTER 耗时 176–223 秒（mutation 不是元数据修改，会物理移动 parts）
- `ON CLUSTER ck3node` 让每条 ALTER 在 6 个节点都执行
- 08:05:24 业务 INSERT 突然涌入，5 秒内 152 条查询
- 槽位瞬间打满，新请求被拒

**雪崩曲线特征**：
- 雪崩前 5 秒：48 → 39 → 20 → 22 → 23 finished/秒（应用突发）
- 雪崩中：finished 跌到 0–3/秒，rejected_202 飙到 24/秒
- 雪崩后：仍有持续 30–70s 的慢查询（说明 ALTER 还在跑）

**解决方案方向**：
1. **清理任务加限流**：DROP PARTITION 并发降到 1–2（用信号量或串行队列）
2. **错峰执行**：清理任务与标签更新错开 30 分钟以上
3. **改用更轻量的清理方式**：考虑 `DETACH PARTITION` + 异步清理 detached 目录
4. **临时调参**（缓释不治本）：`config.xml` 里 `<max_concurrent_queries>` 从 100 调到 200
5. **审查分区策略**：`portrait_tag_0_backup_local` 分区粒度是否过细、保留期是否过长

---

## 5. 解决思路速查

| 根因类型 | 特征 | 解决 |
|---------|------|------|
| 应用突发 | finished 曲线某秒飙升 | 限流、错峰、改异步 |
| 慢查询霸占 | max_dur_ms > 30s | 优化 SQL、加索引、避免全表扫描 |
| ALTER / Mutation | 大量 DROP PARTITION / UPDATE | 限流 ALTER 并发、错峰 |
| 监控轮询 | SELECT merges.* 高频 | 调低监控频率、改用 metric_log |
| CK 配置过低 | 集群硬件有富余 | 调高 max_concurrent_queries |
| JDBC 连接泄漏 | 连接数持续上涨 | 检查代码连接池配置 |

---

## 6. 一页纸速记

```
# 登录正确节点
clickhouse-client --host <报错堆栈里的IP>

# 看集群拓扑
SELECT * FROM system.clusters;

# 看当前查询
SELECT user, type, count() FROM system.processes GROUP BY user, type;

# 看 query_log 是否可用
SELECT type, count() FROM system.query_log WHERE event_time > now() - INTERVAL 1 HOUR GROUP BY type;

# 找 202 异常（注意是 ExceptionBeforeStart）
SELECT * FROM system.query_log WHERE type='ExceptionBeforeStart' AND exception_code=202 ORDER BY event_time DESC LIMIT 50;

# 画曲线
SELECT formatDateTime(event_time,'%H:%M:%S') sec, countIf(type='QueryFinish') finished, countIf(type='ExceptionBeforeStart') rejected, maxIf(query_duration_ms, type='QueryFinish') max_ms FROM system.query_log WHERE event_time > now() - INTERVAL 1 HOUR GROUP BY sec ORDER BY sec;

# 抓慢查询
SELECT * FROM system.query_log WHERE type='QueryFinish' AND query_duration_ms > 30000 ORDER BY query_duration_ms DESC LIMIT 30;
```