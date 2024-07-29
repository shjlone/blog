---
title: Image原理分析
toc: true
tags: Flutter
---


![](./image1.png)

_ImageState是Image所对应的State，也是图片加载的驱动者，它将通过ImageProvider的resolve方法获取一个ImageStream对象。

ImageStream负责提供图片信息的数据流，其对应的监听器由_ImageState的_getListener方法提供。而ImageStream的主要工作将委托给ImageStreamCompleter，因此其持有的ImageStreamListener也将传递给ImageStreamCompleter。图片信息真正的加载由ImageProvider的子类实现，例如NetworkImage将从网络加载图片，加载并解码后的图片信息ImageInfo交由ImageStreamCompleter进行通知。

PaintBinding将持有一个ImageCache实例，用于全局图片缓存的管理。


## 框架分析

_ImageState的didChangeDependencies方法发起图片的解析


```dart
class _ImageState {
  void didChangeDependencies() {
    _updateInvertColors();
    _resolveImage();

    if (TickerMode.of(context)) {
      _listenToStream();
    } else {
      _stopListeningToStream(keepStreamAlive: true);
    }

    super.didChangeDependencies();
  }

  void _resolveImage() {
    final ScrollAwareImageProvider provider = ScrollAwareImageProvider<Object>(
      context: _scrollAwareContext,
      imageProvider: widget.image,//真正加载图片的ImageProvider
    );
    final ImageStream newStream =
      provider.resolve(createLocalImageConfiguration(
        context,
        size: widget.width != null && widget.height != null ? Size(widget.width!, widget.height!) : null,
      ));
    _updateSourceStream(newStream);
  }

/// 更新图片
  void _updateSourceStream(ImageStream newStream) {
    if (_imageStream?.key == newStream.key) {
      return;
    }

    if (_isListeningToStream) {
      _imageStream!.removeListener(_getListener());
    }

    if (!widget.gaplessPlayback) {//新图片加载期间是否维持旧的图片信息
      setState(() { _replaceImage(info: null); });
    }

    setState(() {//重置相关字段
      _loadingProgress = null;
      _frameNumber = null;
      _wasSynchronouslyLoaded = false;
    });

    _imageStream = newStream;//增加监听
    if (_isListeningToStream) {
      _imageStream!.addListener(_getListener());
    }
  }

  ImageStreamListener _getListener({bool recreateListener = false}) {
    if(_imageStreamListener == null || recreateListener) {
      _lastException = null;
      _lastStack = null;
      _imageStreamListener = ImageStreamListener(
        _handleImageFrame,
        onChunk: widget.loadingBuilder == null ? null : _handleImageChunk,
        onError: widget.errorBuilder != null || kDebugMode
            ? (Object error, StackTrace? stackTrace) {
                setState(() {
                  _lastException = error;
                  _lastStack = stackTrace;
                });
              }
            : null,
      );
    }
    return _imageStreamListener!;
  }

/// 真实更新图片的地方
  void _handleImageFrame(ImageInfo imageInfo, bool synchronousCall) {
    setState(() {
      _replaceImage(info: imageInfo);
      _loadingProgress = null;
      _lastException = null;
      _lastStack = null;
      _frameNumber = _frameNumber == null ? 0 : _frameNumber! + 1;
      _wasSynchronouslyLoaded = _wasSynchronouslyLoaded | synchronousCall;
    });
  }


}
```

### ImageProvider.resolve

