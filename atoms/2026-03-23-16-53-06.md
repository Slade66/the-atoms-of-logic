---
created: 2026-03-23T16:53:06+08:00
---

# BCrypt

- **是什么？** BCrypt 是一种专门用于密码加密（更准确说是“密码哈希”）的算法。

- **hash 包含：** 算法版本 + 成本因子 + salt + hash

- **验证流程：**
    - 用户输入：123456
    - 从数据库 hash 中提取 salt
    - 用“同一个 salt + 同样算法”再算一次：`hash(123456 + 原来的salt)`
    - 比较结果是否一致

- **类比：**
    - BCrypt 是 “带配方的锁”，每个密码都会生成一把“锁”（hash），锁里面已经写好了配方（salt）。

- **常用于用户登录系统中存储密码：**
    - **注册时：** 用户输入密码 → 随机 salt 生成 hash → 存数据库
    - **登录时：** 用户输入密码 → 用“存储在哈希值中的原来的 salt” 再算 hash → 是否匹配数据库hash

- **自动加盐：**
    - 每个密码都会生成随机 salt，相同密码 → 不同 hash。

- **抗暴力破解：** BCrypt 设计就是故意的慢算法。
    - 和 MD5 对比：
        - MD5：1秒几亿次 ❌
        - BCrypt：1秒几十次 ✔

- **不可逆：**
    - 无法从 hash 结果反推出原密码。
    - **为什么要不可逆？**
        - 防止数据库泄露后用户密码被破解。
        - 可逆加密（AES），密钥泄露也完蛋。
    - **如何验证？**
        - 只能这样做：`用户输入 → 再算一遍 → 对比`

- **自适应计算成本：**
    - 可以控制“算得有多慢”。
    - **为什么要“慢”？**
        - 密码破解本质是：疯狂尝试（暴力破解）
        - 如果算法很快：MD5：1秒几亿次 ❌（很危险）
        - 如果算法很慢：BCrypt：1秒几十次 ✔（更安全）
    - **为什么叫“自适应”？**
        - 因为未来电脑会变快：
            - 现在：用 10
            - 以后：用 12 / 14
        - 不用换算法，只需调参数。随着硬件性能提升动态增强安全性。
    - **cost 是什么？**
        - cost = 计算复杂度（指数增长）
        - 每 +1，时间大约 翻倍。
        - 8	很快
        - 10	常用
        - 12	更安全
        - 14	很慢
        - **cost 设太大有什么问题？**
            - cost 设太大，登录变慢（用户体验下降），服务器压力变大，需要 安全 vs 性能权衡。

### Spring Boot 中怎么用？

- **代码示例：**

    1️⃣ 引入依赖（如果用 security）
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    ```

    2️⃣ 使用 BCryptPasswordEncoder
    ```java
    import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    // 加密
    String hash = encoder.encode("123456");

    // 校验
    boolean match = encoder.matches("123456", hash);
    ```

### Go 怎么用？

- Go 官方推荐的就是 BCrypt：`import "golang.org/x/crypto/bcrypt"`

- 代码示例：

    1️⃣ 加密密码
    ```go
    password := "123456"

    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        panic(err)
    }

    fmt.Println(string(hash))
    ```

    2️⃣ 校验密码
    ```go
    err := bcrypt.CompareHashAndPassword(hash, []byte("123456"))

    if err == nil {
        fmt.Println("密码正确")
    } else {
        fmt.Println("密码错误")
    ```
