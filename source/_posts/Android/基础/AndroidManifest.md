---
title: AndroidManifest描述文件
toc: true
---

```xml
<?xml version="1.0" encoding="uft-8" ?>
<manifest xmlns:android="http://schemas.android.com/apk/red/android"
               package="com.example.android"
               android:versionCode="1"
               android:versionName="1.0">
    <uses-permission />
    <permission />
    <permission-tree />                                                      //权限配置
    <permission-group />

    <instrumentation android:functionalTest=["true" | "false"]
        android:handleProfiling=["true" | "false"]
        android:icon="drawable resource"
        android:label="string resource"
        android:name="string"
        android:targetPackage="string" />
     <uses-sdk />
     <uses-configuration /> 
     <uses-feature />                                                         //环境配置
     <supports-screens /> 
     <compatible-screens /> 
     <supports-gl-texture /> 

      //应用基本配置
     <application android:allowTaskReparenting=["true" | "false"]
                 android:allowBackup=["true" | "false"]
                 android:backupAgent="string"
                 android:debuggable=["true" | "false"]
                 android:description="string resource"
                 android:enabled=["true" | "false"]
                 android:hasCode=["true" | "false"]
                 android:hardwareAccelerated=["true" | "false"]
                 android:icon="drawable resource"
                 android:killAfterRestore=["true" | "false"]
                 android:largeHeap=["true" | "false"]
                 android:label="string resource"
                 android:logo="drawable resource"
                 android:manageSpaceActivity="string"
                 android:name="string"
                 android:permission="string"
                 android:persistent=["true" | "false"]
                 android:process="string"
                 android:restoreAnyVersion=["true" | "false"]
                 android:requiredAccountType="string"
                 android:restrictedAccountType="string"
                 android:supportsRtl=["true" | "false"]
                 android:taskAffinity="string"
                 android:testOnly=["true" | "false"]
                 android:theme="resource or theme"
                 android:uiOptions=["none" | "splitActionBarWhenNarrow"]
                 android:vmSafeMode=["true" | "false"] >
        //界面组件配置
        <activity android:allowTaskReparenting=["true" | "false"]
              android:alwaysRetainTaskState=["true" | "false"]
              android:clearTaskOnLaunch=["true" | "false"]
              android:configChanges=["mcc", "mnc", "locale",
                                     "touchscreen", "keyboard", "keyboardHidden",
                                     "navigation", "screenLayout", "fontScale", "uiMode",
                                     "orientation", "screenSize", "smallestScreenSize"]
              android:enabled=["true" | "false"]
              android:excludeFromRecents=["true" | "false"]
              android:exported=["true" | "false"]
              android:finishOnTaskLaunch=["true" | "false"]
              android:hardwareAccelerated=["true" | "false"]
              android:icon="drawable resource"
              android:label="string resource"
              android:launchMode=["multiple" | "singleTop" |
                                  "singleTask" | "singleInstance"]
              android:multiprocess=["true" | "false"]
              android:name="string"
              android:noHistory=["true" | "false"] 
              android:parentActivityName="string"
              android:permission="string"
              android:process="string"
              android:screenOrientation=["unspecified" | "behind" |
                                         "landscape" | "portrait" |
                                         "reverseLandscape" | "reversePortrait" |
                                         "sensorLandscape" | "sensorPortrait" |
                                         "userLandscape" | "userPortrait" |
                                         "sensor" | "fullSensor" | "nosensor" |
                                         "user" | "fullUser" | "locked"]
              android:stateNotNeeded=["true" | "false"]
              android:taskAffinity="string"
              android:theme="resource or theme"
              android:uiOptions=["none" | "splitActionBarWhenNarrow"]
              android:windowSoftInputMode=["stateUnspecified",
                                           "stateUnchanged", "stateHidden",
                                           "stateAlwaysHidden", "stateVisible",
                                           "stateAlwaysVisible", "adjustUnspecified",
                                           "adjustResize", "adjustPan"] >  
        . . .
        </activity>

        <activity>
                <intent-filter>
                       <action />
                       <category />
                       <data />
                 </intent-filter>
                 <meta-data />                                 
          </activity>

          <activity-alias>
                 <intent-filter> . . . </intent-filter>
                 <meta-data />
          </activity-alias>
          <service>
                 <intent-filter> . . . </intent-filter>
                 <meta-data/>                                                  //服务组件配置
          </service>
          <receiver>
                 <intent-filter> . . . </intent-filter>                              //触发器组件配置
                 <meta-data />
          </receiver>
          //数据源组件配置



    <provider android:authorities="list"
    android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:grantUriPermissions=["true" | "false"]
    android:icon="drawable resource"
    android:initOrder="integer"
    android:label="string resource"
    android:multiprocess=["true" | "false"]
    android:name="string"
    android:permission="string"
    android:process="string"
    android:readPermission="string"
    android:syncable=["true" | "false"]
    android:writePermission="string" >
    . . .
    </provider>
-------------------------------------------------------------------------------------------------------------------------------------------
    <uses-library />                //依赖库配置
-------------------------------------------------------------------------------------------------------------------------------------------
    </application>
</manifest>
```

