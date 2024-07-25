---
title: Android引擎启动流程
toc: true
tags: Flutter
---



- Embedder启动流程
  - FlutterEngine初始化
  - FlutterView初始化
  - Framework启动
  - Engine入口
- Engine启动流程
  - Engine关键类
  - JNI接口绑定
  - Settings解析
- Surface启动流程
  - Flutter绘制体系介绍
  - PlatformViewAndroid初始化
  - Surface初始化
- Dart Runtime启动流程
  - Dart Runtime介绍
  - Dart VM创建流程
  - Isolate启动流程
- Framework启动流程
  - Binding启动流程

Flutter是如何在Android的基础上启动的呢？查看Flutter项目的Android端代码，会发现Activity是继承FlutterActivity，Application是继承FlutterApplication的，那么他们是如何链接Native和Flutter的呢？

- FlutterApplication： onCreate过程中进行初始化配置，加载libflutter.so，注册JNI方法
- FlutterActivity：onCreate过程中进行创建FlutterView、Dart虚拟机、Enigine、Isolate、taskRunner等对象，最终执行到Dart的main方法

## FlutterApplication启动流程

## FlutterActivity启动流程

## 参考

- [深入理解Flutter引擎启动](http://gityuan.com/2019/06/22/flutter_booting/)
