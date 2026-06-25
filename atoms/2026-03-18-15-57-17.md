---
created: 2026-03-18T15:57:17+08:00
---

# `sql.Result`：表示一次执行 SQL 语句后的结果摘要

## 类型签名

- `type Result interface`

- **所属包：** `database/sql`

### `sql.Result` 里通常关心什么

- 它不是查询出来的数据本身，而是“这次执行产生了什么结果”的摘要对象，通常出现在 `Exec` / `ExecContext` 的返回值里。

- 常用就是继续调用这两个方法：
  - `LastInsertId()`：获取插入后新记录的 ID
  - `RowsAffected()`：获取本次 SQL 影响了多少行

### 使用 `sql.Result` 时要注意

- 不是所有驱动都支持 `LastInsertId()`，具体要看你使用的数据库驱动。

- SQL 执行成功，不代表一定改到了数据；更新或删除时，通常还要再检查 `RowsAffected()` 是否为 `0`。
