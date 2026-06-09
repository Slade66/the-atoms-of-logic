---
created: 2026-03-29T14:17:09+08:00
---

# Network

- **按耗时排序：** 慢在哪个请求？点击 Time 列按耗时从大到小排序，立刻看到最慢的请求是谁。

- **看 Waterfall：** 横向时间轴，是否串行请求（A 请求结束 → B 才开始），前端写法有问题，可以并发优化。

- **Initiator：** 是谁发起了这个请求（哪个文件、哪行代码发起的）。

- **Search（全局搜索）：** 可以搜：URL、Header、Response 内容。

- **Disable cache：** 禁止本地缓存。

- **Preserve Log：** 保存日志，否则刷新页面就清空了。

- **过滤请求：** 按类型过滤请求。Fetch/XHR → 只看接口请求。

- **Block Request（模拟失败）：** 用来测试：某个接口挂了会怎样，是否有降级方案？

- **Replay XHR：** 重新发送请求。不刷新页面调试接口。

- **Copy：** 把请求复制到其它地方发送。
    
    - Copy as cURL (bash)
    
    - Copy as PowerShell

[[Network - Timing]]
