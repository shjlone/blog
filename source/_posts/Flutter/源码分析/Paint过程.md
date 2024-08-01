---
title: Paint过程
toc: true
tags: Flutter
---





经过Build流程，Render Tree中绘制相关的基础信息已经完成更新；经过Layout流程，Render Tree中每个节点的大小和位置完成计算与存储，接下来进入Paint流程：基于Layout的信息生成绘制指令。


![](./paint.png)

Render Tree和Layer Tree的对应关系

使用Layer Tree的好处是可以做Paint流程的局部更新。Render Tree中，每个RenderObject对象都拥有一个needsCompositing属性，用于判断自身及子节点是否有一个要去合成的图层，同是还有一个_needsCompositingBitsUpdate字段
用于标记该属性是否需要更新。Flutter在Paint开始前首先会完成needsCompositing属性的更新，然后开始正式绘制。

![](./layer1.png)



Layer的子类分为以下几种类型：

- ContainerLayer：容器层，用于包含其他Layer。
- PictureLayer：执行实际绘制的节点。通过_picture字段持有一个ui.PictureRecorder对象，用于Engine进行对应绘制指令的记录。
- TextureLayer、PlatformLayer：渲染源将有外部提供


## Compositing-State Mark阶段


当Render Tree需要挂载（mount）或卸载（unmount）一个子节点时，就会调用markNeedsCompositingBitsUpdate方法

```dart

class RenderObject {
  void markNeedsCompositingBitsUpdate() {
    if (_needsCompositingBitsUpdate) {//已经标记过需要更新
      return;
    }
    _needsCompositingBitsUpdate = true;
    if (parent is RenderObject) {//处理父节点
      final RenderObject parent = this.parent! as RenderObject;
      if (parent._needsCompositingBitsUpdate) {//如果父节点标记过，直接返回
        return;
      }

      if ((!_wasRepaintBoundary || !isRepaintBoundary) && !parent.isRepaintBoundary) {//非绘制边界才需要标记父节点
        parent.markNeedsCompositingBitsUpdate();
        return;
      }
    }
    if (owner != null) {
      owner!._nodesNeedingCompositingBitsUpdate.add(this);
    }
  }
}

```

## Compositing-State Flush阶段

Layout完成之后将调用pipelineOwner.flushCompositingBits()

```dart

class PipelineOwner {
    void flushCompositingBits() {
    if (!kReleaseMode) {
      Timeline.startSync('UPDATING COMPOSITING BITS');
    }
    _nodesNeedingCompositingBitsUpdate.sort((RenderObject a, RenderObject b) => a.depth - b.depth);//优先遍历祖先节点
    for (final RenderObject node in _nodesNeedingCompositingBitsUpdate) {
      if (node._needsCompositingBitsUpdate && node.owner == this) {
        node._updateCompositingBits();
      }
    }
    _nodesNeedingCompositingBitsUpdate.clear();
    for (final PipelineOwner child in _children) {
      child.flushCompositingBits();
    }
    if (!kReleaseMode) {
      Timeline.finishSync();
    }
  }
}


class RenderObject {
  void _updateCompositingBits() {
    if (!_needsCompositingBitsUpdate) {//第1步，无更新直接返回
      return;
    }
    final bool oldNeedsCompositing = _needsCompositing;
    _needsCompositing = false;//默认不需要合成，即单独使用一个图层
    visitChildren((RenderObject child) {//第2步，遍历每个子节点
      child._updateCompositingBits();
      if (child.needsCompositing) {//如果子节点需要合成，则父节点也需要，直到遇到绘制边界
        _needsCompositing = true;
      }
    });
    if (isRepaintBoundary || alwaysNeedsCompositing) {//第3步，判断是否需要合成，即是否是一个独立图层
      _needsCompositing = true;
    }
    if (!isRepaintBoundary && _wasRepaintBoundary) {//第4步，
      _needsPaint = false;
      _needsCompositedLayerUpdate = false;
      owner?._nodesNeedingPaint.remove(this);
      _needsCompositingBitsUpdate = false;
      markNeedsPaint();
    } else if (oldNeedsCompositing != _needsCompositing) {//判断_needsCompositing是否发生变化
      _needsCompositingBitsUpdate = false;
      markNeedsPaint();//图层发生变化，需要重绘
    } else {
      _needsCompositingBitsUpdate = false;
    }
  }
}

```


