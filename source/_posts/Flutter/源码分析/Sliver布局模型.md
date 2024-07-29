---
title: Sliver布局模型
toc: true
tags: Flutter
---


在Flutter中，列表的每个Item被称为Sliver。

![](./scroll.png)

Flutter常见的列表类最终都有Scrollable类实现，而该类内部包含RawGestureDetector等一系列负责处理手势、响应滑动的类。

## RenderViewport布局流程分析

![](./renderviewport.png)

RenderViewport关键类及其关系

![](./sliver1.png)

Viewport的大小（主轴方法）为1250（每个Sliver的大小为250），即图中深灰色部分。Viewport前后存在一定长度的缓冲区，用于提升列表滑动的流畅度，即图中浅灰色部分，大小为250。center参数被设置为第4个子节点，但是因为anchor为0.2，所以子节点会向下偏移1/5主轴长度的距离，因此图中第1个显示的为sliver-3.

下面分析RenderViewport的布局过程

```dart

class RenderViewport {
  void performLayout() {
    switch (axis) {//1、记录Viewport在主轴方向的大小
      case Axis.vertical:
        offset.applyViewportDimension(size.height);
      case Axis.horizontal:
        offset.applyViewportDimension(size.width);
    }

    if (center == null) {//2、判断Viewport中是否有列表内容
      assert(firstChild == null);
      _minScrollExtent = 0.0;
      _maxScrollExtent = 0.0;
      _hasVisualOverflow = false;
      offset.applyContentDimensions(0.0, 0.0);
      return;
    }

    final double mainAxisExtent;
    final double crossAxisExtent;
    switch (axis) {//3、计算当前Viewport在主轴和交叉轴方向的大小
      case Axis.vertical:
        mainAxisExtent = size.height;
        crossAxisExtent = size.width;
      case Axis.horizontal:
        mainAxisExtent = size.width;
        crossAxisExtent = size.height;
    }
/// 以上逻辑分为4步：
/// 1、记录Viewport在主轴方向的大小
/// 2、center默认为第1个子节点，如果不存在，则说明该Viewport中没有列表内容
/// 3、计算当前Viewport在主轴和交叉轴方向的大小
/// 4、解析完Viewport本身的信息之后，开始进行子节点的布局


    final double centerOffsetAdjustment = center!.centerOffsetAdjustment;

    double correction;
    int count = 0;
    do {
      correction = _attemptLayout(mainAxisExtent, crossAxisExtent, offset.pixels + centerOffsetAdjustment);//真正的列表布局逻辑
      if (correction != 0.0) {//需要校正，一般在SliverList等动态创建Sliver时进行
        offset.correctBy(correction);
      } else {
        if (offset.applyContentDimensions(
              math.min(0.0, _minScrollExtent + mainAxisExtent * anchor),
              math.max(0.0, _maxScrollExtent - mainAxisExtent * (1.0 - anchor)),
           )) {
          break;
        }
      }
      count += 1;
    } while (count < _maxLayoutCycles);
  }

/// 子节点的布局流程
  double _attemptLayout(double mainAxisExtent, double crossAxisExtent, double correctedOffset) {
    _minScrollExtent = 0.0;
    _maxScrollExtent = 0.0;
    _hasVisualOverflow = false;

    final double centerOffset = mainAxisExtent * anchor - correctedOffset;
    final double reverseDirectionRemainingPaintExtent = clampDouble(centerOffset, 0.0, mainAxisExtent);
    final double forwardDirectionRemainingPaintExtent = clampDouble(mainAxisExtent - centerOffset, 0.0, mainAxisExtent);

    switch (cacheExtentStyle) {
      case CacheExtentStyle.pixel:
        _calculatedCacheExtent = cacheExtent;
      case CacheExtentStyle.viewport:
        _calculatedCacheExtent = mainAxisExtent * _cacheExtent;
    }

    final double fullCacheExtent = mainAxisExtent + 2 * _calculatedCacheExtent!;
    final double centerCacheOffset = centerOffset + _calculatedCacheExtent!;
    final double reverseDirectionRemainingCacheExtent = clampDouble(centerCacheOffset, 0.0, fullCacheExtent);
    final double forwardDirectionRemainingCacheExtent = clampDouble(fullCacheExtent - centerCacheOffset, 0.0, fullCacheExtent);

/// 以上逻辑主要是开始正式布局前进行相关字段计算。correctedOffset通常就是用户的滑动距离offset.pixels，centerOffset时center相对Viewport顶部的偏移值（sliver-3的大小）。
/// reverseDirectionRemainingPaintExtent表示反（reverse）方向的剩余可绘制长度
/// forwardDirectionRemaningPaintExtent表示正（forwar）方向的剩余可绘制长度

    final RenderSliver? leadingNegativeChild = childBefore(center!);

    if (leadingNegativeChild != null) {
      // negative scroll offsets
      final double result = layoutChildSequence(//布局反方向的子节点
        child: leadingNegativeChild,
        scrollOffset: math.max(mainAxisExtent, centerOffset) - mainAxisExtent,
        overlap: 0.0,
        layoutOffset: forwardDirectionRemainingPaintExtent,
        remainingPaintExtent: reverseDirectionRemainingPaintExtent,
        mainAxisExtent: mainAxisExtent,
        crossAxisExtent: crossAxisExtent,
        growthDirection: GrowthDirection.reverse,
        advance: childBefore,
        remainingCacheExtent: reverseDirectionRemainingCacheExtent,
        cacheOrigin: clampDouble(mainAxisExtent - centerOffset, -_calculatedCacheExtent!, 0.0),
      );
      if (result != 0.0) {
        return -result;
      }
    }

    // positive scroll offsets
    return layoutChildSequence(//布局正方向的子节点
      child: center, //表示当前的Sliver节点
      scrollOffset: math.max(0.0, -centerOffset), //表示center Sliver划过Viewport顶部的距离
      overlap: leadingNegativeChild == null ? math.min(0.0, -centerOffset) : 0.0,
      layoutOffset: centerOffset >= mainAxisExtent ? centerOffset: reverseDirectionRemainingPaintExtent,
      remainingPaintExtent: forwardDirectionRemainingPaintExtent,
      mainAxisExtent: mainAxisExtent,
      crossAxisExtent: crossAxisExtent,
      growthDirection: GrowthDirection.forward,
      advance: childAfter,
      remainingCacheExtent: forwardDirectionRemainingCacheExtent,
      cacheOrigin: clampDouble(centerOffset, -_calculatedCacheExtent!, 0.0),
    );
  }

}

```

