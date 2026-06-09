---
created: 2026-03-29T16:58:53+08:00
---

# 如何在对方没有 GitHub 账号的前提下，实现仅允许他人拉取私有仓库而禁止推送的访问控制？

- **问题背景：**
    
    - 我将 GitHub 私有仓库打包成压缩包（删掉了 `.git` 文件夹）发给同学使用，对方解压后的项目代码并不是一个 Git 仓库。现在想把这份 “非 Git 仓库” 的本地代码接入 Git 并与远程仓库同步。

    - 我希望：
        - 他可以持续获取我仓库的更新（pull）
        - 但不能向仓库推送代码（push）
        - 并且不要求他注册 GitHub 账号（很麻烦）

- **解决方案：Deploy Key**

    - Deploy Key 是 GitHub 提供的仓库级 SSH Key，是给一台机器的“只读钥匙”。它不绑定用户账号，而是绑在单个仓库上，所以拉取端可以不注册 GitHub。可以设置成 read-only。

- **操作步骤：**
    
    1. **在对方电脑生成 SSH Key：** `ssh-keygen -t ed25519 -C "readonly"`
    
    2. **获取公钥：** `cat ~/.ssh/id_ed25519.pub`
    
    3. **在 GitHub 仓库里添加 deploy key：** 仓库 Settings → Deploy keys → Add deploy key，粘贴上一步生成的公钥，不要勾选 “Allow write access”。

- **旧代码接入 Git**

    - 对方本地代码来自压缩包，不是 Git 仓库，现在要接入远程仓库并同步。

    ```bash
    git init
    git remote add origin git@github.com:用户名/仓库.git
    git fetch origin # 下载远程更新
    # 情况 A：不保留本地代码
    # 适合：
    #   本地代码只是旧版本
    #   不需要保留修改
    git reset --hard origin/main # 把本地仓库直接变成远程的内容，未跟踪文件会被覆盖
    # 情况 B：保留本地代码
    git add .
    git commit -m "local backup"
    git merge origin/main --allow-unrelated-histories
    ```
