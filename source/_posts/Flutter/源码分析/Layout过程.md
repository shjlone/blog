---
title: Layout过程
toc: true
tags: Flutter
---




```dart
class RenderBinding {
  void drawFrame() {
    pipelineOwner.flushLayout();//布局
    pipelineOwner.flushCompositingBits();//更新所有节点，计算待绘制区域数据
    pipelineOwner.flushPaint();//绘制
    if (sendFramesToEngine) {
      renderView.compositeFrame(); // 发送数据到GPU线程
      pipelineOwner.flushSemantics(); // 更新语义化
      _firstFrameSent = true;
    }
  }
}

```

## 过程




### 标记阶段


markNeedsLayout方法会将当前节点标记为需要Layout。markNeedsLayout方法在很多地方都会被触发，比如UI的高宽发生变化，字体属性发生变化等。

```dart
class RenderObject {
 void markNeedsLayout() {
    if (_needsLayout) {//已经标记过
      return;
    }
    if (_relayoutBoundary == null) {//当前节点不是布局边界，父节点受此影响，也需要被标记
      _needsLayout = true;
      if (parent != null) {
        markParentNeedsLayout();
      }
      return;
    }
    if (_relayoutBoundary != this) {
      markParentNeedsLayout();
    } else {
      _needsLayout = true;
      if (owner != null) {
        owner!._nodesNeedingLayout.add(this);
        owner!.requestVisualUpdate();//请求刷新
      }
    }
  }

  void markParentNeedsLayout() {
    _needsLayout = true;
    final RenderObject parent = this.parent! as RenderObject;
    if (!_doingThisLayoutWithCallback) {
      parent.markNeedsLayout();
    } else {
    }
  }

}
```

### Flush阶段

Layout开始于drawFrame的flushLayout，Layout过程也跟Build类似，先标记再处理。Layout是相对Render Tree而言的

```dart
class PipelineOwner {
  void flushLayout() {
    try {
      while (_nodesNeedingLayout.isNotEmpty) {//存在需要更新Layout信息的节点
        final List<RenderObject> dirtyNodes = _nodesNeedingLayout;
        _nodesNeedingLayout = <RenderObject>[];
        dirtyNodes.sort((RenderObject a, RenderObject b) => a.depth - b.depth);
        for (int i = 0; i < dirtyNodes.length; i++) {
          if (_shouldMergeDirtyNodes) {
            _shouldMergeDirtyNodes = false;
            if (_nodesNeedingLayout.isNotEmpty) {
              _nodesNeedingLayout.addAll(dirtyNodes.getRange(i, dirtyNodes.length));
              break;
            }
          }
          final RenderObject node = dirtyNodes[i];
          if (node._needsLayout && node.owner == this) {
            node._layoutWithoutResize();//真正的Layout逻辑
          }
        }
        _shouldMergeDirtyNodes = false;
      }

      for (final PipelineOwner child in _children) {//更新子节点
        child.flushLayout();
      }
    } finally {
      _shouldMergeDirtyNodes = false;
    }
  }

}


class RenderObject {
  void _layoutWithoutResize() {
    RenderObject? debugPreviousActiveLayout;
    try {
      performLayout();//用于开始具体的Layout逻辑，这个方法中会调用markNeedsPaint进行标记需要进行paint
      markNeedsSemanticsUpdate();
    } catch (e, stack) {
      _reportException('performLayout', e, stack);
    }
    _needsLayout = false;
    markNeedsPaint();
  }
}

```


_nodesNeedingLayout是需要重新构建的列表，存储所有需要重新布局的节点，一般会通过markNeedsLayout将自己添加到待重新layout列表中。

总结：

drawFrame-->flushLayout-->performLayout-->markNeedsPaint


### 具体示例

```dart

class RenderView {
    void performLayout() {
    _size = configuration.size;
    if (child != null) {
      child!.layout(BoxConstraints.tight(_size));
    }
  }

}

class RenderObject {
  void layout(Constraints constraints, { bool parentUsesSize = false }) {
    if (!kReleaseMode && debugProfileLayoutsEnabled) {
      Map<String, String>? debugTimelineArguments;
    }
    final bool isRelayoutBoundary = !parentUsesSize || sizedByParent || constraints.isTight || parent is! RenderObject;
    final RenderObject relayoutBoundary = isRelayoutBoundary ? this : (parent! as RenderObject)._relayoutBoundary!;
    if (!_needsLayout && constraints == _constraints) {
      if (relayoutBoundary != _relayoutBoundary) {
        _relayoutBoundary = relayoutBoundary;
        visitChildren(_propagateRelayoutBoundaryToChild);
      }
      return;
    }
    _constraints = constraints;
    if (_relayoutBoundary != null && relayoutBoundary != _relayoutBoundary) {
      visitChildren(_cleanChildRelayoutBoundary);
    }
    _relayoutBoundary = relayoutBoundary;
    if (sizedByParent) {//子节点大小完全取决于父节点
      try {
        performResize();
      } catch (e, stack) {
      }
    }
    RenderObject? debugPreviousActiveLayout;
    try {
      performLayout();//子节点自身实现布局逻辑
      markNeedsSemanticsUpdate();
    } catch (e, stack) {
      _reportException('performLayout', e, stack);
    }
    _needsLayout = false;
    markNeedsPaint();//标记当前节点需要重绘
  }
}

```

