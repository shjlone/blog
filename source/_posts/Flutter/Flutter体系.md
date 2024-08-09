---
title: Flutter体系
toc: true
tags: Flutter
---



- Flutter的基本控件是如何绘制的？比如Text、Image、Button
- Flutter是如何实现弹窗的？
- Flutter是如何实现动画的？
- Flutter是如何实现滚动的？
- Flutter的路由跳转是如何实现的？
- Flutter是如何跟原生通信的？
- Flutter是如何实现手势事件传递的？


## 概述

![](./11_flutter_arch.png)

Flutter架构最核心的便是Framework（框架）和Engine（引擎）：

- Flutter Framework层：用Dart编写，封装整个Flutter架构的核心功能，包括Widget、动画、绘制、手势等功能，有Material（Android风格UI）和Cupertino（iOS风格）的UI界面， 可构建Widget控件以及实现UI布局。
- Flutter Engine层：用C++编写，用于高质量移动应用的轻量级运行时环境，实现了Flutter的核心库，包括Dart虚拟机、动画和图形、文字渲染、通信通道、事件通知、插件架构等。引擎渲染采用的是2D图形渲染库Skia，虚拟机采用的是面向对象语言Dart VM，并将它们托管到Flutter的嵌入层。shell实现了平台相关的代码，比如跟屏幕键盘IME和系统应用生命周期事件的交互。不同平台有不同的shell，比如Android和iOS的shell。

## 编译产物

![](./12_flutter_artifact.png)

Flutter产物分为Dart业务代码和Engine代码各自生成的产物，图中的Dart Code包含开发者编写的业务代码，Engine Code是引擎代码，如果并没有定制化引擎，则无需重新编译引擎代码。

一份Dart代码，可编译生成双端产物，实现跨平台的能力。经过编译工具处理后可生成双端产物，图中便是release模式的编译产物，Android产物是由vm、isolate各自的指令段和数据段以及flutter.jar组成的app.apk，iOS产物是由App.framework和Flutter.framework组成的Runner.app。

这个过程涉及frontend_server、gen_snapshot、xcrun、ninja编译工具。frontend_server前端编译器会进行词法分析、语法分析以及相关全局转换等工作，将dart代码转换为AST(抽象语法树)，并生成app.dill格式的dart kernel。gen_snapshot经过CHA、内联等一系列执行流的优化，根据中间代码生成优化后的FlowGraph对象，再转换为具体相应系统架构（arm/arm64等）的二进制指令。

## 启动引擎

![](./13_flutter_engine_startup.png)

- FlutterApplication.java的onCreate过程主要完成初始化配置、加载引擎libflutter.so、注册JNI方法；
- FlutterActivity.java的onCreate过程，通过FlutterJNI的AttachJNI()方法来初始化引擎Engine、Dart虚拟机、Isolate、taskRunner等对象。再经过层层处理最终调用main.dart中main()方法，执行runApp(Widget app)来处理整个Dart业务代码。

Flutter引擎启动中会创建有4个TaskRunner以及创建虚拟机，分别来看看它们的工作原理。

### TaskRunner工作原理

![](./14_task_loop.png)

Flutter引擎启动过程，会创建UI/GPU/IO这3个线程，会为这些线程依次创建MessageLoop对象，启动后处于epoll_wait等待状态。对于Flutter的消息机制跟Android原生的消息机制有很多相似之处，都有消息(或者任务)、消息队列(或任务队列)以及Looper；有一点不同的是Android有一个Handler类，用于发送消息以及执行回调方法，相对应Flutter中有着相近功能的便是TaskRunner。

上图是从源码中提炼而来的任务处理流程，比官方流程图更容易理解一些复杂流程的时序问题，Flutter的任务队列处理机制跟Android的消息队列处理相通，只不过Flutter分为Task和MicroTask两种类型，引擎和Dart虚拟机的事件以及Future都属于Task，Dart层执行scheduleMicrotask()所产生的属于Microtask。

每次Flutter引擎在消费任务时调用FlushTasks()方法，遍历整个延迟任务队列delayed_tasks_，将已到期的任务加入task队列，然后开始处理任务。

1. 检查task，当task队列不为空，先执行一个task；
2. 检查microTask，当microTask不为空，则执行microTask；不断循环Step 2 直到microTask队列为空，再回到执行Step 1；

可简单理解为先处理完所有的Microtask，然后再处理Task。因为scheduleMicrotask()方法的调用自身就处于一个Task，执行完当前的task，也就意味着马上执行该Microtask。

了解了其工作机制，再来看看这4个Task Runner的具体工作内容。

