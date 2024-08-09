---
title: APK打包流程
toc: true
tags: Android
---

APK文件的结构

- xxx.apk
  - res：未编译的资源文件
    - anim
    - color
    - drawable
    - layout
    - menu
  - lib
  - assets
  - META-INF
    - CERT.RSA：保存了签名和公钥证书
    - CERT.SF：对每个文件的头3行进行SHA1 hash
    - MANIFEST.MF：版本号以及每一个文件的哈希值（Base64），包括资源文件。
  - com
  - classes.dex
  - resources.arsc
  - AndroidManifest.xml

## 一、资源打包

使用aapt来打包res资源文件，生成R.java、resources.arsc和res文件（二进制 & 非二进制如res/raw和pic保持原样）

res目录有9种目录

- animator。这类资源以XML文件保存在res/animator目录下，用来描述属性动画。
- anim。这类资源以XML文件保存在res/anim目录下，用来描述补间动画。
- color。这类资源以XML文件保存在res/color目录下，用描述对象颜色状态选择子。
- drawable。这类资源以XML或者Bitmap文件保存在res/drawable目录下，用来描述可绘制对象。例如，我们可以在里面放置一些图片（.png, .9.png, .jpg, .gif），来作为程序界面视图的背景图。注意，保存在这个目录中的Bitmap文件在打包的过程中，可能会被优化的。例如，一个不需要多于256色的真彩色PNG文件可能会被转换成一个只有8位调色板的PNG面板，这样就可以无损地压缩图片，以减少图片所占用的内存资源。
- layout。这类资源以XML文件保存在res/layout目录下，用来描述应用程序界面布局。
- menu。这类资源以XML文件保存在res/menu目录下，用来描述应用程序菜单。
- raw。这类资源以任意格式的文件保存在res/raw目录下，它们和assets类资源一样，都是原装不动地打包在apk文件中的，不过它们会被赋予资源ID，这样我们就可以在程序中通过ID来访问它们。例如，假设在res/raw目录下有一个名称为filename的文件，并且它在编译的过程，被赋予的资源ID为R.raw.filename，那么就可以使用以下代码来访问它：

```java
Resources res = getResources();  
InputStream is = res.openRawResource(R.raw.filename);  
```

- values。这类资源以XML文件保存在res/values目录下，用来描述一些简单值，例如，数组、颜色、尺寸、字符串和样式值等，一般来说，这六种不同的值分别保存在名称为arrays.xml、colors.xml、dimens.xml、strings.xml和styles.xml文件中。
- xml。这类资源以XML文件保存在res/xml目录下，一般就是用来描述应用程序的配置信息。

### R.java

```java
public final class R {
    public static final class layout {
        public static int xxx = 0xxxxxx;
    }
    ...
}
```

### resources.arsc

记录了所有的应用程序资源目录的信息，包括每一个资源名称、类型、值、ID以及所配置的维度信息。

**使用资源**

- 程序中通过R.resource_type.resource_name来引用相关资源
- xml文件中的使用格式：@[package:]type/name

**aapt命令**

```shell
/e/Android/sdk/build-tools/28.0.2/aapt.exe package -f -M src/main/AndroidManifest.xml -I "/e/Android/sdk/platforms/android-28/android.jar" -S src/main/res -J gen -m

```

**命令参数：**

-f 如果编译出来的文件已经存在，强制覆盖。
-m 使生成的包的目录放在-J参数指定的目录。
-J 指定生成的R.java的输出目录
-S res文件夹路径
-A assert文件夹的路径
-M AndroidManifest.xml的路径
-I 某个版本平台的android.jar的路径
-F 具体指定apk文件的输出

## 二、aidl阶段

处理.aidl文件，生成对应的Java接口文件

**aidl命令**

## 三、Java编译阶段

编译R.java、Java接口文件、Java源文件，生成.class文件

**javac**

    javac -encoding UTF-8 -bootclasspath -d gen/out src/main/java/com/test/MainActivity.java -classpath 

## 四、dex阶段

通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex

**dx命令**

    dx --dex --output=gen/classes.dex gen/out/java/com/test/

## 五、apkbuilder阶段

将classes.dex、resources.arsc、res文件夹(res/raw资源被原装不动地打包进APK之外，其它的资源都会被编译或者处理)、Other Resources(assets文件夹)、AndroidManifest.xml打包成apk文件。

**注意：**

res/raw和assets的相同点：

1. 两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制。

res/raw和assets的不同点：

1. res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。
2. res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹

**aapt**

    aapt add package/res.apk classes.dex

## 六、签名阶段

对apk进行签名

**apksigner**

```
在Android Studio中点击菜单 Build->Generate signed apk... 打包签名过程中,
可以看到两种签名选项 V1(Jar Signature)  V2(Full APK Signature),
刚开始升级AS看到这个懵了,既然是APK Signature,就放心偷懒选了V2,结果安装失败？？？无奈,只能查资料...

从Android 7.0开始, 谷歌增加新签名方案 V2 Scheme (APK Signature);
但Android 7.0以下版本, 只能用旧签名方案 V1 scheme (JAR signing)

V1签名:
    来自JDK(jarsigner), 对zip压缩包的每个文件进行验证, 签名后还能对压缩包修改(移动/重新压缩文件)
    对V1签名的apk/jar解压,在META-INF存放签名文件(MANIFEST.MF, CERT.SF, CERT.RSA), 
    其中MANIFEST.MF文件保存所有文件的SHA1指纹(除了META-INF文件), 由此可知: V1签名是对压缩包中单个文件签名验证
    
V2签名:
    来自Google(apksigner), 对zip压缩包的整个文件验证, 签名后不能修改压缩包(包括zipalign),
    对V2签名的apk解压,没有发现签名文件,重新压缩后V2签名就失效, 由此可知: V2签名是对整个APK签名验证
    
    V2签名优点很明显:
        签名更安全(不能修改压缩包)
        签名验证时间更短(不需要解压验证),因而安装速度加快

注意: apksigner工具默认同时使用V1和V2签名,以兼容Android 7.0以下版本

```

    apksigner sign --ks key.jks --out package/app-release.apk package/app-unsigned-aligned.apk
    //检查签名
    apksigner verify app.apk

## 七、zipalign阶段

对apk中未压缩的数据进行4字节对齐，对齐后就可以使用mmap函数读取文件，可以像读取内存一样对普通文件进行操作。如果没有4字节对齐，就必须显式的读取，这样比较缓慢并且会耗费额外的内存。

**zipalign**

    zipalign 4 package/res.apk package/app-unsigned-aligned.apk
