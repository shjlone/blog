---
title: Layout过程
toc: true
tags: Flutter
---





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