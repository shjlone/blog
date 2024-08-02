---
title: CustomSingleChildLayout
toc: true
tags: Flutter
---

CustomSingleChildLayout解决以下几个问题：

- 设置child的大小
- 设置child的位置


```dart
const CustomSingleChildLayout({
  super.key,
  required this.delegate,//通过委托类实现相关回调函数
  super.child,
})


abstract class SingleChildLayoutDelegate {
  const SingleChildLayoutDelegate({ Listenable? relayout }) : _relayout = relayout;

  final Listenable? _relayout;

  /// 返回child的大小
  Size getSize(BoxConstraints constraints) => constraints.biggest;

  BoxConstraints getConstraintsForChild(BoxConstraints constraints) => constraints;

  /// 返回child的偏移值
  Offset getPositionForChild(Size size, Size childSize) => Offset.zero;

  /// 是否需要重新布局
  bool shouldRelayout(covariant SingleChildLayoutDelegate oldDelegate);
}

```

可参考ToolTip的具体实现