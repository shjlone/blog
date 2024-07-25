---
title: RenderObject
toc: true
tags: Flutter
---


## 概念

RenderObject表示渲染树的一个对象，其职责包括：Layout、Paint、Hit Testing。


**作用：**

- 布局，从RenderBox开始，对RenderObject Tree从上至下进行布局。
- 绘制，通过Canvas对象，RenderObject可以绘制自身以及其在RenderObject Tree中的子节点。
- 点击测试，RenderObject从上至下传递点击事件，并通过其位置和behavior来控制是否响应点击事件。

插槽（slot）：所谓插槽，就是预留一个接口或位置，由其他对象来接入或占据。

RenderObject拥有一个parent和parentData插槽（slot）：

- parentData：负责存储父节点所需要的子节点的布局信息。该成员只能通过setupParentData方法赋值，RenderObject的子类通过重写该方法将ParentData的子类赋值给parentData，已扩展ParentData功能。
- layout()：布局阶段，父节点会调用子节点的该方法。
- markNeedsLayout()：标记下一个frame重新layout。
- paint()：绘制
- layer：
- isRepaintBoundary：绘制边界点，单独的一层渲染，提升性能
- needsCompositing：


![](./RenderObjectClassDiagram.png)


### RenderObject子类

- RenderView：Render Object Tree的根节点
- RenderBox：采用2D笛卡尔坐标系中的渲染对象。它实现了一个内在的尺寸调整协议，它允许您在没有完全铺设的情况下测量一个子级，以这样的方式，如果该子级改变了尺寸，父级将再次布置（考虑到子级的新尺寸）。若对坐标系统没有限制，可直接继承它来实现自定义RenderObject。size属性用来保存控件的高宽。其layout是通过在组件树中从上往下传递BoxConstraints对象实现的。
  - performResize()：测量
  - performLayout()：布局
- RenderView：渲染对象的根。它有单独的子级，它必须是一个RenderBox。因此，如果你想在渲染树中有一个自定义的RenderObject子类，你有两种选择：你可能需要替换RenderView本身，或者你需要一个RenderBox作为它的子类。
- RenderAbstractViewport：内部较大的渲染对象的界面。其渲染对象（如RenderViewport）显示其内容的一部分，可以通过ViewportOffset进行控制。
- RenderSliver：在视图中实现滚动效果的渲染对象的基类。RenderViewport有一组子Sliver，每个Sliver依次排列，覆盖过程中的视图。而RenderSliver则控制着Sliver的绘制渲染。

RenderObjectWithChildMixin为只有一个child的RenderObject提供child管理模型，ContainerRenderObjectMixin用于为多个child的RenderObject提供child管理模型。

![](./RenderObject_XMind.png)


### 核心函数比较

![](./RendObject_2.png)

|作用|Flutter RenderObject| Android View|
|---|---|---|
|绘制| paint()| draw()/onDraw()|
|布局| performLayout()/layout()| measure()/onMeasure(), layout()/onLayout()|
|布局约束| Constraints| MeasureSpec|
|布局协议1| performLayout() 的 Constraints 参数表示父节点对子节点的布局限制 |measure() 的两个参数表示父节点对子节点的布局限制|
|布局协议2| performLayout() 应调用各子节点的 layout()| onLayout() 应调用各子节点的 layout()|
|布局参数| parentData| mLayoutParams|
|请求布局| markNeedsLayout()| requestLayout()|
|请求绘制| markNeedsPaint()| invalidate()|
|添加| child adoptChild()| addView()|
|移除| child dropChild()| removeView()|
|关联到窗口/树| attach()| onAttachedToWindow()|
|从窗口/树取消关联| detach()| onDetachedFromWindow()|
|获取 parent| parent| getParent()|
|触摸事件| hitTest()| onTouch()|
|用户输入事件| handleEvent()| onKey()|
|旋转事件| rotate()| onConfigurationChanged()|

![](./renderobject_1.png)



## 创建

Widget中有对应的createRenderObject方法，mount时调用，用于创建Widget对应的RenderObject。

```dart
// RenderObjectElement
void mount(Element parent, dynamic newSlot) {
  super.mount(parent, newSlot);
  _renderObject = widget.createRenderObject(this);
  attachRenderObject(newSlot);
  _dirty = false;
}
```

## 布局

当RenderObject需要（重新）布局时调用markNeedLayout，从而被PipelineOwner收集，并在下一帧刷新时触发Layout操作