- Platform Task Runner：运行在Android或者iOS的主线程，尽管阻塞该线程并不会影响Flutter渲染管道，平台线程建议不要执行耗时操作；否则可能触发watchdog来结束该应用。比如Android、iOS都是使用平台线程来传递用户输入事件，一旦平台线程被阻塞则会引起手势事件丢失。
- UI Task Runner: 运行在ui线程，比如1.ui，用于引擎执行root isolate中的所有Dart代码，执行渲染与处理Vsync信号，将widget转换生成Layer Tree。除了渲染之外，还有处理Native Plugins消息、Timers、Microtasks等工作；
- GPU Task Runner：运行在gpu线程，比如1.gpu，用于将Layer Tree转换为具体GPU指令，执行设备GPU相关的skia调用，转换相应平台的绘制方式，比如OpenGL, vulkan, metal等。每一帧的绘制需要UI Runner和GPU Runner配合完成，任何一个环节延迟都可能导致掉帧；
- IO Task Runner：运行在io线程，比如1.io，前3个Task Runner都不允许执行耗时操作，该Runner用于将图片从磁盘读取出来，解压转换为GPU可识别的格式后，再上传给GPU线程。为了能访问GPU，IO Runner跟GPU Runner的Context在同一个ShareGroup。比如ui.image通过异步调用让IO Runner来异步加载图片，该线程不能执行其他耗时操作，否则可能会影响图片加载的性能。

## 虚拟机工作

Flutter引擎启动会创建Dart虚拟机以及Root Isolate。DartVM自身也拥有自己的Isolate，完全由虚拟机自己管理的，Flutter引擎也无法直接访问。Dart的UI相关操作，是由Root Isolate通过Dart的C++调用，或者是发送消息通知的方式，将UI渲染相关的任务提交到UIRunner执行，这样就可以跟Flutter引擎相关模块进行交互。

何为Isolate，从字面上理解是“隔离”，isolate之间是逻辑隔离的。Isolate中的代码也是按顺序执行，因为Dart没有共享内存的并发，没有竞争的可能性，故不需要加锁，也没有死锁风险。对于Dart程序的并发则需要依赖多个isolate来实现。

isolate是Dart对actor并发模式的实现。运行中的Dart程序由一个或多个actor组成，这些actor也就是Dart概念里面的isolate。isolate是有自己的内存和单线程控制的运行实体。isolate本身的意思是“隔离”，因为isolate之间的内存在逻辑上是隔离的。isolate中的代码是按顺序执行的，任何Dart程序的并发都是运行多个isolate的结果。

由于isolate之间没有共享内存，所以他们之间的通信唯一方式只能是通过Port进行，而且Dart中的消息传递总是异步的。

![](./15_isolate_heap.png)

- isolate堆是运该isolate中代码分配的所有对象的GC管理的内存存储；
- vm isolate是一个伪isolate，里面包含不可变对象，比如null，true，false；
- isolate堆能引用vm isolate堆中的对象，但vm isolate不能引用isolate堆；
- isolate彼此之间不能相互引用；
- 每个isolate都有一个执行dart代码的Mutator thread，一个处理虚拟机内部任务(比如GC, JIT等)的helper thread； 可见，isolate是拥有内存堆和控制线程，虚拟机中可以有很多isolate，但彼此之间内存不共享，无法直接访问，只能通过dart特有的Port端口通信；isolate除了拥有一个mutator控制线程，还有一些其他辅助线程，比如后台JIT编译线程、GC清理/并发标记线程；

## 架构概览

![](./16_widget_arch.png)

Widget是所有Flutter应用程序的基石，Widget可以是一个按钮，一种字体或者颜色，一个布局属性等，在Flutter的UI世界可谓是“万物皆Widget”。常见的Widget子类为StatelessWidget(无状态)和StatefulWidget(有状态)；

- StatelessWidget：内部没有保存状态，UI界面创建后不会发生改变；
- StatefulWidget：内部有保存状态，当状态发生改变，调用setState()方法会触发StatefulWidget的UI发生更新，对于自定义继承自StatefulWidget的子类，必须要重写createState()方法。

### 三棵树

![](./17_three_tree.png)

- Widget是为Element描述需要的配置， 负责创建Element，决定Element是否需要更新。Flutter Framework通过差分算法比对Widget树前后的变化，决定Element的State是否改变。当重建Widget树后并未发生改变， 则Element不会触发重绘，否则就是Widget树的重建并不一定会触发Element树的重建。
- Element表示Widget配置树的特定位置的一个实例，同时持有Widget和RenderObject，负责管理Widget配置和RenderObject渲染。Element状态由Flutter Framework管理， 开发人员只需更改Widget即可。
- RenderObject表示渲染树的一个对象，负责真正的渲染工作，比如测量大小、位置、绘制等都由RenderObject完成。

