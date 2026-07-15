# MySQL SELECT 优化（5.7 / 8.0 通用）

---

## MYSQL-SELECT-001：SELECT * → 指定需要的列

**严重性**：⚠️ 警告

减少网络传输、内存占用，增加覆盖索引可能性。

```sql
-- ❌
SELECT * FROM orders WHERE user_id = 1;

-- ✅
SELECT id, order_no, amount, status, created_at FROM orders WHERE user_id = 1;
```

**证据**：MySQL 8.0 Reference Manual §8.2.1.20

---

## MYSQL-SELECT-002：大分页 OFFSET → 游标分页

**严重性**：ℹ️ 提示

OFFSET 越大性能越差，MySQL 需要跳过前 N 行。

```sql
-- ❌ OFFSET 100000
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 100000;

-- ✅ 游标分页（记住上一页最后 id）
SELECT * FROM orders WHERE id > 100000 ORDER BY id LIMIT 20;
```

**详见**：`深度方案/延迟关联.md`

---

## MYSQL-SELECT-003：COUNT(*) vs COUNT(col)

**严重性**：ℹ️ 提示

- `COUNT(*)` — 统计所有行，包含 NULL
- `COUNT(col)` — 忽略 NULL 值
- 优先用 `COUNT(*)`，语义明确

**证据**：MySQL 8.0 Reference Manual §12.20.1 Aggregate Functions

---

## MYSQL-SELECT-004：ORDER BY + LIMIT 缺索引 → filesort

**严重性**：⚠️ 警告

确保 ORDER BY 列有索引且排序方向一致。

```sql
-- ❌ Extra: Using filesort
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC LIMIT 10;

-- ✅ 联合索引 (user_id, created_at)
CREATE INDEX idx_user_created ON orders(user_id, created_at);
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC LIMIT 10;
```

---

## MYSQL-SELECT-005：GROUP BY 非聚合列不一致 → 报错

**严重性**：⚠️ 警告

MySQL 5.7+ 默认开启 `ONLY_FULL_GROUP_BY`。

```sql
-- ❌ MySQL 5.7 报错
SELECT name, MAX(salary) FROM employees GROUP BY dept_id;

-- ✅
SELECT dept_id, MAX(salary) FROM employees GROUP BY dept_id;
```

**证据**：MySQL 5.7 Reference Manual §5.1.8 Server SQL Modes