### markNeedsLayout调用场景

- Render Object 被添加到『 RenderObject Tree 』;
- 子节点 adopt、drop、move；
- 由子节点的markNeedsLayout方法传递调用；
- Render Object 自身与布局相关的属性发生变化，如RenderFlex的排版方向有变化时：


```dart
set direction(Axis value) {
  if (_direction != value) {
    _direction = value;
    markNeedsLayout();
  }
}
```

### Relayout Boundary

若某个 Render Object 的布局变化不会影响到其父节点的布局，则该 Render Object 就是『 Relayout Boundary 』。
Relayout Boundary 是一项重要的优化措施，可以避免不必要的 re-layout。
当某个 Render Object 是 Relayout Boundary 时，会切断 layout dirty 向父节点传播，即下一帧刷新时父节点无需 re-layout。

![](./RelayoutBoundary.png)


- 若RD节点出现 layout dirty，由于其自身、其父节点RA、RRoot都不是 Relayout Boundary，最终 layout dirty 传播到根节点RenderView，导致整颗『 RenderObject Tree 』重新布局；
- 若RF节点出现 layout dirty，由于其父节点RB为 Relayout Boundary，layout dirty 传播到RB即结束，最终需要重新布局的只有RB、RF两个节点；
- 若RG节点出现 layout dirty，由于其自身就是 Relayout Boundary，最终需要重新布局的只有RG自己。

那么，要成为Relayout Boundary，需要什么条件？

```dart
class RenderObject {
  void layout(Constraints constraints, { bool parentUsesSize = false }) {
    RenderObject relayoutBoundary;
    if (!parentUsesSize || sizedByParent || constraints.isTight || parent is! RenderObject) {
      relayoutBoundary = this;
    } 
    else {
      relayoutBoundary = (parent as RenderObject)._relayoutBoundary;
    }

    if (!_needsLayout && constraints == _constraints && relayoutBoundary == _relayoutBoundary) {
      return;
    }

    if (_relayoutBoundary != null && relayoutBoundary != _relayoutBoundary) {
      visitChildren(_cleanChildRelayoutBoundary);
    }
    _relayoutBoundary = relayoutBoundary;
  }
}
```

满足以下条件之一即可：

- parentUsesSize为false，即父节点在 layout 时不会使用当前节点的 size 信息(也就是当前节点的排版信息对父节点无影响)；
- sizedByParent为true，即当前节点的 size 完全由父节点的 constraints 决定，即若在两次 layout 中传递下来的 constraints 相同，则两次 layout 后当前节点的 size 也相同；
- 传给当前节点的 constraints 是紧凑型 (Tight)，其效果与sizedByParent为true是一样的，即当前节点的 layout 不会改变其 size，size 由 constraints 唯一确定；
- 父节点不是 RenderObject 类型(主要针对根节点，其父节点为nil)。


### markNeedsLayout

```dart

void markNeedsLayout() {
  if (_needsLayout) {
    return;
  }
  if (_relayoutBoundary != this) {
    markParentNeedsLayout();
  } 
  else {
    _needsLayout = true;
    if (owner != null) {
      owner._nodesNeedingLayout.add(this);
      owner.requestVisualUpdate();
    }
  }
}

```


- 若当前 Render Object 不是 Relayout Boundary，则 layout 请求向上传播给父节点(即 layout 范围扩大到父节点，这是一个递归过程，直到遇到 Relayout Boundary)；
- 若当前 Render Object 是 Relayout Boundary，则 layout 请求到该节点为此，不会传播到其父节点。


### layout

```dart
void layout(Constraints constraints, { bool parentUsesSize = false }) {
  RenderObject relayoutBoundary;
  if (!parentUsesSize || sizedByParent || constraints.isTight || parent is! RenderObject) {
    relayoutBoundary = this;
  } 
  else {
    relayoutBoundary = (parent as RenderObject)._relayoutBoundary;
  }

  if (!_needsLayout && constraints == _constraints && relayoutBoundary == _relayoutBoundary) {
    return;
  }
  _constraints = constraints;
  if (_relayoutBoundary != null && relayoutBoundary != _relayoutBoundary) {
    visitChildren(_cleanChildRelayoutBoundary);
  }
  _relayoutBoundary = relayoutBoundary;

  if (sizedByParent) {
    performResize();
  }

  performLayout();
  markNeedsSemanticsUpdate();
  
  _needsLayout = false;
  markNeedsPaint();
}
```

