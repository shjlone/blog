---
title: PipelineOwner
toc: true
tags: Flutter
---



在runApp时，RenderBinding创建PipelineOwner。PipelineOwner的作用：

- 不断收集Dirty Render Objects
- 驱动Rendering Pipeline刷新UI

![](./PipelineOwner.PNG)

### Dirty RenderObjects

Render Object 有4种`Dirty State`需要PipelineOwner维护：

- Needing Layout：Render Object需要重新layout
- Needing Compositing Bits Update：Render Object合成标志位有变化
- Needing Paint：Render Object需要重新绘制
- Needing Semantice：Render Object辅助信息有变化



## 驱动

### Dirty RenderObjects

Render Object有4种Dirty State需要PipelineOwner维护：

- Needing Layout：Render Object需要重新layout
- Needing Compositing Bits Update：Render Object合成标志位有变化
- Needing Paint：Render Object需要重新绘制
- Needing Semantice：Render Object辅助信息有变化

![](./RenderObject_PipelineOwner_RendererBinding.png)


- 当 RenderObject 需要重新 layout 时，调用markNeedsLayout方法，该方法会将当前 RenderObject 加入 PipelineOwner#_nodesNeedingLayout或传给父节点去处理；
- 当 RenderObject 的 Compositing Bits 有变化时，调用markNeedsCompositingBitsUpdate方法，该方法会将当前 RenderObject 加入 PipelineOwner#_nodesNeedingCompositingBitsUpdate或传给父节点去处理；
- 当 RenderObject 需要重新 paint 时，调用markNeedsPaint方法，该方法会将当前 RenderObject 加入PipelineOwner#_nodesNeedingPaint或传给父节点处理；
- 当 RenderObject 的辅助信息(Semantics)有变化时，调用markNeedsSemanticsUpdate方法，该方法会将当前 RenderObject 加入 PipelineOwner#_nodesNeedingSemantics或传给父节点去处理

### Request Visual Update

上述4个markNeeds*方法，除了markNeedsCompositingBitsUpdate，其他方法最后都会调用PipelineOwner#requestVisualUpdate。
之所以markNeedsCompositingBitsUpdate不会调用PipelineOwner#requestVisualUpdate，是因为其不会单独出现，一定是伴随其他3个之一一起出现的。


### Handle Draw Frame

![](./RendererBinding_handleDrawFrame.png)

```dart
class RenderBinding {
  //系统vsync触发帧刷新，PipelineOwner构建RenderObject树
  void drawFrame() {
    pipelineOwner.flushLayout();
    pipelineOwner.flushCompositingBits();
    pipelineOwner.flushPaint();
    renderView.compositeFrame(); // this sends the bits to the GPU
    pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
  }
}
```

#### Flush Layout

```dart
class PipelineOwner {
  void flushLayout() {
    while (_nodesNeedingLayout.isNotEmpty) {
      final List<RenderObject> dirtyNodes = _nodesNeedingLayout;
      _nodesNeedingLayout = <RenderObject>[];
      for (RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => a.depth - b.depth)) {
        if (node._needsLayout && node.owner == this)
          node._layoutWithoutResize();
      }
    } 
  }

  void flushCompositingBits() {
    _nodesNeedingCompositingBitsUpdate.sort((RenderObject a, RenderObject b) => a.depth - b.depth);
    for (final RenderObject node in _nodesNeedingCompositingBitsUpdate) {
      if (node._needsCompositingBitsUpdate && node.owner == this) {
        node._updateCompositingBits();
      }
    }
    _nodesNeedingCompositingBitsUpdate.clear();
    for (final PipelineOwner child in _children) {
      child.flushCompositingBits();
    }
  }

  void flushPaint() {
    final List<RenderObject> dirtyNodes = _nodesNeedingPaint;
      _nodesNeedingPaint = <RenderObject>[];
      for (final RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {
        assert(node._layerHandle.layer != null);
        if ((node._needsPaint || node._needsCompositedLayerUpdate) && node.owner == this) {
          if (node._layerHandle.layer!.attached) {
            assert(node.isRepaintBoundary);
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

```

![](./PaintingPipeline.png)


## 参考