![](./sliver2.png)

```dart

class RenderViewportBase {
  /// 计算SliverConstraints实例，并调用child.layout驱动子节点完成布局
  double layoutChildSequence({
    required RenderSliver? child,
    required double scrollOffset,
    required double overlap,
    required double layoutOffset,
    required double remainingPaintExtent,
    required double mainAxisExtent,
    required double crossAxisExtent,
    required GrowthDirection growthDirection,
    required RenderSliver? Function(RenderSliver child) advance,
    required double remainingCacheExtent,
    required double cacheOrigin,
  }) {
    final double initialLayoutOffset = layoutOffset;
    final ScrollDirection adjustedUserScrollDirection =
        applyGrowthDirectionToScrollDirection(offset.userScrollDirection, growthDirection);
    double maxPaintOffset = layoutOffset + overlap;
    double precedingScrollExtent = 0.0;

    while (child != null) {
      final double sliverScrollOffset = scrollOffset <= 0.0 ? 0.0 : scrollOffset;
      final double correctedCacheOrigin = math.max(cacheOrigin, -sliverScrollOffset);
      final double cacheExtentCorrection = cacheOrigin - correctedCacheOrigin;

      assert(sliverScrollOffset >= correctedCacheOrigin.abs());
      assert(correctedCacheOrigin <= 0.0);
      assert(sliverScrollOffset >= 0.0);
      assert(cacheExtentCorrection <= 0.0);

      child.layout(SliverConstraints(//触发子节点的布局
        axisDirection: axisDirection,
        growthDirection: growthDirection,
        userScrollDirection: adjustedUserScrollDirection,
        scrollOffset: sliverScrollOffset,
        precedingScrollExtent: precedingScrollExtent,
        overlap: maxPaintOffset - layoutOffset,
        remainingPaintExtent: math.max(0.0, remainingPaintExtent - layoutOffset + initialLayoutOffset),
        crossAxisExtent: crossAxisExtent,
        crossAxisDirection: crossAxisDirection,
        viewportMainAxisExtent: mainAxisExtent,
        remainingCacheExtent: math.max(0.0, remainingCacheExtent + cacheExtentCorrection),
        cacheOrigin: correctedCacheOrigin,
      ), parentUsesSize: true);

      final SliverGeometry childLayoutGeometry = child.geometry!;
      // If there is a correction to apply, we'll have to start over.
      if (childLayoutGeometry.scrollOffsetCorrection != null) {//如果需要校正，则直接返回
        return childLayoutGeometry.scrollOffsetCorrection!;
      }

      // We use the child's paint origin in our coordinate system as the
      // layoutOffset we store in the child's parent data.
      final double effectiveLayoutOffset = layoutOffset + childLayoutGeometry.paintOrigin;

      if (childLayoutGeometry.visible || scrollOffset > 0) {//判断当前Sliver可见或者在Viewport上面
        updateChildLayoutOffset(child, effectiveLayoutOffset, growthDirection);
      } else {
        updateChildLayoutOffset(child, -scrollOffset + initialLayoutOffset, growthDirection);
      }

      maxPaintOffset = math.max(effectiveLayoutOffset + childLayoutGeometry.paintExtent, maxPaintOffset);
      scrollOffset -= childLayoutGeometry.scrollExtent;
      precedingScrollExtent += childLayoutGeometry.scrollExtent;
      layoutOffset += childLayoutGeometry.layoutExtent;
      if (childLayoutGeometry.cacheExtent != 0.0) {
        remainingCacheExtent -= childLayoutGeometry.cacheExtent - cacheExtentCorrection;
        cacheOrigin = math.min(correctedCacheOrigin + childLayoutGeometry.cacheExtent, 0.0);
      }

      updateOutOfBandData(growthDirection, childLayoutGeometry);

      // move on to the next child
      child = advance(child);
    }

    // we made it without a correction, whee!
    return 0.0;
  }
}


class SliverConstraints {
  //表示列表中forwar Sliver的增长方向，最常用的事AxisDirection.down，表示列表从上到下递增，此时scrollOffset向上增加，remainingPaintExtent向下增加
  final AxisDirection axisDirection;
// 表示Sliver增长的方向，forward表示与axisDirection方向相同，是center Sliver之后的节点；reverse表示与axisDirection方向相反，时center Sliver之前的节点
  final GrowthDirection growthDirection;
  //滑动方向
  final ScrollDirection userScrollDirection;
  //表示center Sliver滑过Viewport的距离，以AxisDirection.down为例，滑过Viewport顶部的距离即scrollOffset
  final double scrollOffset;
  final double precedingScrollExtent;
  //表示上一个Sliver覆盖下一个Sliver的大小
  final double overlap;
  // 表示对当前节点而言，剩余绘制区域大小
  final double remainingPaintExtent;
  //交叉轴方向的大小
  final double crossAxisExtent;
  //交叉轴方向的布局顺序
  final AxisDirection crossAxisDirection;
  //主轴方向的大小
  final double viewportMainAxisExtent;
  //剩余缓冲区大小
  final double remainingCacheExtent;
  //表示当前Sliver可使用的Viewport顶部缓冲区的大小
  final double cacheOrigin;
}


```


