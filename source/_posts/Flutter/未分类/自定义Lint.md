---
title: 自定义Lint
toc: true
tags: Flutter
---


Flutter默认提供了许多的Lint规则，但有时候我们可能有特殊的需求。如果按照官方的方式，你需要了解：

- Introduction：[https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/introduction.md](https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/introduction.md)
- Package Structure：[https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/package_structure.md](https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/package_structure.md)
- Getting Started：[https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/getting_started.md](https://github.com/dart-lang/sdk/blob/main/pkg/analyzer_plugin/doc/tutorial/getting_started.md)


也可以参考[Dart-自定义Lint之路(一)-创建Analyzer Plugin](https://juejin.cn/post/7132671175490011166)系列文章来实现自定义Lint。

但建议使用[custom_lint](https://pub.dev/packages/custom_lint)这个库，有以下好处：

1. 经过封装，使用更加简单
2. 方便debug，调试起来很方便



## 新增检查emit规则

在使用bloc的过程中，如果emit前使用了异步操作，则可能会存在bloc已经销毁了还要emit的情况，一般这种情况需要手动判断一下isClosed。但经常会忘记这个判断，所以我们可以通过自定义Lint来检查这种情况。

```dart

import 'package:analyzer/dart/ast/ast.dart';
import 'package:analyzer/dart/ast/visitor.dart';
import 'package:analyzer/error/error.dart';
import 'package:analyzer/error/listener.dart';
import 'package:custom_lint_builder/custom_lint_builder.dart';

const _code = LintCode(
  name: 'use_emit_synchronously',
  problemMessage: "Please use isClosed to check if the bloc is closed.",
);

/// 异步使用bloc的emit前，需要判断isClosed，否则会报错
class UseEmitSynchronously extends DartLintRule {
  UseEmitSynchronously() : super(code: _code);

  @override
  Future<void> run(
    CustomLintResolver resolver,
    ErrorReporter reporter,
    CustomLintContext context,
  ) async {
    final result = await resolver.getResolvedUnitResult();
    final visitor = _Visitor(reporter);
    result.unit.accept(visitor);
  }

  @override
  List<Fix> getFixes() {
    return [_AddIsClosedCheck()];
  }
}

class _Visitor extends RecursiveAstVisitor<void> {
  _Visitor(this.reporter);

  final ErrorReporter reporter;

  /// 方法调用
  @override
  void visitMethodInvocation(MethodInvocation node) {
    super.visitMethodInvocation(node);
    if (node.methodName.name == 'emit') {
      AstNode? functionNode = node.thisOrAncestorMatching((ancestor) =>
          ancestor is FunctionDeclaration || ancestor is MethodDeclaration);

      if (functionNode != null) {
        BlockFunctionBody? body;
        if (functionNode is FunctionDeclaration) {
          body = functionNode.functionExpression.body as BlockFunctionBody?;
        } else if (functionNode is MethodDeclaration) {
          body = functionNode.body as BlockFunctionBody?;
        }

        if (body != null) {
          var hasAwait = _hasAwaitBeforeAndNoCheckEmit(body, node);
          if (hasAwait) {
            reporter.reportErrorForNode(_code, node, []);
          }
        }
      }
    }
  }

  bool _hasAwaitBeforeAndNoCheckEmit(
      BlockFunctionBody body, MethodInvocation emitNode) {
    var awaitVisitor = _AwaitVisitor(emitNode);
    body.accept(awaitVisitor);
    return awaitVisitor.foundAwait &&
        !awaitVisitor.foundIsClosedCheckBeforeEmit;
  }
}

class _AwaitVisitor extends RecursiveAstVisitor<void> {
  _AwaitVisitor(this.emitNode);

  final MethodInvocation emitNode;
  bool foundAwait = false;
  bool emitEncountered = false;
  bool foundIsClosedCheckBeforeEmit = false;

  @override
  void visitAwaitExpression(AwaitExpression node) {
    if (!emitEncountered) {
      foundAwait = true;
    }
    super.visitAwaitExpression(node);
  }

  @override
  void visitIfStatement(IfStatement node) {
    if (node.expression.toSource().contains('isClosed')) {
      foundIsClosedCheckBeforeEmit = true;
    }
    super.visitIfStatement(node);
  }

  @override
  void visitMethodInvocation(MethodInvocation node) {
    if (node == emitNode) {
      emitEncountered = true;
    }
    super.visitMethodInvocation(node);
  }
}

/// 用于修复的操作
class _AddIsClosedCheck extends DartFix {
  @override
  void run(
    CustomLintResolver resolver,
    ChangeReporter reporter,
    CustomLintContext context,
    AnalysisError analysisError,
    List<AnalysisError> others,
  ) {
    context.registry.addClassDeclaration(
      (node) {
        final changeBuilder = reporter.createChangeBuilder(
            message: 'Add isClosed check', priority: 1);
      },
    );
  }
}


```

## 参考

- [https://pub.dev/packages/custom_lint](https://pub.dev/packages/custom_lint)
- [https://github.com/airshu/practice_flutter/tree/develop/xp_lints](https://github.com/airshu/practice_flutter/tree/develop/xp_lints)