---
title: go_router
toc: true
tags: Flutter
---


官方出品路由管理库[go_router](https://pub.dev/packages/go_router)

路由管理需要的基本功能：

- 命名路由，方便通过系统消息栏等地方打开具体的路由页面，且能带上参数
- 路由嵌套，一个路由页面嵌套多个页面的场景非常多

可以自己封装Navigator，也可以使用第三方库，getx、go_router等都是不错的选择。go_router是官方出品。


## Navigation

```dart

/// url上添加参数
context.go(Uri(path: '/users/123', queryParameters: {'filter': 'abc'}).toString());


//添加附加参数
context.go('/123', extra: 'abc');


/// 获取参数
final String extraString = GoRouterState.of(context).extra! as String;

```



## 转场动画

```dart
GoRoute(
  path: 'details',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: DetailsScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        // Change the opacity of the screen using a Curve based on the the animation's
        // value
        return FadeTransition(
          opacity:
              CurveTween(curve: Curves.easeInOutCirc).animate(animation),
          child: child,
        );
      },
    );
  },
),

```



## 命名路由

```dart

GoRoute(
   name: 'song',
   path: 'songs/:songId',
   builder: /* ... */,
 ),


 TextButton(
  onPressed: () {
    context.goNamed('song', pathParameters: {'songId': 123});
  },
  child: const Text('Go to song 2'),
),

TextButton(
  onPressed: () {
    final String location = context.namedLocation('song', pathParameters: {'songId': 123}, queryParameters: {'filter': 'abc'});
    context.go(location);
  },
  child: const Text('Go to song 2'),
),

```



## 监听路由跳转


```dart
GoRouter router = GoRouter(
  initialLocation: '/splash',
  routes: <RouteBase>[deskAppRoute],
  observers: [wsNavObserver],
  onException: (BuildContext ctx, GoRouterState state, GoRouter router) {
    router.go('/404', extra: state.uri.toString());
  },
);

MyNavObserver wsNavObserver = MyNavObserver();

/// 可以通过subscribe在具体的页面监听路由变化
class MyNavObserver extends RouteObserver {
}


class MyPageState extend State with RouteAware {
  void didChangeDependencies() {
    wsNavObserver.subscribe(this, ModalRoute.of(context)!); //of方法会进行InheritedWidget绑定
    super.didChangeDependencies();
  }
}

```