layout方法是触发Render Object更新布局信息的主要入口点。一般情况下，由父节点调用子节点的layout方法来更新其整体布局。
RenderObject的子类不应重写该方法，可按需重写performResize或/和performLayout方法。
当前 Render Object 的布局受到layout方法参数constraints的约束。

![](./LayputDataFlow.png)

如上图，『 Render Object Tree 』的 layout 是一次深度优先遍历的过程。
优先 layout 子节点，之后 layout 父节点。
父节点向子节点传递 layout constraints，子节点在 layout 时需遵守这些约束。
作为子节点 layout 的结果，父节点在 layout 时可以使用子节点的 size。

在上述layout代码第19~21行，若sizedByParent为true，则调用performResize来计算该 Render Object 的 size。

sizedByParent为true的 Render Object 需重写performResize方法，在该方法中仅根据constraints来计算 size。
如RenderBox中定义的performResize的默认行为：取constraints约束下的最小 size：

```dart
@override
void performResize() {
  // default behavior for subclasses that have sizedByParent = true
  size = constraints.smallest;
  assert(size.isFinite);
}
```

若父节点 layout 依赖子节点的 size，在调用layout方法时需将parentUsesSize参数设为true。
因为，在这种情况下若子节点 re-layout 导致其 size 发生变化，需要及时通知父节点，父节点也需要 re-layout (即 layout dirty 范围需要向上传播)。
这一切都是通过上节介绍过的 Relayout Boundary 来实现。

### performLayout

本质上，layout是一个模板方法，具体的布局工作由performLayout方法完成。
RenderObject#performLayout是一个抽象方法，子类需重写。

关于performLayout有几点需要注意：

- 该方法由layout方法调用，在需要 re-layout 时应调用layout方法，而不是performLayout；
- 若sizedByParent为true，则该方法不应改变当前 Render Object 的 size ( 其 size 由performResize方法计算)；
- 若sizedByParent为false，则该方法不仅要执行 layout 操作，还要计算当前 Render Object 的 size；
- 在该方法中，需对其所有子节点调用layout方法以执行所有子节点的 layout 操作，如果当前 Render Object 依赖子节点的布局信息，需将parentUsesSize参数设为true。

```dart
// RenderFlex
void performLayout() {
  RenderBox child = firstChild;
  while (child != null) {
    final FlexParentData childParentData = child.parentData;
    BoxConstraints innerConstraints = BoxConstraints(minHeight: constraints.maxHeight, maxHeight: constraints.maxHeight);
    child.layout(innerConstraints, parentUsesSize: true);
    child = childParentData.nextSibling;
  }
  
  size = constraints.constrain(Size(idealSize, crossSize));
  
  child = firstChild;
  while (child != null) {
    final FlexParentData childParentData = child.parentData;
    double childCrossPosition = crossSize / 2.0 - _getCrossSize(child) / 2.0;
    childParentData.offset = Offset(childMainPosition, childCrossPosition);
    child = childParentData.nextSibling;
  }
}
```


上述代码片段截取自RenderFlex，可以看到它大概做了3件事：

- 对所有子节点逐个调用layout方法；
- 计算当前 Render Object 的 size；
- 将与子节点布局有关的信息存储到相应子节点的parentData中。

> RenderFlex继承自RenderBox，是常用的Row、Column对应的 Render Object

## 绘制


### markNeedsPaint

```dart 
void markNeedsPaint() {
  if (isRepaintBoundary) {
    assert(_layer is OffsetLayer);
    if (owner != null) {
      owner._nodesNeedingPaint.add(this);
      owner.requestVisualUpdate();
    }
  } 
  else if (parent is RenderObject) {
    final RenderObject parent = this.parent;
    parent.markNeedsPaint();
  } 
  else {
    if (owner != null)
      owner.requestVisualUpdate();
  }
}
```

markNeedsPaint内部逻辑与markNeedsLayout都非常相似：

- 若当前 Render Object 是 Repaint Boundary，则将其添加到PipelineOwner#_nodesNeedingPaint中，Paint request 也随之结束；
- 否则，Paint request 向父节点传播，即需要 re-paint 的范围扩大到父节点(这是一个递归的过程)；
- 有一个特例，那就是『 Render Object Tree 』的根节点，即 RenderView，它的父节点为 nil，此时只需调用PipelineOwner#requestVisualUpdate即可。


