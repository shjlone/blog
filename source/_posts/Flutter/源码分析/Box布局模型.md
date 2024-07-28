---
title: Box布局模型
toc: true
tags: Flutter
---

Lyaout的本质是父节点向子节点传递自己的布局约束Constraints，子节点计算自身的大小（Size），父节点再根据大小信息计算偏移（Offset）。在二维空间中，根据大小和偏移可以唯一确定子节点的位置。


Flutter中主要存在两种布局约束，Box和Sliver：

![](./layout1.png)

BoxConstraints和SliverConstraints分别对应Box布局和Sliver布局模型所需要的约束条件。ParentData是RenderObject持有的一个字段，用于为父节点提供额外的信息。比如RenderBox通过BoxParentData向父节点暴露自身的偏移量。

## Box布局概述

![](./box1.png)




## Align布局流程分析


![](./align.png)


RenderShiftedBox表示一个可以对子节点在自身中的位置进行控制的单子节点容器，最常见是Padding和Align，Align对应RenderObject是RenderPositionedBox

```dart

class RenderPositionedBox {
  void performLayout() {
    final BoxConstraints constraints = this.constraints;
    //shrinkWrapWidth表示当前宽度是否需要折叠（Shrink）
    final bool shrinkWrapWidth = _widthFactor != null || constraints.maxWidth == double.infinity;//就算宽度是infinity，也要变成确定宽度，否则边界无法确定
    final bool shrinkWrapHeight = _heightFactor != null || constraints.maxHeight == double.infinity;

    if (child != null) {//存在子节点
      child!.layout(constraints.loosen(), parentUsesSize: true);//布局子节点
      size = constraints.constrain(Size(//开始布局自身
        shrinkWrapWidth ? child!.size.width * (_widthFactor ?? 1.0) : double.infinity,
        shrinkWrapHeight ? child!.size.height * (_heightFactor ?? 1.0) : double.infinity,
      ));
      alignChild();//计算子节点的偏移
    } else {
      size = constraints.constrain(Size(//没有子节点时，一般大小为0，因为最大约束为infinity时shrinkWrapWidth为true
        shrinkWrapWidth ? 0.0 : double.infinity,
        shrinkWrapHeight ? 0.0 : double.infinity,
      ));
    }
  }

  void alignChild() {
    _resolve();//计算子节点坐标
    final BoxParentData childParentData = child!.parentData! as BoxParentData;//存储位置信息
    childParentData.offset = _resolvedAlignment!.alongOffset(size - child!.size as Offset);
  }

  void _resolve() {
    if (_resolvedAlignment != null) {
      return;
    }
    _resolvedAlignment = alignment.resolve(textDirection);//Alignment是Align对子节点位置的抽象表示。Align实际持有的是AlignmentGeometry，它有多个子类。
  }

}


class BoxConstraints {
  Size constrain(Size size) {
    Size result = Size(constrainWidth(size.width), constrainHeight(size.height));
    return result;
  }

  double constrainWidth([ double width = double.infinity ]) {
    assert(debugAssertIsValid());
    return clampDouble(width, minWidth, maxWidth);//返回一个介于minWidth和maxWidth之间的值，如果是infinity在返回maxWidth
  }

  double constrainHeight([ double height = double.infinity ]) {
    assert(debugAssertIsValid());
    return clampDouble(height, minHeight, maxHeight);
  }


}

```


![](./alignment.png)

alignChild方法最终会调用Alignment的alongOffset方法完成子节点偏移值的计算

```dart
class Alignment {

  Offset alongOffset(Offset other) {//other表示父节点大小减去子节点后剩余的偏移值
    final double centerX = other.dx / 2.0;//定位坐标系的原点
    final double centerY = other.dy / 2.0;
    return Offset(centerX + x * centerX, centerY + y * centerY);
  }

}
```


## Flex布局流程分析


![](./flex.png)

