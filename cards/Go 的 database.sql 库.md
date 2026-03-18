
### `database/sql` 是什么？

- `database/sql` 是 Go 标准库提供的对关系型数据库操作统一抽象层，负责对外暴露一致的数据库操作 API。database/sql 本身是一个接口，不是具体数据库驱动，它必须配合具体驱动使用。

- 你平常用的 API：
  - `sql.Open`
  - `db.QueryContext`
  - `db.ExecContext`
  - `db.BeginTx`

- 真正与具体数据库（如 MySQL、SQLite）通信的，是对应的驱动实现，例如：
  - MySQL: `github.com/go-sql-driver/mysql`
  - SQLite: `github.com/mattn/go-sqlite3`

理解要点：
- `database/sql` = 通用 API（抽象层）
- 驱动 = 具体实现（负责与数据库通信）

### sql.Row：单行结果

row 不需要关，QueryRow 内部其实也是用 rows，但它帮你做了这几件事：只取一行，自动关闭底层资源

### sql.Rows：多行结果集

rows 必须 `defer rows.Close()`，
rows 背后占着：数据库连接，网络资源，结果集游标。忘记关闭 `Rows` 就会导致连接泄露：连接不会及时归还到连接池，连接池被耗尽，最后导致报错崩溃。

### sql.Result

常用两个方法：
LastInsertId()
RowsAffected()

### 事务... // todo

事务里不要混用 db 和 tx
事务开始后，后续 SQL 必须都走 tx.ExecContext / tx.QueryContext / tx.QueryRowContext
因为事务要求这些操作在同一连接上完成。

事务的核心流程只有四步：
1. `BeginTx`
2. 执行一组操作
3. 出错 `Rollback`
4. 成功 `Commit`

```go
func Transfer(ctx context.Context, db *sql.DB, fromID, toID int64, amount int64) (err error) {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return err
	}

	defer func() {
		if err != nil {
			_ = tx.Rollback()
			return
		}
		err = tx.Commit()
	}()

	_, err = tx.ExecContext(ctx,
		`UPDATE accounts SET balance = balance - ? WHERE id = ?`,
		amount, fromID,
	)
	if err != nil {
		return err
	}

	_, err = tx.ExecContext(ctx,
		`UPDATE accounts SET balance = balance + ? WHERE id = ?`,
		amount, toID,
	)
	if err != nil {
		return err
	}

	return nil
}
```


## 🧠 `sql.DB`：表示「一个操作数据库资源的凭证 + 复用多个数据库底层连接的连接池」

[[type DB struct](sql.DB.md)]

### 🧠 `sql.Open`：根据驱动名和数据源信息，创建一个可复用的数据库连接池对象（`*sql.DB`）

[[func Open(driverName, dataSourceName string) (*DB, error)](sql.Open.md)]

### 🧠 `*DB.PingContext`：用上下文执行一次数据库连通性检查

[[func (db *DB) PingContext(ctx context.Context) error](sql.PingContext.md)]

### 🧠 `*DB.QueryRowContext`：用上下文执行查询并返回单行结果

[[func (db *DB) QueryRowContext(ctx context.Context, query string, args ...any) *Row](sql.QueryRowContext.md)]

### 🧠 `*DB.QueryContext`：用上下文执行查询并返回多行结果集

[[func (db *DB) QueryContext(ctx context.Context, query string, args ...any) (*Rows, error)](sql.QueryContext.md)]

### 🧠 `*DB.ExecContext`：用上下文执行不返回结果集（增、删、改）的 SQL 语句

[[func (db *DB) ExecContext(ctx context.Context, query string, args ...any) (Result, error)](sql.ExecContext.md)]

## 最佳实践

### 🧠 永远用 `Context`

- 数据库操作尽量都用 `xxxContext` 版本，这样你能设置超时，也能在请求取消时及时中断 SQL。

- **代码示例：**

  ```go
  ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
  defer cancel()

  err := db.QueryRowContext(ctx, `SELECT name FROM users WHERE id = ?`, id).Scan(&name)
  ```

### 🧠 永远不拼 SQL

- `不要 fmt.Sprintf("...%s...", input)` 拼 SQL。这会带来 SQL 注入风险。

- **正确做法：** 使用占位符 `?`（MySQL）或 `$1`（PostgreSQL）等，配合参数传递。

  ```go
  err := db.QueryRowContext(ctx, `SELECT name FROM users WHERE id = ?`, id).Scan(&name)
  ```

### 🧠 可空字段用 `sql.NullXxx`

- 数据库字段可为 `NULL` 时，不要直接扫进普通 `string/int/time.Time`，否则容易报错。

- **正确做法：** 使用 `sql.NullString`、`sql.NullInt64`、`sql.NullTime`、`sql.NullBool` 等空值类型来接收可能为 `NULL` 的字段。

  ```go
  var name sql.NullString
  err := db.QueryRowContext(ctx, `SELECT name FROM users WHERE id = ?`, id).Scan(&name)
  if err != nil {
      // handle error
  }
  if name.Valid {
      fmt.Println("Name:", name.String)
  } else {
      fmt.Println("Name is NULL")
  }
  ```
