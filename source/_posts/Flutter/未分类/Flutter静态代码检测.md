---
title: Flutter静态代码检测
toc: true
tags: Flutter
---


代码检测是规范写法，提高质量的一种重要方案，几乎所有的主流语言都有相关方案。在Flutter开发过程中，我们可以使用IDE自带的Inspect Code功能，也可以直接使用命令行dart analyze


## 官方方案

Flutter官方提供[analyzer](https://pub.dev/packages/analyzer)来检测代码


### 使用步骤


0. pubspec.yaml配置

```yaml

dev_dependencies:
  lint:  # dart lint
  flutter_lints: ^1.0.0 # flutter 代码检测工具


```

1. 新建analysis_options.yaml配置文件，放在项目的根目录。

```yaml

# 使用 include: url 来从指定的 URL 引入选项 —— 在这种情况下，通常是引入来自 lints 包中的文件。由于 YAML 不支持多个重复的 key，你只能引入最多一个文件
# 常用的
#include: package:flutter_lints/flutter.yaml
#include: package:pedantic/analysis_options.1.9.0.yaml
include: package:lints/recommended.yaml


# 使用 analyzer: 入口来自定义静态分析： 启用更严格的类型检查, 排除文件, 忽略特定规则, 改变规则的警告等级, or 开启实验性功能
analyzer:
  exclude: # 不分析的文件和文件夹
    - [build/**] 
    - lib/*.g.dart
  language: # 启用更严格的类型检查
    strict-casts: true # 类型推理引擎不再将 dynamic 进行隐式类型转换
    strict-inference: true # 确保当类型推理引擎无法确定静态类型时，不再选择dynamic 类型
    strict-raw-types: true # 确保当类型推理引擎，由于省略类型参数而无法确定静态类型时，不再选择dynamic类型
  errors: # 设置严重程度
    invalide_assignment: warning
    missing_return: error
    dead_code: info
  strong-mode: # 强制模式
    implicit-casts: false # 强制使用废弃的选项
# 使用 linter: 入口来配置 linter 规则，可以单独启用或停用某条规则
linter:
  rules:
    - cancel_subscriptions


```

2. 执行命令 

```shell
# 分析项目命令
# 分析从根目录开始遍历，发现analysis_options.yaml配置文件，则执行自定义静态代码分析，若没有则默认配置(https://github.com/flutter/flutter/blob/master/analysis_options.yaml)
dart analyze
```


### 自定义规则



```yaml

include: package:flutter_lints/flutter.yaml

# 使用 analyzer: 入口来自定义静态分析： 启用更严格的类型检查, 排除文件, 忽略特定规则, 改变规则的警告等级, or 开启实验性功能
analyzer:
  exclude: # 不分析的文件和文件夹
    - [build/**] 
    - lib/*.g.dart
    - build/** # 排除整个build文件夹
  language:
    strict-casts: true
    strict-raw-types: true
  errors: # 设置严重程度 分为ignore、info、warning、error
    always_declare_return_types: warning  # 方法必须声明返回类型
    null_closures: warning  # 不要给闭包的参数传null
    invalide_assignment: warning
    missing_return: error # 返回值缺失
    unnecessary_statements: warning # 无效的表达式
    prefer_typing_uninitialized_variables: warning  # 未初始化的变量
    dead_code: info
    prefer_interpolation_to_compose_strings: ignore  # 字符串的写法，不要使用+拼接，使用引号
    use_key_in_widget_constructors: error # Widget的构造方法中默认需要带上key
    control_flow_in_finally: error #不要在finally中使用控制流（break、continue、return），可能会造成难以发现的问题
    no_logic_in_create_state: error # 不要在createState方法中使用任何逻辑
    hash_and_equals: error # 重写==的同时要重写hashCode
    unrelated_type_equality_checks: error # 不相关的类型不要进行比较
    always_use_package_imports: error # 使用包导入
    avoid_print: error # 不使用print，可以用debugPrint、或者自己封装的log
    use_build_context_synchronously: error # 不要在异步操作后使用context，使用的话需要添加mounted判断
  strong-mode:
    implicit-casts: false # 禁止隐式转换，可能会产生Null check异常，通常在Map<String, dynamic>取值、范型方法返回值的转换时易出现

# 使用 linter: 入口来配置 linter 规则
linter:
  rules:
    cancel_subscriptions: true # 新增规范
    avoid_shadowing_type_parameters: false # 删除规范


```


## 使用sonarqube


https://betterprogramming.pub/flutter-and-sonarqube-for-static-code-analysis-51368a85c51c


### 安装sonarqube后台服务

### 本地安装sonar-scanner工具

### 本地项目配置


## 参考

- [自定义静态分析](https://dart.cn/guides/language/analysis-options)
- [官方默认规则](https://github.com/flutter/flutter/blob/master/analysis_options.yaml)
- [Linter for Dart](https://dart-lang.github.io/linter/lints/index.html)
- [flutter_lints](https://pub.dev/packages/flutter_lints)