```dart
class ImageProvider {
  ImageStream resolve(ImageConfiguration configuration) {
    final ImageStream stream = createStream(configuration);
    //解析当前ImageConfiguration所对应的Key，并基于Key进行图片的加载
    _createErrorHandlerAndKey(
      configuration,
      (T key, ImageErrorListener errorHandler) {
        resolveStreamForKey(configuration, stream, key, errorHandler);
      },
      (T? key, Object exception, StackTrace? stack) async {...},
    );
    return stream;
  }

  void _createErrorHandlerAndKey(
    ImageConfiguration configuration,
    _KeyAndErrorHandlerCallback<T> successCallback,
    _AsyncKeyErrorHandler<T?> errorCallback,
  ) {
    T? obtainedKey;
    bool didError = false;
    Future<void> handleError(Object exception, StackTrace? stack) async {
      if (didError) {
        return;
      }
      if (!didError) {
        didError = true;
        errorCallback(obtainedKey, exception, stack);
      }
    }

    Future<T> key;
    try {
      key = obtainKey(configuration);//获取图片的key
    } catch (error, stackTrace) {
      handleError(error, stackTrace);
      return;
    }
    key.then<void>((T key) {
      obtainedKey = key;
      try {//获取key成功后，加载图片
        successCallback(key, handleError);//resolveStreamForKey
      } catch (error, stackTrace) {
        handleError(error, stackTrace);
      }
    }).catchError(handleError);
  }

//加载图片
  void resolveStreamForKey(ImageConfiguration configuration, ImageStream stream, T key, ImageErrorListener handleError) {
    if (stream.completer != null) {
      final ImageStreamCompleter? completer = PaintingBinding.instance.imageCache.putIfAbsent(
        key,
        () => stream.completer!,
        onError: handleError,
      );
      assert(identical(completer, stream.completer));
      return;
    }
    //通过key从缓存中依次加载，没有缓存则执行load方法
    final ImageStreamCompleter? completer = PaintingBinding.instance.imageCache.putIfAbsent(
      key,
      () {
        //如果缓存没有，则通过load创建一个新的实例
        ImageStreamCompleter result = loadImage(key, PaintingBinding.instance.instantiateImageCodecWithSize);
        if (result is _AbstractImageStreamCompleter) {
          result = loadBuffer(key, PaintingBinding.instance.instantiateImageCodecFromBuffer);
          if (result is _AbstractImageStreamCompleter) {
            result = load(key, PaintingBinding.instance.instantiateImageCodec);//子类具体实现，比如NetworkImage
          }
        }
        return result;
      },
      onError: handleError,
    );
    if (completer != null) {
      stream.setCompleter(completer);
    }
  }

}

```

### NetworkImage.load


```dart
class NetworkImage {
  ImageStreamCompleter load(image_provider.NetworkImage key, image_provider.DecoderCallback decode) {
    final StreamController<ImageChunkEvent> chunkEvents = StreamController<ImageChunkEvent>();

//创建MultiFrameImageStreamCompleter，图片加载成功后的处理逻辑
    return MultiFrameImageStreamCompleter(
      codec: _loadAsync(key as NetworkImage, chunkEvents, decodeDeprecated: decode),//decode为图片解码器
      chunkEvents: chunkEvents.stream,//加载进度监听
      scale: key.scale,
      debugLabel: key.url,
      informationCollector: () => <DiagnosticsNode>[
        DiagnosticsProperty<image_provider.ImageProvider>('Image provider', this),
        DiagnosticsProperty<image_provider.NetworkImage>('Image key', key),
      ],
    );
  }

  Future<ui.Codec> _loadAsync(
    NetworkImage key,
    StreamController<ImageChunkEvent> chunkEvents, {
    image_provider.ImageDecoderCallback? decode,
    image_provider.DecoderBufferCallback? decodeBufferDeprecated,
    image_provider.DecoderCallback? decodeDeprecated,
  }) async {
    try {
      assert(key == this);

      final Uri resolved = Uri.base.resolve(key.url);//解析图片资源地址

      final HttpClientRequest request = await _httpClient.getUrl(resolved);

      headers?.forEach((String name, String value) {
        request.headers.add(name, value);
      });
      final HttpClientResponse response = await request.close();
      if (response.statusCode != HttpStatus.ok) {
        await response.drain<List<int>>(<int>[]);
        throw image_provider.NetworkImageLoadException(statusCode: response.statusCode, uri: resolved);
      }

      final Uint8List bytes = await consolidateHttpClientResponseBytes(//下载数据
        response,
        onBytesReceived: (int cumulative, int? total) {//当前进度
          chunkEvents.add(ImageChunkEvent(//对外通知
            cumulativeBytesLoaded: cumulative,
            expectedTotalBytes: total,
          ));
        },
      );
      if (bytes.lengthInBytes == 0) {//没有数据
        throw Exception('NetworkImage is an empty file: $resolved');
      }
      //解析图片信息
      if (decode != null) {
        final ui.ImmutableBuffer buffer = await ui.ImmutableBuffer.fromUint8List(bytes);
        return decode(buffer);
      } else if (decodeBufferDeprecated != null) {
        final ui.ImmutableBuffer buffer = await ui.ImmutableBuffer.fromUint8List(bytes);
        return decodeBufferDeprecated(buffer);
      } else {
        assert(decodeDeprecated != null);
        return decodeDeprecated!(bytes);
      }
    } catch (e) {
      scheduleMicrotask(() {
        PaintingBinding.instance.imageCache.evict(key);
      });
      rethrow;
    } finally {
      chunkEvents.close();
    }
  }
}

```

