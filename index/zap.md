---
title: Zap 知识索引
description: Uber 高性能结构化日志库 Zap 的全部知识卡片索引，按主题分类串联。
date: 2026-02-27T16:24:19+08:00
tags: [Go, Zap]
---

## 入门基础

- [Zap 简介与安装](<../cards/Zap 简介与安装.md>)
- [三种开箱即用的 Logger 预设](<../cards/Zap 三种预设 Logger.md>)

## 核心 API

- [日志级别速查与使用规范](<../cards/Zap 日志级别速查.md>)
- [结构化字段 zap.Field 构造](<../cards/Zap 结构化字段 Field 构造.md>)
- [为什么要使用 SugaredLogger](<../cards/Zap Sugared Logger 的优势.md>)

## 高级用法

- [logger.With() 日志上下文绑定](<../cards/Zap With 日志上下文绑定.md>)
- [ReplaceGlobals 设置全局日志器](<../cards/Zap 全局日志器 ReplaceGlobals.md>)

## 底层机制

- [缓冲落盘与 Sync() 机制](<../cards/Zap 缓冲落盘与 Sync 机制.md>)
- [为什么 Zap 没有 Fatalf](<../cards/Zap 没有 Fatalf 的原因.md>)

## 配置与调试

- [WithCaller 与 AddStacktrace 配置](<../cards/Zap WithCaller 与 AddStacktrace 配置.md>)
- [AddCallerSkip 解决行号失真](<../cards/Zap AddCallerSkip 解决行号失真.md>)
