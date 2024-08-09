---
title: View的事件分发机制
tags: Android
toc: true
---


## 分发过程

事件分发过程由三个方法共同完成：

### dispatchTouchEvent

方法返回值为true表示事件被当前视图消费掉；返回为super.dispatchTouchEvent表示继续分发该事件，返回为false表示交给父类的onTouchEvent处理。

### onInterceptTouchEvent

方法返回值为true表示拦截这个事件并交由自身的onTouchEvent方法进行消费；返回false表示不拦截，需要继续传递给子视图。如果return super.onInterceptTouchEvent(ev)， 事件拦截分两种情况: 　

1. 如果该View存在子View且点击到了该子View, 则不拦截, 继续分发给子View处理, 此时相当于return false。
2. 如果该View没有子View或者有子View但是没有点击中子View(此时ViewGroup 相当于普通View), 则交由该View的onTouchEvent响应，此时相当于return true。

`注意`：一般的LinearLayout、 RelativeLayout、FrameLayout等ViewGroup默认不拦截， 而ScrollView、ListView等ViewGroup则可能拦截，得看具体情况。

### onTouchEvent

方法返回值为true表示当前视图可以处理对应的事件；返回值为false表示当前视图不处理这个事件，它会被传递给父视图的onTouchEvent方法进行处理。
如果return super.onTouchEvent(ev)，事件处理分为两种情况：

1. 如果该View是clickable或者longclickable的,则会返回true, 表示消费了该事件, 与返回true一样;
2. 如果该View不是clickable或者longclickable的,则会返回false, 表示不消费该事件,将会向上传递,与返回false一样。

注意：在Android系统中，拥有事件传递处理能力的类有以下三种：

- Activity：拥有分发和消费两个方法。
- ViewGroup：拥有分发、拦截和消费三个方法。
- View：拥有分发、消费两个方法。

三个方法的关系用伪代码表示如下：

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    boolean consume = false;
    if (onInterceptTouchEvent(ev)) {
        consume = onTouchEvent(ev);
    } else {
        coonsume = child.dispatchTouchEvent(ev);
    }
    
    return consume;
}
```

通过上面的伪代码，我们可以大致了解点击事件的传递规则：对应一个根ViewGroup来说，点击事件产生后，首先会传递给它，这是它的dispatchTouchEvent就会被调用，
如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示它要拦截当前事件，接着事件就会交给这个ViewGroup处理，这时如果它的mOnTouchListener被设置，
则onTouch会被调用，否则onTouchEvent会被调用。在onTouchEvent中，如果设置了mOnCLickListener，则onClick会被调用。
只要View的CLICKABLE和LONG_CLICKABLE有一个为true，onTouchEvent()就会返回true消耗这个事件。如果这个ViewGroup的onInterceptTouchEvent方法
返回false就表示它不拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件被最终处理。

一些重要的结论：

1. 事件传递优先级：onTouchListener.onTouch > onTouchEvent > onClickListener.onClick。

2. 正常情况下，一个时间序列只能被一个View拦截且消耗。因为一旦一个元素拦截了此事件，那么同一个事件序列内的所有事件都会直接交给它处理（即不会再调用这个View的拦截方法去询问它是否要拦截了，而是把剩余的ACTION_MOVE、ACTION_DOWN等事件直接交给它来处理）。特例：通过将重写View的onTouchEvent返回false可强行将事件转交给其他View处理。

3. 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。

4. ViewGroup默认不拦截任何事件（返回false）。

5. View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认都为false，clickable属性要分情况，比如Button的clickable属性默认为true，而TextView的clickable默认为false。

6. View的enable属性不影响onTouchEvent的默认返回值。

7. 通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。

ACTION_CANCEL什么时候触发，触摸button然后滑动到外部抬起会触发点击事件吗，再滑动回去抬起会么？

一般ACTION_CANCEL和ACTION_UP都作为View一段事件处理的结束。如果在父View中拦截ACTION_UP或ACTION_MOVE，在第一次父视图拦截消息的瞬间，父视图指定子视图不接受后续消息了，同时子视图会收到ACTION_CANCEL事件。
如果触摸某个控件，但是又不是在这个控件的区域上抬起（移动到别的地方了），就会出现action_cancel。

**点击事件被拦截，但是想传到下面的View，如何操作？**

重写子类的requestDisallowInterceptTouchEvent()方法返回true就不会执行父类的onInterceptTouchEvent()，即可将点击事件传到下面的View。

如何解决View的事件冲突？举个开发中遇到的例子？

常见开发中事件冲突的有ScrollView与RecyclerView的滑动冲突、RecyclerView内嵌同时滑动同一方向。

滑动冲突的处理规则：

- 对于由于外部滑动和内部滑动方向不一致导致的滑动冲突，可以根据滑动的方向判断谁来拦截事件。
- 对于由于外部滑动方向和内部滑动方向一致导致的滑动冲突，可以根据业务需求，规定何时让外部View拦截事件，何时由内部View拦截事件。
- 对于上面两种情况的嵌套，相对复杂，可同样根据需求在业务上找到突破点。

滑动冲突的实现方法：

- 外部拦截法：指点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，否则就不拦截。具体方法：需要重写父容器的onInterceptTouchEvent方法，在内部做出相应的拦截。
- 内部拦截法：指父容器不拦截任何事件，而将所有的事件都传递给子容器，如果子容器需要此事件就直接消耗，否则就交由父容器进行处理。具体方法：需要配合requestDisallowInterceptTouchEvent方法。

## 参考
