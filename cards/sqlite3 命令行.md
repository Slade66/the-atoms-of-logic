---
created: 2026-03-18T14:22:07+08:00
---

# SQLite 命令行

### 🧠 打开数据库

- 在终端进入项目目录，然后执行：`sqlite3 test.db`

### 🧠 查看所有表

- 在 `sqlite>` 提示符下输入：`.tables`

### 🧠 查看表结构（建表 SQL）

- 在 `sqlite>` 提示符下输入：`.schema 表名`

### 🧠 查看表数据

- 在 `sqlite>` 提示符下输入：`SELECT * FROM 表名;`

### 🧠 美化输出

- 在 `sqlite>` 提示符下输入：`.mode column` 和 `.headers on`，可以让查询结果以表格形式显示，并且显示列名。

### 🧠 退出

- 在 `sqlite>` 提示符下输入 `.exit` 或 2 次 `Ctrl+C` 来退出命令行工具。

