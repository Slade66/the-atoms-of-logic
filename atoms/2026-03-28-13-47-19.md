---
created: 2026-03-28T13:47:19+08:00
---

# Linux 配置代理

1. **编辑 `~/.bashrc`：**

    ```bash
    export http_proxy=http://127.0.0.1:7891
    export https_proxy=http://127.0.0.1:7891
    export all_proxy=socks5://127.0.0.1:7891
    ```

2. **使配置生效：** `source ~/.bashrc`

3. **验证：** `curl -I https://www.google.com`

- **注意：** V2Ray/Clash 需开启 “允许局域网连接”。
