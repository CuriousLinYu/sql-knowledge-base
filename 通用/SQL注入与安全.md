# SQL 注入与安全

---

## SQL-BASE-001：禁止 SQL 注入——参数化查询

**严重性**：🔴 严重

绝不拼接用户输入到 SQL。

```java
// ❌ 拼接字符串
String sql = "SELECT * FROM users WHERE name = '" + userName + "'";

// ✅ 参数化查询
String sql = "SELECT * FROM users WHERE name = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, userName);
```

```xml
<!-- ✅ MyBatis 参数化 -->
<select id="getUser" resultType="User">
    SELECT * FROM users WHERE name = #{userName}
</select>

<!-- ❌ ${} 直接拼接 -->
<select id="getUser" resultType="User">
    SELECT * FROM users WHERE name = '${userName}'
</select>
```

**证据**：OWASP Top 10 — A03:2021 Injection
