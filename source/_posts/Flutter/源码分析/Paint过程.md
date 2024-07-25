---
title: Paint过程
toc: true
tags: Flutter
---


PipelineOwner是『RenderObject Tree』与『RendererBinding』间的桥梁，在两者间起到沟通协调的作用


```dart

void drawFrame() {
  pipelineOwner.flushLayout();
  pipelineOwner.flushCompositingBits();
  pipelineOwner.flushPaint();
  renderView.compositeFrame(); // this sends the bits to the GPU
  pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
}


```



- Root Widget(RenderObjectToWidgetAdapter)
- 『 Element Tree 』的根节点(RenderObjectToWidgetElement)
- 『 RenderObject Tree 』的根节点(RenderView)
- 『 Layer Tree 』的根节点(TransformLayer)




1. 确定当前组件的布局边界。

2. 判断是否需要重新布局，如果没必要会直接返回，反之才需要重新布局。不需要布局时需要同时满足三个条件：

    - 当前组件没有被标记为需要重新布局。

    - 父组件传递的约束没有发生变化。

    - 当前组件的布局边界也没有发生变化时。

3. 调用 performLayout() 进行布局，因为 performLayout() 中又会调用子组件的 layout 方法，所以这时一个递归的过程，递归结束后整个组件树的布局也就完成了。

4. 请求重绘。
