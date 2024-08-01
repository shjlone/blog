---
title: RepaintBoundary
toc: true
tags: Flutter
---


UI系统会进行界面的图层划分，可以进行图层复用，减少绘制量，提升绘制性能。而RepaintBoundary是开放给开发者使用的独立图层。

RenderObject有一个isRepaintBoundary属性，该属性决定这个RenderObject重绘时是否独立于其父元素，如果该属性为true，则独立绘制。

如何独立绘制呢？

```dart

/// PainterContext
void paintChild(RenderObject child, Offset offset) {
    assert(() {
        debugOnProfilePaint?.call(child);
        return true;
    }());

    if (child.isRepaintBoundary) {
        stopRecordingIfNeeded();
        _compositeChild(child, offset);
    } else {
        child._paintWithContext(this, offset);
    }
}

  void _compositeChild(RenderObject child, Offset offset) {
    assert(!_isRecording);
    assert(child.isRepaintBoundary);
    assert(_canvas == null || _canvas!.getSaveCount() == 1);

    // Create a layer for our child, and paint the child into it.
    // 给子节点创建一个layer，然后在上面绘制
    if (child._needsPaint) {
      repaintCompositedChild(child, debugAlsoPaintedParent: true);
    } else {
      assert(() {
        // register the call for RepaintBoundary metrics
        child.debugRegisterRepaintBoundaryPaint();
        child._layerHandle.layer!.debugCreator = child.debugCreator ?? child;
        return true;
      }());
    }
    assert(child._layerHandle.layer is OffsetLayer);
    final OffsetLayer childOffsetLayer = child._layerHandle.layer! as OffsetLayer;
    childOffsetLayer.offset = offset;
    appendLayer(childOffsetLayer);
  }

  void markNeedsPaint() {
    assert(!_debugDisposed);
    assert(owner == null || !owner!.debugDoingPaint);
    if (_needsPaint)
      return;
    _needsPaint = true;
    //如果RenderObject.isRepaintBoundary 为true,则该RenderObject拥有layer，直接绘制
    if (isRepaintBoundary) {
      assert(() {
        if (debugPrintMarkNeedsPaintStacks)
          debugPrintStack(label: 'markNeedsPaint() called for $this');
        return true;
      }());
      // If we always have our own layer, then we can just repaint
      // ourselves without involving any other nodes.
      assert(_layerHandle.layer is OffsetLayer);
      if (owner != null) {
          //找到最近的layer，绘制
        owner!._nodesNeedingPaint.add(this);
        owner!.requestVisualUpdate();
      }
    } else if (parent is RenderObject) {
// 没有自己的layer, 会和一个祖先节点共用一个layer 
      final RenderObject parent = this.parent! as RenderObject;
      parent.markNeedsPaint();
      assert(parent == this.parent);
    } else {
      assert(() {
        if (debugPrintMarkNeedsPaintStacks)
          debugPrintStack(label: 'markNeedsPaint() called for $this (root of render tree)');
        return true;
      }());
      // If we're the root of the render tree (probably a RenderView),
      // then we have to paint ourselves, since nobody else can paint
      // us. We don't add ourselves to _nodesNeedingPaint in this
      // case, because the root is always told to paint regardless.
      if (owner != null)
      // 如果直到根节点也没找到一个Layer，那么便需要绘制自身，因为没有其它节点可以绘制根节点。
        owner!.requestVisualUpdate();
    }
  }
```


而RepaintBoundary就是一个isRepaintBoundary为true的类。



## 使用场景

- 自定义paint的组件
- 会唤起输入框的界面


## 参考

- [Flutter - Using RepaintBoundary Examples](https://www.woolha.com/tutorials/flutter-using-repaintboundary-examples)