---
created: 2026-03-28T13:19:09+08:00
---

# 配置 `apt` 使用代理

- `apt` 需要 HTTP/HTTPS 代理，环境变量对 `apt` 无效，必须单独配置 `/etc/apt/apt.conf.d/`。

- **执行命令：**

    ```bash
    sudo tee /etc/apt/apt.conf.d/99proxy <<EOF
    Acquire::http::Proxy "http://127.0.0.1:7891";
    Acquire::https::Proxy "http://127.0.0.1:7891";
    EOF
    ```

- **为什么要加 `99proxy`？**

    - `/etc/apt/apt.conf.d/` 是 `apt` 包管理器的配置目录，里面每个文件都是模块化配置文件，按文件名数字顺序读取（01 先于 70）。

    - 99 是最高优先级，数字越大越后读，覆盖前面所有配置，确保代理设置最终生效，不会冲突。