### MultiFrameImageStreamCompleter

```dart
class MultiFrameImageStreamCompleter {
    MultiFrameImageStreamCompleter({
    required Future<ui.Codec> codec,
    required double scale,
    String? debugLabel,
    Stream<ImageChunkEvent>? chunkEvents,
    InformationCollector? informationCollector,
  }) : _informationCollector = informationCollector,
       _scale = scale {
    this.debugLabel = debugLabel;
    codec.then<void>(_handleCodecReady, onError: (Object error, StackTrace stack) {
    });
    if (chunkEvents != null) {
      _chunkSubscription = chunkEvents.listen(reportImageChunkEvent,
        onError: (Object error, StackTrace stack) {
        },
      );
    }
  }
/// 处理加载好的数据
  void _handleCodecReady(ui.Codec codec) {
    _codec = codec;
    assert(_codec != null);

    if (hasListeners) {
      _decodeNextFrameAndSchedule();
    }
  }

  Future<void> _decodeNextFrameAndSchedule() async {
    _nextFrame?.image.dispose();
    _nextFrame = null;
    try {
      _nextFrame = await _codec!.getNextFrame();//获取一帧
    } catch (exception, stack) {
      return;
    }
    if (_codec!.frameCount == 1) {//图片只有一帧，则会进入emitFrame
      if (!hasListeners) {
        return;
      }
      _emitFrame(ImageInfo(
        image: _nextFrame!.image.clone(),
        scale: _scale,
        debugLabel: debugLabel,
      ));
      _nextFrame!.image.dispose();
      _nextFrame = null;
      return;//单帧和多帧是互斥的
    }
    _scheduleAppFrame();//图片存在多帧
  }

  void _emitFrame(ImageInfo imageInfo) {
    setImage(imageInfo);
    _framesEmitted += 1;
  }

  void setImage(ImageInfo image) {
    _checkDisposed();
    _currentImage?.dispose();//释放资源
    _currentImage = image;//新的突破信息

    if (_listeners.isEmpty) {
      return;
    }
    // Make a copy to allow for concurrent modification.
    final List<ImageStreamListener> localListeners =
        List<ImageStreamListener>.of(_listeners);
    for (final ImageStreamListener listener in localListeners) {
      try {
        listener.onImage(image.clone(), false);//通知图片更新，_ImageState的_updateSourceStream-->_getListener-->_handleImageFrame
      } catch (exception, stack) {
      }
    }
  }
}
```

## 缓存管理

ImageCache进行缓存管理

