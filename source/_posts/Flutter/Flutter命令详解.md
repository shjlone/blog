---
title: Flutter命令详解
toc: true
tags: Flutter
---



## create

在指定的目录中,创建新的flutter项目,如果没有指定目录,则在当前目录下创建项目

```shell
flutter create ~/flutter #在家目录下的flutter目录项目
flutter create . #在当前目录下创建

flutter create -t module flutter_module  # 创建一个module

```

## -v

查看APP所有日志的输出,对于调试是非常有用处,在调试时需要配合run命令使用

```shell
flutter -v run
```

## -d

切换在不同设备上运行app,如果没有指定设备,默认将会使用设备列表的第一个设备,这对于计算机连接多个设备时非常有用,可以使用设备名称或者设备id作为参数

```shell

flutter run -d NX569J #设备名称
flutter run -d devices_id #设备id
```

## analyze

编辑Flutter代码时，使用分析器检查代码是非常重要,默认是分析整个项目的代码,你也可以通过使用`analysis_options.yaml`文件来排除不需要的代码分析

`analysis_options.yaml`

```
analyzer:
exclude:
- flutter/**
```

有时候你可能需要代码分析一直在运行,可以使用--watch选项

flutter analyze --watch

当运行分析命令flutter都执行一次`pub get`命令,如果不需要运行,可以执行以下命令

flutter analyze --no-pub

## attach

相当于命令flutter run命令,不同之处在很多执行都是自己手动,比如热重载,

## build

构建应用程序的apk,appbundle,aot,iOS, iOS应用需要在Mac上构建

```shell
flutter build apk
flutter build apk --debug

flutter build appbundle
```

## channel

切换flutter不同的版本,在执行flutter channel会输出不同分支信息,默认使用stable分支。

有四个分支：

- master
- dev
- beta
- stable

```shell

flutter channel # 输出channel
flutter channel dev # 切换到dev channel
```

## clean

删除`build/`和`.dart_tool/`目录,清除缓存信息,避免之前不同版本代码的影响

## config

可以用于指定gradle,android sdk,android studio的目录或者开启,禁用analytics选项,analytics选项用于flutter工具的报告

```shell
flutter config --gradle-dir /gradle/
```

## devices

列出已经连接到计算机的设备

```shell
flutter devices
NX569J                    • 192.168.43.1:5555 • android-arm64 • Android 7.1.2 (API 25)
Android SDK built for x86 • emulator-5554     • android-x86   • Android 9 (API 28) (emulator)
```

## doctor

检查开发工具链是否完整安装,对于安装环境非常有用处

```shell
flutter doctor

Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, v1.2.1, on Linux, locale en_US.UTF-8)
[✓] Android toolchain - develop for Android devices (Android SDK version 28.0.3)
[✓] Android Studio (version 3.3)
[✓] VS Code (version 1.33.1)
[✓] Connected device (2 available)

• No issues found!
```

## drive

执行flutter ui测试,该工具类似与web端的`Selenium`,`WebDriver`,`Protractor`.你可以指定不同模式进行测试,可以是debug,profile,release,flavor模式,flavor模式可以指定平台规范,你还可以指定在不同的平台测试,甚至可以指定页面路由

```shell
flutter drive --debug --target-platform android-x86
```

## emulators

列出,创建,启动模拟器,默认是列出模拟器

```shell

flutter emulators --launch flutter_emulator #启动
flutter emulators # 列出
flutter emulators --create Pixel_API_28 # 创建
```

## format

按照dart代码规范格式项目代码文件,`flutter format .`是当前项目所有文件,也可以指定目录或者文件

```shell
flutter format dartfile
```

## run

参考：[源码解读Flutter run机制](http://gityuan.com/2019/09/07/flutter_run/)

1. flutter build apk：通过gradle来构建APK，由以下两部分组成：
    - flutter build aot，分为如下两个核心过程，该过程详情见下一篇文章
    - frontend_server前端编译器生成kernel文件
    - gen_snapshot来编译成AOT产物 - flutter build bundle，将相关文件放入flutter_assets目录
2. 通过adb install来安装APK
3. 通过adb am start来启动应用

```shell
# profile方式运行，对于Profile模式，启动Observatory调试器，可通过127.0.0.1:xxx地址，打开网页版性能分析工具Observatory，其中包含有timeline工具，类似于Android的systrace。
flutter run --profile --disable-service-auth-codes --local-engine-src-path=<FLUTTER_ROOT>/engine/src --local-engine android_profile

# 关闭鉴权
flutter run --disable-service-auth-codes

```

参数 | 说明
--- | ---
–debug | 调试版本，这是默认模式
–profile|  profile版本
–release | 发布版本
–target-platform|  指定app运行的目标平台，比如android-arm/android-arm64，iOS平台不可用
–target=|  主入口，默认值lib/main.dart
–observatory-port|  指定observatory端口，默认为0（随机生成可用端口）
–disable-service-auth-codes|  关闭observatory服务鉴权
–trace-startup|  跟踪应用启动/退出，并保存trace到文件
–trace-skia | 跟踪skia，用于调试GPU线程
–trace-systrace|  转为systrace，适用于Android/Fuchsia
–dump-skp-on-shader-compilation|  转储触发着色器编译的skp，默认关闭
–verbose-system-logs | 包括来自flutter引擎的详细日志记录
–enable-software-rendering | 开启软件渲染，默认采用OpenGL或者Vulkan
–skia-deterministic-rendering | 确定性采用skia渲染
–no-hot | 可关闭热重载，默认是开启
–start-paused|  应用启动后暂停
–local-engine-src-path | 指定本地引擎源码路径，比如xxx/engine/src
–local-engine | 指定本地引擎类型，比如android_profile

## verion

列出或者切换flutter版本,默认是列出所有版本

```shell
flutter version
flutter version v1.5.8
```

## upgrade

更新flutter代码,实质就是git代码更新拉取,下载flutter sdk是git仓库的打包

## test

运行flutter单元测试,可以使用`--start-paused`模式等待调试器的连接,`--concurrency`可以指定并发任务数默认值是6

```shell
flutter test --concurrency=8
```

## install

安装app到一个已经连接的设备

```shell
flutter install
```

## screenshot

截取当前屏幕,默认是将图片输出到家目录下,使用-o指定输出目录

```shell
flutter screenshot -o /home/work
```

## packages

获取,测试,更新依赖包,`flutter pub` 将会传递剩余参数到dart工具的pub

## pub

```shell
flutter pub get #获取依赖

flutter pub deps    #用于显示当前Package使用的所有依赖项

flutter pub cache add/clear/repair  #管理本地Package缓存

flutter pub upgrade #检索当前Package所依赖的其他Package的最新版本，如果pubspec.lock存在，则忽略其保存的版本并以pubspec文件中指定的最新版本为主。



pub run 命令可以调用当前 Package 或当前 Package 所依赖的其它 Package 中的 Dart 脚本。

pub global 命令可以让你使用那些可以全局可用的 Package。

pub publish #发布

```

## 修改源码调试

> /path/to/flutter/bin/cache/flutter_tools.stamp

当你修改了flutter tools的源码，删除上面这个文件，代码即可生效

## 参考

- [Flutter Tools 断点调试](https://fucknmb.com/2019/03/21/Flutter-Tools-%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95/)
