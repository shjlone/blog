---
title: Android系统架构
tags: Android
toc: true
---


![](./android-stack_2x系统架构.png)

## 系统架构

### Linux内核层

Linux Kernel：Android 的核心系统服务基于Linux 内核，在此基础上添加了部分Android专用的驱动。系统的安全性、内存管理、进程管理、网络协议栈和驱动模型等都依赖于该内核。

### 硬件抽象层

Hardware Abstraction Layer：对Linux内核驱动程序的封装，向上提供接口，向下屏蔽了具体的实现细节。硬件抽象层是位于操作系统内核与硬件电路之间的接口层，
其目的在于将硬件抽象化，为了保护硬件厂商的知识产权，它隐藏了特定平台的硬件接口细节，为操作系统提供虚拟硬件平台，使其具有硬件无关性，可在多种平台上进行移植。
从软硬件测试的角度来看，软硬件的测试工作都可分别基于硬件抽象层来完成，使得软硬件测试工作的并行进行成为可能。通俗来讲，就是将控制硬件的动作放在硬件抽象层中。

### 系统运行层

Native C/C++ Libraries：系统运行层分为C/C++运行时库和Android运行时环境。

Android运行时环境在4.4以前使用的是Dalvik，之后使用ART。从5.0开始，正式废弃了Dalvik。

#### Dalvik

**什么是Dalvik？**

- Dalvik是用于Android平台的Java虚拟机
- Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一
- 它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行
- dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统
- Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行
- 独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭

**特点**

- Dalvik是依靠一个Just-In-Time(JIT编译)编译器去解释字节码
- Dalvik虚拟机下运行Java时，要将字节码通过即时编译器（just in time ，JIT）转换为机器码（机器码才是能真正运行的），这会拖慢应用的运行效率
- 应用安装时，执行dexopt指令，将dex文件优化为odex文件
- 应用运行时，会将二进制翻译成机器码流程

#### Android Runtime

**Android Runtime特点**

- 应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用，这个过程叫做预编译（AOT,Ahead-Of-Time）
- 在移除解释代码这一过程后，应用程序执行将更有效率，启动更快
- 系统性能的显著提升
- 垃圾回收方面的优化

### 应用框架层

Application Framework：应用框架层，提供了应用开发的核心功能。在实际开发中会使用里面的API。

名称 | 描述
--- | ---
Activity Manager(活动管理器)  | 管理各个应用程序生命周期以及通常的导航回退功能
Location Manager(位置管理器) | 提供地理位置以及定位功能服务
Package Manager(包管理器)  |管理所有安装在Android系统中的应用程序
Notification Manager(通知管理器) | 使得应用程序可以在状态栏中显示自定义的提示信息
Resource Manager（资源管理器） | 提供应用程序使用的各种非代码资源，如本地化字符串、图片、布局文件、颜色文件等
Telephony Manager(电话管理器) | 管理所有的移动设备功能
Window Manager（窗口管理器）|  管理所有开启的窗口程序
Content Providers（内容提供器）|  使得不同应用程序之间可以共享数据
View System（视图系统）|  构建应用程序的基本组件

### 应用层

System Apps：这里存放的是Android自带的一些App，比如：电话、短信、图库、拍摄等。

## 源码目录

