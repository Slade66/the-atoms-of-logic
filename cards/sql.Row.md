---
created: 2026-03-18T16:44:38+08:00
---

# `sql.Row`：表示一次“预期只返回一行”的查询结果

## 类型签名

- `type Row struct`

- **所属包：** `database/sql`

### `sql.Row` 有什么特点

- 它通常来自 `QueryRow` 或 `QueryRowContext`，适合按主键、唯一键这类“最多只应返回一条记录”的查询场景。

- `sql.Row` 不需要你手动 `Close()`。它底层虽然也是基于 `Rows` 实现的，但已经帮你处理好了“只取一行”和“自动释放底层资源”这件事。

### 使用 `sql.Row` 时要注意

- `QueryRow` / `QueryRowContext` 本身不会直接返回错误，很多错误会延迟到你调用 `row.Scan(...)` 时才暴露出来。

- 如果查询结果为空，`Scan(...)` 会返回 `sql.ErrNoRows`。这通常表示“没查到数据”，而不是程序崩了。
