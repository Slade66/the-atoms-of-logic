---
created: 2026-03-15T18:08:10+08:00
---

# 在 AWS 上部署 OpenClaw

## 操作步骤

### 起一台服务器

1. 登录 AWS
2. 进入 EC2 控制台
3. 切换到美国 弗吉尼亚北部 us-east-1 区域
4. 点击启动实例
5. 填写实例信息：
   1. 名称：OpenClaw
   2. 应用程序和操作系统映像选择 ubuntu server 24.04 LTS
   3. 实例类型选择 c7i-flex.large（重要！少于 4G 会在安装过程中报内存溢出错误！）
   4. 创建一个密钥对：密钥对类型选择 ED25519，私钥文件格式选择 .pem，点击「创建密钥对」，然后保存好下载的密钥对
   5. 密钥对选择刚刚创建的密钥对
   6. 防火墙（安全组）选择「创建安全组」，勾选「允许来自以下对象的 SSH 流量」「任何位置 0.0.0.0/0」
   7. 配置存储 填20G
6. 点击「启动实例」

### 配置 OpenClaw

1. **登录 EC2 实例：** 在密钥所在目录打开 PowerShell，输入 `ssh -i "OpenClaw.pem" ubuntu@ec2-100-26-196-241.compute-1.amazonaws.com`
2. **运行官方的一键安装脚本：**
   1. 执行 `curl -fsSL https://openclaw.ai/install.sh | bash`。
   2. 当出现：
      ```plain
      🦞 OpenClaw installed successfully (OpenClaw 2026.3.13 (61d171a))!
      The lobster has landed. Your terminal will never be the same.
      ```
      时，说明安装成功。
3. 配置 OpenClaw：
```plain
   I understand this is personal-by-default and shared/multi-user use requires lock-down. Continue?
│  Yes

   Onboarding mode
│  QuickStart

   Model/auth provider
│  Kilo Gateway

How do you want to provide this API key?
│  Paste API key now

Enter Kilo Gateway API key
...

Default model
│  kilocode/minimax/minimax-m2.5:free

Select channel (QuickStart)
│  Skip for now

 Search provider
│  Skip for now

 Configure skills now? (recommended)
│  No

Enable hooks?
│  📝 command-logger, 💾 session-memory


How do you want to hatch your bot?
│  Hatch in TUI (recommended)

我叫小泽，以后你要叫我主人！

```
4. 配置远程访问
   1. 查看网关状态：先执行 `bash`，再执行 `openclaw gateway status`
   ```bash
   🦞 OpenClaw 2026.3.13 (61d171a) — I run on caffeine, JSON5, and the audacity of "it worked on my machine."

    │
    ◇
    Service: systemd (enabled)
    File logs: /tmp/openclaw/openclaw-2026-03-15.log
    Command: /usr/bin/node /home/ubuntu/.npm-global/lib/node_modules/openclaw/dist/index.js gateway --port 18789
    Service file: ~/.config/systemd/user/openclaw-gateway.service
    Service env: OPENCLAW_GATEWAY_PORT=18789

    Config (cli): ~/.openclaw/openclaw.json
    Config (service): ~/.openclaw/openclaw.json

    Gateway: bind=loopback (127.0.0.1), port=18789 (service args)
    Probe target: ws://127.0.0.1:18789
    Dashboard: http://127.0.0.1:18789/
    Probe note: Loopback-only gateway; only local clients can connect.

    Runtime: running (pid 5405, state active, sub running, last exit 0, reason 0)
    RPC probe: ok

    Listening: 127.0.0.1:18789
    Troubles: run openclaw status
    Troubleshooting: https://docs.openclaw.ai/troubleshooting
   ```
   2. 访问界面：执行 `openclaw dashboard`
      ```
      🦞 OpenClaw 2026.3.13 (61d171a) — I can't fix your code taste, but I can fix your build and your backlog.

    Dashboard URL: http://127.0.0.1:18789/#token=e27a3cf5e9506779b719aa8154e8a4861639077ea8812ab4
    Copy to clipboard unavailable.
    No GUI detected. Open from your computer:
    ssh -N -L 18789:127.0.0.1:18789 ubuntu@172.31.89.191
    Then open:
    http://localhost:18789/
    http://localhost:18789/#token=e27a3cf5e9506779b719aa8154e8a4861639077ea8812ab4
    Docs:
    https://docs.openclaw.ai/gateway/remote
    https://docs.openclaw.ai/web/control-ui
      ```
   3. 在密钥所在目录执行： `ssh -i "OpenClaw.pem" -N -L 18789:127.0.0.1:18789 ubuntu@ec2-52-91-20-108.compute-1.amazonaws.com`，然后浏览器访问： `http://127.0.0.1:18789/#token=e27a3cf5e9506779b719aa8154e8a4861639077ea8812ab4`
       - -N 表示不执行远程命令，仅做端口转发。保持这个终端运行，关闭后隧道断开。

