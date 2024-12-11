---
title: pub get过程
toc: true
tags: Flutter
---

flutter pub get实际上调用的是dart pub get，pub是Dart的包管理工具，用于管理Dart项目的依赖关系。pub get命令会根据项目根目录下的pubspec.yaml文件中的dependencies和dev_dependencies字段，下载并安装依赖包。

Flutter SDK中包含了pub工具，所以在Flutter项目中可以直接使用pub命令。而具体的pub代码在[https://github.com/dart-lang/pub/blob/master](https://github.com/dart-lang/pub/blob/master)

pub get会生成pubspec.lock文件，存储当前项目的依赖，减少后续的pub get时间。依赖存储在.pub-cache文件夹中。cache文件夹存储了git相关信息，会根据项目依赖情况下载对应源码。resolved-ref保存项目依赖对应的git commit hash。

这个过程也会生成一些临时文件，比如.dart_tool中的flutter_build/dart_plugin_registrant.dart，会注册所有的Flutter插件

## 参考

- [https://github.com/dart-lang/pub/blob/master](https://github.com/dart-lang/pub/blob/master)
- [Dart的包管理命令解析-pub get](https://juejin.cn/post/6856402321966891022)
