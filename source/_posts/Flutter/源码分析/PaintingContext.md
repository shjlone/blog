---
title: PaintingContext
toc: true
tags: Flutter
---


## 概要

![](PaintingContext_1.png)

- 继承自ClipContext，提供裁剪相关辅助方法
- PictureLayer _currentLayer、_recorder、_canvas用于具体的绘制操作
- ContainerLayer _containerLayer, Layer树的根节点


## 基本概念

### Canvas

Canvas是 Engine(C++) 层到 Framework(Dart) 层的桥接，真正的功能在 Engine 层实现。Canvas 向 Framework 层曝露了与绘制相关的基础接口，如：draw*、clip*、transform以及scale等，RenderObject 正是通过这些基础接口完成绘制任务的。

> 通过这套接口进行的所有操作都将被PictureRecorder记录下来。

除了正常的绘制操作(draw*)，Canvas 还支持矩阵变换(transformation matrix)、区域裁剪(clip region)，它们将作用于其后在该 Canvas 上进行的所有绘制操作。

```dart

void scale(double sx, [double sy]);
void rotate(double radians) native;
void transform(Float64List matrix4);

void clipRect(Rect rect, { ClipOp clipOp = ClipOp.intersect, bool doAntiAlias = true });
void clipPath(Path path, {bool doAntiAlias = true});

void drawColor(Color color, BlendMode blendMode);
void drawLine(Offset p1, Offset p2, Paint paint);
void drawRect(Rect rect, Paint paint);
void drawCircle(Offset c, double radius, Paint paint);
void drawImage(Image image, Offset p, Paint paint);
void drawParagraph(Paragraph paragraph, Offset offset);

```

### Picture

其本质是一系列「graphical operations」的集合，对 Framework 层透明。
Future<Image> toImage(int width, int height)，通过toImage方法可以将其记录的所有操作经光栅化后生成Image对象。

### PictureRecorder

其主要作用是记录在Canvas上执行的「graphical operations」，通过Picture#endRecording最终生成Picture。

### Scene

一系列 Picture、Texture 合成的结果。UI 帧刷新时，在 Rendering Pipeline 中 Flutter UI 经 build、layout、paint 等步骤后最终生成 Scene。
其后通过window.render将该 Scene 送入 Engine 层，最终经 GPU 光栅化后显示在屏幕上。

### SceneBuilder

用于将多个图层(Layer)、Picture、Texture 合成为 Scene。

```dart

void main() {
  PictureRecorder recorder = PictureRecorder();
  // 初始化 Canvas 时，传入 PictureRecorder 实例
  // 用于记录发生在该 canvas 上的所有操作
  //
  Canvas canvas = Canvas(recorder);

  Paint circlePaint= Paint();
  circlePaint.color = Colors.blueAccent;

  // 调用 Canvas 的绘制接口，画一个圆形
  //
  canvas.drawCircle(Offset(400, 400), 300, circlePaint);

  // 绘制结束，生成Picture
  //
  Picture picture = recorder.endRecording();

  SceneBuilder sceneBuilder = SceneBuilder();
  sceneBuilder.pushOffset(0, 0);
  // 将 picture 送入 SceneBuilder
  //
  sceneBuilder.addPicture(Offset(0, 0), picture);
  sceneBuilder.pop();

  // 生成 Scene
  //
  Scene scene = sceneBuilder.build();

  window.onDrawFrame = () {
    // 将 scene 送入 Engine 层进行渲染显示
    //
    window.render(scene);
  };
  window.scheduleFrame();
}

```


## PaintingContext