在 kilo 的 profile 里复制 API Key

## Q & A

### 内存不足

#### 现象

```plain
· Starting setup


<--- Last few GCs --->

[5275:0x3300a000]    43888 ms: Scavenge (interleaved) 943.5 (969.7) -> 942.7 (970.4) MB, pooled: 0 MB, 39.16 / 0.00 ms  (average mu = 0.489, current mu = 0.439) allocation failure;
[5275:0x3300a000]    43921 ms: Scavenge 944.3 (970.4) -> 943.9 (980.9) MB, pooled: 0 MB, 8.80 / 0.00 ms  (average mu = 0.489, current mu = 0.439) allocation failure;


<--- JS stacktrace --->

FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
----- Native stack trace -----

 1: 0xe42d60 node::OOMErrorHandler(char const*, v8::OOMDetails const&) [openclaw]
 2: 0x121ddd0 v8::Utils::ReportOOMFailure(v8::internal::Isolate*, char const*, v8::OOMDetails const&) [openclaw]
 3: 0x121e0a7 v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, v8::OOMDetails const&) [openclaw]
 4: 0x144ba05  [openclaw]
 5: 0x1465299 v8::internal::Heap::CollectGarbage(v8::internal::AllocationSpace, v8::internal::GarbageCollectionReason, v8::GCCallbackFlags) [openclaw]
 6: 0x1439948 v8::internal::HeapAllocator::AllocateRawWithLightRetrySlowPath(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [openclaw]
 7: 0x143a875 v8::internal::HeapAllocator::AllocateRawWithRetryOrFailSlowPath(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [openclaw]
 8: 0x141354e v8::internal::Factory::NewFillerObject(int, v8::internal::AllocationAlignment, v8::internal::AllocationType, v8::internal::AllocationOrigin) [openclaw]
 9: 0x18740bc v8::internal::Runtime_AllocateInYoungGeneration(int, unsigned long*, v8::internal::Isolate*) [openclaw]
10: 0x1dd17c9  [openclaw]
```

#### 解析

这是 Node.js 的 "JavaScript heap out of memory" 内存不足错误，OpenClaw 在安装脚本的 setup 步骤里跑的 Node 进程把可用内存吃满了，V8 垃圾回收多次后仍然分配失败，就直接崩溃了。

OpenClaw 安装流程涉及 Node 22 安装、构建工具安装、pnpm 安装和运行、依赖解析和构建等很多步骤；

OpenClaw 的安装脚本会启动一个名为 openclaw 的 Node 可执行文件，里面做依赖安装、项目初始化等重任务。

Node 默认可用的 JS 堆内存是有限的（和机器内存、Node 版本有关，一般 1–2GB 左右），当 setup 过程中需要的对象太多时，就会不停 GC 并最终报：FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory。

机器 / 虚拟机物理内存太小（例如 1GB RAM 的小 VPS），安装 OpenClaw 时非常容易 OOM。
在同一台机器上同时跑了其他吃内存的进程（浏览器、Docker 容器、大模型等），导致给 openclaw 剩下的可用内存不足。

我使用的是 2GB 内存的服务器，

解决方案：
1. 把安装环境的可用内存提升到至少 4GB。

