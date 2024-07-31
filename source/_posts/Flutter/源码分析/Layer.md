---
title: Layer
toc: true
tags: Flutter
---


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