![](./sliver3.png)



以上便是Viewport布局的核心流程，确定每个子节点的大小和偏移值，下一个子节点基于此计算自己的大小和偏移值。



```dart
class SliverGeometry with Diagnosticable {
  final double scrollExtent;//当前Sliver在列表中的可滚动长度，一般就是Sliver本身的长度
  final double paintOrigin;//当前Sliver开始绘制的起点，相对当前Sliver的布局起点而言
  final double paintExtent;//当前Sliver需要绘制在可视区域Viewport中的长度

  final double layoutExtent;//当前Sliver需要布局的长度，默认为paintExtent
  final double maxPaintExtent;//当前Sliver的最大绘制长度
  final double maxScrollObstructionExtent;//当Sliver被固定在Viewport边缘时占据的最大长度
  final double hitTestExtent;//相应点击的区域长度，默认值为paintExtent
  final bool visible;//当前Sliver是否可见，不可见则paintExtent为0
  final bool hasVisualOverflow;//当前Sliver是否溢出Viewport，通常是在滑入、滑出时发生
  final double? scrollOffsetCorrection;//校正值
  final double cacheExtent;//当前Sliver消耗的缓冲区大小

}
```

下面分析SLiver的ParentData实例的更新

```dart

class RenderViewport {
  void updateChildLayoutOffset(RenderSliver child, double layoutOffset, GrowthDirection growthDirection) {
    final SliverPhysicalParentData childParentData = child.parentData! as SliverPhysicalParentData;
    childParentData.paintOffset = computeAbsolutePaintOffset(child, layoutOffset, growthDirection);
  }

/// 计算不同方向下的子节点偏移
  Offset computeAbsolutePaintOffset(RenderSliver child, double layoutOffset, GrowthDirection growthDirection) {
    switch (applyGrowthDirectionToAxisDirection(axisDirection, growthDirection)) {
      case AxisDirection.up:
        return Offset(0.0, size.height - (layoutOffset + child.geometry!.paintExtent));
      case AxisDirection.right:
        return Offset(layoutOffset, 0.0);
      case AxisDirection.down:
        return Offset(0.0, layoutOffset);
      case AxisDirection.left:
        return Offset(size.width - (layoutOffset + child.geometry!.paintExtent), 0.0);
    }
  }

/// 更新可滚动距离
  void updateOutOfBandData(GrowthDirection growthDirection, SliverGeometry childLayoutGeometry) {
    switch (growthDirection) {
      case GrowthDirection.forward:
        _maxScrollExtent += childLayoutGeometry.scrollExtent;
      case GrowthDirection.reverse:
        _minScrollExtent -= childLayoutGeometry.scrollExtent;
    }
    if (childLayoutGeometry.hasVisualOverflow) {
      _hasVisualOverflow = true;
    }
  }


}
```

