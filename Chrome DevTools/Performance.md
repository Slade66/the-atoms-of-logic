---
created: 2026-03-29T16:51:00+08:00
---

# Performance

- **作用：** 用来分析页面卡（JS 执行慢）的原因。

- **使用方法：**

    1. 打开 Performance
    2. 点击录制（Record）
    3. 操作页面
    4. 停止

- 按 Save trace 把录制的性能追踪保存下来。

- 按 Load trace 加载之前保存的性能追踪。

- **长任务（Long Task）：** 超过 50ms 的任务就是 Long Task。黄色块很宽，上面有红色标记。

- **INP：** 用户点击 → 页面响应（页面产生视觉反馈）的总时间。
    - Input delay（输入延迟）：用户点击 → JS开始执行，如果这个高 → 说明主线程被占用（别的 JS 在跑）
    - Processing duration（处理时间）：JS执行时间（事件回调处理函数执行时间），你的代码慢
    - Presentation delay（渲染延迟）：JS执行完 → 页面渲染出来，如果这个高 → 是渲染慢（DOM/CSS问题）

- 找具体是哪段代码卡
1. Call Tree / Bottom-up 只显示“选中时间范围”的数据，在时间轴，拖动框选中那段黄色块
2. 再看 Bottom-up / Call Tree

- 黄色表示 JavaScript 执行

- Bottom-up（找“最卡函数”）：用来找“哪个函数耗时最多”，按 Total time 排序

- Call Tree（看调用链）：Call Tree 展示函数调用关系（谁调用谁）。
    - Self time：函数自身耗时（最重要）
    - Total time：包含子调用
    - 点击 Activity，看右侧的函数 → 跳源码 
