---
title: React Native项目
tags: Harmony 
toc: true
---

[https://gitee.com/openharmony-sig/ohos_react_native](https://gitee.com/openharmony-sig/ohos_react_native)

如何适配鸿蒙？

- 在现行的 React Native 中，有很多属性是在 React 侧完成的封装，也有很多属性是平台独有的。为了达成这个效果，React Native 在 JS 侧根据 Platform 增加了很多判断。所以，React Native 的鸿蒙化适配也需要增加 HarmonyOS 相关的平台判断，与相应的组件属性的封装。为此，鸿蒙化团队提供了 react-native-harmony 的 tgz 包，并通过更改 metro.config.js 配置，将该 tgz 包应用到 Metro Bundler 中。
- React Native 还提供了很多库的封装，例如 Codegen、打包工具等。为此，鸿蒙化团队提供了 react-native-harmony-cli 的包，对这些库进行了 HarmonyOS 平台的适配，用于向开发者提供相关的功能。

Fabric
TurboModule：

- ArkTSTurboModule：
- cxxTurboModule：

RNOH线程模型：

- MAIN：
- JS：
- BACKGROUND：
- WORKER：

相关API

- RNAbility：继承自UIAbility，封装了启动RN的必要操作
- RNApp：是用于启动和管理 RNInstance 和 RNSurface 的模块，封装了创建与启动单个 RNInstance 和单个 RNSuface 的行为
  - 配置appkey，和JS侧registerComponent注册的appkey关联；
  - 配置初始化参数initialProps，传递给JS；
  - 配置jsBundleProvider，指定bundle加载路径；
  - 配置rnInstanceConfig，指定开发者自定义package，注入字体文件fontResourceByFontFamily，设置BG线程开关，设置C-API开关
  - 持有RNSurface，作为RN页面容器
- RNSurface：
  - 页面容器，持有XComponent用于挂载ArkUI的C-API节点和响应手势事件
- SurfaceConfig：RNSurface的配置参数，拥有两个子类： SurfaceConfig1、 SurfaceConfig2，开发者在使用的时候可以根据需要分别选择不同的 config
  - SurfaceConfig2:
    - appKey：AppRegister.registerComponent 注册的 appKey
    - initialProps：传递给 JS 的初始化参数
- RNInstance：
  - 创建RN实例的步骤：
    1. 获取RNInstance的id
    2. 注册TurboModule：在RNInstance.ts中调用processPackage方法注册系统自带和自定义在UI线程上的TurboModule
    3. 注册字体
    4. 注册RN官方能力和开发者自定义能力：RNInstanceFactory.h 中通过 PackageProvider.cpp 的 getPackage 方法获取 RN 系统自带和开发者自定义 TurboModule，接着注册系统 View、系统自带 TurboModule、开发者自定义 View、开发者自定义 TurboModule
    5. 注册ArkTS混合组件
    6. 初始化JS引擎
    7. 注册TM的JSI通道
    8. 注入Scheduler
    9. 注册Fabric的JSI通道
- RNOHCoreContext：提供可跨RNInstances共享的依赖项和实用程序，还包括创建和销毁RNInstance的方法
- RNComponentContext：是 React Native for OpenHarmony 构造组件时使用的上下文信息，是 RNOHContext 的子类
- JSBundleProvider：JS Bundle 提供者，用于初始化 bundle 信息，获取 bundle 具体内容。本节主要介绍了 RNOHCoreContext 的接口类型。
  - AnyJSBundleProvider：JSBundleProvider 的抽象类，用于提供 JS Bundle 的抽象方法
  - MetroJSBundleProvider: 使用Metro服务加载bundle
  - FileJSBundleProvider: 从沙箱目录下加载bundle
  - ResourceJSBundleProvider： 加载reources/rawfile下的bundle文件
- RNInstancesCoordinator：跟灵活的控制RN启动

## native侧接入流程

1. entry中添加react-native-openharmony依赖
2. CMakeLists.txt配置相关依赖
3. entry\build-profile.json5配置CMakeLists.txt
4. 入口文件EntryAbility继承RNAbility
5. 添加RNPackagesFactory.ets文件，注册createRNPackages
6. RN的Page中包装RNSurface

## 如何编写native module

```shell

BaseRN load bundle
BaseRN({
  rnInstance: this.instance,
  moduleName: this.moduleName,//bundle name
  bundlePath: this.bundlePath,//本地的bundle路径，resources/rawfile/bundle
})


```



## 第三方库

[https://gitee.com/react-native-oh-library/docs](https://gitee.com/react-native-oh-library/docs)
[https://gitee.com/react-native-oh-library/usage-docs](https://gitee.com/react-native-oh-library/usage-docs)
[RNOH三方库源码地址](https://github.com/orgs/react-native-oh-library/repositories)




## ohos_react_native官方demo运行注意事项


1. 代码clone后，需要更新子仓库代码

```shell
git clone https://gitee.com/openharmony-sig/ohos_react_native

# 一定要更新子仓库代码
git submodule update --init

# 更新boost子仓库代码
pushd tester/harmony/react_native_openharmony/src/main/cpp/third-party/boost
git submodule update --init
popd

```

2. 本地编译react_native_harmony_cli和react_native_harmony两个项目，先构建cli，后者依赖前者，pack之前先修改package.json里面的依赖，改成本地依赖


3. 打包tester源码har



## 如何引入第三方库

[https://gitee.com/react-native-oh-library/usage-docs/](https://gitee.com/react-native-oh-library/usage-docs/)