Column和Row是常见的支持弹性布局的Widget，它们都继承自Flex，而Flex对应的RenderObject是RenderFlex。RenderFlex实现弹性布局的关键，在于其子节点的parentData字段的类型为FlexParentData，其内部含有子节点的弹性系数（flex）等信息。RenderFlex控制的是子节点的parentData字段的类型。

```dart

class RenderFlex {

  @override
  void performLayout() {
    final BoxConstraints constraints = this.constraints;

    final _LayoutSizes sizes = _computeSizes(//1、对子节点进行布局
      layoutChild: ChildLayoutHelper.layoutChild,
      constraints: constraints,//当前节点给子节点的约束
    );

    final double allocatedSize = sizes.allocatedSize;
    double actualSize = sizes.mainSize;
    double crossSize = sizes.crossSize;
    double maxBaselineDistance = 0.0;
    if (crossAxisAlignment == CrossAxisAlignment.baseline) {
      RenderBox? child = firstChild;
      double maxSizeAboveBaseline = 0;
      double maxSizeBelowBaseline = 0;
      while (child != null) {
        final double? distance = child.getDistanceToBaseline(textBaseline!, onlyReal: true);
        if (distance != null) {
          maxBaselineDistance = math.max(maxBaselineDistance, distance);
          maxSizeAboveBaseline = math.max(
            distance,
            maxSizeAboveBaseline,
          );
          maxSizeBelowBaseline = math.max(
            child.size.height - distance,
            maxSizeBelowBaseline,
          );
          crossSize = math.max(maxSizeAboveBaseline + maxSizeBelowBaseline, crossSize);
        }
        final FlexParentData childParentData = child.parentData! as FlexParentData;
        child = childParentData.nextSibling;
      }
    }

    // Align items along the main axis.
    switch (_direction) { // 确定主轴和交叉轴的实际大小
      case Axis.horizontal:
        size = constraints.constrain(Size(actualSize, crossSize));
        actualSize = size.width;
        crossSize = size.height;
      case Axis.vertical:
        size = constraints.constrain(Size(crossSize, actualSize));
        actualSize = size.height;
        crossSize = size.width;
    }
    final double actualSizeDelta = actualSize - allocatedSize;//计算剩余空间大小
    _overflow = math.max(0.0, -actualSizeDelta);//判断是否溢出，用于Paint阶段绘制提示信息
    final double remainingSpace = math.max(0.0, actualSizeDelta);//剩余空间，用于计算间距
    late final double leadingSpace;//每一个节点的前部间距
    late final double betweenSpace;//每个节点的间距
    final bool flipMainAxis = !(_startIsTopLeft(direction, textDirection, verticalDirection) ?? true);//根据各direction参数，判断是否反转子节点排列方向
    switch (_mainAxisAlignment) {//根据主轴的对齐方式，计算间距等信息
      case MainAxisAlignment.start:
        leadingSpace = 0.0;
        betweenSpace = 0.0; // 没有间距
      case MainAxisAlignment.end://每个子节点按序尽可能靠近主轴的结束点排列
        leadingSpace = remainingSpace;//其实位置保证剩余空间填充满，即靠近结束点排列
        betweenSpace = 0.0;
      case MainAxisAlignment.center://中点排列
        leadingSpace = remainingSpace / 2.0;
        betweenSpace = 0.0;
      case MainAxisAlignment.spaceBetween://子节点等距排列，两端的子节点边距为0
        leadingSpace = 0.0;
        betweenSpace = childCount > 1 ? remainingSpace / (childCount - 1) : 0.0;
      case MainAxisAlignment.spaceAround://每个子节点等距排列，两端的子节点边距为间距的一半
        betweenSpace = childCount > 0 ? remainingSpace / childCount : 0.0;
        leadingSpace = betweenSpace / 2.0;
      case MainAxisAlignment.spaceEvenly://每个子节点等距排列，两端的子节点边距也为该距离
        betweenSpace = childCount > 0 ? remainingSpace / (childCount + 1) : 0.0;
        leadingSpace = betweenSpace;
    }

  // 至此，已经确定了每个子节点在主轴的偏移值，下面开始确定在交叉轴的偏移值
    // Position elements
    double childMainPosition = flipMainAxis ? actualSize - leadingSpace : leadingSpace;
    RenderBox? child = firstChild;
    while (child != null) {
      final FlexParentData childParentData = child.parentData! as FlexParentData; 
      final double childCrossPosition;//计算交叉轴方向的偏移值
      switch (_crossAxisAlignment) {
        case CrossAxisAlignment.start://每个子节点紧贴着交叉轴的起始点排列
        case CrossAxisAlignment.end://每个子节点紧贴着交叉轴的结束点排列
          childCrossPosition = _startIsTopLeft(flipAxis(direction), textDirection, verticalDirection)
                               == (_crossAxisAlignment == CrossAxisAlignment.start)
                             ? 0.0
                             : crossSize - _getCrossSize(child.size);
        case CrossAxisAlignment.center:
          childCrossPosition = crossSize / 2.0 - _getCrossSize(child.size) / 2.0;
        case CrossAxisAlignment.stretch:
          childCrossPosition = 0.0;
        case CrossAxisAlignment.baseline://每个子节点的Baseline对齐
          if (_direction == Axis.horizontal) {
            assert(textBaseline != null);
            final double? distance = child.getDistanceToBaseline(textBaseline!, onlyReal: true);//计算子节点顶部到Baseline的距离
            if (distance != null) {
              childCrossPosition = maxBaselineDistance - distance;
            } else {
              childCrossPosition = 0.0;
            }
          } else {
            childCrossPosition = 0.0;
          }
      }
      if (flipMainAxis) {//如果方向翻转，则布局的实际位置需要减去自身所占空间
        childMainPosition -= _getMainSize(child.size);
      }
      switch (_direction) {
        case Axis.horizontal:
          childParentData.offset = Offset(childMainPosition, childCrossPosition);
        case Axis.vertical:
          childParentData.offset = Offset(childCrossPosition, childMainPosition);
      }
      if (flipMainAxis) {//更新主轴方向的偏移量
        childMainPosition -= betweenSpace;
      } else {
        childMainPosition += _getMainSize(child.size) + betweenSpace;//正常方向，累加当前节点大小和间距
      }
      child = childParentData.nextSibling;//下一个节点
    }
  }


///对子节点布局
  _LayoutSizes _computeSizes({required BoxConstraints constraints, required ChildLayouter layoutChild}) {
    // Determine used flex factor, size inflexible items, calculate free space.
    int totalFlex = 0;
    final double maxMainSize = _direction == Axis.horizontal ? constraints.maxWidth : constraints.maxHeight;//计算主轴方向的最大值
    final bool canFlex = maxMainSize < double.infinity;

    double crossSize = 0.0;
    double allocatedSize = 0.0; // Sum of the sizes of the non-flexible children. 分配给非弹性节点的总大小
    RenderBox? child = firstChild;
    RenderBox? lastFlexChild;//最后一个Flex类型子节点
    while (child != null) {
      final FlexParentData childParentData = child.parentData! as FlexParentData;
      final int flex = _getFlex(child);// 1、获取当前子节点的弹性系数
      if (flex > 0) { 
        totalFlex += flex; //计算flex之和，用于后续分配对应比例的空间
        lastFlexChild = child;
      } else {
        final BoxConstraints innerConstraints; //2、计算对子节点的约束
        if (crossAxisAlignment == CrossAxisAlignment.stretch) {//strctch是比较特殊的交叉轴对齐类型，需要特殊处理
          switch (_direction) {
            case Axis.horizontal://强度自身高度为最大高度，达到拉伸效果
              innerConstraints = BoxConstraints.tightFor(height: constraints.maxHeight);
            case Axis.vertical:
              innerConstraints = BoxConstraints.tightFor(width: constraints.maxWidth);
          }
        } else {
          switch (_direction) {
            case Axis.horizontal:
              innerConstraints = BoxConstraints(maxHeight: constraints.maxHeight);
            case Axis.vertical:
              innerConstraints = BoxConstraints(maxWidth: constraints.maxWidth);
          }
        }
        final Size childSize = layoutChild(child, innerConstraints);//布局子节点
        allocatedSize += _getMainSize(childSize);//累计非弹性节点占用的空间
        crossSize = math.max(crossSize, _getCrossSize(childSize));//更新交叉轴方向的最大值
      }
      child = childParentData.nextSibling;//遍历下一个子节点
    }

    //完成所有非弹性节点的布局之后，这些节点占用的主轴空间便确定了，如果还有弹性节点，那么剩余空间会用于弹性节点的布局
    // Distribute free space to flexible children.
    final double freeSpace = math.max(0.0, (canFlex ? maxMainSize : 0.0) - allocatedSize);// 计算剩余空间
    double allocatedFlexSpace = 0.0;
    if (totalFlex > 0) {
      final double spacePerFlex = canFlex ? (freeSpace / totalFlex) : double.nan; //单位距离
      child = firstChild;
      while (child != null) { //遍历每个子节点
        final int flex = _getFlex(child);//获取弹性系数
        if (flex > 0) {
          final double maxChildExtent = canFlex ? (child == lastFlexChild ? (freeSpace - allocatedFlexSpace) : spacePerFlex * flex) : double.infinity;
          late final double minChildExtent;
          switch (_getFit(child)) {
            case FlexFit.tight:
              assert(maxChildExtent < double.infinity);
              minChildExtent = maxChildExtent;
            case FlexFit.loose:
              minChildExtent = 0.0;
          }
          final BoxConstraints innerConstraints;
          if (crossAxisAlignment == CrossAxisAlignment.stretch) {
            switch (_direction) {
              case Axis.horizontal:
                innerConstraints = BoxConstraints(
                  minWidth: minChildExtent,
                  maxWidth: maxChildExtent,
                  minHeight: constraints.maxHeight,
                  maxHeight: constraints.maxHeight,
                );
              case Axis.vertical:
                innerConstraints = BoxConstraints(
                  minWidth: constraints.maxWidth,
                  maxWidth: constraints.maxWidth,
                  minHeight: minChildExtent,
                  maxHeight: maxChildExtent,
                );
            }
          } else {
            switch (_direction) {
              case Axis.horizontal:
                innerConstraints = BoxConstraints(
                  minWidth: minChildExtent,
                  maxWidth: maxChildExtent,
                  maxHeight: constraints.maxHeight,
                );
              case Axis.vertical:
                innerConstraints = BoxConstraints(
                  maxWidth: constraints.maxWidth,
                  minHeight: minChildExtent,
                  maxHeight: maxChildExtent,
                );
            }
          }
          final Size childSize = layoutChild(child, innerConstraints);
          final double childMainSize = _getMainSize(childSize);
          assert(childMainSize <= maxChildExtent);
          allocatedSize += childMainSize;
          allocatedFlexSpace += maxChildExtent;//已经分配的弹性布局空间
          crossSize = math.max(crossSize, _getCrossSize(childSize));//计算交叉轴的最大值
        }
        final FlexParentData childParentData = child.parentData! as FlexParentData;
        child = childParentData.nextSibling;
      }
    }

// 计算每个非Flex子节点占用空间的大小和弹性系数之和
    final double idealSize = canFlex && mainAxisSize == MainAxisSize.max ? maxMainSize : allocatedSize;
    return _LayoutSizes(
      mainSize: idealSize,
      crossSize: crossSize,
      allocatedSize: allocatedSize,
    );
  }

}


```


根据对齐方式确定布局的起始位置和间距

![](./mainAxisAlignment.png)


![](./crossAlignment.png)

