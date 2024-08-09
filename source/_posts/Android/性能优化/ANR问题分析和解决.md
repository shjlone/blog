---
title: ANR问题分析和解决
date: 2021-06-01
toc: true
tags:
---


## ANR类型

### 按键或触摸事件在特定时间内无响应

```java
//ActivityTaskManagerService.java

// How long we wait until we timeout on key dispatching.
public static final int KEY_DISPATCHING_TIMEOUT_MS = 5 * 1000;
```

### BroadcastRecevier超时

```java
//ActivityManagerService.java

// How long we allow a receiver to run before giving up on it.
static final int BROADCAST_FG_TIMEOUT = 10*1000;
static final int BROADCAST_BG_TIMEOUT = 60*1000;
```

前台广播超时时间是 10s，后台广播超时是 60s，这类超时没有提示框弹出。

### Service超时

```java
ActiveServices.java

// How long we wait for a service to finish executing.
static final int SERVICE_TIMEOUT = 20 * 1000 * Build.HW_TIMEOUT_MULTIPLIER;

// How long we wait for a service to finish executing.
static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;

```

当发生ANR时，会将相应信息记录到/data/anr/traces.txt

## 降低ANR的一些技巧

- 将所有耗时操作，比如访问网络，Socket通信，查询大量SQL语句、IO操作、复杂逻辑计算等都放在子线程中去
- onCreate 和 onResume 回调中尽量避免耗时的代码
- View 的 onOnTouchevent 和 onclick 中避免耗时的代码

AMS系统时间调节原理

程序等待原理分析

## 参考
