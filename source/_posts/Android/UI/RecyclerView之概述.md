---
title: RecyclerView之概述
toc: true
tags: [Android, RecyclerView]
---

## 目录

- [RecyclerView之概述](/Android/UI/RecyclerView之概述/)
- [RecyclerView之绘制流程](/Android/UI/RecyclerView之绘制流程/)
- [RecyclerView之Adapter](/Android/UI/RecyclerView之Adapter/)  
- [RecyclerView之ItemDecoration](/Android/UI/RecyclerView之ItemDecoration/)
- [RecyclerView之ItemAnimator](/Android/UI/RecyclerView之ItemAnimator/)
- [RecyclerView之DiffUtil](/Android/UI/RecyclerView之DiffUtil/)
- [RecyclerView之缓存策略](/Android/UI/RecyclerView之缓存策略/)

## 相关类

- RecyclerView
  - SmoothScroller(RecyclerView内部类)：平滑的速度处理
    - LinearSmoothScroller：水平方向的处理，用于LinearSnapHelper中
  - ItemAnimator(RecyclerView内部类)：元素动画类
    - DefaultItemAnimator：默认动画类
    - SimpleItemAnimator：抽象类
  - Recycler(RecyclerView内部类)：保存缓存的类
- RecyclerView.State：测量状态
- RecyclerListener：当ViewHolder回收时的监听器
- LayoutManager：布局管理器基类
  - LinearLayoutManager：水平布局管理器
    - GridLayoutManager：方格布局管理器
  - StaggeredGridLayoutManager：不规则高度的方格管理器

- Adapter：视图和数据绑定的适配器，视图复用
  - ConcatAdapter
- ViewHolder：视图容器
- AdapterHelper： 管理和执行更新操作的帮助类，RecyclerView将每一次更新操作封装成了一个UpdateOp操作，然后通过AdapterHelper进行管理和执行。
- OpReorderer：记录操作指令
- ChildHelper：布局管理器和RecyclerView的child的处理器，有Callback接口可以对相应事件进行回调

- RecyclerViewPool：缓存池
- ItemDecoration：元素隔间，比如绘制分隔符
  - DividerItemDecoration
  - FastScroller
- ItemTouchHelper
- DiffUtil：用于比较前后数据的工具类，提升多个item更新的效率
- AsyncListDiffer
- AsyncListUtil
- ScrollbarHelper
- SnapHelper：用于辅助RecyclerView在滚动结束时将Item对齐到某个位置
  - LinearSnapHelper：水平速率计算的帮助类
  - PagerSnapHelper：类似ViewPage滑动速率帮助类
- StableIdStorage
- ItemTouchHelper
- AdapterListUpdateCallback
- OrientationHelper：LayoutManager用于测量child的一个辅助类，可以根据Layoutmanager的布局方式和布局方向来计算得到ItemView的大小位置等信息。
- ViewBoundsCheck
- ViewInfoStore：存放当前VH和相关的InfoRecord
  - InfoRecord（ViewInfoStore内部类）：VH的状态：出现、消失、预布局、实际布局

## 概述

RecyclerView 会回收这些单个的元素。当列表项滚动出屏幕时，RecyclerView 不会销毁其视图。相反，RecyclerView 会对屏幕上滚动的新列表项重用该视图。
这种重用可以显著提高性能，改善应用响应能力并降低功耗。

### 布局管理

- LinearLayoutManager：将各个项排列在一维列表中
- GridLayoutManager：将各个项排列在二维网格中，就像棋盘上的方格一样。
- StaggeredGridLayoutManager：将各个项排列在二维网格中，每一列都在前一列基础上稍微偏移，就像美国国旗中的星星一样。

### Adapter

通过Adapter来完成数据和视图的绑定，这里使用了模版方法模式，我们只需要完成具体的实现即可。

- onCreateViewHolder()：每当 RecyclerView 需要创建新的 ViewHolder 时，它都会调用此方法。此方法会创建并初始化 ViewHolder 及其关联的 View，
  但不会填充视图的内容，因为 ViewHolder 此时尚未绑定到具体数据。
