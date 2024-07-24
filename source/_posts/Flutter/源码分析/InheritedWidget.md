---
title: InheritedWidget
toc: true
tags: Flutter
---



## 相关API

- InheritedWidget
- ProxyWidget
- InheritedElement


## 基本用法


提供一种在widget树中从上到下共享数据的方式。当前节点和关联节点同时注册。当build时，会检查是否需要刷新。

```dart

class InheritedWidgetTestRoute extends StatefulWidget {
  @override
  _InheritedWidgetTestRouteState createState() => _InheritedWidgetTestRouteState();
}

class _InheritedWidgetTestRouteState extends State<InheritedWidgetTestRoute> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: ShareDataWidget(
        //使用ShareDataWidget
        data: count,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.only(bottom: 20.0),
              child: TestInheritWidget(), //子widget中依赖ShareDataWidget
            ),
            ElevatedButton(
              child: const Text("Increment"),
              //每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新
              onPressed: () => setState(() => ++count),
            )
          ],
        ),
      ),
    );
  }
}

class TestInheritWidget extends StatefulWidget {
  @override
  _TestInheritWidgetState createState() => _TestInheritWidgetState();
}

class _TestInheritWidgetState extends State<TestInheritWidget> {
  @override
  Widget build(BuildContext context) {
    //使用InheritedWidget中的共享数据
    return Text(ShareDataWidget.of(context)!.data.toString());
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("_TestInheritWidgetState  Dependencies change");
  }

  @override
  void didUpdateWidget(covariant TestInheritWidget oldWidget) {
    print('_TestInheritWidgetState   didUpdateWidget');
    super.didUpdateWidget(oldWidget);
  }
}

class ShareDataWidget extends InheritedWidget {
  ShareDataWidget({Key? key, required Widget child, required this.data}) : super(key: key, child: child);

  final int data;

  static ShareDataWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
  }

  @override
  bool updateShouldNotify(covariant ShareDataWidget oldWidget) {
    return oldWidget.data != data;
  }
}
```


## 核心函数

### dependOnInheritedWidgetOfExactType

在需要使用数据的的地方，会调用dependOnInheritedWidgetOfExactType这个方法来获取相应的数据。而这个方法的Element中的实现如下：

```dart
  T? dependOnInheritedWidgetOfExactType<T extends InheritedWidget>({Object? aspect}) {
    final InheritedElement? ancestor = _inheritedWidgets == null ? null : _inheritedWidgets![T];//先看以前有没有这个祖先
    if (ancestor != null) {
      return dependOnInheritedElement(ancestor, aspect: aspect) as T;
    }
    _hadUnsatisfiedDependencies = true;
    return null;
  }

  InheritedWidget dependOnInheritedElement(InheritedElement ancestor, { Object? aspect }) {
    _dependencies ??= HashSet<InheritedElement>();
    _dependencies!.add(ancestor);//将祖先添加到依赖列表
    ancestor.updateDependencies(this, aspect);//祖先也将自己添加到他的依赖列表中
    return ancestor.widget as InheritedWidget;
  }

```


### updateShouldNotify

控制依赖于InheritedWidget的组件是否需要重建。如果为true，则当InheritedWidget发生变化时，依赖于它的组件会被rebuild，其Element的didChangeDependencies会被调用。


### updated

当进行build的时候，ProxyElement这个节点会调用updated方法，如下：

```dart
  void update(ProxyWidget newWidget) {
    final ProxyWidget oldWidget = widget as ProxyWidget;
    super.update(newWidget);
    updated(oldWidget);
    rebuild(force: true);
  }

  void updated(InheritedWidget oldWidget) {
    if ((widget as InheritedWidget).updateShouldNotify(oldWidget)) {//是否需要通知，这个方法我们要重写
      super.updated(oldWidget);
    }
  }

  void updated(covariant ProxyWidget oldWidget) {
    notifyClients(oldWidget);
  }

  void notifyClients(InheritedWidget oldWidget) {
    for (final Element dependent in _dependents.keys) {
      notifyDependent(oldWidget, dependent);
    }
  }

  void notifyDependent(covariant InheritedWidget oldWidget, Element dependent) {
    dependent.didChangeDependencies();//生命周期回调方法
  }

```

Element中，_inheritedWidgets保存了所有上级节点的InheritedElement。

```dart
Map<Type, InheritedElement>? _inheritedWidgets;
/// Element中
void _updateInheritance() {
  assert(_lifecycleState == _ElementLifecycle.active);
  _inheritedWidgets = _parent?._inheritedWidgets;
}

/// InheritedElement中
void _updateInheritance() {
  assert(_lifecycleState == _ElementLifecycle.active);
  final Map<Type, InheritedElement>? incomingWidgets = _parent?._inheritedWidgets;
  if (incomingWidgets != null)
    _inheritedWidgets = HashMap<Type, InheritedElement>.of(incomingWidgets);
  else
    _inheritedWidgets = HashMap<Type, InheritedElement>();
  _inheritedWidgets![widget.runtimeType] = this;
}
```



获取InheritedWidget的方式

```dart
  T? dependOnInheritedWidgetOfExactType<T extends InheritedWidget>({Object? aspect}) {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement? ancestor = _inheritedWidgets == null ? null : _inheritedWidgets![T];
    if (ancestor != null) {
      return dependOnInheritedElement(ancestor, aspect: aspect) as T;
    }
    _hadUnsatisfiedDependencies = true;
    return null;
  }

  InheritedElement? getElementForInheritedWidgetOfExactType<T extends InheritedWidget>() {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement? ancestor = _inheritedWidgets == null ? null : _inheritedWidgets![T];
    return ancestor;
  }

```

从根节点到子节点，以runtimeType作为key，保存最新的Element对象。getElementForInheritedWidgetOfExactType方法可以通过类型查找离自己最近的类型的对象。
dependOnInheritedWidgetOfExactType方法会注册依赖，当InheritedWidget发生变化时就会更新依赖它的子组件。




## 参考

- [InheritedWidget的使用和源码分析](https://juejin.cn/post/6943515602191384613)