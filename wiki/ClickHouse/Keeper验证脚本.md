---
title: Keeper 验证脚本
type: script
created: 2026-05-27
tags:
  - ClickHouse
  - Keeper
  - script
  - verification
related:
  - [[ClickHouse 知识指南]]
  - [[CK 22.2→23.8 升级与 Keeper 迁移执行计划]]
source: ck 项目空间 / scripts/ck-keeper-verify.sh
---

# ClickHouse + Keeper 状态验证脚本

> 来源: `scripts/ck-keeper-verify.sh`
> 用法: `bash ck-keeper-verify.sh [all|ck|keeper|replica|zkdata]`

## 功能

- **ck**: 检查 6 台 CK 版本是否为 23.8.11.28
- **keeper**: 检查 Keeper raft 集群 (1 leader + 2 followers)
- **replica**: 检查副本状态 (is_readonly, queue_size, exception)
- **zkdata**: 检查 Keeper 中 /clickhouse 路径数据完整性
- **all**: 以上全部

## 脚本源码

```bash
#!/bin/bash
#
# ClickHouse 升级与 Keeper 迁移验证脚本
# 用法: bash ck-keeper-verify.sh [all|ck|keeper|replica]
#

set -euo pipefail

# ============ 参数配置（生产部署前必须修改）============
CH_HOST="${CH_HOST:-15.28.140.41}"
CH_USER="${CH_USER:-default}"
CH_PASSWORD="${CH_PASSWORD:-6RQ382CjVqhrsDWj}"
CLUSTER_NAME="${CLUSTER_NAME:-ck3node}"
KEEPER_HOSTS=("15.28.140.41" "15.28.140.42" "15.28.140.43")
KEEPER_PORT="${KEEPER_PORT:-9181}"
EXPECTED_CK_VERSION="23.8.11.28"
EXPECTED_KEEPER_VERSION="23.8.11.28"
# =====================================================

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

pass() { echo -e "${GREEN}[PASS]${NC} $1"; }
fail() { echo -e "${RED}[FAIL]${NC} $1"; exit 1; }
warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
info() { echo -e "[INFO]  $1"; }

run_query() {
    clickhouse-client --host "$CH_HOST" --user "$CH_USER" --password "$CH_PASSWORD" \
        --query "$1" 2>/dev/null
}

check_ck_version() {
    info "=== 检查 CK 版本 ==="
    for h in 15.28.140.41 15.28.140.42 15.28.140.43 15.28.140.44 15.28.140.45 15.28.140.46; do
        local ver
        ver=$(clickhouse-client --host "$h" --user "$CH_USER" --password "$CH_PASSWORD" \
            --query "SELECT version()" 2>/dev/null) || { fail "CK $h 不可访问"; }
        if [[ "$ver" == "$EXPECTED_CK_VERSION" ]]; then
            pass "CK $h 版本: $ver"
        else
            fail "CK $h 版本异常: $ver (期望 $EXPECTED_CK_VERSION)"
        fi
    done
}

check_keeper() {
    info "=== 检查 Keeper 集群 ==="
    local leader=0
    local followers=0
    for h in "${KEEPER_HOSTS[@]}"; do
        local mntr
        mntr=$(echo mntr | nc -w 3 "$h" "$KEEPER_PORT" 2>/dev/null) || \
            { fail "Keeper $h:$KEEPER_PORT 不可访问"; }
        local mode
        mode=$(echo "$mntr" | grep "zk_server_state" | awk '{print $2}')
        local outstanding
        outstanding=$(echo "$mntr" | grep "zk_outstanding_requests" | awk '{print $2}')
        if [[ "$mode" == "leader" ]]; then
            ((leader++)) || true
            local followers_count
            followers_count=$(echo "$mntr" | grep "zk_followers" | awk '{print $2}' || echo "N/A")
            pass "Keeper $h: leader, followers=$followers_count, outstanding=$outstanding"
        elif [[ "$mode" == "follower" ]]; then
            ((followers++)) || true
            pass "Keeper $h: follower, outstanding=$outstanding"
        else
            warn "Keeper $h: 状态=$mode (outstanding=$outstanding)"
        fi
    done
    if [[ "$leader" -eq 1 && "$followers" -eq 2 ]]; then
        pass "Keeper raft 集群配置正确（1 leader + 2 followers）"
    else
        fail "Keeper raft 集群异常: leader=$leader, followers=$followers"
    fi
}

check_replicas() {
    info "=== 检查副本状态 ==="
    local readonly_cnt queue_cnt exc_cnt
    readonly_cnt=$(run_query "
        SELECT count()
        FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
        WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
          AND is_readonly != 0
    ")
    queue_cnt=$(run_query "
        SELECT count()
        FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
        WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
          AND queue_size > 0
    ")
    exc_cnt=$(run_query "
        SELECT count()
        FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
        WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
          AND last_queue_update_exception != ''
    ")
    echo ""
    info "汇总: readonly=$readonly_cnt, queue=$queue_cnt, exception=$exc_cnt"
    if [[ "$readonly_cnt" -eq 0 && "$queue_cnt" -eq 0 && "$exc_cnt" -eq 0 ]]; then
        pass "所有副本正常（is_readonly=0, queue_size=0, 无异常）"
    else
        if [[ "$readonly_cnt" -gt 0 ]]; then
            warn "存在 $readonly_cnt 个只读副本"
            run_query "
                SELECT hostName(), database, table, is_readonly, last_queue_update_exception
                FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
                WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
                  AND is_readonly != 0
                LIMIT 5
            " | while read -r line; do warn "  $line"; done
        fi
        if [[ "$exc_cnt" -gt 0 ]]; then
            warn "存在 $exc_cnt 个异常副本"
            run_query "
                SELECT hostName(), database, table, last_queue_update_exception
                FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
                WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
                  AND last_queue_update_exception != ''
                LIMIT 5
            " | while read -r line; do warn "  $line"; done
        fi
    fi
}

check_zookeeper_data() {
    info "=== 检查 Keeper 数据完整性 ==="
    local root_items
    root_items=$(run_query "SELECT name FROM system.zookeeper WHERE path='/' FORMAT TabSeparated" 2>/dev/null)
    local has_clickhouse
    has_clickhouse=$(echo "$root_items" | grep -c "clickhouse" || echo 0)
    if [[ "$has_clickhouse" -gt 0 ]]; then
        pass "Keeper 包含 /clickhouse 路径"
    else
        fail "Keeper 缺少 /clickhouse 路径（可能 snapshot 转换失败）"
    fi
    local tables_cnt
    tables_cnt=$(run_query "SELECT count() FROM system.zookeeper WHERE path='/clickhouse/tables'" 2>/dev/null)
    info "  /clickhouse/tables 路径数: $tables_cnt"
}

check_migrations() {
    info "=== 检查 pending merges ==="
    local pending
    pending=$(run_query "
        SELECT sum(merges_in_queue)
        FROM clusterAllReplicas('$CLUSTER_NAME', system, replicas)
        WHERE database NOT IN ('system','INFORMATION_SCHEMA','information_schema')
    " 2>/dev/null)
    if [[ "${pending:-0}" -gt 50 ]]; then
        warn "pending merges 较多: $pending（可能正常，需要观察）"
    else
        pass "pending merges 正常: $pending"
    fi
}

main() {
    local target="${1:-all}"
    echo ""
    echo "========================================"
    echo " ClickHouse + Keeper 状态验证"
    echo "========================================"
    echo "CK Host: $CH_HOST"
    echo "Cluster: $CLUSTER_NAME"
    echo "Keeper:  ${KEEPER_HOSTS[*]}:$KEEPER_PORT"
    echo "期望版本: CK=$EXPECTED_CK_VERSION"
    echo "========================================"
    echo ""

    case "$target" in
        all)
            check_ck_version
            echo ""
            check_keeper
            echo ""
            check_zookeeper_data
            echo ""
            check_replicas
            echo ""
            check_migrations
            echo ""
            info "=== 验证完成 ==="
            ;;
        ck)      check_ck_version ;;
        keeper)  check_keeper ;;
        replica) check_replicas ;;
        zkdata) check_zookeeper_data ;;
        *) echo "用法: $0 [all|ck|keeper|replica|zkdata]" ;;
    esac
}

main "$@"
```
