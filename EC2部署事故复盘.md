# EC2 部署事故复盘

## 事件日志

起因：

为了将校园图书共享系统部署到 EC2，新增并执行了根目录下的 `deploy.sh`。脚本目标是自动完成代码拉取、后端 Spring Boot 构建、前端 Bun/Vite 构建，并启动后端 `16666` 端口与前端 `3000` 端口服务。

经过：

1. 首次远程执行部署脚本时，脚本在服务器上执行系统依赖安装、代码拉取和后端 Maven 构建。
2. 当时 EC2 规格较小，内存约 2GB，且没有 swap。
3. 服务器上同时运行 Docker、MySQL、Appsmith 等服务，其中 Appsmith 自身包含 Java、Node、MongoDB、Redis、Postgres 等进程。
4. 部署过程中，系统出现严重内存压力。内核日志显示：

```text
May 01 02:48:03 kernel: oom-kill
May 01 02:48:03 kernel: Out of memory: Killed process 14873 (java)
```

5. 同一时间，`apt-get` 与系统后台检查进程仍在运行，进一步加剧了内存压力。
6. SSH 服务随后进入异常状态，日志显示：

```text
May 01 02:48:54 systemd: Stopping ssh.service
May 01 02:54:40 ssh.service: start-pre operation timed out
May 01 03:00:57 Failed to start ssh.service
May 01 03:33:50 sshd: Server listening on 0.0.0.0 port 22
```

7. 外部表现为 EC2 可以 ping 通，端口 `22/3000/16666` 也能建立 TCP 连接，但 SSH 卡在：

```text
Connection timed out during banner exchange
```

8. HTTP 请求前端 `3000` 和后端 `16666` 也长时间无响应。
9. 通过 AWS 控制台重启实例后，SSH 恢复。
10. 登录服务器后，确认上一轮部署中断在构建阶段，服务器仓库仍停留在旧提交，后端未生成 jar，前端未生成 dist。
11. 随后在服务器上创建 2GB swap：

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

12. 拉取最新代码并重新执行修正版 `deploy.sh`。

结果：

1. 后端成功构建并运行在 `16666` 端口。
2. 前端成功使用 Bun 构建并通过 Vite Preview 运行在 `3000` 端口。
3. 验证结果：

```text
http://47.128.236.133:3000 -> HTTP 200
http://47.128.236.133:16666/api/announcements?page=1&pageSize=1 -> 正常返回 JSON
```

4. 当前服务进程：

```text
java -jar target/campus-book-sharing-system-0.0.1-SNAPSHOT.jar --server.port=16666
bun run preview -- --host 0.0.0.0 --port 3000
vite preview --host 0.0.0.0 --port 3000
```

5. 服务器已启用 2GB swap，后续短期部署具备基本兜底能力。

## 根因分析

本次问题的直接根因是 EC2 内存不足触发 OOM。服务器只有约 2GB 内存，没有 swap，同时承载了数据库、Appsmith、Docker 以及部署构建任务。Maven 首次构建需要下载依赖、编译 Java 项目，前端构建也需要 Node/Bun 运行时；这些任务叠加后超过了服务器的可用内存。

OOM 并没有直接杀死 SSH 主进程，而是杀掉了当时占用较高内存的 Java 进程。日志中被杀的 Java 进程位于 Docker/Appsmith 相关 cgroup 下。但系统整体已经处于高内存压力状态，导致 `sshd` 后续重启、启动预处理、认证握手都出现超时。因此用户看到的现象不是典型的 `Connection refused`，而是 `banner exchange timeout`：TCP 连接可以建立，但 SSH 服务不能及时返回协议 banner。

另一个重要诱因是首次部署脚本同时承担了“服务器初始化”和“应用发布”两类工作。初始化阶段执行了 `apt-get install`，这会触发系统包安装、后台检查、服务刷新等额外开销。早期脚本里还包含 `openssh-client`，实际安装过程中系统升级了 `openssh-server` 和 `openssh-sftp-server`，导致 SSH 服务被系统重载或重启。虽然没有执行过主动关闭 SSH 的命令，但在一个内存紧张的服务器上升级 SSH 相关包，放大了 SSH 会话中断风险。

