---
created: 2026-03-17T12:58:00+08:00
---

# Go 的 time.Time

讲清 Go 的 `time.Time` 本质、常见操作（获取时间、时间运算、比较、格式化与解析）与易错点。

### `time.Time` 是“一个时间点”

不是“日期字符串”，也不是“时间戳整数”，而是 Go 里的时间对象。

## 常见操作

你最常做的事只有 4 类：拿当前时间、做加减、比较先后、格式化/解析。

### 获取时间

- **拿当前时间：** `now := time.Now()`

- **解析时间字符串：** `t, err := time.Parse("2006-01-02 15:04:05", "2026-03-17 14:30:00")`

    - **⚠️注意：** Go 的格式化模板不是 `%Y-%m-%d`（`YYYY-MM-DD`），而是一串神奇数字（`2006-01-02 15:04:05`）。

### 时间运算

- 加一段时间： `expireAt := now.Add(30 * time.Minute)`

- 求两个时间差：

  - `d := end.Sub(start)`

  - `time.Since(start)`
  
    ```go
    start := time.Now()
    // do something
    cost := time.Since(start) // 本质上等于：time.Now().Sub(start)
    ```

### 比较时间

- 不要自己拆年月日比较。

- 
    ```go
    if t1.Before(t2) { ... }
    if t1.After(t2) { ... }
    if t1.Equal(t2) { ... }
    ```

### 格式化输出

- `s := now.Format("2006-01-02 15:04:05")`

### 时区

- 存储时间统一使用 UTC，展示给用户时再转本地时区。

- **转 UTC：** `utc := time.Now().UTC()`

- **指定时区：**
  
    ```go
    loc, err := time.LoadLocation("Asia/Shanghai")
    t := time.Now().In(loc)
    ```
