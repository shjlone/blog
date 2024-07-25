---
title: Build过程
toc: true
tags: Flutter
---



## 概括

1. runApp，初始化构建Widget树，Widget对应Element，创建Element树；
2. 调用setState方法时，会将对应的element添加到dirtyElement队列中；触发WidgetsBinding中的_handleBuildScheduled方法，下一帧drawFrame会调用buildScope方法；
3. 在buildScope方法中，会对dirtyElement队列中的element进行排序，然后逐个调用rebuild方法，触发对应的生命周期方法，State的didChangeDependencies、build等；
4. 当Widget从树中移除时，会调用deactivate方法，将对应的element添加到inactiveElement队列中；调用unmount方法

## 相关类

开发者用来绘制UI的配置挂件，每一个Widget都对应一个Element(通过createElement创建)。Widget是不可变的。

```dart
class WidgetsBinding {

  //初始化
  void initInstances() {
    _buildOwner = BuildOwner(onBuildScheduled: _handleBuildScheduled); 管理整个build过程
  }
  

  void attachRootWidget(Widget rootWidget) {
    rootElement = RenderObjectToWidgetAdapter<RenderObjectToWidgetAdapter>(
      container: renderView,
      debugShortDescription: '[root]',
      owner: this,
    ).attachToRenderTree(rootElement);//初始化
  }

  /// 每一帧都会调用该方法
  void drawFrame() {
    buildOwner.buildScope(rootElement);
  }
}

class RenderObjectToWidgetAdapter {
  attachToRenderTree(RenderObjectToWidgetElement<RenderObjectToWidgetAdapter> element) {
    owner.buildScope(element, (){
      element!.mount(null, null);
    });
  }
}


class BuildOwner {
  void buildScope(Element context, [VoidCallback? callback]) {
    callback();
    element.rebuild();
  }
}

class Element {
  void rebuild() {
    performRebuild();
  }
}

class StatefulElement extends ComponentElement {
  void performRebuild() {
    state.didChangeDependencies();//生命周期回调
    super.performRebuild();
  }
}

class ComponentElement {
  void performRebuild() {
    build();//生命周期回调
  }
}
```


## 流程分析


### 标脏阶段

我们在需要触发刷新时会调用setState方法，会对该element进行标脏，然后擦除脏标记。

```dart

class State {
  void setState(VoidCallback fn) {
    final Object? result = fn() as dynamic;//执行用户的逻辑
    _element!.markNeedsBuild();
  }
}

class Element {
  void markNeedsBuild() {
    _dirty = true;
    owner.scheduleBuildFor(this);//在BuildOwner中进一步标记
  }
}
```


### BuildOwner

作用：

1. 在 UI 更新过程中跟踪、管理需要 rebuild 的 Element (「dirty elements」);
2. 在有「dirty elements」时，及时通知引擎，以便在下一帧安排上对「dirty elements」的 rebuild，从而去刷新 UI；
3. 管理处于 “inactive” 状态的 Element。

