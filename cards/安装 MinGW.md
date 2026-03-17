---
created: 2026-03-17T15:11:00+08:00
---

# 安装 MinGW

- **MinGW 是什么？** MinGW 是 Windows 上的 GCC 编译器，用于 编译 C / C++。

- 现在推荐用 MinGW-w64，支持 64 位，而不是老的 MinGW。

### 用 MSYS2 安装 MinGW

- **MSYS2 是什么？** MSYS2 是一个 Windows 上的「类 Linux 终端环境 + 包管理器 `pacman`」，里面可以安装 MinGW。

- **安装步骤：**
 
  1. 打开官网：`https://www.msys2.org/`。
  
  2. 下载并安装 `msys2-x86_64-20251213.exe`。
  
  3. 从开始菜单打开 `MSYS2 UCRT64` 终端。
  
  4. 执行安装命令：`pacman -S mingw-w64-ucrt-x86_64-gcc`。
  
  5. 在 `MSYS2 UCRT64` 终端中执行 `gcc --version`。如果能看到版本号，说明 `gcc` 已安装成功。
  
  6. 配置 Windows 的 `Path` 环境变量（可选）：如果你希望在普通 PowerShell 中也能找到 `gcc`，请把 `A:\msys64\ucrt64\bin` 加入系统环境变量 `Path`。配置完成后，重新打开一个新的 PowerShell，再次执行 `gcc --version`。如果也能看到版本号，说明 `gcc` 已可在 Windows 全局环境中使用。

### MSYS2 的三种环境

MSYS2 中这三种环境的本质区别在于：你编译出来的程序最终依赖哪套运行时。

| 环境 | 用途 | 推荐程度 |
| --- | --- | --- |
| MSYS | 模拟 Linux 环境，不适合编译 Windows 程序。 | ❌ 基本不用 |
| MINGW64 | 编译可直接在 Windows 运行的程序（旧运行时）。 | ✅ 最常用 |
| UCRT64 | 基于 UCRT（Universal C Runtime）的新环境，可视为 MINGW64 的升级版。 | ✅ 推荐 |