可以从 [这里](https://cs.android.com/android/platform/superproject) 在线阅读源码，也可以从 [https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/) 下载源码。

以下是Android9的源码目录结构：

- art：全新的ART运行环境
  - dalvlkvm：dalvik 运行时代码
  - dex2oat：
  - dexdump：
  - dexlayout：
  - disassembler：
  - openjdk|vm：
  - openjdk|jvmti：
  - runtime：
  - simulator：模拟器
  - tools：
    - ahat：Android堆栈分析工具
    - amm：Actionable Memory Metric
- bionic：google自己开发的内核库，比GNU的内核更适合移动设备
  - apex：
  - libc：对系统调用的封装
    - arch-arm：
    - arch-arm64：
    - arch-common：
    - arch-x86：
    - arch-x86_64：
    - stdio：标准IO
    - tools：一些python工具脚本
  - libdl：   用于动态库的装载
  - libm：数学库
    - upstream-freebsd：很多来自FreeBSD的函数库
  - libstdc++：标准的C++的功能库
  - linker：链接模块
  - tools：
- bootable：启动引导相关代码
  - recovery：这个目录用于创建恢复程序
- build：存放系统编译规则及generic等基础开发包配置
  - bazel：
  - blueprint：
  - make：
    - common：
    - core：构建系统的核心目录
      - clang：
      - combo：编译、编译设置
      - tasks：
    - envsetup.sh
    - target：
      - board：构建目标设备的配置
      - product： 哪些apps需要编译
    - tools：编译过程中需要用到的工具
      - acp：
      - aplcheck：
      - atree：
      - check_prereq：
      - drolddoc：
      - rbb2565：
      - zipalign：
  - pesto：
  - soong：

- compatibility
- cts：Android兼容性测试套件标准
- dalvik：dalvik虚拟机
  - dexdump：
  - dexgen：
  - dexlist：
  - dx：
  - opcode-gen：
- developers：开发者目录  
  - build：
  - demos：
  - docs：
  - samples：
- development：应用程序开发相关
  - apps：包含没有部署到系统到应用
    - BluetoothDebug：
    - SdkSetup：
  - build：
    - tools：构建过程中需要用到的一些工具
  - cmds：包含monkey tool
  - python-packages：
  - host：
    - windows：包含Windows版USB驱动
  - ide：包含对IDE一些配置信息
    - clion
    - eclipse
    - intellij
  - sdk：
  - scripts：
  - tools：
    - apkcheck：APK检查工具
    - axl：TCP、HTTP测试
    - elftree：
    - idegen：
    - emulator：
    - bugreport：
    - ndk：
    - ota_analysis：
    - otagui：
    - winscope：
    - monkey：模拟用户点击的测试工具
  - vendor_snapshot：
  - vndk：
- device：设备相关配置
  - amlogic：
  - common：
  - generic：包含不同设备的配置信息
    - arm64：
    - art：
    - goldfish：
    - goldfish-opengl：
    - x86：
  - google：
  - google_car：
  - linaro：
  - mediatek：
  - ti：
- external：开源模组相关文件，可以理解成第三方库的依赖
  - ImageMagick：
  - FXdiv：
  - OpenCL-CTS：
  - aac：
  - adhd：
  - adt-infra：
  - android-clat：
  - androidplot：
  - angle：
  - antlr：[http://www.antlr.org](http://www.antlr.org)
  - apache-commons-bcel：
  - apache-commons-compress：
  - apache-commons-math：
  - apache-harmony：
  - apache-http：
  - apache-xml：
  - auto：
  - clang：
  - bsdiff：
  - chromium-libpac：
  - chromium-trace：
  - chromium-webview：
  - cpuinfo：
  - curl：
  - dagger2：
  - dexmaker：
  - exoplayer：
  - libogg：
  - libopus：
  - libcap：
  - libpng：
  - lzma：
  - skia：[http://code.google.com/p/skia/](http://code.google.com/p/skia/)
  - v8：Javascript引擎
  - webp：[http://code.google.com/speed/webp](http://code.google.com/speed/webp)
  - webrtc：[http://www.webrtc.org](http://www.webrtc.org)
- frameworks：应用程序框架，Android系统核心部分，由Java和C++编写
  - av：
    - camera：
    - media：C实现系统媒体库
  - base：
    - api：
    - boot：
    - cmds：重要命令
      - am：
      - app_process：
      - pm：包管理工具
    - config：
    - data：包含字体文件、音频文件、视频文件等
    - location：
    - media：多媒体相关库
      - java：
      - jni：
        - audioeffect：
        - soundpool：
      - mca：
      - native：
    - multidex：
    - native：本地库
    - opengl：2D/3D 图形API
    - sax：XML解析器
    - wifi：wifi无线网络
    - packages：
      - BackupRestoreConfirmation：
      - DefaultContainerService：
      - SystemUI：
      - Shell：
      - VpnDialogs：
  - compile：
  - ex：
  - hardware：
  - layoutlib：
  - libs：
  - native：
    - opengl：第三方图形渲染库
    - services：
      - audiomanager：
      - batteryservice：
      - displayservice：
      - gpuservice：
      - inputflinger：
      - surfaceflinger：图形显示库，主要负责图形的渲染、叠加和绘制等功能
      - sensorservice：
      - vr：
- hardware：主要是硬件抽象层的代码
  - broadcom：
  - google：
  - libhardware：
  - libhardware_legacy：
  - nxp：
  - samsung：
  - st：
- kernel：
- libcore：核心库相关文件
  - api：
  - dalvik：
  - dom：
  - json：
  - xml：
- libnativehelper：动态库，实现JNI库的基础
- packages：自带Apps应用程序包
  - apps：系统App
    - Bluetooth：
    - Calendar：
    - Camera2：
    - Dialer：
    - Launcher3：
  - inputmethods：输入法目录
  - modules：
    - ArtPrebuilt:
    - DnsResolver: DNS解析
    - Permission: 权限
    - adb：adb工具
  - providers：内容提供者目录
    - CalendarProvider:
    - DownloadProvider:
    - MediaProvider:
    - TelephonyProvider:
  - screensavers：屏幕保护
  - services：通信服务
    - AlternativeNetworkAccess：
    - Telephony
  - wallpapers：墙纸
- pdk：Plug Development Kit 的缩写，本地开发套件
- platform_testing：平台测试
- prebuilts：x86和arm架构下预编译的一些资源
  - bazel：
  - clang：
  - devtools：
  - gcc：
  - go：
  - gradle-plugin：
  - jdk：
  - python：
  - mlsc：
  - ndk：
  - runtime：
  - tools：
    - common：
    - darwln-x86：
    - linux-x86：
    - linux-x86_64：
- sdk：sdk和模拟器
  - annotations：
  - apkbuilder：这个废弃了，推荐直接使用com.android.sdklib.build.ApkBuilder
  - find_java：
  - find_java2：
  - sdklauncher：
  - hierarchyviewer：视图查看器
- system：底层文件系统库、应用和组件
  - apex：
  - bpf：
  - core：
    - debuggerd：
    - logcat：
    - toolbox：
    - watchdogd：
  - extras：
    - ANRdaemon：
    - app-launcher：
    - su：
    - sound：播放wav文件工具
    - toolchain-extras：
- test：
- toolchain：工具链文件
- tools：工具文件

## 参考

- [https://elinux.org/Master-android](https://elinux.org/Master-android)
- [Android 虚拟机Art和Dalvik的区别](https://juejin.cn/post/6844903958918463495)
- [https://source.android.com/devices/tech/dalvik?hl=zh-cn](https://source.android.com/devices/tech/dalvik?hl=zh-cn)
- [Android 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
