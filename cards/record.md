---
created: 2026-03-23T20:33:55+08:00
---

# record

只存不可变数据的类。

```java
public record User(String name, int age) {}
```

**它自动帮你生成常用方法：**
构造方法
getter（不是 getName()，而是 name()）
equals()
hashCode()
toString()

**不可变：**
所有字段默认：`private final`
创建对象后不允许改值。

**不能继承其他类**

**用在哪？**
请求参数 DTO
返回 VO

**本质：** record 还是 class（语法糖）

