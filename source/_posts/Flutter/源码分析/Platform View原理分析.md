---
title: Platform View原理分析
toc: true
tags: Flutter
---

![](./platform1.png)

AndroidView、PlatformViewLink、UiKitView是表示Platform View的Widget接口，它们底层分别对应的RenderObject是RenderAndroidView、PlatformViewRenderBox、RenderUiKitView，前者基于TextureLayer进行真正的绘制，后两者给予PlatformViewLayer进行绘制。使用TextureLayer方式被称为Virtual Display，仅Android支持。使用PlatformViewLayer被称为Hybrid Composition，Android和iOS都支持。PlatformViewController和UiKitViewController时对Platform View中使用的Platform Channel的抽象封装，用于控制Platform View在Embedder中的各种属性和表现。


## Virtual Display原理分析

```dart
Widget build(BuildContext context) {
  return AndroidView(
    viewType: 'webview',//用于Embedder侧查找对应的Platform View
    createionParams: xx, //初始化参数
    createionParamsCodec: StandardMessageCodec(),//编解码规则
  );
}
```

AndroidView对应RenderObject是RenderAndroidView

```dart
class RenderAndroidView {
  void paint(PaintingContext context, Offset offset) {
    if (_viewController.textureId == null || _currentTextureSize == null) {
      return;
    }
    final bool isTextureLargerThanWidget = _currentTextureSize!.width > size.width ||
                                           _currentTextureSize!.height > size.height;
    if (isTextureLargerThanWidget && clipBehavior != Clip.none) {
      _clipRectLayer.layer = context.pushClipRect(
        true,
        offset,
        offset & size,
        _paintTexture,
        clipBehavior: clipBehavior,
        oldLayer: _clipRectLayer.layer,
      );
      return;
    }
    _clipRectLayer.layer = null;
    _paintTexture(context, offset);//真正的绘制过程
  }

  void _paintTexture(PaintingContext context, Offset offset) {
    if (_currentTextureSize == null) {
      return;
    }

    context.addLayer(TextureLayer(//添加一个独立图层
      rect: offset & _currentTextureSize!,
      textureId: _viewController.textureId!,
    ));
  }
}
```


Virtual Display方案存在一些问题，比如无法响应复杂的手势。于是出现了Hybrid Composition


## Hybrid Composition原理分析

```dart
Widget build(BuildContext context) {
  const String viewType = '<platform-view-type>';
  const Map<String, dynamic> creationParams = <String, dynamic>{};

  return PlatformViewLink(
    viewType: viewType,
    surfaceFactory: (context, controller) {
      return AndroidViewSurface(
        controller: controller as AndroidViewController,
        gestureRecognizers: const <Factory<OneSequenceGestureRecognizer>>{},
        hitTestBehavior: PlatformViewHitTestBehavior.opaque,
      );
    },
    onCreatePlatformView: (params) {
      return PlatformViewsService.initSurfaceAndroidView(
        id: params.id,
        viewType: viewType,
        layoutDirection: TextDirection.ltr,
        creationParams: creationParams,
        creationParamsCodec: const StandardMessageCodec(),
        onFocus: () {
          params.onFocusChanged(true);
        },
      )
        ..addOnPlatformViewCreatedListener(params.onPlatformViewCreated)
        ..create();
    },
  );
}
```


PlatformViewLink对应RenderObject时PlatformViewRenderBox

```dart
class PlatformViewRenderBox {
  void paint(PaintingContext context, Offset offset) {
    context.addLayer(PlatformViewLayer(
      rect: offset & size,
      viewId: _controller.viewId,//该id由onCreatPlatformView决定
    ));
  }
}

class _PlatformViewLinkState {
  void _initialize() {
    _id = platformViewsRegistry.getNextPlatformViewId();
    _controller = widget._onCreatePlatformView(
      PlatformViewCreationParams._(
        id: _id!,
        viewType: widget.viewType,
        onPlatformViewCreated: _onPlatformViewCreated,
        onFocusChanged: _handlePlatformFocusChanged,
      ),
    );
  }
}

```

Hybrid Composition的流程和Virtual Display相差很大，下面从Engine和Embedder两个阶段进行分析

### Engine处理阶段



Flutter的UI最终绘制于Raster线程，而Hybrid Composition模式下的Platform View是原生View，需要绘制于Platform线程，由于两者需要同帧绘制，因此Flutter会做出牺牲，将自身UI也通过Platform线程进程绘制。




### Embedder处理阶段

1. 将当前用于渲染Flutter UI的FlutterSurfaceView转换为FlutterImageView，Hybrid Composition模式下将使用后者渲染Flutter UI


## 参考

- [Hosting native Android views in your Flutter app with Platform Views](https://docs.flutter.dev/platform-integration/android/platform-views)