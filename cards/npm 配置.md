---
created: 2026-03-19T10:44:04+08:00
---

# npm 配置

- **查看 HTTP 代理：** `npm config get proxy`

- **查看 HTTPS 代理：** `npm config get https-proxy`

- **查看所有配置（包括代理和源）：** `npm config list`

- **清除 npm 自身配置的代理：**

    ```bash
    npm config delete proxy
    npm config delete https-proxy
    ```