```dart
class BuildOwner {
  BuildOwner({ this.onBuildScheduled, FocusManager? focusManager }) :
      focusManager = focusManager ?? (FocusManager()..registerGlobalHandlers());

  VoidCallback? onBuildScheduled;

  final _InactiveElements _inactiveElements = _InactiveElements();

  final List<Element> _dirtyElements = <Element>[];
  bool _scheduledFlushDirtyElements = false;

  bool? _dirtyElementsNeedsResorting;

  bool get _debugIsInBuildScope => _dirtyElementsNeedsResorting != null;

  FocusManager focusManager;

  /// setState方法会调用该方法
  void scheduleBuildFor(Element element) {
    if (element._inDirtyList) {//已经在脏列表中，重排
      _dirtyElementsNeedsResorting = true;
      return;
    }
    if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
      _scheduledFlushDirtyElements = true;
      onBuildScheduled!();//通知下一帧要更新，对应WidgetsBinding中的_handleBuildScheduled方法，调用ensureVisualUpdate，下一帧drawFrame会调用buildScope方法
    }
    _dirtyElements.add(element);//添加到脏列表
    element._inDirtyList = true;
  }

  int _debugStateLockLevel = 0;
  bool get _debugStateLocked => _debugStateLockLevel > 0;

  bool get debugBuilding => _debugBuilding;
  bool _debugBuilding = false;
  Element? _debugCurrentBuildTarget;

  void lockState(VoidCallback callback) {
    try {
      callback();
    } finally {
    }
  }

// drawFrame会调用该方法
  @pragma('vm:notify-debugger-on-exception')
  void buildScope(Element context, [ VoidCallback? callback ]) {
    if (callback == null && _dirtyElements.isEmpty) {//第1步
      return;
    }
    try {//第2步
      _scheduledFlushDirtyElements = true;
      if (callback != null) {
        _dirtyElementsNeedsResorting = false;
        try {
          callback();
        } finally {
        }
      }
      _dirtyElements.sort(Element._sort);
      _dirtyElementsNeedsResorting = false;
      int dirtyCount = _dirtyElements.length;
      int index = 0;
      while (index < dirtyCount) {//第3步，开始遍历脏节点
        final Element element = _dirtyElements[index];
        final bool isTimelineTracked = !kReleaseMode && _isProfileBuildsEnabledFor(element.widget);
        try {
          element.rebuild();//第4步，重新构建，触发对应生命周期方法，State的didChangeDependencies、build等
        } catch (e, stack) {
        }
        index += 1;//第5步
        if (dirtyCount < _dirtyElements.length || _dirtyElementsNeedsResorting!) {
          _dirtyElements.sort(Element._sort);
          _dirtyElementsNeedsResorting = false;
          dirtyCount = _dirtyElements.length;
          while (index > 0 && _dirtyElements[index - 1].dirty) {
            index -= 1;
          }
        }
      }
    } finally {
      for (final Element element in _dirtyElements) {
        element._inDirtyList = false;
      }
      _dirtyElements.clear();
      _scheduledFlushDirtyElements = false;
      _dirtyElementsNeedsResorting = null;
    }
  }

  Map<Element, Set<GlobalKey>>? _debugElementsThatWillNeedToBeRebuiltDueToGlobalKeyShenanigans;

  void _debugTrackElementThatWillNeedToBeRebuiltDueToGlobalKeyShenanigans(Element node, GlobalKey key) {
    _debugElementsThatWillNeedToBeRebuiltDueToGlobalKeyShenanigans ??= HashMap<Element, Set<GlobalKey>>();
    final Set<GlobalKey> keys = _debugElementsThatWillNeedToBeRebuiltDueToGlobalKeyShenanigans!
      .putIfAbsent(node, () => HashSet<GlobalKey>());
    keys.add(key);
  }

  void _debugElementWasRebuilt(Element node) {
    _debugElementsThatWillNeedToBeRebuiltDueToGlobalKeyShenanigans?.remove(node);
  }

  final Map<GlobalKey, Element> _globalKeyRegistry = <GlobalKey, Element>{};

  @_debugOnly
  final Set<Element>? _debugIllFatedElements = kDebugMode ? HashSet<Element>() : null;

  @_debugOnly
  final Map<Element, Map<Element, GlobalKey>>? _debugGlobalKeyReservations = kDebugMode ? <Element, Map<Element, GlobalKey>>{} : null;

  int get globalKeyCount => _globalKeyRegistry.length;

  void _debugRemoveGlobalKeyReservationFor(Element parent, Element child) {
  }

  void _registerGlobalKey(GlobalKey key, Element element) {
    _globalKeyRegistry[key] = element;
  }

  void _unregisterGlobalKey(GlobalKey key, Element element) {
    if (_globalKeyRegistry[key] == element) {
      _globalKeyRegistry.remove(key);
    }
  }

  void _debugReserveGlobalKeyFor(Element parent, Element child, GlobalKey key) {
  }

  void _debugVerifyGlobalKeyReservation() {
  }

  void _debugVerifyIllFatedPopulation() {
  }

  @pragma('vm:notify-debugger-on-exception')
  void finalizeTree() {
    try {
      lockState(_inactiveElements._unmountAll); // this unregisters the GlobalKeys
    } catch (e, stack) {
    } finally {
    }
  }

  void reassemble(Element root, DebugReassembleConfig? reassembleConfig) {
    try {
      root._debugReassembleConfig = reassembleConfig;
      root.reassemble();
    } finally {
    }
  }
}

```



### Flush阶段

我们知道Vsync信号到达后会触发BuildOwner的buildScope方法（参考上面的代码解释），该方法分为以下几个步骤：

1. 检查参数，并标记当前进入Build流程
2. callback的回调，一般用于首帧渲染时3棵树的创建，更新阶段该参数为null
3. 脏节点排序，优先更新父节点效率更高
4. 遍历脏节点，执行rebuild方法
5. 第5步，先将index自增，再检查当前是否满足以下两种情况之一：
  - dirtyCount < _dirtyElements.length：即在处理Element脏节点的过程中又有新的节点标记为脏
  - _dirtyElementsNeedsResorting：通常由GlobalKey的复用导致，如果当前节点已经在列表中，则会将该字段设置为true(scheduleBuildFor方法中设置)

