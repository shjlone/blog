---
title: adb工具
date: 2021-06-01
toc: true
tags:
---



**查询设备**

```shell
  adb devices
  List of devices attached
  emulator-5554  device
  emulator-5556  device
  emulator-5558  device
```

**将命令发送至特定设备**

```shell
    adb -s serial_number command
    adb -s emulator-5556 install helloWorld.apk
```

**拉取/推送文件**

```shell
    adb pull remote local
    adb push local remote
    adb push foo.txt /sdcard/foo.txt
```

**开启/停止adb服务器**

```shell
    adb start-server
    adb kill-server
```

**将日志输出到屏幕**

```shell
    adb logcat [option] [filter-specs]

```

**获取实例序列号**

```shell
    adb get-serialno
```

**查看应用版本号**

```shell
    adb shell dumpsys package 包名 | findstr version
```

**调用Activity Manager**

```shell
    adb shell am start -a android.intent.action.VIEW
```

| 命令                               | 说明                                                                                                                                                                |
| ---------------------------------- | ----------------------------------------- |
| start [options] intent             | 启动 intent 指定的 Activity。                                                                                                                                       |
| startservice [options] intent      | 启动 intent 指定的 Service                                                                                                                                          |
| force-stop package                 | 强行停止与 package（应用的包名称）关联的所有应用。                                                                                                                  |
| kill [options] package             | 终止与 package（应用的包名称）关联的所有进程。此命令仅终止可安全终止且不会影响用户体验的进程。                                                                      |
| kill-all                           | 终止所有后台进程。                                                                                                                                                  |
| instrument [options] component     | 使用 Instrumentation 实例启动监控。通常，目标 component 是表单 test_package/runner_class。                                                                          |
| profile start process file         | 启动 process 的分析器，将结果写入 file。                                                                                                                            |
| set-debug-app [options] package    | 将应用 package 设为调试。                                                                                                                                           |
| clear-debug-app                    | 使用 set-debug-app 清除以前针对调试用途设置的软件包。                                                                                                               |
| monitor [options]                  | 启动对崩溃或 ANR 的监控。                                                                                                                                           |
| screen-compat {on\| off} package   | 控制 package 的屏幕兼容性模式                                                                                                                                       |
| display-size [reset\|widthxheight] | 替换模拟器/设备显示尺寸。此命令对于在不同尺寸的屏幕上测试您的应用非常有用，它支持使用大屏设备模仿小屏幕分辨率（反之亦然）。示例：am display-size 1280x800           |
| display-density dpi                | 替换模拟器/设备显示密度。此命令对于在不同密度的屏幕上测试您的应用非常有用，它支持使用低密度屏幕在高密度环境环境上进行测试（反之亦然）。示例：am display-density 480 |

**软件包管理器**

```shell
    adb shell pm uninstall com.example.MyApp
```

| 命令                           | 说明                                                             |
| ------------------------------ | ---------------------------------------------------------------- |
| list packages [options] filter | 输出所有软件包，或者，仅输出包名称包含filter中的文本的软件包。 |
| list permission-groups         | 输出所有已知的权限组。                                           |
| list features                  | 输出系统的所有功能。                                             |
| list libraries                 | 输出当前设备支持的所有库。                                       |
| install [options] path         | 将软件包（通过path指定）安装到系统。                           |
| clear package                  | 删除与软件包关联的所有数据。                                     |
| set-install-location location  | 更改默认安装位置。                                               |

**屏幕截图**

```shell
    adb shell screencap /sdcard/screen.png
```

**录制视频**

```shell
    adb shell screenrecord /sdcard/demo.mp4
```

**其他Shell命令**

```shell
    adb shell ls /sdcard
```

Shell命令 | 说明
dumpsys | 将系统数据转储到屏幕
dumpstate | 将状态转储到文件
dmesg | 将内核调试消息输出到屏幕。
start |启动（重启）模拟器/设备实例。
stop |停止执行模拟器/设备实例。

<br/>

```shell
    adb shell dumpsys activity log anr 2   可以打开anr的messagequeue
    adb shell dumpsys activity a           查看activity详细信息，如显示大小，布局等等
    adb shell dumpsys activity | grep mFocusedActivity
    adb shell "dumpsys activity top | grep ACTIVITY"    显示当前的Activity
    adb shell dumpsys activity processes   进程信息 trimmemory
    adb shell "ps | grep 包名"   查看应用的进程id
    adb shell dumpsys activity recents
    adb shell dumpsys activity broadcasts  可以查看前台和后台broadcast详细信息(发送时间，
                                        所有应用处理时间，监听广播的所有列表)
    adb shell dumpsys activity intents
    adb shell dumpsys activity oom
    adb shell dumpsys input | grep Focus   查看焦点窗口

    //查看当前activity名字
    adb shell dumpsys  activity activities | grep mFocusedActivity # 8.0以下
    adb shell dumpsys activity activities | grep mResumedActivity # 8.0

    adb shell ps | findstr com.xxxx.xxx  查看进程信息
    adb shell kill -10 pid  触发垃圾回收

    adb kill-server && adb server && adb shell  重制adb连接
    adb shell dumpsys activity activities   查看当前任务栈
```

## gfxinfo

```shell
adb shell dumpsys gfxinfo < PACKAGE_NAME >


Applications Graphics Acceleration Info:
Uptime: 83871844 Realtime: 83871844

** Graphics info for pid 23289 [com.xxx.xxx] **

Stats since: 74696703302664ns
Total frames rendered: 0    本次dump搜集了0帧的信息
Janky frames: 0 (0.00%)     卡顿率
50th percentile: 4950ms
90th percentile: 4950ms
95th percentile: 4950ms
99th percentile: 4950ms
Number Missed Vsync: 0       垂直同步失败的帧
Number High input latency: 0        处理input时间超时的帧数
Number Slow UI thread: 0            因UI线程上的工作导致超时的帧数
Number Slow bitmap uploads: 0       因bitmap的加载耗时的帧数
Number Slow issue draw commands: 0      因绘制导致耗时的帧数
Number Frame deadline missed: 0



//Android 6.0版本为gfxinfo提供了一个新的命令——framestats，其作用是可以从最近的帧中获取非常详细的帧耗时信息
adb shell dumpsys gfxinfo < PACKAGE_NAME > framestats

```

### 模拟用户行为

```shell
发送键盘事件
adb shell input keyevent “value”

发送鼠标事件(Touch)：
adb shell sendevent [device] [type] [code] [value]

input text 
input keyevent 
input tap 
input swipe 

```

## **参考**

- [官方文档](https://developer.android.com/studio/command-line/adb?hl=zh-cn)
- [http://blog.hsujee.com/2017/09/25/Android/Framework/2017-09-25-Android%E6%A8%A1%E6%8B%9F%E8%BE%93%E5%85%A5%E6%93%8D%E4%BD%9C%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E5%8E%9F%E7%90%86/](http://blog.hsujee.com/2017/09/25/Android/Framework/2017-09-25-Android%E6%A8%A1%E6%8B%9F%E8%BE%93%E5%85%A5%E6%93%8D%E4%BD%9C%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E5%8E%9F%E7%90%86/)
- [https://www.cnblogs.com/zhengna/p/10032078.html](https://www.cnblogs.com/zhengna/p/10032078.html)