更深层的问题是部署前缺少资源预检查和部署隔离。脚本没有先检查内存、swap、磁盘和当前运行服务，也没有判断服务器是否适合现场构建。对于 2GB 内存的小型 EC2 来说，数据库、Appsmith、后端构建、前端构建不应无保护地叠加运行。

因此，本次事故不是单一命令错误，而是以下因素叠加：

1. 小规格 EC2 内存不足。
2. 没有 swap。
3. Appsmith 和数据库已占用较多常驻内存。
4. 首次 Maven 构建资源开销较大。
5. 部署脚本混合了系统初始化和应用发布。
6. `apt-get install` 间接触发 SSH 相关服务变化。
7. 缺少部署前资源检查和失败保护。

## 肃正协议

以后部署必须遵守以下规则。

1. 初始化和部署分离

服务器初始化只能在明确维护窗口手动执行一次。应用部署脚本不得承担大范围系统初始化职责。

允许的初始化命令：

```bash
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y git curl unzip openjdk-21-jdk-headless
```

部署脚本只允许做：

```text
git pull / git reset
后端构建
前端构建
重启应用进程
```

2. 禁止在部署脚本中安装或升级 SSH 相关包

部署脚本中禁止出现：

```text
openssh-client
openssh-server
openssh-sftp-server
```

如确需处理 SSH，必须先确认 AWS Console、EC2 Instance Connect 或 Session Manager 可用。

3. 小内存服务器必须启用 swap

2GB 内存及以下实例必须至少启用 2GB swap：

```bash
free -h
swapon --show
```

如果没有 swap，不允许执行 Maven/前端构建。

4. 部署前必须检查资源

部署前执行：

```bash
free -h
df -h /
uptime
```

最低要求：

```text
available memory + free swap >= 1GB
根分区可用空间 >= 3GB
load average 不应长期高于 CPU 核数 2 倍
```

5. 构建必须限制内存

Maven 构建必须限制 JVM 内存：

```bash
MAVEN_OPTS="-Xmx512m" ./mvnw clean package -DskipTests
```

如服务器资源不足，应改为本地或 CI 构建产物，再上传到服务器运行。

6. 部署脚本不得升级核心系统组件

部署过程中不得安装或升级：

```text
openssh
systemd
linux kernel
libc
docker
mysql
```

这些操作必须作为单独维护任务执行。

7. SSH 异常时立即停止部署动作

如果出现：

```text
Connection timed out during banner exchange
```

不得继续重试部署。应按以下顺序恢复：

```text
AWS 控制台确认实例 Public IPv4
检查 Status checks 是否 2/2 passed
尝试 EC2 Instance Connect
必要时 Reboot instance
恢复后查看 journalctl 和内存状态
```

8. 恢复后先止血再部署

恢复 SSH 后先执行：

```bash
uptime
free -h
df -h /
journalctl -k -b -1 --no-pager | grep -Ei 'oom|out of memory|killed process'
pgrep -af 'mvn|java|bun|vite' || true
```

确认没有异常构建进程、内存稳定后，才能重新部署。

9. 部署完成必须验证

部署完成后必须验证：

```bash
curl -I http://47.128.236.133:3000
curl http://47.128.236.133:16666/api/announcements?page=1&pageSize=1
pgrep -af 'java -jar|vite.*preview|bun.*preview'
```

10. 后续改进方向

短期：

继续使用简单 `deploy.sh`，保留 swap，避免再次在部署脚本中触碰 SSH。

中期：

用 `systemd` 管理后端与前端服务，替代 `nohup`，实现崩溃自动拉起和开机自启。

长期：

绑定 Elastic IP，使用 CI 或本地构建产物，EC2 只负责拉取产物和重启服务，避免在小规格服务器上现场编译。
