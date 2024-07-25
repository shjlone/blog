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





## 绘制

**大致流程：**

第一次绘制时，从上到下递归绘制子节点，每当遇到一个边界节点，判断如果该节点的layer属性是否为空，是就创建一个新的OffsetLayer并赋值给它；不是则使用。然后将layer传递给子节点，接下来：

1. 如果子节点是非边界节点，且需要绘制，则：
    - 第一次绘制：创建一个Canvas对象和一个PictureLayer，然后将它们绑定，后续调用Canvas绘制都会落到和其绑定的PictureLayer上，接着这个PictureLayer会加入到边界节点的layer中；
    - 不是第一次绘制：复用已有的边界节点和Canvas对象；
2. 如果子节点是边界节点，则对子节点递归上述过程。当子树递归完成后，就要将子节点的layer添加到父级layer中。

RenderObject调用markNeedsRepaint来发起重绘：

1. 从当前节点一直往父级查找，直到找到一个**绘制边界点**时终止查找，然后会将该绘制边界点添加到其PiplineOwner的_nodesNeedingPaint列表中。
2. 在查找的过程中，会将自己到绘制边界点路径上所有节点的_needPaint属性设置为true，表示需要重绘。
3. 请求新的frame，执行重绘流程。下一个frame就会走drawFrame流程，涉及到flushCompositingBits、flushPaint 和 compositeFrame 三个函数。

```dart

void markNeedsPaint() {
  if (_needsPaint) return;
  _needsPaint = true;
  if (isRepaintBoundary) { // 如果是当前节点是边界节点
      owner!._nodesNeedingPaint.add(this); //将当前节点添加到需要重新绘制的列表中。
      owner!.requestVisualUpdate(); // 请求新的frame，该方法最终会调用scheduleFrame()
  } else if (parent is RenderObject) { // 若不是边界节点且存在父节点
    final RenderObject parent = this.parent! as RenderObject;
    parent.markNeedsPaint(); // 递归调用父节点的markNeedsPaint
  } else {
    // 如果是根节点，直接请求新的 frame 即可
    if (owner != null)
      owner!.requestVisualUpdate();
  }
}


```