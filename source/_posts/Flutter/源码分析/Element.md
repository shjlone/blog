---
title: Element
toc: true
tags: Flutter
---


## 概念

Widget是UI元素的配置数据，Element代表屏幕显示元素。主要作用：

- 维护这棵Element Tree，根据Widget Tree的变化来更新Element Tree，包括：节点的插入、更新、删除、移动等；
- 将Widget和RenderObject关联到Element Tree上。

![](element_1.png)

- ComponentElement：用来组合其他更基础的Element，开发时常用到的StatelessWidget和StatefulWidget相对应的Element：StatelessElement和StatefulElement。其子节点对应的Widget需要通过build方法创建，该类型Element只有一个子节点。
- RenderObjectElement：渲染类Element，对应Renderer Widget，是框架最核心的Element。RenderObjectElement主要包括LeafRenderObjectElement，SingleChildRenderObjectElement，和MultiChildRenderObjectElement。
  - LeafRenderObjectElement对应的Widget是LeafRenderObjectWidget，没有子节点；
  - SingleChildRenderObjectElement对应的Widget是SingleChildRenderObjectWidget，有一个子节点；
  - MultiChildRenderObjectElement对应的Widget是MultiChildRenderObjecWidget，有多个子节点，这些子节点是Widget对应的Element，当前Element只负责计算布局

![](element_9.png)


- Element 通过 parent、child 指针形成「Element Tree」；
- Element 持有 Widget、「Render Object」；
- State 是绑定在 Element 上的，而不是绑在「Stateful Widget」上(这点很重要)。

> 上述这些关系并不是所有类型的 Element 都有，如：「Render Object」只有「RenderObject Element」才有，State 只有「Stateful Element」才有。



## 重要属性和方法

Element的子类：

- ComponentElement
  - StatelessElement
  - StatefulElement
  - ProxyElement
    - ParentDataElement
    - InheritedElement
- RenderObjectElement
  - LeafRenderObjectElement
  - SingleChildRenderObjectElement
  - MultiChildRenderObjectElement
    - TheatreElement
  - RootRenderObjectElement
- _NullElement
- RenderObjectToWidgetElement：element树的根节点

### 属性

```dart

Map<Type, InheritedElement>? _inheritedWidgets;
Set<InheritedElement> _dependencies;

class InheritedElement extends ProxyElement {
  final Map<Element, Object?> _dependents = HashMap<Element, Object?>();

  void _updateInheritance() {
    assert(_lifecycleState == _ElementLifecycle.active);
    final Map<Type, InheritedElement>? incomingWidgets = _parent?._inheritedWidgets;
    if (incomingWidgets != null)
      _inheritedWidgets = HashMap<Type, InheritedElement>.of(incomingWidgets);
    else
      _inheritedWidgets = HashMap<Type, InheritedElement>();
    _inheritedWidgets![widget.runtimeType] = this;
  }
}

abstract class Element extends DiagnosticableTree implements BuildContext {


  // 槽，用来存储一些额外信息，比如坐标
  Object? get slot => _slot;
  Object? _slot;

  // element tree上的深度
  late int _depth;

  // 开发人员需要处理的Widget
  Widget _widget;

  BuildOwner _owner;//用来处理Element的对象，全局一个，将element tree转换成renderobject tree

  @override
  InheritedWidget dependOnInheritedElement(InheritedElement ancestor, { Object? aspect }) {
    assert(ancestor != null);
    _dependencies ??= HashSet<InheritedElement>();
    _dependencies!.add(ancestor);
    ancestor.updateDependencies(this, aspect);
    return ancestor.widget;
  }

  @override
  T? dependOnInheritedWidgetOfExactType<T extends InheritedWidget>({Object? aspect}) {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement? ancestor = _inheritedWidgets == null ? null : _inheritedWidgets![T];
    if (ancestor != null) {
      return dependOnInheritedElement(ancestor, aspect: aspect) as T;
    }
    _hadUnsatisfiedDependencies = true;
    return null;
  }

  @override
  InheritedElement? getElementForInheritedWidgetOfExactType<T extends InheritedWidget>() {
    assert(_debugCheckStateIsActiveForAncestorLookup());
    final InheritedElement? ancestor = _inheritedWidgets == null ? null : _inheritedWidgets![T];
    return ancestor;
  }
}
```

从根节点到子节点，以runtimeType作为key，保存最新的Element对象。getElementForInheritedWidgetOfExactType方法可以通过类型查找离自己最近的类型的对象。
dependOnInheritedWidgetOfExactType方法会注册依赖，当InheritedWidget发生变化时就会更新依赖它的子组件。

### updateChild

