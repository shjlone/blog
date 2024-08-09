---
title: WindowManager
tags: Android
toc: true
---


## 概述

Window是一个抽象的概念，每一个Window都对应着一个View和一个ViewRootImpl，Window和View通过ViewRootImpl来建立联系，因此Window并不是实际存在的，
它是以View的形式存在。

Android系统中Window有三种类型，分别是应用Window、子Window和系统Window。应用类Window对应着一个Activity。子Window不能单独存在，它需要附属在特定的父Window之中，
比如常见的一些Dialog就是一个子Window。系统Window是需要声明权限在能创建的Window，比如Toast和系统状态栏这些都是系统Window。

Window是分层的，每个Window都有对应的z-ordered，层级大的会覆盖在层级小的Window的上面。在三类Window中，
应用Window的层级范围是1~99，子Window的层级范围是1000~1999，系统Window的层级范围是2000~2999，这些层级范围对应着WindowManager.LayoutParams的type参数。
如果想要Window位于所有Window的最顶层，那么采用较大的层级即可。很显然系统Window的层级是最大的，而且系统层级有很多值，一般我们可以选用TYPE_SYSTEM_OVERLAY
或者TYPE_SYSTEM_ERROR，如果采用TYPE_SYSTEM_ERROR，只需要为type参数指定这个层级即可:mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR;
同时声明权限:<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />。因为系统类型的Window是需要检查权限的，
如果不在AndroidManifest中使用相应的权限，那么创建Window的时候就会报错，

![](./3.jpg)

## 流程分析

WindowManager实现了ViewManager接口，ViewManager接口提供以下三个方法：

```java
public void addView(View view, ViewGroup.LayoutParams params);
public void updateViewLayout(View view, ViewGroup.LayoutParams params);
public void removeView(View view);
```

### addView过程

WindowManager在Android中的实现类为WindowManagerImpl，WindowManagerImpl内部使用桥接模式封装了WindowManagerGlobal。

```java
//WindowManagerGlobal.java
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow) {
    if (view == null) {
        throw new IllegalArgumentException("view must not be null");
    }
    if (display == null) {
        throw new IllegalArgumentException("display must not be null");
    }
    if (!(params instanceof WindowManager.LayoutParams)) {
        throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
    }

    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        // If there's no parent, then hardware acceleration for this view is
        // set from the application's hardware acceleration setting.
        final Context context = view.getContext();
        if (context != null
                && (context.getApplicationInfo().flags
                        & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
            wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
        }
    }

    ViewRootImpl root;
    View panelParentView = null;

    synchronized (mLock) {
        // Start watching for system property changes.
        if (mSystemPropertyUpdater == null) {
            mSystemPropertyUpdater = new Runnable() {
                @Override public void run() {
                    synchronized (mLock) {
                        for (int i = mRoots.size() - 1; i >= 0; --i) {
                            mRoots.get(i).loadSystemProperties();
                        }
                    }
                }
            };
            SystemProperties.addChangeCallback(mSystemPropertyUpdater);
        }

        int index = findViewLocked(view, false);//判断是否已经添加
        if (index >= 0) {
            if (mDyingViews.contains(view)) {
                // Don't wait for MSG_DIE to make it's way through root's queue.
                mRoots.get(index).doDie();
            } else {
                throw new IllegalStateException("View " + view
                        + " has already been added to the window manager.");
            }
            // The previous removeView() had not completed executing. Now it has.
        }

        // If this is a panel window, then find the window it is being
        // attached to for future reference.
        if (wparams.type >= WindowManager.LayoutParams.FIRST_SUB_WINDOW &&
                wparams.type <= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
            final int count = mViews.size();
            for (int i = 0; i < count; i++) {
                if (mRoots.get(i).mWindow.asBinder() == wparams.token) {
                    panelParentView = mViews.get(i);
                }
            }
        }

        root = new ViewRootImpl(view.getContext(), display);

        view.setLayoutParams(wparams);

        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);

        // do this last because it fires off messages to start doing things
        try {
            root.setView(view, wparams, panelParentView);//PhoneWindow要开始创建
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            if (index >= 0) {
                removeViewLocked(index, true);
            }
            throw e;
        }
    }
}
```

mViews存储的是所有Window所对应的View，mRoots存储的是所有Window所对应的ViewRootImpl，mParams存储的是所有Window所对应的布局参数，
mDyingViews则存储了那些正在被删除的View对象，或者说是那些已经调用removeView方法但是删除操作还未完成的Window对象。

总结如下：

1. 检查参数是否合法，如果是子Window那么还需要调整一些布局参数
2. 创建ViewRootImpl并将View添加到列表中
3. 通过ViewRootImpl来更新界面并完成Window的添加过程
    1. 在setView内部会通过requestLayout来完成异步刷新请求
    2. WindowSession最终来完成Window的添加过程
    3. 在Session内部会通过WindowManagerService来实现Window的添加

![](./2.jpg)

### removeView过程

### updateView过程

## Window的创建过程

### Activity的Window创建过程

由[Activity工作过程](/wiki/Android/基础/Activity工作过程/)可知，Activity启动时会调用ActivityThread的performLaunchActivity方法来完成
整个启动流程。

performLaunchActivity会调用Activity的attach方法。在这个方法里，会创建PhoneWindow对象。

![](./1.jpg)

接下来看看setContentView做了哪些事情

```java

//PhoneWindow.java
public void setContentView(int layoutResID) {
    // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
    // decor, when theme attributes and the like are crystalized. Do not check the feature
    // before this happens.
    if (mContentParent == null) {
        installDecor();
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }

    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                getContext());
        transitionTo(newScene);
    } else {
        mLayoutInflater.inflate(layoutResID, mContentParent);
    }
    mContentParent.requestApplyInsets();
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        cb.onContentChanged();
    }
    mContentParentExplicitlySet = true;
}

private void installDecor() {
    mForceDecorInstall = false;
    if (mDecor == null) {
        mDecor = generateDecor(-1);
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        
    ...
}

protected DecorView generateDecor(int featureId) {
    // System process doesn't have application context and in that case we need to directly use
    // the context we have. Otherwise we want the application context, so we don't cling to the
    // activity.
    Context context;
    if (mUseDecorContext) {
        Context applicationContext = getContext().getApplicationContext();
        if (applicationContext == null) {
            context = getContext();
        } else {
            context = new DecorContext(applicationContext, getContext());
            if (mTheme != -1) {
                context.setTheme(mTheme);
            }
        }
    } else {
        context = getContext();
    }
    return new DecorView(context, featureId, this, getAttributes());
}


```

大致步骤如下：

1. 如果没有DecorView，那么就创建它
2. 将View添加到DecorView的mContentParent中
3. 回调Activity的onContentChanged方法通知Activity视图已经发生改变
4. ActivityThread中的handleResumeActivity只通过WindowManager添加视图到DecorView中

## 参考

- 《Android开发艺术探索》
- [https://juejin.cn/post/6844904073586556935](https://juejin.cn/post/6844904073586556935)
- [https://blog.csdn.net/hzw19920329/article/details/52423771](https://blog.csdn.net/hzw19920329/article/details/52423771)
