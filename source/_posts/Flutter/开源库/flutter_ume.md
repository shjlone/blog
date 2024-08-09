---
title: flutter_ume
toc: true
tags: Flutter
---

字节出品的开发调试工具，具备以下功能：

- 查看具体Widget的信息（WidgetInfo）
- 查看Widget树形结构，点击查询RenderObject树（WidgetDetail）
- 测量尺寸（AlignRuler）
- 展示某个位置的颜色
- Dio网络监听（DioInspector）
- 性能面板展示（PerOverlay）
- 设备信息（DeviceInfo）
- 展示具体代码（ShowCode）
- 获取某个位置的颜色（ColorSucker）
- 内存信息展示（MemoryInfo）
- CPU信息展示（CPUInfo）
- 用于录制时显示手势操作
- 监听debugPrint，输出到Console
- 平台消息的监听（ChannelMonitor）

代码中熟练使用了Flutter的渲染机制，自定义Layer进行相关绘制，源码值得阅读。其支持自定义插件，可以添加一些属于自己项目的插件，比如（环境切换、设置代理等）

## 参考

- [https://github.com/bytedance/flutter_ume](https://github.com/bytedance/flutter_ume)
