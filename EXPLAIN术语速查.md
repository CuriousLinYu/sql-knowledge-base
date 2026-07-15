# EXPLAIN 术语速查

---

## type 列（性能关键，从好到差）

| type | 含义 | 说明 |
|------|------|------|
| **NULL** | 最优 | 不访问表（如 `SELECT 1`） |
| **const** | 常数引用 | 主键/唯一索引等值匹配，最多一行 |
| **eq_ref** | 唯一索引引用 | JOIN 时对每个前表行最多匹配一行 |
| **ref** | 非唯一索引引用 | 普通索引等值匹配，可能多行 |
| **range** | 范围扫描 | `>`、`<`、`BETWEEN`、`IN`、`LIKE 'xxx%'` |
| **index** | 全索引扫描 | 扫描整个索引树（比 ALL 快但不理想） |
| **ALL** | 全表扫描 | 最差，必须优化 |

---

## Extra 列（诊断关键列）

| Extra 值 | 含义 | 行动 |
|----------|------|------|
| **Using index** | 覆盖索引，无需回表 | ✅ 好 |
| **Using where** | Server 层额外过滤 | ⚠️ 关注 filtered 列 |
| **Using index condition** | ICP（索引条件下推） | ✅ 优化生效 |
| **Using temporary** | 创建临时表 | ❌ 需优化（通常 GROUP BY/DISTINCT 无索引） |
| **Using filesort** | 文件排序 | ❌ 需优化（ORDER BY 无索引） |
| **Using join buffer** | JOIN 列无索引 | ❌ 需加索引 |

---

## select_type 列

| 值 | 含义 |
|----|------|
| **SIMPLE** | 简单查询（无子查询/UNION） |
| **PRIMARY** | 最外层 SELECT |
| **SUBQUERY** | 子查询中的第一个 SELECT |
| **DERIVED** | FROM 子句中的子查询（派生表） |
| **DEPENDENT SUBQUERY** | 依赖外部查询的子查询（每行执行一次，危险！） |
| **MATERIALIZED** | 子查询结果物化（MySQL 8.0+） |
| **UNION** | UNION 的后续 SELECT |
| **UNION RESULT** | UNION 合并结果 |

---

## 快速诊断流程

```
EXPLAIN 输出
  ├── type=ALL 且 rows>10000 → 必须加索引
  ├── Extra=Using filesort → ORDER BY 列加索引
  ├── Extra=Using temporary → GROUP BY/DISTINCT 列加索引
  ├── Extra=Using join buffer → JOIN 列加索引
  ├── select_type=DEPENDENT SUBQUERY → 改写为 JOIN
  └── type=ALL 但 rows<100 → 数据量小，可接受
```
