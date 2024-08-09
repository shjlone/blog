---
title: Intent
toc: true
tags: Android
---


## 介绍

Intent是一个消息传递对象，他的作用：

- 启动Activity。不同的业务场景启动Activity又分为显示调用和隐式调用。
- 启动Service
- 传递广播

Intent的属性有:component(组件)、action、category、data、type、extras、flags;所有的属性也是各显神通，满足开发者的各种需要满足不同场景;

- component: 显然就是设置四大组件的， 将直接使用它指定的组件，借助这一属性可以实现不同应用组件之间通讯;
- action: 是一个可以指定目标组件行为的字符串，开发人员可以自定义action通过匹配action实现组件之间的隐式跳转，当然Android系统也已经预定部分String作为系统应用Action，例如打开系统设置页面等等;
- data: 通常是URI类型或者MIME类型格式定义的操作数据;表示与动作要操纵的数据
- Category: 属性用于指定当前动作(Action)被执行的环境;
- type: 对于data范例的描写;

Intent.setFlags(int flags)设置启动模式

1. FLAG_ACTIVITY_CLEAR_TOP : 等同于mainfest中配置的singleTask
2. FLAG_ACTIVITY_SINGLE_TOP: 同样等同于mainfest中配置的singleTop;
3. FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS: 其对应在AndroidManifest中的属性为android:excludeFromRecents=“true”,当用户按了“最近任务列表”时候,
   该Task不会出现在最近任务列表中，可达到隐藏应用的目的。
4. FLAG_ACTIVITY_NO_HISTORY: 对应在AndroidManifest中的属性
   为:android:noHistory=“true”，这个FLAG启动的Activity，一旦退出，它不会存在于栈中。
5. FLAG_ACTIVITY_NEW_TASK : 这个属性需要在被start的目标Activity在AndroidManifest.xml文件 配置taskAffinity的值必须和startActivity
   发其者Activity的包名不一样，如果是跳转另一个 App的话可以taskAffinity可以省略】，则会在新标记的Affinity所存在的taskAffinity中压入这个Activity。

## 参考

- []()
- []()
- [Intent和Intent过滤器](https://developer.android.google.cn/guide/components/intents-filters?hl=zh-cn)
