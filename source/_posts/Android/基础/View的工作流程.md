---
title: View的工作流程
tags: Android
toc: true
---

DecorView是视图的顶级View，我们添加的布局文件是它的一个子布局，而ViewRootImpl则负责渲染视图，它调用了一个performTraveals方法使得ViewTree开始
三大工作流程，然后使得View展现在我们面前。

![](./绘制流程.png)

## 绘制的流程概要

![](./draw_1.jpeg)

**注意**：这里的三个步骤是每次从根视图到最上层视图依次执行完，再进行下一步骤。

三个步骤：

- 测量（measure）：测量视图大小。从顶层父View到子View递归调用measure方法，measure方法又回调OnMeasure。
- 布局（layout）：确定View位置，进行页面布局。从顶层父View向子View的递归调用view.layout方法的过程，即父View根据上一步measure子View所得到的布局大小
  和布局参数，将子View放在合适的位置上。
- 绘制（draw）：绘制视图。ViewRoot创建一个Canvas对象，然后调用OnDraw()。六个步骤：
  
    1. 绘制视图的背景；
    2. 保存画布的图层（Layer）；
    3. 绘制View的内容；
    4. 绘制View子视图，如果没有就不用；
    5. 还原图层（Layer）；
    6. 绘制滚动条

绘制从ViewRootImpl的performTraversals()方法开始，从上到下遍历整个视图树，每个View控件负责绘制自己，而ViewGroup还需要负责通知自己的子View进行绘制操作。

```java
private void performTraversals() {
    ...
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    ...
    //执行测量流程
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    ...
    //执行布局流程
    performLayout(lp, desiredWindowWidth, desiredWindowHeight);
    ...
    //执行绘制流程
    performDraw();
}


```

## measure

### MeasureSpec

MeasureSpec表示的是一个32位的整形值，它的高2位表示测量模式SpecMode，低30位表示某种测量模式下的规格大小SpecSize。

![](./draw_2.png)

mode的模式分为：

- EXACTLY：对应LayoutParams中的match_parent和具体数值这两种模式。检测到View所需要的精确大小，这时候View的最终大小就是SpecSize所指定的值，
- AT_MOST ：对应LayoutParams中的wrap_content。View的大小不能大于父容器的大小。
- UNSPECIFIED ：不对View进行任何限制，要多大给多大，一般用于系统内部，如ListView，ScrollView

```java
public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        /**
          * UNSPECIFIED 模式：
          * 父View不对子View有任何限制，子View需要多大就多大
          */ 
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;

        /**
          * EXACTYLY 模式：
          * 父View已经测量出子Viwe所需要的精确大小，这时候View的最终大小
          * 就是SpecSize所指定的值。对应于match_parent和精确数值这两种模式
          */ 
        public static final int EXACTLY     = 1 << MODE_SHIFT;

        /**
          * AT_MOST 模式：
          * 子View的最终大小是父View指定的SpecSize值，并且子View的大小不能大于这个值，
          * 即对应wrap_content这种模式
          */ 
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        //将size和mode打包成一个32位的int型数值
        //高2位表示SpecMode，测量模式，低30位表示SpecSize，某种测量模式下的规格大小
        public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        //将32位的MeasureSpec解包，返回SpecMode,测量模式
        public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }

        //将32位的MeasureSpec解包，返回SpecSize，某种测量模式下的规格大小
        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }
        //...
    }

```

### ViewGroup的measure

由于DecorView继承自FrameLayout，是PhoneWindow的一个内部类，而FrameLayout没有measure方法，因此调用的是父类View的measure方法。

### View的measure

## layout

### View的layout流程

```java
// ViewRootImpl.java
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth, int desiredWindowHeight) {
    ...
    host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
    ...
}

// View.java
public void layout(int l, int t, int r, int b) {
    ...
    // 通过setFrame方法来设定View的四个顶点的位置，即View在父容器中的位置
    boolean changed = isLayoutModeOptical(mParent) ? 
    set OpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    ...
    onLayout(changed, l, t, r, b);
    ...
}

// 空方法，子类如果是ViewGroup类型，则重写这个方法，实现ViewGroup
// 中所有View控件布局流程
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {

}
```

### Layout的onLayout

```java
protected void onlayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l,)
    }
}

// layoutVertical核心源码
void layoutVertical(int left, int top, int right, int bottom) {
    ...
    final int count = getVirtualChildCount();
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        if (child == null) {
            childTop += measureNullChild(i);
        } else if (child.getVisibility() != GONE) {
            final int childWidth = child.getMeasureWidth();
            final int childHeight = child.getMeasuredHeight();

            final LinearLayout.LayoutParams lp = 
                    (LinearLayout.LayoutParams) child.getLayoutParams();
            ...
            if (hasDividerBeforeChildAt(i)) {
                childTop += mDividerHeight;
            }

            childTop += lp.topMargin;
            // 为子元素确定对应的位置
            setChildFrame(child, childLeft, childTop + getLocationOffset(child), childWidth, childHeight);
            // childTop会逐渐增大，意味着后面的子元素会被
            // 放置在靠下的位置
            childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

            i += getChildrenSkipCount(child,i)
        }
    }
}

private void setChildFrame(View child, int left, int top, int width, int height) {
    child.layout(left, top, left + width, top + height);
}

```

## draw

```java
private void performDraw() {
    ...
    draw(fullRefrawNeeded);
    ...
}

private void draw(boolean fullRedrawNeeded) {
    ...
    if (!drawSoftware(surface, mAttachInfo, xOffest, yOffset, 
    scalingRequired, dirty)) {
        return;
    }
    ...
}

private boolean drawSoftware(Surface surface, AttachInfo attachInfo, 
int xoff, int yoff, boolean scallingRequired, Rect dirty) {
    ...
    mView.draw(canvas);
    ...
}

// 绘制基本上可以分为六个步骤
public void draw(Canvas canvas) {
    ...
    // 步骤一：绘制View的背景
    drawBackground(canvas);

    ...
    // 步骤二：如果需要的话，保持canvas的图层，为fading做准备
    saveCount = canvas.getSaveCount();
    ...
    canvas.saveLayer(left, top, right, top + length, null, flags);

    ...
    // 步骤三：绘制View的内容
    onDraw(canvas);

    ...
    // 步骤四：绘制View的子View
    dispatchDraw(canvas);

    ...
    // 步骤五：如果需要的话，绘制View的fading边缘并恢复图层
    canvas.drawRect(left, top, right, top + length, p);
    ...
    canvas.restoreToCount(saveCount);

    ...
    // 步骤六：绘制View的装饰(例如滚动条等等)
    onDrawForeground(canvas)
}
```

## 常见问题

### 如何在onCreate中获取View的高宽

```java
//方法1：
view.post(new Runnable() {            
            @Override
            public void run() {
                int width = view.getWidth();
                int measuredWidth = view.getMeasuredWidth();
                Log.i(TAG, "width: " + width);
                Log.i(TAG, "measuredWidth: " + measuredWidth);
            }
        });

//方法2：
ViewTreeObserver vto = view.getViewTreeObserver();       
       vto.addOnGlobalLayoutListener(new OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                view.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                Log.i(TAG, "width: " + view.getWidth());
                Log.i(TAG, "height: " + view.getHeight());
            }
        });

```

## 参考

- [https://jsonchao.github.io/2018/10/28/Android%20View%E7%9A%84%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B/](https://jsonchao.github.io/2018/10/28/Android%20View%E7%9A%84%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B/)