## manifest标签

属性 | 描述
----|----
android:installLocation|设置应用的安装环境，如preferExternal表示安装到外部存储设备中；但运行时产生的数据还是会在/data/data目录下

### uses-permission标签

请求应用使用到的权限

属性 | 描述
----|----
android:name|使用的权限名字

### permission标签

限定权限来限制第三方应用的访问

属性 | 描述
----|----
android:name|
android:permissionGroup|权限的分组，如android.permission-group.COST_MONEY提示用户该应用可能会消耗通信费用，如果没有预定义该权限的分组，也可以通过配置项<permission-group>定义
android:label|标签
android:description|描述


### instrumentation标签

这个元素声明了一个Instrumentation类，这个类能够监视应用程序跟系统的交互。Instrumentation对象会在应用的其他所有组件被实例化之前实例化。


属性 | 描述
----|----
android:functionalTest|这个属性用于指定Instrumentation类是否应该作为一个功能性的测试来运行，如果设置为true，这要运行，否则不应该运行。默认值是false。
android:handleProfiling|这个属性用于指定Instrumentation对象是否会开启和关闭分析功能。如果设置为true，那么由Instrumentation对象来决定分析功能的启动和终止时机，如果设置为false，则分析功能会持续到Instrumentation对象整个运行周期。如果设置为true，会使Instrumentation对象针对一组特定的操作来进行分析。默认值是false。
android:targetPackage|制定Intrumentation对象所监视的应用程序。

### uses-feature标签

声明应用所依赖的外设或Android的特色功能。比如，一款主打拍照功能的应用需要声明所安装的设备需要有相机
<uses-feature android:name="android.hardware.camera" />


### supports-screens标签

属性 | 描述
----|----
android:resizeable|指明应用程序是否根据不同的屏幕尺寸进行缩放。如果设为否，则在较大屏幕上系统将以屏幕兼容模式运行应用程序。本属性已过时。为了帮助程序从Android 1.5升级为1.6才引入本属性，当时第一次引入了对多种屏幕的支持。不应再使用本属性。
android:smallScreens|指明应用程序是否支持较小屏幕。较小的屏幕是指小于“normal”（传统的HVGA）大小的屏幕。不支持小屏幕的应用程序将在外部服务（比如Android Market）中禁止用于小屏幕设备，因为只有很少一部分平台能让程序运行在小屏幕上。缺省值是“true”。
android:normalScreens|指明应用程序是否支持“normal”屏幕尺寸。传统意义上指的是中等密度的HVGA 屏幕，但低密度的WQVGA和高密度的WVGA一般也被视为是正常尺寸。缺省属性是“true”。
android:largeScreens|指明应用程序是否支持大屏幕尺寸。大屏幕是指明显比“normal”手持设备屏幕更大的尺寸。虽然依赖于系统的缩放也能全屏显示，但为了更好的用户体验可能需要对程序组件进行特定的处理。本属性的缺省值依版本而各不相同，因此最好是一直都明确声明这个属性。注意设为“false”将总是启用屏幕兼容模式。
android:xlargeScreens|指明应用程序是否支持超大屏幕尺寸。超大屏幕是指明显比“large”屏幕更大的尺寸，比如平板设备（或更大），虽然依赖于系统的缩放也能全屏显示，但为了更好的用户体验可能需要对程序组件进行特定的处理。本属性的缺省值依版本而各不相同，因此最好是一直都明确声明这个属性。注意设为“false”将总是启用屏幕兼容模式。本属性自API level 9引入。
android:anyDensity|指明应用程序是否包含适用于任何屏幕密度的资源。对于支持Android 1.6 (API level 4)以上版本的应用程序而言，本属性缺省值是“true”。除非绝对确认程序必须要能运行，不应设为“false”。只有应用程序要直接操作位图时（详情参阅支持多种屏幕文档），才可能需要禁用此选项。
android:requiresSmallestWidthDp|指定程序所需的smallestWidth最小值。smallestWidth是指可被程序用户界面使用的屏幕可用空间的最小值（单位为dp）——指可用屏幕两边中最短的那条边长。为了保证与应用程序兼容，设备的smallestWidth必须大于等于本属性值。（通常此值对应于布局layout所支持的“最小宽度”，而与屏幕当前的方向无关。）例如，典型的手持设备的最小宽度是320dp，7英寸的平板设备的最小宽度是600dp，10英寸的平板设备的最小宽度是720dp。因为这些值即为屏幕可用空间的最小值，所以一般也即是smallestWidth的值。在计算屏幕上的组件排列和系统用户界面大小时会与本属性值进行比较。例如，如果设备屏幕上需要显示一些永久性的用户界面元素，这些元素占用的屏幕位置对于其它用户界面元素是不可用的，通过对这些元素尺寸进行计算，系统声明的设备smallestWidth会比实际屏幕尺寸要小些。因此，应该用layout所需的最小宽度来设置此值，而与屏幕的方向无关。如果应用程序能在小屏幕上正确缩放（最低是small尺寸或最小宽度320dp），那就不需要用到本属性。否则，就应该为最小屏幕宽度标识符设置本属性来匹配应用程序所需的最小尺寸。警告：Android系统并不关心本属性，因此它不会影响程序运行时的表现。它是用于为诸如Android Market之类的服务启用过滤功能。不过，Android Market 当前还不支持对这个属性的过滤（Android 3.2），因此如程序不支持小屏幕的话还应继续使用其它屏幕尺寸的属性来进行限制。本属性自API level 13引入。
android:compatibleWidthLimitDp|通过指定程序支持的“最小屏幕宽度”的最大值，本属性可启用屏幕兼容模式作为用户可选项。如果设备可用屏幕的最小边长大于在此设置的值，用户将仍可以安装程序，但会运行在屏幕兼容模式。缺省情况下，屏幕兼容模式将被关闭，layout将如常缩放至全屏显示，但系统状态栏中会出现一个按钮，用户可以用此按钮来开关屏幕兼容模式。如果应用程序能兼容所有的屏幕尺寸，layout也能正确缩放，那就不需要用到本属性。注意：目前屏幕兼容模式只能在手持设备上仿真320dp宽度的屏幕，因此ndroid:compatibleWidthLimitDp大于320时屏幕兼容模式将不会生效。本属性自API level 13引入。
android:largestWidthLimitDp|通过指定程序支持的“最小屏幕宽度”的最大值，本属性可强制开启屏幕兼容模式。如果设备屏幕的最小边长大于本属性值，应用程序将运行在屏幕兼容模式，且用户无法将其关闭。如果应用程序能兼容所有的屏幕尺寸，layout也能正确缩放，那就不需要用到本属性。不然也应优先考虑使用android:compatibleWidthLimitDp属性。仅当应用程序缩放到大屏幕时会崩溃，屏幕兼容模式是用户使用的唯一方式，才会用到android:largestWidthLimitDp属性。注意：目前屏幕兼容模式只能在手持设备上仿真320dp宽度的屏幕，因此android:largestWidthLimitDp大于320时屏幕兼容模式将不会生效。本属性自API level 13引入。

