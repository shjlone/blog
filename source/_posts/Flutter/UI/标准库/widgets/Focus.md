---
title: Focus
toc: true
tags: Flutter
---

用于光标管理

## 相关类

### FocusNode

用于Widget获取键盘焦点和处理键盘事件的对象

```dart
class FucusNode {

  /// 请求焦点
 void requestFocus([FocusNode? node]) {}

  /// 释放焦点
  /// disposition表示释放后的行为， scope表示向上寻找最近的FocusScopeNode，previouslyFocusedChild表示寻找上一个焦点位置
 void unfocus({    UnfocusDisposition disposition = UnfocusDisposition.scope,  }) {}

}
```

### FocusScopeNode

FocusScopeNode继承自FocusNode

### Focus

Focus是一个Widget，内部管理者一个FocusNode，监听焦点的变化。源码中很多地方都使用了Focus，比如MaterialApp、InkWell等。

### FocusScope

FocusScope继承Focus，允许你在多个焦点节点之间进行导航和管理焦点状态。FocusScope 通常用于处理复杂的焦点管理场景，例如在表单中有多个输入字段时。

## 参考

- [说说Flutter中的无名英雄 —— Focus](https://blog.csdn.net/qq_17766199/article/details/107132031)
