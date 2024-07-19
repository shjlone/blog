---
title: Flutter动画
toc: true
tags: Flutter
---


https://flutter.cn/docs/development/ui/animations

## 动画的基本组成

### Animation

作用：保存动画的差值和状态。整个动画执行过程可以是线性的、曲线的、一个步进函数活着任何其他曲线，由Curve来决定。

- addListener

给Animation添加帧监听器

- addStatusListener

动画开始、结束、正向或反向时的回调

### Curve

作用：定义动画曲线

内置Curves曲线：

- linear：匀速
- decelerate：匀减速
- ease：开始加速后面减速
- easeIn：开始慢后面快
- easeOut：开始快后面慢
- easeInOut：开始慢，然后加速，最后再减速

### AnimationController

作用：用于控制动画，包含启动forward、停止stop、反向播放reverse等

### Ticker

当创建AnimationController时，需要传递一个vsync参数，它接收一个TickerProvider类型。通常我们会将SingleTickerProviderStateMixin添加到State的定义中，然后将State对象作为vsync的值。

SingleTickerProviderStateMixin和TickerProviderStateMixin，这两个类的区别就是是否支持创建多个TickerProvider

### Tween

默认情况下，AnimationController对象值的范围是[0.0，1.0]。如果我们需要构建UI的动画值在不同的范围或不同的数据类型，则可以使用Tween来添加映射以生成不同的范围或数据类型的值。例如，像下面示例，Tween生成[-200.0，0.0]的值

```
final Tween doubleTween = Tween<double>(begin: -200.0, end: 0.0);
```

Tween构造函数需要begin和end两个参数。Tween的唯一职责就是定义从输入范围到输出范围的映射。输入范围通常为[0.0，1.0]，但这不是必须的，我们可以自定义需要的范围。Tween继承自Animatable<T>，而不是继承自Animation<T>，Animatable中主要定义动画值的映射规则。

```
final Tween colorTween =
    ColorTween(begin: Colors.transparent, end: Colors.black54);

final AnimationController controller = AnimationController(
  duration: const Duration(milliseconds: 500), 
  vsync: this,
);
Animation<int> alpha = IntTween(begin: 0, end: 255).animate(controller);

final Animation curve = CurvedAnimation(parent: controller, curve: Curves.easeOut);
Animation<int> alpha = IntTween(begin: 0, end: 255).animate(curve);

```

Tween对象不存储任何状态，相反，它提供了evaluate(Animation<double> animation)方法，它可以获取动画当前映射值。 Animation对象的当前值可以通过value()方法取到。evaluate函数还执行一些其他处理，例如分别确保在动画值为0.0和1.0时返回开始和结束状态

**Tween的子类：**

- ColorTween
- ConstantTween
- CurveTween
- IntTween
- RectTween
- ReverseTween
- SizeTween
- StepTween


## 基本用法

```dart
class ScaleAnimationRoute extends StatefulWidget {
  const ScaleAnimationRoute({Key? key}) : super(key: key);
  
  @override
  _ScaleAnimationRouteState createState() => _ScaleAnimationRouteState();
}

//需要继承TickerProvider，如果有多个AnimationController，则应该使用TickerProviderStateMixin。
class _ScaleAnimationRouteState extends State<ScaleAnimationRoute>
    with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;
  
  @override
  initState() {
    super.initState();
    controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    //匀速
    //图片宽高从0变到300
    animation = Tween(begin: 0.0, end: 300.0).animate(controller);
    animation.addListener(() {
        setState(() => {});
      });

    //启动动画(正向执行)
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Image.asset(
        "imgs/avatar.png",
        width: animation.value,
        height: animation.value,
      ),
    );
  }
  
  @override
  dispose() {
    //路由销毁时需要释放动画资源
    controller.dispose();
    super.dispose();
  }
}


```

以上写法封装后便是AnimatedBuilder

```dart
AnimatedBuilder({
    child: xx,
    animation: yy,//绑定Animation
});
```




## 隐式动画

Implicit Animations

ImplicitlyAnimatedWidget的子类，可以方便的设置各种各种属性的动画。隐式动画只需要传递duration，即可自行驱动。简单的动画场景可使用。

**系统实现的隐式动画**

- AnimatedContainer：Container属性变化过渡动画
- AnimatedAlign：alignment变化过渡动画
- AnimatedOpacity：透明度
- AnimatedPositioned：配合Stack使用
- AnimatedRotation
- AnimatedScale
- AnimatedSlide
- AnimatedSwitcher
- AnimatedSize
- AnimatedCrossFade
- AnimatedTheme
- AnimatedPositionedDirectional
- AnimatedPhysicalModel
- AnimatedPadding

**常用属性**

- duration：设置动画时长
- curve：设置动画曲线
- onEnd：动画结束回调

### 自定义隐式动画

使用TweenAnimationBuilder，该 Widget 使用的时候我们需要传递 duration 参数动画时间、tween 参数动画要设置的值的范围（补间）、重要的还有 builder 参数，builder函数的参数包含context、补间参数tween的类型、还有child


```dart

TweenAnimationBuilder<double>(
    tween: Tween<double>(begin: 0, end: 2 * pi),
    duration: Duration(seconds: 2),
    builder: (BuildContext context, double angle, Widget child) {
        return Transform.rotate(
        angle: angle,
        child: Container(
            color: Colors.red,
            width: 100,
            height: 100,
        ),
        );
    },
),



```

## 显示动画

Explicit Animations

继承自AnimatedWidget，通过传入listenable来驱动视图变化。自己来控制动画的运行

- AlignTransition
- AnimatedBuilder
- DecoratedBoxTransition
- DefaultTextStyleTransition
- RelativePositionedTransition
- RotationTransition：旋转
- ScaleTransition：缩放动画，可以指定中心点
- FadeTransition：透明度动画
- SizeTransition：宽高变化，不会引起内容的变形
- SlideTransition：位移动画


## 交织动画

多个动画交错在一起，通过一个controller控制。

```dart
// 多个tween绑定controller
AnimationController controller = AnimationController();
Tween<double>(begin: 0, end: 100).animate(CurvedAnimation(parent: controller, curve: Interval(0.0, 0.5, curve: Curves.ease)));
ColorTween(begin: Colors.green, end: Colors.red,).amimate(CurvedAnimation(parent: controller, curve: Interval(0.5, 0.8, curve: Curves.ease,)));

```


## 自定义动画



## 参考

- [Flutter实战-动画](https://book.flutterchina.club/chapter9/intro.html#_9-1-1-%E5%8A%A8%E7%94%BB%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86)
- [深入理解Flutter动画原理](http://gityuan.com/2019/07/13/flutter_animator/)