### Repaint Boundary

Repaint Boundary 有以下特点：

- 每个 Repaint Boundary 都有一个独属于自己的 OffsetLayer (ContainerLayer)，其自身及子孙节点的绘制结果都将 attach 到以该 layer 为根节点的子树上；
- 每个 Repaint Boundary 都有一个独属于自己的 PaintingContext (包括背后的 Canvas)，从而使得其绘制与父节点完全隔离开。

![](./RepaintBoundary.png)

如上图，由于Root/RA/RC/RG/RI是 Repaint Boundary，所以它们都有对应的 OffsetLayer。
同时，由于每个 Repaint Boundary 都有属于自己的 PaintingContext，所以它们都有对应的 PictureLayer，用于呈现具体的绘制结果。
对于那些不是 Repaint Boundary 的节点，将会绘制到最近的 Repaint Boundary 祖先节点提供的 PictureLayer 上。

> Repaint Boundary 会影响兄弟节点的绘制，如由于RC是 Repaint Boundary，导致RB、RD被绘制到不同的 PictureLayer 上。

> 实现中，『 Layer Tree 』往往会比上图所示更复杂，由于每个 Render Object 在绘制过程中都可以自主引入更多的 layer。

### paint

```dart
void paint(PaintingContext context, Offset offset) { }
```

paint方法主要有2项任务：

- 当前 Render Object 本身的绘制，如：RenderImage，其paint方法主要职责就是 image 的渲染

```dart
void paint(PaintingContext context, Offset offset) {
  paintImage(
    canvas: context.canvas,
    rect: offset & size,
    image: _image,
    ...
  );
}
```

- 绘制子节点，如：RenderTable，其paint方法主要职责是依次对每个子节点调用PaintingContext#paintChild方法进行绘制：

```dart
void paint(PaintingContext context, Offset offset) {
  for (int index = 0; index < _children.length; index += 1) {
    final RenderBox child = _children[index];
    if (child != null) {
      final BoxParentData childParentData = child.parentData;
      context.paintChild(child, childParentData.offset + offset);
    }
  }
}
```

### 串起来

![](./RenderObject-Paint-Line.png)

#### PipelineOwner#flushPaint

当新一帧开始时，会触发PipelineOwner#flushPaint方法，进而对dirty Render Object进行re-paint

#### PaintingContext#repaintCompositedChild

作用：

1. 创建layer
2. 为RenderObject的绘制准备context并发起绘制流程


#### RenderObject#_paintWithContext

```dart
void _paintWithContext(PaintingContext context, Offset offset) {
  paint(context, offset);//子类实现
}
```

#### PaintingContext#paintChild

对于当前绘制子节点，若是 Repaint Boundary，则需要在独立的 layer 上进行绘制，否则直接调用子节点的_paintWithContext方法在当前上下文(paint context)中绘制：

```dart
void paintChild(RenderObject child, Offset offset) {
    if (child.isRepaintBoundary) {
      stopRecordingIfNeeded();
      _compositeChild(child, offset);
    } else if (child._wasRepaintBoundary) {
      child._layerHandle.layer = null;
      child._paintWithContext(this, offset);
    } else {
      child._paintWithContext(this, offset);
    }
  }

void stopRecordingIfNeeded() {
  if (!_isRecording)
    return;
  _currentLayer.picture = _recorder.endRecording();
  _currentLayer = null;
  _recorder = null;
  _canvas = null;
}
```


#### PaintingContext#_compositeChild

在_compositeChild中，通过repaintCompositedChild对子节点发起新一轮的绘制，并将绘制结果(child._layer)添加到『 Layer Tree 』中：

```dart
void _compositeChild(RenderObject child, Offset offset) {
  assert(!_isRecording);
  assert(child.isRepaintBoundary);
  assert(_canvas == null || _canvas.getSaveCount() == 1);

  repaintCompositedChild(child, debugAlsoPaintedParent: true);

  final OffsetLayer childOffsetLayer = child._layer;
  childOffsetLayer.offset = offset;
  appendLayer(child._layer);
}
```


## 跑起来

## 参考

- [深入浅出 Flutter Framework 之 RenderObject](https://zxfcumtcs.github.io/2021/03/27/deepinto-flutter-renderobject/)
- [Flutter实战-RenderObject（布局、绘制、点击测试）](https://book.flutterchina.club/chapter14/render_object.html)
