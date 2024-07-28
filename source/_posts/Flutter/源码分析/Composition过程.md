---
title: Composition过程
toc: true
tags: Flutter
---



经过Build、Layout、Paint后，Render Tree变成Layer Tree，那么Layer Tree是如何合成，以变成最终的渲染数据呢？这就是Composition过程。


## Mark阶段


Framework使用_neesaAddToScene字段标识当前图层是否需要进行合成，通常当一个Layer节点有子节点的变化（adoptChild、dropChild）或者Layer节点本身有变化时，需要将该标识设置为true，表示当前图层发生改变，需要重新合成。


```dart 
class Layer {
  set alpha(int? value) {
    if (value != _alpha) {
      if (value == 255 || _alpha == 255) {
        engineLayer = null;
      }
      _alpha = value;
      markNeedsAddToScene();
    }
  }

  set elevation(double? value) {
    if (value != _elevation) {
      _elevation = value;
      markNeedsAddToScene();
    }
  }

  void markNeedsAddToScene() {
    if (_needsAddToScene) {
      return;
    }
    _needsAddToScene = true;
  }

}

```

## Flush阶段

合成的Flush阶段是从renderView.compositeFrame方法开始

```dart

class RenderView {
  void compositeFrame() {
    try {
      final ui.SceneBuilder builder = ui.SceneBuilder();
      final ui.Scene scene = layer!.buildScene(builder);//Layer Tree的最终产物
      if (automaticSystemUiAdjustment) {
        _updateSystemChrome();
      }
      _view.render(scene);//请求渲染
      scene.dispose();//渲染完成，释放资源
    } finally {
    }
  }
}


class ContainerLayer {
  ui.Scene buildScene(ui.SceneBuilder builder) {
    updateSubtreeNeedsAddToScene();//1、计算哪些Layer需要合成
    addToScene(builder);//2、将Layer Tree映射到Engine
    if (subtreeHasCompositionCallbacks) {
      _fireCompositionCallbacks(includeChildren: true);
    }
    _needsAddToScene = false;//合成完成
    final ui.Scene scene = builder.build();//真正的合成逻辑
    return scene;
  }

  void updateSubtreeNeedsAddToScene() {
    super.updateSubtreeNeedsAddToScene();//先执行父类逻辑，更新自身标记
    Layer? child = firstChild;
    while (child != null) {//遍历每个子节点，完成更新
      child.updateSubtreeNeedsAddToScene();
      _needsAddToScene = _needsAddToScene || child._needsAddToScene;
      child = child.nextSibling;
    }
  }

  void addToScene(ui.SceneBuilder builder) {
    addChildrenToScene(builder);
  }

  void addChildrenToScene(ui.SceneBuilder builder) {
    Layer? child = firstChild;
    while (child != null) {
      child._addToSceneWithRetainedRendering(builder);
      child = child.nextSibling;
    }
  }
}



class Layer {
  void _addToSceneWithRetainedRendering(ui.SceneBuilder builder) {
    if (!_needsAddToScene && _engineLayer != null) {
      builder.addRetained(_engineLayer!);
      return;
    }
    addToScene(builder);//上屏操作，不同子类不同实现
    _needsAddToScene = false;
  }
}
```


