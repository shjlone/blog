---
title: DevTools
toc: true
tags: Flutter
---

开发者工具介绍

- Flutter Inspector：检查 Flutter 应用程序的 UI 组件布局和状态
- Performance View：在 Flutter 应用程序中诊断 UI 性能过低的问题
- CPU Profiler View：Flutter 和 Dart 应用的 CPU 性能检测
- Network View：为 Flutter 应用进行网络性能检测
- 为 Flutter 或 Dart 应用进行源码级的调试
- Memory View：在 Flutter 或 Dart 命令行应用中测试内存问题
- Logging View：查看正在运行的 Flutter 或 Dart 的命令行应用程序相关的常规日志和诊断信息
- App Size Tool：分析代码和应用的大小

## Flutter inspector 工具

![](./inspector_screenshot.webp)

查看 widget 树，诊断布局问题

### Select Widget Mode

启动此按钮，可在应用中选中某个Widget进行查看。通过此工具可以快速定位UI的详细信息

### Slow Animations

以五分之一的速度运行动画以便对它们进行优化

### Show Guidelines 显示引导线

覆盖一层引导线以帮助调整布局问题

### Show Baselines 显示基线

针对文字对齐展示文字的基线。对检查文字是否对齐有帮助。

### Highlight Repaints 高亮重绘区域

该选项会为所有的 RenderBox 绘制一层边框，在它们重新绘制时改变颜色。重新绘制时在图层上依次显示不同的颜色。例如，一个小动画可能会导致整个页面一直在重绘。将动画使用RepaintBoundary widget嵌套，可以保证动画只会导致其本身重绘。

### Highlight Oversized Images 高亮尺寸过大的图片

在运行的应用程序中高亮并反转消耗过多内存的图像。

### 技巧

- 对于loading、toast这种动画UI，使用RepaintBoundary包裹
- 对于大尺寸的图片，使用cacheHeight、cacheWidth等属性进行优化

## 性能视图 (Performance view)

- Flutter 帧图表（仅 Flutter 应用）
- 帧分析标签页（仅 Flutter 应用）
- 光栅统计标签页（仅 Flutter 应用）
- 时间轴事件跟踪查看器（所有原生 Dart 应用）
- 高级调试工具（仅 Flutter 应用）

### Flutter 帧图表

### 光栅统计标签页 Raster Stats

如果帧的卡顿来自光栅线程，这个工具也许能够帮助你诊断性能缓慢的原因。生成光栅统计的步骤：

1. 在应用程序中导航到你看见光栅线程卡顿的画面。
2. 点击 Take Snapshot 生成快照。
3. 查看不同图层和它们各自的渲染时间。

![Raster Stats Tab](./raster-stats-tab.webp)

### 时间线事件表 Timeline Events

时间线事件图表显示了应用程序的所有事件追踪。 Flutter 底层框架在构建帧、绘制场景和跟踪其他活动（如 HTTP 请求时间和垃圾回收）时，会发出时间线事件。这些事件会在时间线中显示出来。你也可以使用 dart:developer Timeline 和 TimelineTask API 发送你自己的时间线事件

![](./timeline-events-tab.webp)

#### 技巧

- 时间线添加更多的信息

```dart
void main() {
  debugProfileBuildsEnabled = true;//timeline中会显示具体的widget构建时间
  debugProfilePaintsEnabled = true;//timeline中会显示具体的绘制时间
  runApp();
}

```

- 自定义信息

```dart
Timeline.startSync("Doing Something");
doSomething();
Timeline.finishSync();
```

### 增强的追踪选项 Enhance Tracing

![](./enhanced-tracing.webp)

你可以重复操作你想要追踪的行为来查看新的时间线事件，操作后可以在时间线中选择一个构建帧进行查看。

#### 追踪 widget 的构建 Track Widget Builds

想要在时间线中查看 build() 方法的事件，启用 Track Widget Builds 选项，时间线中将出现 widget 对应名称的事件。

![](./track-widget-builds.webp)

通过该视图，分析出哪些widget的build构建的次数异常，进而进行优化

#### 追踪布局 Track Layouts

想要在时间线中查看 RenderObject 布局构建的事件，启用 Track Layouts 选项：

![](./track-layouts.webp)

#### 追踪绘制 Track Paints

想要在时间线中查看 RenderObject 的绘制事件，启用 Track Paints 选项：

![](./track-paints.webp)

### 更多调试选项 More debugging options

想要诊断渲染图层相关的问题，请先关闭渲染层。下述的选项将会默认启动。

想要查看你的应用的性能影响，请尝试以相同的操作重现性能问题。在渲染层关闭的情况下，于构建帧图表里选择一个新的构建帧，查看它的时间线细节。如果光栅线程的时间消耗有显著降低，那么你禁用的效果的滥用可能是导致卡顿的主要原因。

#### 渲染裁剪的图层 Render Clip layers

