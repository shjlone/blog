---
title: Dalvik、ART虚拟机
tags: Android
toc: true
---

## 概要

Dalvik 是 Google 公司自己设计用于 Android 平台的虚拟机。它可以支持已转换为**.dex 格式**的 Java 应用程序的运行，.dex 格式是专为Dalvik 设计的一种压缩格式，适合内存和处理器速度有限的系统。

Dalvik 经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为一个独立的 Linux 进程执行。独立的进程可以防止在虚拟机 崩溃的时候所有程序都被关闭。

2014 年 6 月 25 日，Android L 正式亮相于召开的谷歌 I/O 大会，Android L 改 动幅度较大，谷歌将直接删除 Dalvik，代替它的是传闻已久的 ART。

## Dalvik与JVM的区别

- Dalvik 是基于寄存器的，而 JVM 是基于栈的。
- Dalvik 运行 dex 文件，而 JVM 运行 java 字节码
- 自 Android 2.2 开始，Dalvik 支持 JIT(just-in-time，即时编译技术)。

## ART（Android Runtime）

ART 的机制与 Dalvik 不同。在 Dalvik 下，应用每次运行的时候，字节码都需要通过即时编译器(just in time ，JIT)转换为机器码，这会拖慢应用的运行效率，
而在 ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码， 使其成为真正的本地应用。这个过程叫做预编译(AOT,Ahead-Of-Time)。这样 的话，应用的启动(首次)和执行都会变得更加快速。

### 优点

1. 系统性能的显著提升。
2. 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
3. 更长的电池续航能力。
4. 支持更低的硬件。

### 缺点

1. 机器码占用的存储空间更大，字节码变为机器码之后，可能会增加 10%-20%
2. 应用的安装时间会变长。
