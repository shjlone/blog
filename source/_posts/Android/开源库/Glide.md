---
title: Glide
tags: Android
toc: true
---


> 此篇文章只是用于记录使用Glide时要注意的点和一些使用技巧，如需查看基本使用，
请查阅[官方文档](http://bumptech.github.io/glide/)。也可以看看[Glide原理分析](../Glide原理分析/)。

## 注意点

### 占位符

```java
Glide.with(this)
    .load(url)
    .asBitmap() //指定加载的图片是位图，这样底层不会进行图片格式判断，Gif就无法播放，也可以设置asGif()指明是Gif图片
    .placeholder(R.drawable.loading)//下载过程中的占位符
    .error(R.drawable.error)//下载失败后的显示
    .diskCacheStrategy(DiskCacheStrategy.NONE)//磁盘策略，这里不使用缓存
    .into(imageView);
```

### **缓存等级**

#### 活动缓存

#### 内存缓存

默认开启，可以通过skipMemoryCache(true)关闭内存缓存。
  
#### 磁盘缓存

InternalCacheDiskCacheFactory（默认）

- DiskCacheStrategy.NONE 不缓存文件
- DiskCacheStrategy.DATA 对应 Glide 3 中的 DiskCacheStrategy.SOURCE， 只缓存原图
- DiskCacheStrategy.RESOURCE 对应 Glide 3 中的 DiskCacheStrategy.RESULT， 只缓存最终加载的图
- DiskCacheStrategy.ALL 同时缓存原图和结果图
- DiskCacheStrategy.AUTOMATIC:表示让 Glide 根据图片资源智能地选择使用哪一 种缓存策略(默认选项)。

### **缓存键**

```java
//缓存的键包括图片的宽、高、signature等参数
EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
resourceClass, transcodeClass, options)
```

### submit

submit()方法其实就是对应的 Glide 3 中的 downloadOnly()方法，和 preload()方法 类似，submit()方法也是可以替换 into()方法的，
不过 submit()方法的用法明显要 比 preload()方法复杂不少。这个方法只会下载图片，而不会对图片进行加载。当 图片下载完成之后，
我们可以得到图片的存储路径，以便后续进行操作。

## 重要知识点

- 通过attach一个Fragment来监听Context的生命周期，合理的管理图片的加载和释放。
- Glide默认采用的是RGB-565，相比ARGB-8888内存占用会减小一半。
- 会根据ImageView的尺寸来缓存图片。

## 常用配置

### 整合OkHttp3

为了提升请求效率，使用更高效的OkHttp3来进行资源的下载。接入方式很简单，直接在gradle中引入

> compile "com.github.bumptech.glide:okhttp3-integration:4.11.0"

它会自动添加以下配置

```java
@GlideModule
public final class OkHttpLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(@NonNull Context context, @NonNull Glide glide,
      @NonNull Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}
```

## 注意点

- 使用时尽量传生命周期所对应的Context（比如Activity、Fragment）
- 如果使用ApplicationContext，则使用弱引用

## 参考

- [https://github.com/bumptech/glide](https://github.com/bumptech/glide)
- [官方文档](http://bumptech.github.io/glide/)