禁用该选项来检查已使用的裁剪图层是否影响了性能。如果禁用后性能有显著提升，请尝试减少你的应用中裁剪效果的使用。

#### 渲染透明度图层 Render Opacity layers

禁用该选项来检查已使用的透明度图层是否影响了性能。如果禁用后性能有显著提升，请尝试减少你的应用中透明度效果的使用。

#### 渲染物理形状图层 Render Physical Shape layers

禁用该选项来检查已使用的物理形状图层是否影响了性能，例如阴影和背景特效。如果禁用后性能有显著提升，请尝试减少你的应用中物理效果的使用。

![](./more-debugging-options.webp)

## CPU探测视图 CPU profiler

单击“Record”开始记录 CPU 剖析。 当完成录制后，单击“Stop”。  此时，CPU 分析数据将从 VM 中提取并显示在分析器视图中（Call tree, Bottom up, Method table, and Flame chart）

### Bottom Up

此表提供了 CPU 配置文件的自下而上表示。 这意味着自下而上表中的每个顶级方法或根实际上是一个或多个 CPU 样本的调用堆栈中的顶级方法。 换句话说，自下而上的表中的每个顶级方法都是自上而下的表（调用树）的叶节点。 在此表中，可以展开方法以显示其调用者。

#### Total Time

#### Self Time

#### Method

### Call tree 调用树

自上而下的调用展示

### CPU Flame Chart CPU火焰图

![](./cpu-flame-chart.webp)

火焰图是一种可视化工具，用于显示方法调用的时间分布。 矩形的宽度表示方法的执行时间。 矩形的颜色表示方法的深度，即方法调用堆栈的深度。上面调用下面的方法。

### CPU sampling rate

## 内存视图 Memory view

![](./memory_chart_anatomy.webp)

### Root object, retaining path, and reachability

### Shallow size vs retained size

### Dart中的内存泄漏

## 技巧

- 小心闭包函数的使用
- 小心context的传递。如果闭包的生命周期在widget内，则可以传递
- 注意widget和state，state中不要引用widget中的context，widget是短生命的，而state是长生命的

Flutter Framework的每一层都提供了函数来输出当前状态或事件到控制台

- debugDumpApp：输出整个应用程序的状态
- debugDumpRenderTree：输出渲染树，用于分析布局问题
- debugDumpLayerTree：输出图层树，用于分析compositing问题

- debugPaintSize：设置为true时，每个box外都显示一个亮蓝色边界
- debugPaintBaselineEnabled： 文本的基线会高亮
- debugPaintPointersEnabled： 被点击的对象以蓝绿色显示，用于检查hit test是否正确
- debugPaintLayerBordersEnabled：橙色显示图层边界，用于检查是否需要使用RepaintBoundary
- debugPaintLayerBordersEnabled - 查看 layer 界线

- debugRepaintRainbowEnabled：每次重绘时，都会有不同的颜色，用于检查重绘区域

- debugProfileBuildsEnabled - 在观测台里显示构建树
- debugProfilePaintsEnabled

- debugPrintBuildScope：打印buildScope时的信息
- debugPrintScheduleBuildForStacks：打印调度构建堆栈
- debugPrintGlobalKeyedWidgetLifecycle：GlobalKey的生命周期信息
- debugPrintRebuildDirtyWidgets：打印重建的widget
  - 结合 debugPrintScheduleBuildForStacks，可以观察 widget 的 dirty/clean 生命周期
  - 结合 debugProfileBuildsEnabled，可以在 DevTools Timeline 中观察到详细事件信息
- debugPrintMarkNeedsLayoutStacks：打印markNeedsLayout方法调用时的堆栈信息
- debugPrintMarkNeedsPaintStacks：打印markNeedsPaint方法调用时的堆栈信息

### 添加

## 参考

- [Flutter 调试工具篇 | 壹 - 使用 Flutter Inspector 分析界面](https://juejin.cn/post/7260499321983893565)
- [性能视图](https://flutter.cn/docs/tools/devtools/performance)
- [https://flutter.cn/docs/tools/devtools/overview](https://flutter.cn/docs/tools/devtools/overview)
- [Flutter性能分析工具使用](https://blog.csdn.net/qq_41818873/article/details/130618157)
- [Mastering Dart & Flutter DevTools — Part 6: CPU Profiler View](https://medium.com/@fluttergems/mastering-dart-flutter-devtools-cpu-profiler-view-part-6-of-8-31e24eae6bf8)
- [Mastering Dart & Flutter DevTools — Part 7: Memory View](https://medium.com/@fluttergems/mastering-dart-flutter-devtools-part-7-memory-view-e7f5aaf07e15)
- [Mastering Dart & Flutter DevTools — Part 8: Performance View](https://medium.com/@fluttergems/mastering-dart-flutter-devtools-performance-view-part-8-of-8-4ae762f91230)
- [Flutter Performance 分析工具简介](https://www.sunmoonblog.com/2020/01/10/flutter-performance-tools/)