## 渲染原理

![](./18_gpu_pipeline.png)

渲染过程，UI线程完成布局、绘制操作，生成Layer Tree；GPU线程执行合成并光栅化后交给GPU来处理，其中几个关键步骤：

- Animate: 遍历_transientCallbacks，执行动画回调方法；
- Build: 对于dirty的元素会执行build构造，没有dirty元素则不会执行，对应于buildScope()
- Layout: 计算渲染对象的大小和位置，对应于flushLayout()，这个过程可能会嵌套再调用build操作；
- Compositing bits: 更新具有脏合成位的任何渲染对象， 对应于flushCompositingBits()；
- Paint: 将绘制命令记录到Layer， 对应于flushPaint()；
- Compositing: 将Compositing bits发送给GPU， 对应于compositeFrame()；

GPU线程通过skia向GPU硬件绘制一帧的数据，GPU将帧信息保存到FrameBuffer里面，然后视频控制器会根据VSync信号从FrameBuffer取帧数据传递给显示器，从而显示出最终的画面。

## Platform Channels

![](./19_platform_channels.png)

Flutter框架提供了UI的控件支持，对于APP除了UI还有其他依赖于Native平台的支持，比如调用Camera的功能，该怎么办呢？为此，Flutter通过提供Platform Channel的功能，使得Dart代码具备与Native交互的能力。

Platform Channel用于Flutter与Native之间的消息传递，整个过程的消息与响应是异步执行，不会阻塞用户界面。Flutter引擎框架已完成桥接的通道，这样开发者只需在Native层编写定制的Android/iOS代码，即可在Dart代码中直接调用，这也就是Flutter Plugin插件的一种形式。

## Engine环境搭建

- [官方文档](https://github.com/flutter/flutter/wiki/Setting-up-the-Framework-development-environment)
- [https://github.com/flutter/flutter/wiki/Compiling-the-engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)
- [https://github.com/flutter/flutter/wiki/The-flutter-tool](https://github.com/flutter/flutter/wiki/The-flutter-tool)

## 源码目录结构

- flutter 根目录
 	- packages：
  		- flutter：
   			- lib：
    				- src：标准库的源码文件夹
     					- animation：动画库
     					- cupertino：iOS风格UI库
     					- foundation：基础
     					- gestures：手势识别
     					- material：Android风格UI库
     					- painting：自绘
     					- physics：移动的物理效果，Android、iOS不一样
     					- rendering：渲染相关
     					- scheduler：调度相关，底层传入数据给到Flutter进行操作
     					- semantics：语义化的东西？
     					- services
     					- widgets：UI库
  		- flutter_driver:
  		- flutter_goldens:
  		- flutter_goldens_client:
  		- flutter_localizations:国际化相关
  		- flutter_test:
  		- flutter_tools:
  		- flutter_web_plugins:
  		- fuchsia_remote_debug_protocol:
  		- integration_test:
 	- bin:
  		- internal
  		- cache:
   			- artifacts:
    				- engine:
     					- android-arm:
     					- android-arm64:
     					- android-arm64-profile:
     					- android-arm64-release:
     					- android-arm-profile:
     					- android-arm-release:
     					- android-x64:
     					- android-x64-profile:
     					- android-x86:
     					- android-x86-jit-release:
     					- common:
     					- windows-x64:
    				- gradle_wrapper:
    				- ios-deploy:
    				- libimobiledevice:
    				- libplist:
    				- material_fonts:
    				- openssl:
    				- usbmuxd:
   			- dart-sdk:
   			- downloads:
   			- flutter_web_sdk:
   			- pkg:
    				- sky_engine:
  		- mingit:
   			- cmd
   			- etc
   			- mingw64
   			- usr
   			- dart：dart命令
   			- dart.bat
   			- flutter:linux相关平台的命令
   			- flutter.bat：windows上的命令
  		- dev:Flutter团队开发框架时用到的工具
   			- automated_tests:
   			- benchmarks:
   			- bots:
   			- ci:
   			- conductor:
   			- customer_testing:
   			- devicelab:
   			- docs:
   			- forbidden_from_release_tests:
   			- integration_tests:
   			- manual_tests:
   			- missing_dependency_tests:
   			- snippets:
   			- tools:
   			- tracing_tests:
 	- examples

## 参考

- [Introduction to Dart VM](https://mrale.ph/dartvm/)
- [Flutter 跨平台演进及架构开篇](http://gityuan.com/flutter/)
- [Flutter System Architecture](https://docs.google.com/presentation/d/1cw7A4HbvM_Abv320rVgPVGiUP2msVs7tfGbkgdrTy0I/edit#slide=id.gbb3c3233b_0_187)