```dart
/// 父节点通过该方法来修改子节点对应的Widget
/// newWidget == null   说明子节点对应的Widget已被移除，直接remove child element
/// child == null   说明newWidget是新插入的，通过inflateWidget创建子节点
/// child != null   分以下几种情况：
///     1. child.widget == newWidget，说明没变化，若child.slot != newSlot 表明子节点在兄弟节点间移动了位置，通过updateSlotForChild修改child.slot即可；
///     2. widget.canUpdate判断是否可以用newWidget修改child element，若可以则调用update方法；
///     3. 否则先将child element移除，并通过newWidget创建新的element子节点。
Element? updateChild(Element? child, Widget? newWidget, Object? newSlot) { }

```

### update

#### Element

```dart
void update(covariant Widget newWidget) {
    _widget = newWidget;
}

void rebuild() {
  if (_lifecycleState != _ElementLifecycle.active || !_dirty)
    return;
  Element? debugPreviousBuildTarget;
  performRebuild();//ComponentElement中调用build、updateChild
}

Element? updateChild(Element? child, Widget? newWidget, Object? newSlot) {}

```

#### StatelessElement

```dart
  @override
  void update(StatelessWidget newWidget) {
    super.update(newWidget);
    assert(widget == newWidget);
    _dirty = true;
    rebuild();
  }
```

rebuild调用performRebuild，调用当前build方法和updateChild。

#### StatefulElement

```dart

class StatefulElement extends ComponentElement {

  //开发者操作的对象，同样有相应生命周期，参考_StateLifecycle
  State<StatefulWidget>? _state;

  @override
  void update(StatefulWidget newWidget) {
    super.update(newWidget);
    final StatefulWidget oldWidget = state._widget!;
    _dirty = true;
    state._widget = widget as StatefulWidget;
    try {
      _debugSetAllowIgnoredCallsToMarkNeedsBuild(true);
      final Object? debugCheckForReturnedFuture = state.didUpdateWidget(oldWidget) as dynamic;
      assert(() {
        if (debugCheckForReturnedFuture is Future) {
        }
        return true;
      }());
    } finally {
      _debugSetAllowIgnoredCallsToMarkNeedsBuild(false);
    }
    rebuild();
  }
}
  
```

处理State：

- 修改_widget属性
- 调用didUpdateWidget更新属性

然后触发rebuild操作。

#### ProxyElement

```dart
  @override
  void update(ProxyWidget newWidget) {
    final ProxyWidget oldWidget = widget;
    super.update(newWidget);
    updated(oldWidget);
    _dirty = true;
    rebuild();
  }

  @protected
  void updated(covariant ProxyWidget oldWidget) {
    notifyClients(oldWidget);
  }


```

update方法会通知关联对象Widget有更新。不同子类的notifyClients实现不同。

#### RenderObjectElement

```dart
  @override
  void update(covariant RenderObjectWidget newWidget) {
    super.update(newWidget);
    _performRebuild(); // calls widget.updateRenderObject()
  }

  void _performRebuild() {
    widget.updateRenderObject(this, renderObject);//修改RenderObject的属性
    _dirty = false;
  }
```

#### SingleChildRenderObject

```dart
  @override
  void update(SingleChildRenderObjectWidget newWidget) {
    super.update(newWidget);
    _child = updateChild(_child, widget.child, null);
  }

```

#### MultiChildRenderObject

```dart
  @override
  void update(MultiChildRenderObjectWidget newWidget) {
    super.update(newWidget);
    _children = updateChildren(_children, widget.children, forgottenChildren: _forgottenChildren);
    _forgottenChildren.clear();
  }
```

updateChildren中处理子节点的插入、移动、更新、删除等操作。

### inflateWidget

```dart
  Element inflateWidget(Widget newWidget, Object? newSlot) {
    final Key? key = newWidget.key;
    if (key is GlobalKey) {
    //如果带有GlobalKey，首先在inactive Elements列表中查找是否有处于inactive状态的节点（即刚从树上移除），如找到就直接复活该节点。
      final Element? newChild = _retakeInactiveElement(key, newWidget);
      if (newChild != null) {
        newChild._activateWithParent(this, newSlot);
        final Element? updatedChild = updateChild(newChild, newWidget, newSlot);
        return updatedChild!;
      }
    }
    final Element newChild = newWidget.createElement();
    newChild.mount(this, newSlot);//挂载到树上
    return newChild;
  }

```

### mount

#### Element

