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
    if (element._inDirtyList) {
      _dirtyElementsNeedsResorting = true;
      return;
    }
    if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
      _scheduledFlushDirtyElements = true;
      onBuildScheduled!();//通知下一帧要更新，对应WidgetsBinding中的_handleBuildScheduled方法，下一帧drawFrame会调用buildScope方法
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
    if (callback == null && _dirtyElements.isEmpty) {
      return;
    }
    try {
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
      while (index < dirtyCount) {
        final Element element = _dirtyElements[index];
        final bool isTimelineTracked = !kReleaseMode && _isProfileBuildsEnabledFor(element.widget);
        try {
          element.rebuild();//触发对应生命周期方法，State的didChangeDependencies、build等
        } catch (e, stack) {
        }
        index += 1;
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

## 流程分析

我们在需要触发刷新时会调用setState方法，会对该element进行标脏，然后擦除脏标记。

```dart

class State {
  void setState(VoidCallback fn) {
    final Object? result = fn() as dynamic;
    _element!.markNeedsBuild();
  }
}

class Element {
  void markNeedsBuild() {
    _dirty = true;
    owner.scheduleBuildFor(this);
  }
}
```

## 参考

- [深入浅出 Flutter Framework 之 BuildOwner](https://zxfcumtcs.github.io/2020/05/16/deepinto-flutter-buildowner/)
