---
title: Layer
toc: true
tags: Flutter
---

Layer是Flutter中针对SceneBuilder的一些方法做的一个封装，每种Layer都对应了一个或多个SceneBuilder的方法。

Layer分类：

- 有孩子节点的Layer：
  - OffsetLayer/TransformLayer：位移类
  - OpacityLayer：透明度类
  - ClipRectLayer/ClipRRectLayer/ClipPathLayer：裁剪类
  - PhysicalModelLayer：
- 无孩子节点的Layer：
  - PictureLayer：绘制类，Flutter的组件基本都是通过这个Layer来绘制的
  - TextureLayer：纹理类，比如视频播放
  - PlatformViewLayer： 用于iOS上的PlatformView嵌入纹理

XXXLayer对应还有XXXEngineLayer，通过addToScene中创建。

RenderObject是渲染树中的一个节点，包含了布局和绘制逻辑。Layer是合成树的一个节点，他代表了GPU的一次绘制指令。Layer主要负责将渲染树中的绘制操作转换为GPU可以理解的指令。

RenderObject的paint方法中，会创建新的Layer或更新已有的Layer。这些Layer会被添加到合成树中，然后在后续的合成阶段，会被转换为GPU的绘制指令。

![](./Element_RenderObject_LayerTree.png)

![](./Layer.png)

```dart
abstract class Layer with DiagnosticableTreeMixin {

/// SceneBuilder.pushXXX创建
ui.EngineLayer? _engineLayer;

///用于记录该 Layer 自上次渲染后(addToScene)是否发生了变化
bool _needsAddToScene = true;

/// 用于将 layer 送入 engine 进行渲染
void addToScene(ui.SceneBuilder builder);
}
```

![](./LayerBuild.png)

## 参考

- [深入浅出 Flutter Framework 之 Layer](https://zxfcumtcs.github.io/2020/06/07/deepinto-flutter-layer/)
