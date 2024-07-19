---
title: setState具体做了什么
toc: true
tags: Flutter
---


![](./lifecycle_1.png)

## setState流程

1. 标脏，将对应element添加到dirtyElement队列中
2. 触发vsync
3. 下一帧drawFrame

- buildScope
  - rebuild
    - performRebuild
      - didChangeDependencies
      - build

### updateChild

调用setState()之后，它所有的子节点调用updateChild(Element child, Widget newWidget, dynamic newSlot)：

- 如果之前的位置child为null
  - 如果newWidget为null的话，说明这个位置始终没有子节点，直接返回null即可。
  - 如果newWidget不为null，说明这个位置新增加了子节点调用inflateWidget(newWidget, newSlot)生成一个新的Element返回
- 如果之前的child不为null
  - 如果newWidget为null的话，说明这个位置需要移除以前的节点，调用deactivateChild(child)移除并且返回nullD、如果newWidget不为null的话，先调用Widget.canUpdate(child.widget, newWidget)对比是否能更新。这个方法会对比两个Widget的runtimeType和key，
    1. 如果一致则说明子Widget没有改变，只是需要根据newWidget(配置清单)更新下当前节点的数据child.update(newWidget)；
    2. 如果不一致说明这个位置发生变化，则**deactivateChild(child)**后返回**inflateWidget(newWidget, newSlot)**;

### GlobalKey

## 参考

- [面试官问我State的生命周期，该怎么回答](https://juejin.cn/post/6908574202253541389)
