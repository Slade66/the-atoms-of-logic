---
created: 2026-03-28T14:01:04+08:00
---

# 安装 appsmith

1. `mkdir appsmith && cd appsmith`
2. **创建 `docker-compose.yml` 文件：**

    ```yaml
    version: "3"
    services:
    appsmith:
        image: index.docker.io/appsmith/appsmith-ce
        container_name: appsmith
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - ./stacks:/appsmith-stacks
        restart: unless-stopped
    ```

3. `docker compose up -d`
4. 访问：http://localhost
