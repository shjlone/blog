---
title: 一些Flutter开发技巧
toc: true
tags: Flutter
---



## 使用Text的注意事项

### 如果字符串中有特殊字符，在有多行的情况下，可能会出现异常截断的情况，建议封装一个扩展函数处理这种情况

```dart
import 'package:flutter/material.dart';

// https://github.com/flutter/flutter/issues/18761
extension StringExt on String {
  String get overflow => Characters(this).replaceAll(Characters(''), Characters('\u{200B}')).toString();
}
```


## 编写列表页时的注意事项

- 骨架屏的设置
- 上下拉刷新的设置
- 各种异常页面的展示（无数据、请求异常、正常情况）


## 生命周期的监听等要记得释放监听器

## 使用XXController时记得在dispose方法中释放资源


## 使用GestureDetector时设置behavior: HitTestBehavior.opaque,可以避免点击事件穿透

```dart
GestureDetector(
  behavior: HitTestBehavior.opaque,
  child: Container(),
  onTap: () {
    // do something
  },
)
```

