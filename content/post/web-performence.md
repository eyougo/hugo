---
title: "三种方式分析监测web前端性能"
date: 2021-07-20T14:18:22+08:00
tags:
 - Web
 - Performance
---
> 原文地址：<https://juejin.cn/post/6981315563104501790>
>
> 原作者：[这些年你跑哪去了](https://juejin.cn/user/2735240658560408)

从Performance、Performance API、LightHouse三个方向来分析前端性能。

### 1 Performance
> 运行时性能表现（runtime performance）指的是当你的页面在浏览器运行时的性能表现，而不是在下载页面的时候的表现。用 Chrome DevToos Performance 功能去分析运行时性能表现。
<!--more-->

#### 1.1 面板初始化简介

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/789a98bef13e4041b545ab316f922584~tplv-k3u1fbpfcp-watermark.image)

#### 1.2 demo 简介

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/269a351816cb4c7ba49a96885d90cdca~tplv-k3u1fbpfcp-watermark.image)

#### 1.3 FPS图表 - Frames Per Seconds

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/477f08b837b6458aad3c554a2078e620~tplv-k3u1fbpfcp-watermark.image)

> 分析每一秒的帧，FPS（frames per second）是用来分析动画的一个主要性能指标。让页面效果能够达到>=60fps(帧)/s的刷新频率以避免出现卡顿。

> 为什么是60fps? 目前大多数显示器的刷新率相吻合(60Hz)。如果网页动画能够做到每秒60帧，就会跟显示器同步刷新，达到最佳的视觉效果。60 HZ/s，每个帧的工作（从 JS 到绘制）完成时间小于 16ms,达到人眼顺滑（例如滚动 拖动）。

> 观察FPS图表，如果你发现了一个红色的长条，那么就说明这些帧存在严重问题（有掉帧情况），有可能导致非常差的用户体验。一般来说，绿色的长条越高，说明FPS越高，用户体验越好。

#### 1.4 Frames

![FBS-3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b8ca76678174cec82f0bbc3e6e16ac5~tplv-k3u1fbpfcp-watermark.image)

> more -> more tools -> Rendering ->Frame Rending Status 可看实时帧率。

> more -> more tools -> Rendering ->Paint flashing 页面上有哪些内容被重绘了。


![FBS-2.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02cf4adb8a67479a9cd6e75ec0c0d6de~tplv-k3u1fbpfcp-watermark.image)

> 点击三角箭头展开 Frames 区域，鼠标悬浮/点击绿色方块，可以看到该特定帧的帧率和渲染耗时。

#### 1.5 CPU

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b885ed80f5ad47bb9d3d7251943775e5~tplv-k3u1fbpfcp-watermark.image)


![CPU-3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b825d120dc344e85bdab55d495a522b7~tplv-k3u1fbpfcp-watermark.image)

顏色块的表示：
1. 蓝色为HTML文件
2. 黃色为脚本
3. 紫色为样式
4. 绿色为媒体文件
5. 灰色为其他資源

> 总结：当 CPU 长时间被占满，就是当前网页性能需要优化的信号。CPU 图表中，可以根据颜色填充的饱满程度，确定 CPU 的忙闲，进而了解该页面的总的任务量(鼠标悬停可见)。而 Summary 饼图则以一种直观的方式告诉了我们，哪个类型的任务最耗时。

#### 1.6 HEAP

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55cc8f619f38467190582c15c88476d6~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45e0d75db73b42ed8f1872fd7e2a2c62~tplv-k3u1fbpfcp-watermark.image)

> 在 HEAP 图表中可以看到 JS 内存占用情况，与下方的 memory 窗格中的JS Heap相对应。在 Memory 窗格还可以看到 Document 文档、Nodes DOM 节点、监听器、GPU 内存的习份内存统计。

#### 1.7 Network

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e4bee0bfe5742eb966587bdfcd60b0c~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/147b46af44ed4e0fbb3abad9eee5b496~tplv-k3u1fbpfcp-watermark.image)

> 鼠标悬停可以看到具体的网络请求以及获取请求的时间。网络请求时间过长会影响白屏时间。

#### 1.8 事件时间点

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44b2f60ed07642ceb4d7724890fe58f4~tplv-k3u1fbpfcp-watermark.image)

1. FP：First Paint(页面在导航后首次呈现出不同于导航前内容的时间点)；
2. FCP：First Contentful Paint(首次绘制任何文本，图像，非空白canvas或SVG的时间点)；
3. LCP：Largest Contenful Paint(页面开始加载到最大文本块内容或图片显示在页面中的时间)；
4. DCL：DOMContentLoaded Event(HTML加载完成事件)
5. L：OnLoad Event(页面所有资源加载完成事件)。

#### 1.9 Main

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eec6a4c05178425280110b963dc7176b~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d6ce7ede6d040d2a8ef9948f63e7405~tplv-k3u1fbpfcp-watermark.image)

1. 展开 Main 图表，Devtools 展示了主线程运行状况。X轴代表着时间。每个长条代表着一个 event。长条越长就代表这个 event花费的时间越长。Y轴代表了调用栈（call stack）。在栈里，上面的 event 调用了下面的event。
2. 在事件长条的右上角出，如果出现了红色小三角，说明这个事件是存在问题的，需要特别注意。
3. 双击这个带有红色小三角的的事件。在Summary 面板会看到详细信息。注意 reveal这个链接，双击它会让高亮触发这个事件的 event。如果点击代码提示链接，就会跳转到对应的代码处。

### 2 Performance API

> 在 performance 的 timing 属性中，我们可以查看到时间戳。在 performance 的 memory属性中可以查看浏览器内存情況。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d725e69d6690463ea5e64d0c1272d174~tplv-k3u1fbpfcp-watermark.image)

#### 2.1 这些时间戳与页面整个加载流程中的关键时间节点有着一一对应的关系

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/958afb38444a438e8e64f64fd938b8ad~tplv-k3u1fbpfcp-watermark.image)

#### 2.2 通过求两个时间点之间的差值，我们可以得出某个过程花费的时间

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a61a06f38a8448abbb9d72661417158~tplv-k3u1fbpfcp-watermark.image)

#### 2.3 关键性能指标：firstbyte、fpt、tti、ready 和 load 时间

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6520eb4520f644e18f7b2151b25ccd40~tplv-k3u1fbpfcp-watermark.image)

### 3 LightHouse

> Lighthouse 是一个开源的自动化工具，用于改进网络应用的质量。你可以将其作为一个 Chrome 扩展程序运行，或从命令行运行。为Lighthouse 提供一个需要审查的网址，它将针对此页面运行一连串的测试，然后生成一个有关页面性能的报告。

#### 3.1 指标
> 页面性能、可访问性（无障碍）、最佳实践、SEO、PWA（渐进式 Web 应用） 五项指标的跑分。向下拉动 Report 页，我们还可以看到每一个指标的细化评。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e7f786747284b7fab106ea41144b103~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f668d56216984b40a080e5323bdf6f8e~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a101a585a6474bad807859331bf082aa~tplv-k3u1fbpfcp-watermark.image)

> 在 “Opportunities” 中，LightHouse 甚至针对我们的性能问题给出了可行的建议、以及每一项优化操作预期会帮我们节省的时间。











