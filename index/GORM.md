---
created: 2026-03-20T15:09:29+08:00
---

# GORM

## find


## preload

### 自定义查询

```go
db.Preload("Orders", func(db *gorm.DB) *gorm.DB {
    return db.Order("price DESC").Limit(5)
}).Find(&users)
```
场景：每个用户只要最近 5 条订单

不全查，只查几个字段
```go
Preload("Site", func(tx *gorm.DB) *gorm.DB {
			return tx.Select("id", "uuid", "name")
		}).
```