```dart
  void mount(Element? parent, Object? newSlot) {
    _parent = parent;
    _slot = newSlot;
    _lifecycleState = _ElementLifecycle.active;
    _depth = _parent != null ? _parent!.depth + 1 : 1;
    if (parent != null) {
      _owner = parent.owner;//传递owner给子节点
    }
    final Key? key = widget.key;
    if (key is GlobalKey) {
      owner!._registerGlobalKey(key, this);//GlobalKey注册自己，方便其他地方使用
    }
    _updateInheritance();
  }


```

#### ComponentElement

```dart
  @override
  void mount(Element? parent, Object? newSlot) {
    super.mount(parent, newSlot);
    assert(_child == null);
    assert(_lifecycleState == _ElementLifecycle.active);
    _firstBuild();
    assert(_child != null);
  }

  void _firstBuild() {
    // StatefulElement overrides this to also call state.didChangeDependencies.
    rebuild(); // This eventually calls performRebuild.
  }


```

组合型 Element 在挂载时会执行_firstBuild->rebuild操作。

#### RenderObjectElement

```dart
void mount(Element parent, dynamic newSlot) {
  super.mount(parent, newSlot);
  _renderObject = widget.createRenderObject(this);
  attachRenderObject(newSlot);
  _dirty = false;
}
```

创建RenderObject并插入到树上。

#### SingleChildRenderObjectElement

```dart
@override
void mount(Element parent, dynamic newSlot) {
  super.mount(parent, newSlot);
  _child = updateChild(_child, widget.child, null);//创建新Element实例
}
```

#### MultiChildRenderObjectElement

```dart
class MultiChildRenderObjectElement {

  void mount(Element parent, dynamic newSlot) {
    super.mount(parent, newSlot);
    _children = List<Element>(widget.children.length);
    Element previousChild;
    for (int i = 0; i < _children.length; i += 1) {
      final Element newChild = inflateWidget(widget.children[i], previousChild);//将Widget转换成Element
      _children[i] = newChild;
      previousChild = newChild;
    }
  }
}
```

对每个子节点调用inflateWidget。

### markNeedsBuild

```dart
void markNeedsBuild() {
  if (!_active)
    return;

  if (dirty)
    return;

  _dirty = true;
  owner.scheduleBuildFor(this);
}
```

标记需要重建，其作用是将当前Element加入_dirtyElements，以便在下一帧可以rebuild。以下场景会调用markNeedsBuild：

- State.setState
- Element.reassemble：debug hot reload
- Element.didChangeDependencies：
- StatefulElement.activate

### rebuild

```dart
void rebuild() {
  if (!_active || !_dirty)
    return;

  performRebuild();
}

```

活跃的或脏节点会执行performRebuild，以下场景会调用rebuild：

- 对于dirty element，在新一帧绘制过程中由BuildOwner.buildScope
- 在element挂载时，由Element.mount调用
- 在update方法内被调用

### performRebuild

#### ComponentElement

```dart
void performRebuild() {
  Widget built;
  built = build();

  _child = updateChild(_child, built, slot);
}
```

组合型Element，先build自己，再更新子节点

#### RenderObjectElement

```dart
void performRebuild() {
  widget.updateRenderObject(this, renderObject);
  _dirty = false;
}
```

## 生命周期

Element有4种状态：initial，active，inactive，defunct。其对应的意义如下：

- initial：初始状态，Element刚创建时就是该状态。、
- active：激活状态。此时Element的Parent已经通过mount将该Element插入Element Tree的指定的插槽处（Slot），Element此时随时可能显示在屏幕上。
- inactive：未激活状态。当Widget Tree发生变化，Element对应的Widget发生变化，同时由于新旧Widget的Key或者的RunTimeType不匹配等原因导致该Element也被移除，因此该Element的状态变为未激活状态，被从屏幕上移除。并将该Element从Element Tree中移除，如果该Element有对应的RenderObject，还会将对应的RenderObject从Render Tree移除。但是，此Element还是有被复用的机会，例如通过GlobalKey进行复用。
- defunct：失效状态。如果一个处于未激活状态的Element在当前帧动画结束时还是未被复用，此时会调用该Element的unmount函数，将Element的状态改为defunct，并对其中的资源进行清理。

![](./element_2.png)

## ComponentElement

### 创建

![](./element_3.png)

### 更新

![](./element_4.png)

### 销毁

![](./element_5.jpg)

## RenderObjectElement

### 创建

![](./element_6.png)

### 更新

![](./element_7.png)

### 销毁

![](./element_8.jpg)

## 总结

Element继承自BuildContext，所以我们在平常使用的context其实就是Element。各种of方法其实就是操作Element树来获取相应对象。

## 参考

- [深入浅出 Flutter Framework 之 Element](http://zxfcumtcs.github.io/2020/05/17/deepinto-flutter-element/)
- [Keys! What are they good for?](https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d)
