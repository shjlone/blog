---
title: ContentProvider
tags: Android
toc: true
---

## 概述

`ContentProvider`是一种数据共享型组件，可以在应用之间共享数据。所以与BroadcastReceiver一样，其可以脱离Activity实现。在实现ContentProvider时，
需要继承ContentProvider抽象类，然后在AndroidManifest.xml中注册类名和ContentProvider的域名。同样的，不需要重写onCreat()方法，而是实现CRUD操作，
所以ContentProvider没有启动和停止的概念，更像是一个系统级的监听器。与前三个组件不同的是，ContentProvider并没有使用intent，
而是使用URI来判定能否为ContentResolver提供数据共享。

关于ContentProvider，用来提供其他地方（包括其他App）调用的一种全局（系统级）方式。有了ContentProvider，我们就能方便的调用相册的东西、
进行文件选择，在我们自己的App中，也可以提供一个数据中心。


当数据量比较大的时候，继续用Parcel做容器效率会比较低，因此它还使用了匿名共享内存的方式。

但是有一个问题是，ContentProvider的提供者进程不再存活时，其他进程通过Provider读一个非常简单的数据时，都需要先把提供者进程启动起来(除非
指定multiprocess=true)，这对用户是相当不友好的。又因为其是间接通过db进行数据操作，所以效率也远不如直接操作db。因此在用户app中，
不是很建议经常使用ContentProvider。不过对于系统级的app，它统一了数据操作的规范，利是远大于弊的。

在很多开源库中，也运用了ContentProvider的特性来进行初始化。

## 一些注意点

> 对于面向 Android 7.0 的应用，Android 框架执行的 StrictMode API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常。

因为Android7的变化，所以在使用FileProvider时需要做一些处理，关于如何处理，网上一大把资料，总结出来需要以下步骤：

1. 配置Manifest文件，添加provider
2. 对于Android7以上，在FileProvider.getUriForFile时使用配置的authority




## 参考

- [官方使用手册](https://developer.android.com/guide/topics/providers/content-provider-basics?hl=zh-cn)
- [https://blog.csdn.net/lmj623565791/article/details/72859156](https://blog.csdn.net/lmj623565791/article/details/72859156)
- [https://developer.android.com/topic/libraries/app-startup](https://developer.android.com/topic/libraries/app-startup)
