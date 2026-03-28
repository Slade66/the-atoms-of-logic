---
created: 2026-03-23T17:50:01+08:00
---

# Maven

[[Maven 的 scope 是什么？]]

### 常用命令

`-DskipTests`：编译测试代码，但不运行测试。

`mvn compile`

会做这些事：
1️⃣ 下载依赖
2️⃣ 编译代码（src/main/java）
3️⃣ 输出 .class 文件

不会：
打包 jar
启动项目
