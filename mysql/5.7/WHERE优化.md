# MySQL WHERE 优化（5.7 / 8.0 通用）

---

## MYSQL-WHERE-001：OR 替代 IN → 性能下降

**严重性**：⚠️ 警告

等值查询优先用 IN，IN 会被优化器转为范围查询，OR 不行。

```sql
-- ❌ OR 连接
SELECT * FROM orders WHERE status = 'pending' OR status = 'processing' OR status = 'shipped';

-- ✅ IN 列表
SELECT * FROM orders WHERE status IN ('pending', 'processing', 'shipped');
```

**证据**：MySQL 8.0 Reference Manual §8.2.1.17 Range Optimization

---

## MYSQL-WHERE-002：NULL 列做等值比较 → 永远不命中

**严重性**：🔴 严重

```sql
-- ❌ 永远返回空
SELECT * FROM users WHERE deleted_at = NULL;

-- ✅ 正确写法
SELECT * FROM users WHERE deleted_at IS NULL;
```

**证据**：MySQL 8.0 Reference Manual §3.3.4.6 Working with NULL Values

---

## MYSQL-WHERE-003：条件顺序——高筛选率优先

**严重性**：ℹ️ 提示

把能过滤最多行的条件写在前。

```sql
-- ✅ status 过滤 90%，优先
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2024-01-01';
```

**证据**：MySQL 8.0 Reference Manual §8.2.1.1 WHERE Clause Optimization

---

## MYSQL-WHERE-004：SELECT * + 无条件 LIMIT

**严重性**：⚠️ 警告

无条件 LIMIT 可能扫描到 LIMIT 行后停止，但加上 ORDER BY 仍需全排序。

```sql
-- ❌ 有 ORDER BY 时，全排序后才取 10 行
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;

-- ✅ 确保 ORDER BY 列有索引
CREATE INDEX idx_orders_created ON orders(created_at);
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;
```

---

## MYSQL-WHERE-005：否定条件 → 索引可能不生效

**严重性**：ℹ️ 提示

`!=`、`NOT IN`、`NOT LIKE` 通常不走索引。

```sql
-- ❌ 可能不走索引
SELECT * FROM orders WHERE status != 'cancelled';

-- ✅ 用 IN 明确列出
SELECT * FROM orders WHERE status IN ('pending', 'processing', 'shipped');
```
