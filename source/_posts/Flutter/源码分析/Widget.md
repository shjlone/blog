---
title: Widget
toc: true
tags: Flutter
---

## Widget

```dart

@immutable // 不可变
abstract class Widget extends DiagnosticableTree { //DiagnosticableTree提供调试信息
  const Widget({ this.key });

  final Key? key;//canUpdate中判断前后的key是否相同，则使用新的Widget配置更新Element对象，否则创建新的Element对象

  @protected
  @factory
  Element createElement();//每个Widget都有对应的Element。Widget树根Element树对应

  
  @override
  String toStringShort() {
    final String type = objectRuntimeType(this, 'Widget');
    return key == null ? type : '$type-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  @override
  @nonVirtual
  bool operator ==(Object other) => super == other;

  @override
  @nonVirtual
  int get hashCode => super.hashCode;


  // 是否可以用newWidget修改前一帧oldWidget生成的Element，而不是创建新的Element
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }

}

```

开发过程中，我们会使用`StatelessWidget`和`StatefulWidget`，这两个类都继承自`Widget`。而整个过程就是将Widget树转换成Element树，再转换成RenderObject树，最终通过底层skia绘制。

如果我们绘制的UI也是不可变的，那么我们可以使用`StatelessWidget`，这样在创建的时候绘制一次即可。如果UI需要根据状态发生变化，那么我们可以使用`StatefulWidget`。对于`StatefulWidget`，我们需要实现`State`类，这个类持有`Widget`和`Element`，用来管理`Widget`的状态。



## StatelessWidget

```dart

class FooWidget extends StatelessWidget {
  const FooWidget({super.key});

// 以下三种情况会被调用：
// 1. Widget第一次插入到树中时，mount时调用
// 2. Parent Widget修改了配置信息
// 3. InheritedWidget发生变化时
  @override
  Widget build(BuildContext context) {
    return const Placeholder();//通过不同Widget的组合来构建UI
  }
}

```


## StatefulWidget

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);
    
  @override
  StatefulElement createElement() => StatefulElement(this);//Element持有该Widget，回调State对应的生命周期方法
    
  @protected
  State createState();//开发者通过不同的生命周期方法来管理Widget的状态
}

```


## State


```dart

abstract class State<T extends StatefulWidget> with Diagnosticable {
  
  T get widget => _widget!;//持有Widget对象
  T? _widget;

  _StateLifecycle _debugLifecycleState = _StateLifecycle.created;

  bool _debugTypesAreRight(Widget widget) => widget is T;

  // 持有对应Element
  BuildContext get context {
    return _element!;
  }
  StatefulElement? _element;

  //是否挂载在树上
  bool get mounted => _element != null;

// 生命周期初始化方法
  @protected
  @mustCallSuper
  void initState() {
  }

// canUpdate返回true则会调用此方法
  @mustCallSuper
  @protected
  void didUpdateWidget(covariant T oldWidget) { }

  // hot reload时触发
  @protected
  @mustCallSuper
  void reassemble() { }

  // 调用这个方法会重新构建当前Widget
  @protected
  void setState(VoidCallback fn) {
    final Object? result = fn() as dynamic;
    _element!.markNeedsBuild();
  }

// 从树中移除时回调，如果没有重新添加到树上，那么会调用dispose
  @protected
  @mustCallSuper
  void deactivate() { }
  
  @protected
  @mustCallSuper
  void activate() { }

// 销毁时回调
  @protected
  @mustCallSuper
  void dispose() {
    if (kFlutterMemoryAllocationsEnabled) {
      MemoryAllocations.instance.dispatchObjectDisposed(object: this);
    }
  }

  // 构建UI
  @protected
  Widget build(BuildContext context);

  // 初始化，依赖的InheritedWidget发生变化时会调用
  @protected
  @mustCallSuper
  void didChangeDependencies() { }

}

```


## Widget的子类


![](./Widget子类.png)


Widget的子类：

- StatelessWidget、StatefulWidget：组合类Widget
- ProxyWidget：提供一些附加的功能
  - ParentDataWidget：布局信息
  - InheritedWidget：传递共享信息
- RenderObjectWidget：渲染类Widget，参与layout、paint流程，组合Widget和代理Widget都会映射到Render Widget
  - LeafRenderObjectWidget
  - SingleChildRenderObjectWidget
  - MultiChildRenderObjectWidget
- RenderObjectToWidgetAdapter：Widget树的根节点

### RenderObjectWidget


```dart

abstract class RenderObjectWidget extends Widget {
  
  const RenderObjectWidget({ super.key });

  @override
  @factory
  RenderObjectElement createElement();

  // 创建Widget对应的RenderObject，mount时调用
  @protected
  @factory
  RenderObject createRenderObject(BuildContext context);

  // Widget更新后，修改对应的RenderObject
  @protected
  void updateRenderObject(BuildContext context, covariant RenderObject renderObject) { }

  @protected
  void didUnmountRenderObject(covariant RenderObject renderObject) { }
}

```

