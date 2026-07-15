# MySQL JOIN 优化（5.7 / 8.0 通用）

---

## MYSQL-JOIN-001：LEFT JOIN 后 WHERE 子表条件

**严重性**：⚠️ 警告

WHERE 中对右表列做等值过滤会使 LEFT JOIN 退化为 INNER JOIN。

```sql
-- ❌ WHERE 对右表过滤 → LEFT JOIN 退化为 INNER JOIN
SELECT * FROM orders o
LEFT JOIN users u ON u.id = o.user_id
WHERE u.status = 'active';

-- ✅ 移至 ON 子句（保留 LEFT JOIN 语义）
SELECT * FROM orders o
LEFT JOIN users u ON u.id = o.user_id AND u.status = 'active';

-- ✅ 或直接改为 INNER JOIN（如果确实需要过滤）
SELECT * FROM orders o
INNER JOIN users u ON u.id = o.user_id AND u.status = 'active';
```

**证据**：MySQL 8.0 Reference Manual §13.2.13

---

## MYSQL-JOIN-002：JOIN 顺序——小结果集驱动大表

**严重性**：ℹ️ 提示

NLJ（Nested Loop Join）下，驱动表越小，内层循环次数越少。用 EXPLAIN 的 `rows` 列判断。

```sql
-- EXPLAIN rows: orders=100, users=10000
-- 正确：orders 驱动（小表驱动大表）
SELECT * FROM orders o INNER JOIN users u ON u.id = o.user_id;

-- EXPLAIN rows: orders=10000, users=100
-- 错误：orders 驱动（大表驱动小表）
-- MySQL 优化器通常会自动调整，但复杂 JOIN 时可能失效
```

**证据**：MySQL 8.0 Reference Manual §8.2.1.7 Nested-Loop Join Algorithms

---

## MYSQL-JOIN-003：缺少 JOIN 条件 → 笛卡尔积

**严重性**：⚠️ 警告

每对表之间至少需要一个连接条件。

```sql
-- ❌ 笛卡尔积
SELECT * FROM orders o, users u;

-- ✅ 必须有关联条件
SELECT * FROM orders o INNER JOIN users u ON u.id = o.user_id;
```

**证据**：MySQL 8.0 Reference Manual §13.2.13

---

## MYSQL-JOIN-004：JOIN 字段类型不一致 → 索引失效

**严重性**：⚠️ 警告

JOIN 双方列类型必须一致，隐式类型转换导致索引失效。

```sql
-- ❌ user_id 是 INT，传入 VARCHAR → 索引失效
SELECT * FROM orders o INNER JOIN users u ON u.id = o.user_id
WHERE o.user_id = '12345';

-- ✅ 确保类型一致
SELECT * FROM orders o INNER JOIN users u ON u.id = o.user_id
WHERE o.user_id = 12345;
```

**证据**：MySQL 8.0 Reference Manual §12.3 Type Conversion in Expression Evaluation

---

## MYSQL-JOIN-005：排查 SQL 应主动 JOIN 头表

**严重性**：⚠️ 警告

查询有头-行关系的表时，先 JOIN 头表拿到完整上下文，不要多条 SQL 分开查再手动对。

```sql
-- ❌ 分两次查
SELECT * FROM order_header WHERE order_id = 1;
SELECT * FROM order_lines WHERE order_id = 1;

-- ✅ 一次 JOIN
SELECT * FROM order_header h
INNER JOIN order_lines l ON l.order_id = h.order_id
WHERE h.order_id = 1;
```
