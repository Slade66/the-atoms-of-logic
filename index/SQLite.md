
### 🧠 安装 SQLite

1. 访问： `https://sqlite.org/download.html`
2. 下载并解压 `sqlite-tools-win-x64-3510300.zip`（这个包里包含最关键的命令行工具 `sqlite3`）
3. 将解压后的文件夹路径添加到系统环境变量 `Path` 中。

### 🧠 `DEFAULT CURRENT_TIMESTAMP`：自动填写当前时间。
    
- **作用：** 插入数据时，如果你没传这个字段，SQLite 会自动填写预先设置的数据。

- **本质：** SQLite 的内置函数，相当于：`datetime('now')`。实际存储是 `TEXT`，会存成："2026-03-18 10:23:45" 这样的字符串。

- 默认是 UTC 时间。

- **不会自动更新 `updated_at`：** 不像 MySQL 的 `ON UPDATE CURRENT_TIMESTAMP`，SQLite 不支持这个功能。你需要在应用层面手动更新 `updated_at` 字段。

### 🧠 SQLite 没有真正 `BOOLEAN` 类型

- `BOOLEAN` 实际上是 `INTEGER`，`FALSE` 会被转换成 `0`，`TRUE` 会被转换成 `1`。

- `is_active BOOLEAN DEFAULT FALSE` 等价于 `is_active INTEGER DEFAULT 0`。
    
### 🧠 大多数情况下，不需要 `AUTOINCREMENT`

- `id INTEGER PRIMARY KEY` 已经是自增主键了。

- **`AUTOINCREMENT` 的作用：** 保证 ID 不会被重用。

    - 不加 `AUTOINCREMENT`：如果你删除了某条记录，SQLite 可能会重用那个 ID。
    
    - 加了 `AUTOINCREMENT`：即使删除了记录，ID 也不会被重用，下一条记录会继续递增。

- **适用场景：** 只有在必须全局唯一递增，不允许 ID 回退的场景下，才用 `AUTOINCREMENT`。

### 🧠 SQLite 的字段类型

INTEGER	整数
TEXT	字符串
REAL	浮点数
BLOB	二进制
DATETIME	实际是 TEXT

## SQL

### 🧠 插入数据（INSERT）

- 往指定列，塞对应值。

- **SQLite 使用 `?` 占位符：**

    - Go 里要写成：`db.Exec("INSERT INTO users(name, age) VALUES(?, ?)", "Alice", 25)`

- **自增主键可以不写：** SQLite 会自动生成主键。

- **`NULL` 不能加引号：** 

    ```sql
    NULL   ✅
    'NULL' ❌（变字符串）
    ```

- **字符串必须加引号：**

    ```sql
    'Alice' ✅
    Alice   ❌
    ```

#### 🧠 插入一行

- **代码示例：**

    ```sql
    INSERT INTO users (name, age)
    VALUES ('Alice', 25);
    ```

#### 🧠 插入全部列（省略列名）

- **代码示例：**

    ```sql
    INSERT INTO users
    VALUES (1, 'Alice', 25);
    ```

#### 🧠 一次插入多行

- **代码示例：**

    ```sql
    INSERT INTO users (name, age)
    VALUES 
    ('Alice', 25),
    ('Bob', 30);
    ```

#### 🧠 插入或忽略冲突

- **作用：** 避免唯一键冲突报错，直接跳过这条记录。

- **代码示例：**

    ```sql
    INSERT OR IGNORE INTO users (name, age)
    VALUES ('Alice', 25);
    ```

#### 🧠 插入或替换

- **本质：** 先 `DELETE` 再 `INSERT`。

- **注意：** 会影响外键、触发器。

- **代码示例：**

    ```sql
    INSERT OR REPLACE INTO users (name, age)
    VALUES ('Alice', 25);
    ```

### 🧠 查询数据（SELECT）

- **代码示例：**

    ```sql
    SELECT * FROM users
    ORDER BY age DESC
    LIMIT 10;
    ```

### 🧠 更新数据（UPDATE）

- **代码示例：**

    ```sql
    UPDATE users
    SET age = 30
    WHERE id = 1;
    ```

- ⚠️ 不写 WHERE = 全表更新

### 🧠 删除数据（DELETE）

- **代码示例：**

    ```sql
    DELETE FROM users WHERE id = 1;
    ```

### 🧠 删除表

- **代码示例：**

    ```sql
    DROP TABLE users;
    ```

### 🧠 修改表结构（ALTER TABLE）

- SQLite 只支持添加列；不支持删除列、修改列类型，只能建新表 + 拷贝数据。

- **代码示例（添加列）：**

    ```sql
    ALTER TABLE users ADD COLUMN email TEXT;
    ```

### 🧠 约束（Constraints）

- **代码示例：**

    ```sql
    CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE,
        age INTEGER CHECK(age > 0)
    );
    ```

### 🧠 外键（Foreign Key）

- **代码示例：**

    ```sql
    CREATE TABLE orders (
        id INTEGER PRIMARY KEY,
        user_id INTEGER,
        FOREIGN KEY(user_id) REFERENCES users(id)
    );
    ```

- ⚠️ SQLite 默认不开启外键约束，需要：`PRAGMA foreign_keys = ON;`

### 🧠 索引（Index）

- **代码示例：**

    ```sql
    CREATE INDEX idx_users_name ON users(name);
    ```