![](./sliver4.png)



## RenderSliverToBoxAdapter布局流程分析


![](./sliver5.png)


RenderSliver的子类分类：

- RenderSliverSingleBoxAdapter：可以封装一个子节点
- RenderSliverMultiBoxAdaptor：可以封装多个子节点，比如SliverList、SliverGrid
- RenderSliverPersistentHeader：它是SliverAppBar的底层实现，是对overlap属性的典型应用


下面以RenderSliverToBoxAdapter为例，分析RenderSliver节点自身的布局，它可以将一个Box类型的Widget放在列表中使用，那么其中必然涉及SliverConstraints到BoxConstraints的转换，因为Box类型的RenderObject只接受BoxConstraints作为约束，此外Box类型的RenderObject返回的Size信息也需要转换为SliverGeometry，否则Viewport无法解析。


```dart
class RenderSliverToBoxAdapter extends RenderSliverSingleBoxAdapter {
  void performLayout() {
    if (child == null) {
      geometry = SliverGeometry.zero;
      return;
    }
    final SliverConstraints constraints = this.constraints;//1、将SliverConstraints转换为BoxConstraints
    child!.layout(constraints.asBoxConstraints(), parentUsesSize: true);
    final double childExtent;
    switch (constraints.axis) {//2、根据主轴方向确定子节点所占用的空间大小
      case Axis.horizontal:
        childExtent = child!.size.width;
      case Axis.vertical:
        childExtent = child!.size.height;
    }
    //3、根据子节点在主轴占据的空间大小以及当前约束绘制大小
    final double paintedChildSize = calculatePaintOffset(constraints, from: 0.0, to: childExtent);
    final double cacheExtent = calculateCacheOffset(constraints, from: 0.0, to: childExtent);

    geometry = SliverGeometry(//4、计算SliverGeometry
      scrollExtent: childExtent,
      paintExtent: paintedChildSize,
      cacheExtent: cacheExtent,
      maxPaintExtent: childExtent,
      hitTestExtent: paintedChildSize,
      hasVisualOverflow: childExtent > constraints.remainingPaintExtent || constraints.scrollOffset > 0.0,
    );
    setChildParentData(child!, constraints, geometry!);//5、为子节点绘制偏移
  }
}
```

### SliverConstraints转换为BoxConstraints

```dart
class SliverConstraints extends Constraints {
  ///
  BoxConstraints asBoxConstraints({
    double minExtent = 0.0,
    double maxExtent = double.infinity,
    double? crossAxisExtent,
  }) {
    crossAxisExtent ??= this.crossAxisExtent;
    switch (axis) {
      case Axis.horizontal:
        return BoxConstraints(
          minHeight: crossAxisExtent,
          maxHeight: crossAxisExtent,
          minWidth: minExtent,
          maxWidth: maxExtent,
        );
      case Axis.vertical:
        return BoxConstraints(
          minWidth: crossAxisExtent,
          maxWidth: crossAxisExtent,
          minHeight: minExtent,
          maxHeight: maxExtent,
        );
    }
  }
}

```

paintedChildSize是如何计算的呢？


### setChildParentData

```dart

class RenderSliverSingleBoxAdapter {

  /// 用于解决一些特殊的边界情况
  void setChildParentData(RenderObject child, SliverConstraints constraints, SliverGeometry geometry) {
    final SliverPhysicalParentData childParentData = child.parentData! as SliverPhysicalParentData;
    switch (applyGrowthDirectionToAxisDirection(constraints.axisDirection, constraints.growthDirection)) {
      case AxisDirection.up:
        childParentData.paintOffset = Offset(0.0, -(geometry.scrollExtent - (geometry.paintExtent + constraints.scrollOffset)));
      case AxisDirection.right:
        childParentData.paintOffset = Offset(-constraints.scrollOffset, 0.0);
      case AxisDirection.down:
        childParentData.paintOffset = Offset(0.0, -constraints.scrollOffset);
      case AxisDirection.left:
        childParentData.paintOffset = Offset(-(geometry.scrollExtent - (geometry.paintExtent + constraints.scrollOffset)), 0.0);
    }
  }

  void paint(PaintingContext context, Offset offset) {
    if (child != null && geometry!.visible) {
      final SliverPhysicalParentData childParentData = child!.parentData! as SliverPhysicalParentData;
      context.paintChild(child!, offset + childParentData.paintOffset);
    }
  }
}

```

![](./sliver6.png)

如图，对完全处于Viewport内的Sliver而言，constraints.scrollOffset为0，子节点的paintOffset为(0,0)。此时子节点从Sliver的左上角开始绘制。


## 参考

- [Flutter源码内核剖析]()