```dart
class ImageCache {
  ImageStreamCompleter? putIfAbsent(Object key, ImageStreamCompleter Function() loader, { ImageErrorListener? onError }) {
    TimelineTask? timelineTask;
    TimelineTask? listenerTask;
    ImageStreamCompleter? result = _pendingImages[key]?.completer;
    if (result != null) {//正在加载，直接返回
      return result;
    }
    final _CachedImage? image = _cache.remove(key);//使用缓存
    if (image != null) {
      if (!kReleaseMode) {
        timelineTask!.finish(arguments: <String, dynamic>{'result': 'keepAlive'});
      }
      _trackLiveImage(
        key,
        image.completer,
        image.sizeBytes,
      );
      _cache[key] = image;//注册新的key
      return image.completer;
    }

    final _LiveImage? liveImage = _liveImages[key];
    if (liveImage != null) {
      _touch(
        key,
        _CachedImage(
          liveImage.completer,
          sizeBytes: liveImage.sizeBytes,
        ),
        timelineTask,
      );
      if (!kReleaseMode) {
        timelineTask!.finish(arguments: <String, dynamic>{'result': 'keepAlive'});
      }
      return liveImage.completer;
    }

    try {
      result = loader();//没有缓存，开始加载
      _trackLiveImage(key, result, null);
    } catch (error, stackTrace) {
      if (onError != null) {
        onError(error, stackTrace);
        return null;
      } else {
        rethrow;
      }
    }

    if (!kReleaseMode) {
      listenerTask = TimelineTask(parent: timelineTask)..start('listener');
    }
    bool listenedOnce = false;

    final bool trackPendingImage = maximumSize > 0 && maximumSizeBytes > 0;
    late _PendingImage pendingImage;

    void listener(ImageInfo? info, bool syncCall) {
      int? sizeBytes;//图片大小
      if (info != null) {
        sizeBytes = info.sizeBytes;
        info.dispose();
      }
      //构造缓存
      final _CachedImage image = _CachedImage(
        result!,
        sizeBytes: sizeBytes,
      );

      _trackLiveImage(key, result, sizeBytes);

      // Only touch if the cache was enabled when resolve was initially called.
      if (trackPendingImage) {
        _touch(key, image, listenerTask);
      } else {
        image.dispose();
      }

      _pendingImages.remove(key);
      if (!listenedOnce) {
        pendingImage.removeListener();
      }
      if (!kReleaseMode && !listenedOnce) {
        listenerTask!.finish(arguments: <String, dynamic>{
          'syncCall': syncCall,
          'sizeInBytes': sizeBytes,
        });
        timelineTask!.finish(arguments: <String, dynamic>{
          'currentSizeBytes': currentSizeBytes,
          'currentSize': currentSize,
        });
      }
      listenedOnce = true;
    }

    final ImageStreamListener streamListener = ImageStreamListener(listener);//定义监听器
    pendingImage = _PendingImage(result, streamListener);
    if (trackPendingImage) {
      _pendingImages[key] = pendingImage;
    }
    // Listener is removed in [_PendingImage.removeListener].
    result.addListener(streamListener);

    return result;
  }

  void _touch(Object key, _CachedImage image, TimelineTask? timelineTask) {
    if (image.sizeBytes != null && image.sizeBytes! <= maximumSizeBytes && maximumSize > 0) {
      _currentSizeBytes += image.sizeBytes!;//已缓存图片大小
      _cache[key] = image;//添加缓存
      _checkCacheSize(timelineTask);//检查缓存大小，并清理缓存
    } else {
      image.dispose();//图片过大，释放资源
    }
  }

  void _trackLiveImage(Object key, ImageStreamCompleter completer, int? sizeBytes) {
    //加入缓存队列
    _liveImages.putIfAbsent(key, () {
      return _LiveImage(
        completer,
        () {
          _liveImages.remove(key);
        },
      );
    }).sizeBytes ??= sizeBytes;
  }

  void _checkCacheSize(TimelineTask? timelineTask) {
    final Map<String, dynamic> finishArgs = <String, dynamic>{};
    TimelineTask? checkCacheTask;
    while (_currentSizeBytes > _maximumSizeBytes || _cache.length > _maximumSize) {
      final Object key = _cache.keys.first;//取出最先加入的图片
      final _CachedImage image = _cache[key]!;
      _currentSizeBytes -= image.sizeBytes!;//更新大小
      image.dispose();//释放资源
      _cache.remove(key);//移除缓存注册信息
    }
  }

} 
```


## 参考

- [Flutter源码内核剖析]()