---
title: Android sdk目录结构
tags: Android
toc: true
---


- sdk
  - add-ons：第三方公司为Android平台开发的附加功能系统
  - build-tools：构建工具
    - 28.0.3：
      - aapt.exe：打包res资源文件，生成R.java、resources.arsc和res
      - aapt2.exe
      - aidl.exe
      - apksigner.bat
      - bcc_compat.exe
      - d8.bat
      - dexdump.exe
      - dx.bat
      - llvm-rs-cc.exe
      - mainDexClasses.bat
      - split-select.exe
      - zipalign.exe
  - cmake
  - docs：API文档
  - emulator
  - extras：存放Android support v4、v7、v13、v17包
  - fonts
  - licenses
  - lldb：C/C++调试器，它与LLVM编译器一起使用，提供了丰富的流程控制和数据检测
  - ndk-bundle
    - ndk-build.cmd
    - ndk-depends.cmd
    - ndk-gdb.cmd
    - ndk-stack.cmd
    - ndk-which.cmd
  - patcher
  - platforms：根据API level存放不同版本的Android系统
    - android-28：28表示版本
      - data：系统资源
      - optional
      - skin：Android模拟器的皮肤
      - android.jar
      - uiautomator.jar
  - platform-tools：Android平台通用工具
    - adb.exe
    - dmtracedump.exe
    - etc1tool.exe：PNG图像压缩为etc1标准
    - fastboot.exe：刷机工具
    - hprof-conv.exe：hprof文件转换命令，将Android Studio工具生成的hprof文件转换成一个标准格式
    - make_f2fs.exe
    - mke2fs.exe：建立ext2文件系统
    - sqlite3.exe：数据库工具
  - sources
  - system-images：模拟器映像文件
  - tools：Android开发和调试工具
    - bin
      - archquery.bat
      - avdmanager.bat
      - jobb.bat：处理APK扩展文件的工具
      - lint.bat：代码检测
      - monkeyrunner.bat：测试工具
      - sdkmanager.bat:SDK管理器
      - uiautomatorviewer.bat：测试工具
    - android.bat
    - emulator.exe
    - emulator-check.exe
    - mksdcard.exe：使用模拟器时，用来创建sd卡
    - monitor.bat
  
  - compileSdkVersion：告诉Gradle用哪个Android SDK版本编译你的应用，使用任何新添加的API都需要对应等级的Android SDK。修改compileSdkVersion不会改变运行时的行为。如果使用Support Library，那么使用最新发布的 Support Library 就需要使用最新的 SDK 编译。例如，要使用 23.1.1 版本的 Support Library ，compileSdkVersion 就必需至少是 23 （大版本号要一致！）。
  - minSdkVersion：设置应用可以运行的最低版本要求
  - targetSdkVersion：系统会根据这个值来应用最新的行为变化。比如API 23会把你的应用转换到运行时权限模型。当设置targetSdkVersion小于23时，在6.0的机器上不会动态申请权限。
  - buildToolsVersion：构建工具的版本，其中包括aapt、dx等，这个工具的目录在Android sdk/build-tools/

## 参考

- [https://medium.com/androiddevelopers/picking-your-compilesdkversion-minsdkversion-targetsdkversion-a098a0341ebd#.tz5zzucma](https://medium.com/androiddevelopers/picking-your-compilesdkversion-minsdkversion-targetsdkversion-a098a0341ebd#.tz5zzucma)