为了性能考虑，layout过程会使用_relayoutBoundary来优化性能。








## 布局

### 布局过程

1. 父节点向子节点传递约束（constraints）信息，限制子节点的最大和最小宽高。
2. 子节点根据约束信息确定自己的大小。
3. 父节点根据特定布局规则确定每一个子节点在父节点布局控件中的位置，用偏移offset表示。
4. 递归整个过程，确定出每一个节点的大小和位置。

### layout流程

1. 确定当前组件的布局边界。
2. 判断是否需要重新布局，如果没必要会直接返回，反之才需要重新布局。不需要布局要同时满足以下三个条件：
    1. 当前组件没有被标记为需要重新布局。
    2. 父组件传递的约束没有发生变化。
    3. 当前组件的布局边界没有发生变化。
3. 调用performLayout进行布局，其内部会调用子组件的layout方法。
4. 请求绘制。

### performLayout流程

1. 如果有子组件，则对子组件进行递归布局。
2. 确定当前组件的大小，通常会依赖子组件的大小。
3. 确定子组件在当前组件中的起始偏移。

```dart
 
void layout(Constraints constraints, { bool parentUsesSize = false }) {
  RenderObject? relayoutBoundary;
  // 先确定当前组件的布局边界
  if (!parentUsesSize || sizedByParent || constraints.isTight || parent is! RenderObject) {
    relayoutBoundary = this;
  } else {
    relayoutBoundary = (parent! as RenderObject)._relayoutBoundary;
  }
  // _needsLayout 表示当前组件是否被标记为需要布局
  // _constraints 是上次布局时父组件传递给当前组件的约束
  // _relayoutBoundary 为上次布局时当前组件的布局边界
  // 所以，当当前组件没有被标记为需要重新布局，且父组件传递的约束没有发生变化，
  // 且布局边界也没有发生变化时则不需要重新布局，直接返回即可。
  if (!_needsLayout && constraints == _constraints && relayoutBoundary == _relayoutBoundary) {
    return;
  }
  // 如果需要布局，缓存约束和布局边界
  _constraints = constraints;
  _relayoutBoundary = relayoutBoundary;

  // sizedByParent表示当前的Widget虽然不是isTight，但是通过其他约束属性，也可以明确的知道size，比如Expanded，并不一定需要明确的size
  if (sizedByParent) {
    performResize();
  }
  // 执行布局
  performLayout();
  // 布局结束后将 _needsLayout 置为 false
  _needsLayout = false;
  // 将当前组件标记为需要重绘（因为布局发生变化后，需要重新绘制）
  markNeedsPaint();
}

```

## 命中测试

命中测试用来判断某个组件是否需要响应一个点击事件，其入口是RenderObject Tree的根节点RenderView的hitTest函数

```dart
bool hitTest(HitTestResult result, { Offset position }) {
  if (child != null)
    child.hitTest(BoxHitTestResult.wrap(result), position: position);//child为RenderView
  result.add(HitTestEntry(this));
  return true;
}
```

查看RenderView源码

```dart
bool hitTest(BoxHitTestResult result, { @required Offset position }) {
  if (_size.contains(position)) {
    if (hitTestChildren(result, position: position) || hitTestSelf(position)) {
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
  }
  return false;
}
```

如果点击事件位置处于RenderObject之内，如果在其内，并且hitTestSelf或者hitTestChildren返回true，则表示RenderObject通过了命中测试，需要响应事件，此时需要将被点击的RenderObject加入BoxHitTestResult列表，同时点击事件不再向下传递。否则认为没有通过命中测试，事件继续向下传递。其中，hitTestSelf函数表示节点是否通过命中测试，hitTestChildren表示子节点是否通过命中测试。




1. 父节点向子节点传递约束（constraints）信息，限制子节点的最大和最小宽高。
2. 子节点根据约束信息确定自己的大小（size）。
3. 父节点根据特定布局规则（不同布局组件会有不同的布局算法）确定每一个子节点在父节点布局空间中的位置，用偏移 offset 表示。
4. 递归整个过程，确定出每一个节点的大小和位置。