```dart

class PaintingContext extends ClipContext {

  @protected
  PaintingContext(this._containerLayer, this.estimatedBounds);

  final ContainerLayer _containerLayer;//Layer树的根节点

  final Rect estimatedBounds;

  static void repaintCompositedChild(RenderObject child, { bool debugAlsoPaintedParent = false }) {
    _repaintCompositedChild(
      child,
      debugAlsoPaintedParent: debugAlsoPaintedParent,
    );
  }

  static void _repaintCompositedChild(
    RenderObject child, {
    bool debugAlsoPaintedParent = false,
    PaintingContext? childContext,
  }) {
    OffsetLayer? childLayer = child._layerHandle.layer as OffsetLayer?;
    if (childLayer == null) {
      final OffsetLayer layer = child.updateCompositedLayer(oldLayer: null);//创建layer
      child._layerHandle.layer = childLayer = layer;
    } else {
      Offset? debugOldOffset;
      childLayer.removeAllChildren();
      final OffsetLayer updatedLayer = child.updateCompositedLayer(oldLayer: childLayer);
    }
    child._needsCompositedLayerUpdate = false;
    childContext ??= PaintingContext(childLayer, child.paintBounds);
    child._paintWithContext(childContext, Offset.zero);

    childContext.stopRecordingIfNeeded();
  }

  static void updateLayerProperties(RenderObject child) {
    final OffsetLayer childLayer = child._layerHandle.layer! as OffsetLayer;
    Offset? debugOldOffset;
    final OffsetLayer updatedLayer = child.updateCompositedLayer(oldLayer: childLayer);
    child._needsCompositedLayerUpdate = false;
  }

  static void debugInstrumentRepaintCompositedChild(
    RenderObject child, {
    bool debugAlsoPaintedParent = false,
    required PaintingContext customContext,
  }) {
  }

  void paintChild(RenderObject child, Offset offset) {
    if (child.isRepaintBoundary) {
      stopRecordingIfNeeded();
      _compositeChild(child, offset);
    } else if (child._wasRepaintBoundary) {
      child._layerHandle.layer = null;
      child._paintWithContext(this, offset);
    } else {
      child._paintWithContext(this, offset);
    }
  }

  void _compositeChild(RenderObject child, Offset offset) {

    // Create a layer for our child, and paint the child into it.
    if (child._needsPaint || !child._wasRepaintBoundary) {
      repaintCompositedChild(child, debugAlsoPaintedParent: true);
    } else {
      if (child._needsCompositedLayerUpdate) {
        updateLayerProperties(child);
      }
    }
    final OffsetLayer childOffsetLayer = child._layerHandle.layer! as OffsetLayer;
    childOffsetLayer.offset = offset;
    appendLayer(childOffsetLayer);
  }

  @protected
  void appendLayer(Layer layer) {
    layer.remove();
    _containerLayer.append(layer);
  }

  bool get _isRecording {
    final bool hasCanvas = _canvas != null;
    return hasCanvas;
  }

/// 用于具体的绘制操作
  PictureLayer? _currentLayer;
  ui.PictureRecorder? _recorder;
  Canvas? _canvas;

  @override
  Canvas get canvas {
    if (_canvas == null) {
      _startRecording();
    }
    return _canvas!;
  }

  void _startRecording() {
    _currentLayer = PictureLayer(estimatedBounds);
    _recorder = ui.PictureRecorder();
    _canvas = Canvas(_recorder!);
    _containerLayer.append(_currentLayer!);
  }

  VoidCallback addCompositionCallback(CompositionCallback callback) {
    return _containerLayer.addCompositionCallback(callback);
  }

  @protected
  @mustCallSuper
  void stopRecordingIfNeeded() {
    if (!_isRecording) {
      return;
    }
    _currentLayer!.picture = _recorder!.endRecording();
    _currentLayer = null;
    _recorder = null;
    _canvas = null;
  }

  void setIsComplexHint() {
    _currentLayer?.isComplexHint = true;
  }

  void setWillChangeHint() {
    _currentLayer?.willChangeHint = true;
  }

  void addLayer(Layer layer) {
    stopRecordingIfNeeded();
    appendLayer(layer);
  }

  void pushLayer(ContainerLayer childLayer, PaintingContextCallback painter, Offset offset, { Rect? childPaintBounds }) {
    if (childLayer.hasChildren) {
      childLayer.removeAllChildren();
    }
    // 在 append sub layer 前先终止现有的绘制操作
    // stopRecordingIfNeeded 所执行的操作见上文
    stopRecordingIfNeeded();
    appendLayer(childLayer);
      // 为 childLayer 创建新的 PaintingContext，以便独立进行绘制操作
    final PaintingContext childContext = createChildContext(childLayer, childPaintBounds ?? estimatedBounds);

    painter(childContext, offset);
    childContext.stopRecordingIfNeeded();
  }

  @protected
  PaintingContext createChildContext(ContainerLayer childLayer, Rect bounds) {
    return PaintingContext(childLayer, bounds);
  }

  ClipRectLayer? pushClipRect(bool needsCompositing, Offset offset, Rect clipRect, PaintingContextCallback painter, { Clip clipBehavior = Clip.hardEdge, ClipRectLayer? oldLayer }) {
    if (clipBehavior == Clip.none) {
      painter(this, offset);
      return null;
    }
    final Rect offsetClipRect = clipRect.shift(offset);
    if (needsCompositing) {
      final ClipRectLayer layer = oldLayer ?? ClipRectLayer();
      layer
        ..clipRect = offsetClipRect
        ..clipBehavior = clipBehavior;
      pushLayer(layer, painter, offset, childPaintBounds: offsetClipRect);
      return layer;
    } else {
      clipRectAndPaint(offsetClipRect, clipBehavior, offsetClipRect, () => painter(this, offset));
      return null;
    }
  }

  ClipRRectLayer? pushClipRRect(bool needsCompositing, Offset offset, Rect bounds, RRect clipRRect, PaintingContextCallback painter, { Clip clipBehavior = Clip.antiAlias, ClipRRectLayer? oldLayer }) {
    if (clipBehavior == Clip.none) {
      painter(this, offset);
      return null;
    }
    final Rect offsetBounds = bounds.shift(offset);
    final RRect offsetClipRRect = clipRRect.shift(offset);
    if (needsCompositing) {
      final ClipRRectLayer layer = oldLayer ?? ClipRRectLayer();
      layer
        ..clipRRect = offsetClipRRect
        ..clipBehavior = clipBehavior;
      pushLayer(layer, painter, offset, childPaintBounds: offsetBounds);
      return layer;
    } else {
      clipRRectAndPaint(offsetClipRRect, clipBehavior, offsetBounds, () => painter(this, offset));
      return null;
    }
  }

  ClipPathLayer? pushClipPath(bool needsCompositing, Offset offset, Rect bounds, Path clipPath, PaintingContextCallback painter, { Clip clipBehavior = Clip.antiAlias, ClipPathLayer? oldLayer }) {
    if (clipBehavior == Clip.none) {
      painter(this, offset);
      return null;
    }
    final Rect offsetBounds = bounds.shift(offset);
    final Path offsetClipPath = clipPath.shift(offset);
    if (needsCompositing) {
      final ClipPathLayer layer = oldLayer ?? ClipPathLayer();
      layer
        ..clipPath = offsetClipPath
        ..clipBehavior = clipBehavior;
      pushLayer(layer, painter, offset, childPaintBounds: offsetBounds);
      return layer;
    } else {
      clipPathAndPaint(offsetClipPath, clipBehavior, offsetBounds, () => painter(this, offset));
      return null;
    }
  }

  ColorFilterLayer pushColorFilter(Offset offset, ColorFilter colorFilter, PaintingContextCallback painter, { ColorFilterLayer? oldLayer }) {
    final ColorFilterLayer layer = oldLayer ?? ColorFilterLayer();
    layer.colorFilter = colorFilter;
    pushLayer(layer, painter, offset);
    return layer;
  }

  TransformLayer? pushTransform(bool needsCompositing, Offset offset, Matrix4 transform, PaintingContextCallback painter, { TransformLayer? oldLayer }) {
    final Matrix4 effectiveTransform = Matrix4.translationValues(offset.dx, offset.dy, 0.0)
      ..multiply(transform)..translate(-offset.dx, -offset.dy);
    if (needsCompositing) {
      final TransformLayer layer = oldLayer ?? TransformLayer();
      layer.transform = effectiveTransform;
      pushLayer(
        layer,
        painter,
        offset,
        childPaintBounds: MatrixUtils.inverseTransformRect(effectiveTransform, estimatedBounds),
      );
      return layer;
    } else {
      canvas
        ..save()
        ..transform(effectiveTransform.storage);
      painter(this, offset);
      canvas.restore();
      return null;
    }
  }

  OpacityLayer pushOpacity(Offset offset, int alpha, PaintingContextCallback painter, { OpacityLayer? oldLayer }) {
    final OpacityLayer layer = oldLayer ?? OpacityLayer();
    layer
      ..alpha = alpha
      ..offset = offset;
    pushLayer(layer, painter, Offset.zero);
    return layer;
  }

  @override
  String toString() => '${objectRuntimeType(this, 'PaintingContext')}#$hashCode(layer: $_containerLayer, canvas bounds: $estimatedBounds)';
}
```

## 绘制流程

![](./RenderObject_2.png)



## Compositing

Compositing，合成，属于 Rendering Pipeline 中的一环，表示是否要生成新的 Layer 来实现某些特定的图形效果

## 参考

- [深入浅出 Flutter Framework 之 PaintingContext](https://zxfcumtcs.github.io/2020/05/23/deepinto-flutter-paintingcontext/)