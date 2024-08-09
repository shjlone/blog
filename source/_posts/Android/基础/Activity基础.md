---
title: Activity
tags: Android
toc: true
---

## 由来

我们在做带UI的软件时，一般的做法是先创建一个窗口，然后在窗口上添加各种Button、Text、List等其他UI控件。Android、iOS也是类似，但代码的设计上跟PC端
有些差别，Android使用Activity来管理UI、iOS使用ViewController。一般软件的入口都是main函数开始，Android中则通过描述文件[AndroidManifest.xml](./AndroidManifest.html)配置一个Activity的属性作为入口。用户操作手机的时候使得一个界面可能处于`可视`状态，也可能处理`隐藏`状态，对应着Activity会有自己的生命周期。不同UI的嵌套
也是需要维护的，所以就有了Activity`任务栈`，对应着不同Activity有不同的`启动模式`。不同的Activity之间又可能需要数据传递，因而有了[Intent](./Intent.html)。




## 生命周期

![](./activity_lifecycle.jpg)


### 回调函数

#### onCreate

生命周期中的第一个函数，整个生命周期中只会调用一次。savedInstanceState参数如果不为空，表示Activity暂时销毁时有存储一些数据，此时可以恢复。

#### onRestart

当前Activity从不可见重新变为可见状态时，会调用。

#### onStart

此时准备进入前台了

#### onResume

可见了

#### onPause

表示Activity正在停止

#### onStop

表示Activity即将停止

#### onDestroy

表示Activity即将被销毁，一般在这个方法中进行资源释放。

#### savedInstanceState

界面销毁时可保存数据

#### onRestoreInstanceState

恢复数据


### 不同场景的生命周期流程

#### 正常的开启和结束

从Activity1中打开Activity2

```java
2021-06-02 21:03:06.820 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onCreate
2021-06-02 21:03:06.831 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onStart
2021-06-02 21:03:06.835 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onResume
2021-06-02 21:03:26.542 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onPause
2021-06-02 21:03:26.649 30945-30945/com.lqd.androidpractice D/LaunchActivity2: onCreate
2021-06-02 21:03:26.659 30945-30945/com.lqd.androidpractice D/LaunchActivity2: onStart
2021-06-02 21:03:26.663 30945-30945/com.lqd.androidpractice D/LaunchActivity2: onResume
```

在Activity2中点击返回键，当一个后台Activity回到前台时，会执行onRestart->onResume->onDestroy。当回到桌面，再次进入的应用的时候也是此流程

```java
2021-06-02 21:06:54.500 30945-30945/com.lqd.androidpractice D/LaunchActivity2: onPause
2021-06-02 21:06:54.521 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onRestart
2021-06-02 21:06:54.523 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onStart
2021-06-02 21:06:54.526 30945-30945/com.lqd.androidpractice D/LaunchActivity1: onResume
2021-06-02 21:06:54.762 30945-30945/com.lqd.androidpractice D/LaunchActivity2: onDestroy

```

#### 屏幕旋转时

```java
2021-06-02 21:14:02.755 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onPause
2021-06-02 21:14:02.763 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onSaveInstanceState
2021-06-02 21:14:02.768 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onDestroy
2021-06-02 21:14:02.997 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onCreate
2021-06-02 21:14:03.011 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onStart
2021-06-02 21:14:03.014 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onRestoreInstanceState
2021-06-02 21:14:03.018 1351-1351/com.lqd.androidpractice D/LaunchActivity1: onResume
```


### 异常情况下的数据保存和恢复

某些情况下，Activity会被销毁，此时系统会调用savedInstanceState(Bundle bundle)方法，我们可以在这个方法中存储一些数据，等到Activity恢复时，
bundle对象会被传递给onCreate和onRestoreInstanceState方法，我们就可以恢复到原来的状态了。




## 启动模式和任务栈


任务栈Task，是一种用来放置Activity实例的容器，他是以栈的形式进行盛放，也就是所谓的先进后出，主要有2个基本操作：压栈和出栈，
其所存放的Activity是不支持重新排序的，只能根据压栈和出栈操作更改Activity的顺序。

启动一个Application的时候，系统会为它默认创建一个对应的Task，用来放置根Activity。默认启动Activity会放在同一个Task中，
新启动的Activity会被压入启动它的那个Activity的栈中，并且显示它。当用户按下回退键时，这个Activity就会被弹出栈，按下Home键回到桌面，
再启动另一个应用，这时候之前那个Task就被移到后台，成为后台任务栈，而刚启动的那个Task就被调到前台，成为前台任务栈，
手机页面显示的就是前台任务栈中的栈顶元素。

