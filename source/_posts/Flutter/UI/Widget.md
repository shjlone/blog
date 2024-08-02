---
title: Widget
toc: true
tags: Flutter
---


## Widget

Flutter中的一切都是Widget。关于Flutter的UI绘制原理可以参考[纷争再起：Flutter-UI绘制解析](https://juejin.cn/post/6844903794627575822)。

简要概括就是，我们写各种widget，Flutter框架帮我们解析成element树，最终转换成renderobject树，再通过底层skia绘制。

![](./widget_1.png)

- Component Widget：组合类Widget，这类Widget都继承StatelessWidget或StatefulWidget；
- Render Widget：渲染类Widget，参与layout、paint流程，有与之对应的Render Object；
- Proxy Widget：代理类Widget，提供一些附加的功能，比如InheritedWidget用于共享信息，ParentDataWidget用于为其他Widget提供信息；


### context

如果 widget `A` 拥有子 widget，那么 widget `A` 的 context 将成为其直接关联子 context 的父 context。

```
context.ancestorWidgetOfExactType(Scaffold) => 通过从 context 得到树结构来返回第一个 Scaffold
```

Widget主要有两种类型：

- StatelessWidget：只有在创建的时候绘制一次
- StatefulWidget：根据状态会发生变化

感觉这样设计还是处于性能考虑，当某些控件不需要改变UI时，使用StatelessWidget就不会重绘。

## Widget

```dart
abstract class Widget extends DiagnosticableTree {

   final Key? key;//标识 https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d

   Element createElement();//每个Widget对应一个Element

   /// 是否需要更新，根据runtimeType和key来判断,新旧Widget相同就只需要更新数据
   static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }

}
```


`为什么widget都是immutable?`

@immutable 代表 Widget 是不可变的，这会限制 Widget 中定义的属性（即配置信息）必须是不可变的（final），为什么不允许 Widget 中定义的属性变化呢？这是因为，Flutter 中如果属性发生变化则会重新构建Widget树，即重新创建新的 Widget 实例来替换旧的 Widget 实例，所以允许 Widget 的属性变化是没有意义的，因为一旦 Widget 自己的属性变了自己就会被替换。这也是为什么 Widget 中定义的属性必须是 final 的原因。


## StatelessWidget

对于StatelessWidget，只需要重写build方法即可。

## StatefulWidget

StatefulWidget是通过State来管理状态，State的生命周期也就是State的生命周期。参考：[State](/wiki/Flutter/UI/标准库/widgets/framework/State/)

### Key

在 Fultter 中，每一个 Widget 都是被唯一标识的。这个唯一标识在 build/rendering 阶段由框架定义。该唯一标识对应于可选的 Key 参数。如果省略该参数，Flutter 将会为你生成一个。

参考：[key](/wiki/Flutter/UI/标准库/widgets/framework/Key/)

### 访问子Widget

通过key

```dart
...
GlobalKey<MyStatefulWidgetState> myWidgetStateKey = new GlobalKey<MyStatefulWidgetState>();
...
@override
Widget build(BuildContext context){
    return new MyStatefulWidget(
        key: myWidgetStateKey,
        color: Colors.blue,
    );
}

///访问子State
myWidgetStateKey.currentState
```

### 访问父Widget

```dart
class MyExposingWidgetState extends State<MyExposingWidget>{
   Color _color;

   Color get color => _color;
   ...
}

class MyChildWidget extends StatelessWidget {
   @override
   Widget build(BuildContext context){
   // 通过context的方法
      final MyExposingWidget widget = context.ancestorWidgetOfExactType(MyExposingWidget);
      final MyExposingWidgetState state = widget?.myState;

      return new Container(
         color: state == null ? Colors.blue : state.color,
      );
   }
}

```

### StatelessWidget、StatefulWidget选择策略：

* 优先使用 StatelessWidget
* 含有大量子 Widget（如根布局、次根布局）慎用 StatefulWidget
* 尽量在叶子节点使用 StatefulWidget
* 将会调用到setState((){}) 的代码尽可能的和要更新的视图封装在一个尽可能小的模块里。
* 如果一个Widget需要reBuild，那么它的子节点、兄弟节点、兄弟节点的子节点应该尽可能少

## InheritedWidget

允许树中较低层次的widget向上查找，获得祖先widget的引用，以及在祖先改变的时候重建自身。Theme.of()和Navigator.of()都是InheritedWidget的例子。

InheritedWidget 组件的所有子组件都可以直接通过 BuildContext.dependOnInheritedWidgetOfExactType 获取数据。

updateShouldNotify方法来决定是否通知子树中依赖data的Widget。 如果返回true，则子树中依赖(build函数中有调用)本widget的子widget的`state.didChangeDependencies`会被调用



## 参考

- [纷争再起：Flutter-UI绘制解析](https://juejin.cn/post/6844903794627575822)
- [数据共享（InheritedWidget）](https://book.flutterchina.club/chapter7/inherited_widget.html)
- [源码分析系列之InheritedWidget](https://segmentfault.com/a/1190000039030651)
