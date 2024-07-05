---
title: flutter_easy_refresh
toc: true
tags: Flutter
---


Flutter的刷新组件以前比较流行的是pull_to_refresh，现在已经不再维护了，新的刷新组件是easy_refresh，支持Flutter SDK 3.x。它的特点：


- 支持所有的滚动组件
- 滚动物理作用域，精确匹配滚动组件
- 集成多个炫酷的 Header 和 Footer
- 支持自定义样式，实现各种动画效果
- 支持下拉刷新、上拉加载(可使用控制器触发和结束)
- 支持指示器位置设定，结合监听器也放置在任何位置
- 支持页面启动时刷新，并自定义视图
- 支持安全区域，不再有遮挡
- 自定义滚动参数，让列表具有不同的滚动反馈和惯性


```dart
///控制刷新和加载
  EasyRefreshController _controller = EasyRefreshController(
    controlFinishRefresh: true,
    controlFinishLoad: true,
  );
  ....
  EasyRefresh(
    header: MaterialHeader(),//设置自定义头部
    footer: MaterialFooter(),//设置自定义底部
    controller: _controller,
    onRefresh: () async {
      ....
      _controller.finishRefresh();//刷新完成
      _controller.resetFooter();
    },
    onLoad: () async {
      ....
      _controller.finishLoad(IndicatorResult.noMore);//根据数据来判断是否还有更多
    },
    ....
  );
  ....
  _controller.callRefresh();//触发刷新
  _controller.callLoad();//触发加载

```


## 参考

- [https://github.com/xuelongqy/flutter_easy_refresh](https://github.com/xuelongqy/flutter_easy_refresh)
- [https://github.com/peng8350/flutter_pulltorefresh](https://github.com/peng8350/flutter_pulltorefresh)
- [https://pub.dev/packages/easy_refresh](https://pub.dev/packages/easy_refresh)