启动模式则是用来管理Activity如何添加到任务栈里的。可以在描述文件中配置，也可以代码中设置。

### 启动模式launchMode


#### standard

默认模式，每次启动都会创建一个新的Activity实例。在这个模式中，谁启动了这个Activity，那么新的Activity会添加到启动的Activity所在的任务栈中。
当使用非Activity的Context打开一个Activity时，则会创建一个新的任务栈，这个时候需要指定FLAG_ACTIVITY_NEW_TASK标识。

#### singleTop

栈顶复用模式。在这种模式下，如果新Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被回调，
通过此方法的参数我们可以取出当前请求的信息。需要注意的是，这个Activity的onCreate、onStart不会被系统调用，因为它并没有发生改变。
如果新Activity的实例已存在但不是位于栈顶，那么新Activity仍然会重新重建。举个例子，假设目前栈内的情况为ABCD，其中ABCD为四个Activity，
A位于栈底，D位于栈顶，这个时候假设要再次启动D，如果D的启动模式为singleTop，那么栈内的情况仍然为ABCD;如果D的启动模式为standard，
那么由于D被重新创建，导致栈内的情况就变为ABCDD。

#### singleTask

栈内复用模式。 这是一种单实例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，和singleTop一样，
系统也会回调其onNewIntent。如果当前栈中启动，当前栈如果已经存在，则会将其上面的Activity全部出栈，自己排到栈顶。如果不存在，则创建新任务栈。

比如：

- 比如目前任务栈S1中的情况为ABC，这个时候Activity D以singleTask模式请求启动，其所需要的任务栈为S2，由于S2和D的实例均不存在，所以系统会先创建任务栈S2，
  然后再创建D的实例并将其入 栈到S2。
- 另外一种情况，假设D所需的任务栈为S1，其他情况如上面例子1所示，那么由于S1已经存在，所以系统会直接创建D的实例并将其入栈到S1。 
- 如果D所需的任务栈为S1，并且当前任务栈S1的情况为ADBC，根据栈内复用的原则，此时D不会重新创建，系统会把D切换到栈顶并调用其onNewIntent方法，
  同时由于singleTask默认具有clearTop的效果，会导致栈内所有在D上面的Activity全部出栈，于是最终S1中的情况为AD。

#### singleInstance

单实例模式。这是一种加强的singleTask模式，它除了具有singleTask模式的所有特性外，还加强了一点，那就是具有此种模式的Activity只能单独地位于一个任务栈中，
换句话说，比如Activity A是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A独自在这个新的任务栈中，由于栈内复用的特性，
后续的请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了。"来电显示"界面就可以使用该模式。


## Activity中的Flags

标记位的作用很多，有的标记位可以设定Activity的启动模式，比如FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_SINGLE_TOP等; 
还有的标记位可以影响Activity的运行状态，比如FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS等。
下面主要介绍几个比较常用的标记位，剩下的标记位读者可以查看官方文档去了解，大部分情况下，我们不需要为Activity指定标记位，因此，对于标记位理解即可。
在使用标记位的时候，要注意有些标记位是系统内部使用的，应用程序不需要去手动设置这些标记位以防出现问题。

FLAG_ACTIVITY_NEW_TASK

这个标记位的作用是为Activity指定“singleTask”启动模式，其效果和在XML中指定该启动模式相同。 

FLAG_ACTIVITY_SINGLE_TOP

这个标记位的作用是为Activity指定“singleTop”启动模式，其效果和在XML中指定该启动模式相同。 

FLAG_ACTIVITY_CLEAR_TOP

具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个模式一般需要和FLAG_ACTIVITY_NEW_TASK配合使用，
在这种情况下，被启动Activity的实例如果已经存在，那么系统就会调用它的onNewIntent。如果被启动的Activity采用standard模式启动，
那么它连同它之上的Activity都要出栈，系统会创建新的Activity实例并放入栈顶。

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

具有这个标记的Activity不会出现在历史Activity的列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用。
它等同于在XML中指定Activity的属性 android:excludeFromRecents="true"。

## Activity之间如何通信



## 参考

- [https://developer.android.com/guide/components/activities/intro-activities?hl=zh-cn](https://developer.android.com/guide/components/activities/intro-activities?hl=zh-cn)
- 《Android开发艺术探索》