## Paint Mark阶段

Paint和Layout的脏节点标记逻辑比较类似，RenderObject中对绘制有影响的属性更新了就会进行标记，比如RenderImage的image属性

```dart
  set image(ui.Image? value) {
    if (value == _image) {//没有改变
      return;
    }
    if (value != null && _image != null && value.isCloneOf(_image!)) {
      value.dispose();
      return;
    }
    _image?.dispose();
    _image = value;
    markNeedsPaint();
    if (_width == null || _height == null) {
      markNeedsLayout();
    }
  }
```


markNeedsPaint方法的逻辑如下：

```dart
class RenderObject {
    void markNeedsPaint() {
    if (_needsPaint) {
      return;
    }
    _needsPaint = true;
    if (isRepaintBoundary && _wasRepaintBoundary) {//第一种情况，如果是绘制边界
      if (owner != null) {
        owner!._nodesNeedingPaint.add(this);//重新加入到需要绘制的列表中
        owner!.requestVisualUpdate();//请求渲染
      }
    } else if (parent is RenderObject) {//第二种情况，父节点不是绘制边界
      final RenderObject parent = this.parent! as RenderObject;
      parent.markNeedsPaint();//父节点也受影响，直接向上标记
    } else {
      if (owner != null) {//第三种情况，非RenderObject
        owner!.requestVisualUpdate();
      }
    }
  }
}

```

## Paint Flush阶段


当flushCompositingBits完成之后，会调用pipelineOwner.flushPaint()

```dart

class PipelineOwner {
  void flushPaint() {
    try {
      final List<RenderObject> dirtyNodes = _nodesNeedingPaint;
      _nodesNeedingPaint = <RenderObject>[];

      // Sort the dirty nodes in reverse order (deepest first).
      for (final RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {//从深到浅遍历
        if ((node._needsPaint || node._needsCompositedLayerUpdate) && node.owner == this) {
          if (node._layerHandle.layer!.attached) {
            if (node._needsPaint) {
              PaintingContext.repaintCompositedChild(node);
            } else {
              PaintingContext.updateLayerProperties(node);
            }
          } else {
            node._skippedPaintingOnLayer();
          }
        }
      }
      for (final PipelineOwner child in _children) {
        child.flushPaint();
      }
    }
  }
}


class PaintingContext {
  static void repaintCompositedChild(RenderObject child, { bool debugAlsoPaintedParent = false }) {
    _repaintCompositedChild(
      child,
      debugAlsoPaintedParent: debugAlsoPaintedParent,
    );
  }

  static void _repaintCompositedChild(
    RenderObject child, {
    bool debugAlsoPaintedParent = false,
    PaintingContext? childContext,
  }) {
    OffsetLayer? childLayer = child._layerHandle.layer as OffsetLayer?;
    if (childLayer == null) {
      final OffsetLayer layer = child.updateCompositedLayer(oldLayer: null);//没有就创建一个OffsetLayer
      child._layerHandle.layer = childLayer = layer;
    } else {
      childLayer.removeAllChildren();//移除子节点
      final OffsetLayer updatedLayer = child.updateCompositedLayer(oldLayer: childLayer);
    }
    child._needsCompositedLayerUpdate = false;
    childContext ??= PaintingContext(childLayer, child.paintBounds);
    child._paintWithContext(childContext, Offset.zero);//绘制当前图层
    childContext.stopRecordingIfNeeded();
  }

}


class RenderObject {
  void _paintWithContext(PaintingContext context, Offset offset) {
    if (_needsLayout) {//存在Layout未处理的节点
      return;
    }
    RenderObject? debugLastActivePaint;
    _needsPaint = false;
    _needsCompositedLayerUpdate = false;
    _wasRepaintBoundary = isRepaintBoundary;
    try {
      paint(context, offset);//开始绘制，具体的绘制逻辑在子类中
    } catch (e, stack) {
      _reportException('paint', e, stack);
    }
  }
}


```










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



## 参考

- [Flutter内核源码剖析]()