### uses-sdk标签

属性 | 描述
----|----
targetSdkVersion|
minSdkVersion|

### application

属性 | 描述
----|----
android:uiOptions|"none" 默认值"splitActionBarWhenNarrow”，分裂ActionBar用来分开action item，在屏幕底部出现一个actionbar来显示顶部显示不完的items
android:backupAgent|云端存储组件声明，该组件一个应用只有一个。其值为BackupAgent的子类
android:allowbackup|是否允许备份
android:allowClearUserData| 用户是否能选择自行清除数据，默认为true，程序管理器包含一个选择允许用户清除数据。当为true时，用户可自己清理用户数据，反之亦然 
android:testOnly| 用于判断该应用是否用于测试
android:largeHeap|可以给程序分配大内存。 getMemoryClass() 或getLargeMemoryClass()查询可用内存

#### activity

属性 | 描述
----|----
android:launchMode|运行模式
android:configChanges|当配置变化时，activity将会重启，但声明了这个属性会阻止activity重新启动，而调用onConfigurationChanged()
android:screenOrientation|activity在设别中的显示方向
android:parentActivityName|指定父级的activity。系统根据这个属性来决定当使用actionbar的向上按钮时那个activity响应
android:taskAffinity|表示当前activity进入的task
adnroid:finishOnTaskLanuch|离开这个Task栈, 则系统会马上清除这个Activity,不管这个Activity在堆栈的任何位置.
android:allowTaskReparenting|允许当前Activity可以在任务栈中移动
android:alwaysRetaainTaskState|
android:excludeFromRecents|
android:stateNotNeeded|
android:exported|是否可以导出，是否可以被外部访问
android:clearTaskOnLaunch|当值为true, 只要你一离开这个Task栈, 则系统会马上清理除了Root Activity的全部Activity.
android:windowSoftInputMode|
android:permission|
android:alwaysRetainTaskState|

##### intent-filter

##### meta-data

#### activity-alias

targetActivity属性所指activity的别名。别名会作为一个独立的实体来代表目标Activity。它能够有自己的Intent过滤器。该标签的其他属性是<activity>属性的一个子集，对于子集中的属性，不会把目标Activity中所设置的任何值转交给别名Activity，但对于子集中所没有的属性，则目标Activity所设置的值有会应用到别名Activity。

属性 | 描述
----|----
android:enabled|
android:exported|

#### service

#### receiver

#### provider

声明内容存储组件

属性 | 描述
----|----
android:authorities|
android:enabled|是否可以被实例化。只有当application和provider中的enabled都为true时，provider才可以
android:name|

#### uses-library

引用类库

属性 | 描述
----|----
android:name|指定类库的名称
android:required|是否一定要找个库，true的时候，如果机器上没有这个库则不能安装



## 参考

- [https://developer.android.com/guide/topics/manifest](https://developer.android.com/guide/topics/manifest)