满足任何一个都会导致_dirtyElements列表重新排序，然后将index重制到最近的一个非脏节点，并继续从该Element节点的索引进行rebuild方法

6. 将_dirtyElements列表中的每个节点的_inDirtyList字段重置为false，然后清空列表，并重置相关字段



对于rebuild，会调用performRebuild方法，该方法不同子类不一样的实现

```dart
class Element {
    @protected
  @mustCallSuper
  void performRebuild() {
    _dirty = false;
  }
}

///对于StatefulElement来说，rebuild会触发ComponentElement的performRebuild方法
class ComponentElement {
  void performRebuild() {
    Widget? built;
    try {
      built = build();
    } catch (e, stack) {
      _debugDoingBuild = false;
    } finally {
      super.performRebuild(); // clears the "dirty" flag
    }
    try {
      _child = updateChild(_child, built, slot);//完成子节点的更新
      assert(_child != null);
    } catch (e, stack) {
      _child = updateChild(null, built, slot);
    }
  }
}

class Element {
  Element? updateChild(Element? child, Widget? newWidget, Object? newSlot) {
    if (newWidget == null) {
      if (child != null) {
        deactivateChild(child);
      }
      return null;
    }

    final Element newChild;
    if (child != null) {
      bool hasSameSuperclass = true;
      if (hasSameSuperclass && child.widget == newWidget) {//第1种情况
        if (child.slot != newSlot) {
          updateSlotForChild(child, newSlot);
        }
        newChild = child;
      } else if (hasSameSuperclass && Widget.canUpdate(child.widget, newWidget)) {//第2种情况
        if (child.slot != newSlot) {
          updateSlotForChild(child, newSlot);
        }
        final bool isTimelineTracked = !kReleaseMode && _isProfileBuildsEnabledFor(newWidget);
        if (isTimelineTracked) {
          Map<String, String>? debugTimelineArguments;
        }
        child.update(newWidget);
        newChild = child;
      } else {//第3种情况
        deactivateChild(child);
        newChild = inflateWidget(newWidget, newSlot);
      }
    } else {//第4种情况
      newChild = inflateWidget(newWidget, newSlot);
    }
    return newChild;
  }
}
```

- 第1种：Widget节点相同，直接同步slot，并复用现有的Element节点。
- 第2种：科直接基于新的Widget节点更新，复用并更新现有的Element节点即可。不同的子类不一样的update，比如

```dart
class Element {
  void update(Widget newWidget) {
    _widget = newWidget;
  }
}


class StatelessElement {
  void update(Widget newWidget) {
    super.update(newWidget);
    _dirty = true;
    rebuild();
  }
}

class RenderObjectElement {
  void update(Widget newWidget) {
    super.update(newWidget);
    widget.updateRenderObject(this, renderObject);//更新对应RenderObject
    rebuild();
  }
}
```

- 第3种：当前Widget Tree的子节点完全不一样，需要移除原有的Element节点，并新建新的Element节点进行挂载

```dart
class Element {
    void deactivateChild(Element child) {
    child._parent = null;
    child.detachRenderObject();
    owner!._inactiveElements.add(child); // this eventually calls child.deactivate()
  }
}
```


### 清理阶段

在Build流程中，对于执行了deactivate方法的节点，其_lifecycleState字段的属性为inactive，当Build、Layout、Paint、Composition在UI线程的工作结束后，BuildOwner会调用finalizeTree方法进行最后的处理，其会调用_unmountAll

```dart
class _InactiveElements {
  void _unmountAll() {
    _locked = true;
    final List<Element> elements = _elements.toList()..sort(Element._sort);
    _elements.clear();//对每一个inactive状态的节点进行清理
    try {
      elements.reversed.forEach(_unmount);
    } finally {
      _locked = false;
    }
  }

  void _unmount(Element element) {
    element.visitChildren((Element child) {
      _unmount(child);
    });
    element.unmount();
  }
}


class Element {
    void unmount() {
    if (kFlutterMemoryAllocationsEnabled) {
      MemoryAllocations.instance.dispatchObjectDisposed(object: this);
    }
    final Key? key = _widget?.key;
    if (key is GlobalKey) {
      owner!._unregisterGlobalKey(key, this);//移除注册
    }
    _widget = null;
    _dependencies = null;
    _lifecycleState = _ElementLifecycle.defunct;//更新状态
  }
}

```



## 参考

- [深入浅出 Flutter Framework 之 BuildOwner](https://zxfcumtcs.github.io/2020/05/16/deepinto-flutter-buildowner/)
