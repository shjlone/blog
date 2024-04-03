---
title: RN常用的开源库
tags: React Native 
---


## antd-mobile-rn

- [https://rn.mobile.ant.design/](https://rn.mobile.ant.design/)

antd-mobile-rn是一个基于React Native的UI组件库，由蚂蚁金服团队开发和维护。包含了许多常用的UI组件，如按钮、列表、输入框、导航栏、轮播图、对话框等。这些组件都遵循了Ant Design的设计规范，具有一致的视觉样式和交互行为。


## async-retry

async-retry是一个用于在异步操作失败时进行重试的JavaScript库。它提供了一个简单的API，你可以使用它来包装一个异步函数，当这个函数抛出错误时，async-retry会自动重试这个函数。

```js
const retry = require('async-retry')

async function fetchData() {
  // 这里是你的异步操作
}

retry(fetchData, {
  retries: 5,  // 重试次数
  factor: 2,  // 每次重试之间的时间间隔将乘以这个因子
  minTimeout: 1000,  // 两次重试之间的最小时间间隔
  maxTimeout: Infinity,  // 两次重试之间的最大时间间隔
  onRetry: error => { console.log(error) }  // 在每次重试前调用
})
```

## axios 

- [https://www.axios-http.cn/docs/intro](https://www.axios-http.cn/docs/intro)

网络请求库

## babel-jest

babel-jest是一个Jest插件，用于使用Babel转换你的JavaScript代码。当你在Jest测试中使用ES6或其他需要转换的JavaScript语法时，你需要使用babel-jest。

## Jest

- [https://jestjs.io/](https://jestjs.io/)

Jest是一个流行的JavaScript测试框架，由Facebook开发并开源。它被设计为提供完整的测试解决方案，包括单元测试、集成测试和快照测试。

## Metro

- [https://facebook.github.io/metro/](https://facebook.github.io/metro/)

Metro是Facebook开发的一个JavaScript模块打包器，用于React Native应用。它负责将你的JavaScript代码和依赖项打包成一个单一的文件，这个文件可以在React Native应用中运行。

Metro的主要功能包括：

- 代码转换：Metro使用Babel将你的JavaScript代码转换为可以在React Native环境中运行的代码。

- 模块打包：Metro将你的所有JavaScript模块打包成一个单一的bundle文件。

- 资源管理：Metro可以处理图片和其他静态资源，将它们包含在bundle中。

- 热更新：Metro支持热更新，这意味着你可以在不重新打包整个应用的情况下，更新你的JavaScript代码。

- 源码映射：Metro可以生成源码映射，这对于调试和错误跟踪非常有用。


## react-hook-form

- [https://react-hook-form.com/](https://react-hook-form.com/)

react-hook-form是一个用于在React应用中处理表单的库。它使用React的钩子（Hooks）API，提供了一种简单、高效的方式来验证和收集表单数据。


## redux

状态管理框架



## rematch

Rematch是一个基于Redux的状态管理库，它提供了一种更简单的方式来使用Redux。它有以下插件：

- @rematch/core：它提供了Rematch的主要功能，包括定义models、创建store等。

- @rematch/select：它提供了一种简单的方式来创建和使用selectors。Selectors可以帮助你从store中选择和计算派生数据。

- @rematch/loading：它自动跟踪你的异步action的加载状态。这可以帮助你在UI中显示加载指示器。

- @rematch/updated：它提供了一种方式来跟踪你的state的更新状态。

- @rematch/persist：它提供了一种方式来持久化和恢复你的state。这可以帮助你在页面刷新或重新加载时保持用户的状态。

- @rematch/immer：它让你可以在reducers中使用Immer库来更简单地处理不可变状态。



## react-native

- [https://reactnative.dev/](https://reactnative.dev/)

React Native是一个由Facebook开发的开源框架，用于构建跨平台的移动应用。它允许你使用JavaScript和React来编写本地移动应用，而无需学习Swift或Java。

### react-native-code-push

react-native-code-push是一个用于React Native应用的开源库，它允许开发者通过Microsoft的CodePush服务直接向用户的设备推送代码更新。

### react-native-fast-image

它提供了比React Native内置的Image组件更高效的图片加载。iOS使用SDWebImage，Android使用Glide。

### react-native-get-random-values

它提供了一个实现了Web Crypto API的getRandomValues方法。这个方法可以生成一个包含随机数的类型化数组。在Web浏览器中，getRandomValues方法是Web Crypto API的一部分，用于生成加密安全的随机数。但在React Native中，Web Crypto API并不可用，因此我们需要react-native-get-random-values这个库来提供类似的功能。

### react-native-gesture-handler

它提供了一种在JavaScript中处理触摸手势的方式。这个库的目标是提供一个更强大、更灵活的手势系统，以替代React Native内置的手势系统。

### react-native-highlight-words

它可以在文本中高亮显示指定的单词或短语。

### react-native-drag-sort

提供了一个可以拖动排序的列表组件

### react-native-image-crop-picker

图片的选择和裁剪

### react-native-linear-gradient

线性渐变背景的功能

### react-native-modal

模态窗口组件

### react-native-ratings

评分组件

### react-native-reanimated

提供了一个更强大和灵活的方式来创建动画，有以下特点：

- 性能优化：react-native-reanimated在原生线程上执行动画，避免了JavaScript线程的阻塞，从而提高了动画的性能。

- 灵活的API：react-native-reanimated提供了一套低级API，你可以使用这些API来创建复杂的动画和交互。

- 与手势库集成：react-native-reanimated可以与react-native-gesture-handler库集成，使你可以创建与手势交互的动画。

### react-native-safe-area-context

提供一个安全区域的上下文，可以帮助你避免在设备的刘海、圆角、虚拟家庭指示器等特殊区域内放置内容。

### react-native-safe-area-view

视图组件，可以帮助你的应用适应设备的安全区域。

### react-native-screens

提供原生的屏幕组件

### react-native-size-matters

提供一个简单的API，可以帮助你在不同的设备上适配不同的尺寸。

### react-native-sound

### react-native-svg

### react-native-swiper

轮播图组件

### react-native-video

### react-native-view-shot

可以捕获某个视图的截图

### react-native-vector-icons

使用矢量图标

### react-native-collapsible

可折叠组件

### react-navigation

导航组件

### react-navigation-transitions

自定义屏幕之间的转场动画


## rne

React Native Elements是一个用于React Native应用的UI组件库，它提供了一套丰富的UI组件，包括按钮、图标、输入框、列表、卡片、轮播图等。这些组件都遵循了Material Design的设计规范，具有一致的视觉样式和交互行为。


## react-native-fs


## react-native-permissions

权限管理库

## react-native-device-info

获取硬件设备信息

react-native-fs是一个用于React Native应用的文件系统库，它提供了一套简单的API，可以帮助你在React Native应用中读写文件、创建目录、删除文件等。

## react-native-share

系统分享库，支持本地、网络文件。[https://www.npmjs.com/package/react-native-share](https://www.npmjs.com/package/react-native-share)

### @rneui/base

### @rneui/envinfo

### #rneui/themed

### @rneui/layout

### @rneui/kit

### #rneui/template