- onBindViewHolder()：RecyclerView 调用此方法将 ViewHolder 与数据相关联。此方法会提取适当的数据，并使用该数据填充 ViewHolder 的布局。
  例如，如果 RecyclerView 显示的是一个名称列表，该方法可能会在列表中查找适当的名称，并填充 ViewHolder 的 TextView 微件。
- getItemCount()：RecyclerView 调用此方法来获取数据集的大小。例如，在通讯簿应用中，这可能是地址总数。RecyclerView 使用此方法来确定什么时候
  没有更多的列表项可以显示。

## 常用API

### ItemDecoration

设置单元格间的布局

### ItemAnimator

### RecycledViewPool

用于多个RecyclerView之间共享View

### ViewCacheExtension

### setNestedScrollingEnabled

禁止滑动

### setHasFixedSize

数据变化不会导致Item高度变化时，可设置此值。

### Header/Footer

## 数据操作

### API

- notifyItemChanged(int position) 更新列表position位置上的数据可以调用
- notifyItemInserted(int position) 列表position位置添加一条数据时可以调用，伴有动画效果
- notifyItemRemoved(int position) 列表position位置移除一条数据时调用，伴有动画效果
- notifyItemMoved(int fromPosition, int toPosition) 列表fromPosition位置的数据移到toPosition位置时调用，伴有动画效果
- notifyItemRangeChanged(int positionStart, int itemCount) 列表从positionStart位置到itemCount数量的列表项进行数据刷新
- notifyItemRangeInserted(int positionStart, int itemCount) 列表从positionStart位置到itemCount数量的列表项批量添加数据时调用，伴有动画效果
- notifyItemRangeRemoved(int positionStart, int itemCount) 列表从positionStart位置到itemCount数量的列表项批量删除数据时调用，伴有动画效果
- notifyDataSetChanged

### DiffUtils

比较两个数据集，找出差异，再调用notifyItemXXX方法进行更新。

## 缓存机制

![](./缓存机制.jpg)

1. Scrap：屏幕内部的ItemView，通过数据集的position来找到对应的Item，可以直接取过来用；
2. Cache：刚移出屏幕的ItemView，放到Cache里，当Cache里的ItemView重新进入屏幕时，也是通过position来找到对应的Item，直接可以使用，
   不需要走bindViewHolder()。Cache和 Scrap 一样，都是可以直接通过position来找到对应的Item，不需要重新绑定；
3. ViewCacheExtension：自定义缓存，如果有自定义，需要在这里面找，没有的话直接跳过；
4. RecycledViewPool：所有被废弃的ItemView的Pool，该pool里面的Item都是dirty的，需要通过ViewType来找到数据，找到数据的话，需要重新绑定，
   不走createViewHodler()，走bindViewHolder()。

### RecycledViewPool的用法

- 不同的RecyclerView共用一个RecycledViewPool
- RecyclerView的布局管理器需要设置recycleChildrenOnDetach
- setMaxRecycledViews的设置，设置不同类型的ViewHolder的缓存个数
  - Adapter中getItemViewType的方法设置ViewHolder的类型

## 优化设置

- onBindViewHolder内不要设置监听器
  - 如果在这里设置，会频繁的重复创建监听器。
- 高度固定时设置onBindViewHolder
- 共用RecycleViewPool

## 参考

- [https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=zh-cn](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=zh-cn)
- [RecycledViewPool使用](https://www.jianshu.com/p/122e68e9ddac)
- [一点点有助于巧用RecyclerView的小技巧](https://www.jianshu.com/p/bafd9ac9ffbd)
- [RecyclerView的好伴侣：详解DiffUtil](https://blog.csdn.net/zxt0601/article/details/52562770)
- [Speed up Your Android RecyclerView Using DiffUtil](https://www.raywenderlich.com/21954410-speed-up-your-android-recyclerview-using-diffutil)
