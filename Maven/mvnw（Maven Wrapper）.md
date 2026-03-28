---
created: 2026-03-28T21:26:51+08:00
---

# mvnw（Maven Wrapper）

- 在 IntelliJ IDEA 创建完 Spring Boot 项目之后，会看到 `mvnw` 和 `mvnw.cmd`，它们让你不用安装 Maven，也能直接运行 Spring Boot 项目。

- **运行 Spring Boot 项目：**

    ```bash
    mvnw spring-boot:run
    ```

- **解决了什么问题？**

    1. `mvnw` 保证团队每个人 Maven 版本一致。
    
    2. `mvnw` 让你的项目代码即使拿到其它没有安装 Maven 的电脑上，也能直接运行。
