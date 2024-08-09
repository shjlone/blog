---
title: ConstraintLayout
tags: Android
toc: true
---

## 概述

约束布局，可以降低布局的层次，对性能提升有帮助。

可以把约束理解成一根绳子，比如，如果设置了左侧与父级依赖，则相当于左边有一根绳子拉着自己，就会贴着左边。当右边也有一根绳子拉着自己，则会居中。
这跟绳子还可以指向同一个点，比如设置在某个控件的中间位置。也可以设置偏移量bias。

使用约束布局，主要要理解不同控件的约束。比如A控件在B控件上方，则可以使用layout_constraintBottom_toTopOf="B"，同理可设置不同方位。
如果是挨着边，则可以设置控件与父级约束。layout_constraintLeft_toLeftOf="parent"表示挨着父级的左边。

### layout_constraintLeft_toRightOf

constraintXXX表示约束View自己，XXX分别表示上下左右等位置，toXXXOf表示约束依赖等对象，可以是同级的View，也可以是parent。

### layout_constraintBaseline_toBaselineOf

对于TextView，可以使用基线对齐，这样文字就能对齐了。

### margin

两个item之间的间距

**不同方位的边距**

- android:layout_marginStart
- android:layout_marginEnd
- android:layout_marginLeft
- android:layout_marginTop
- android:layout_marginRight
- android:layout_marginBottom

### goneMargin

- goneMargin 要和 margin 一起使用才有效
- 在约束的布局 Gone 时，View 自身的 margin 属性会被 goneMargin 替换
- Gone 隐藏的控件，会被解析为一个点，其 Margin 会被忽略，它对其他控件的约束仍会保留

**常用属性**

- layout_goneMarginStart
- layout_goneMarginEnd
- layout_goneMarginLeft
- layout_goneMarginTop
- layout_goneMarginRight
- layout_goneMarginBottom

### bias

- layout_constraintHorizontal_bias 水平偏移
- layout_constraintVertical_bias 垂直偏移

两个属性的取值范围在0-1。在水平偏移中，0表示最左，1表示最右；在垂直偏移，0表示最上，1表示最下；0.5表示中间。

## Group

##

## 参考

- [https://add7.cc/android-base/android-constraintlayout](https://add7.cc/android-base/android-constraintlayout)
