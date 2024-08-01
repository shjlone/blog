---
title: CustomPaint
toc: true
tags: Flutter
---


## 基本概念

我们的手机屏幕可以当成一块画板（Canvas），使用我们自定义的Painter，就能绘制出各种图形。而Flutter中提供了CustomPaint组件来方便使用自定义动画。

### CustomPaint

```dart
const CustomPaint({
  super.key,
  this.painter,//CustomPainter对象，如果设置了child，则painter绘制的内容会被覆盖
  this.foregroundPainter,//如果设置了child，该painter的内容会覆盖child
  this.size = Size.zero,//画板大小
  this.isComplex = false,//如果设置为true，会对canvas的绘制进行一些必要的缓存来优化性能
  this.willChange = false,//配合isComplex使用，控制组件是否在下一帧需要重绘
  super.child,
}) 
```

### CustomPainter

CustomPainter是一个抽象类，用于自定义绘制逻辑。需要实现paint和shouldRepaint方法。

```dart


class XXXPainter extend CustomPainter {


  @override
  void paint(Canvas canvas, Size size) {
  }

  ///返回 true 才会进行重绘，否则就只会绘制一次
  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
  }
}

```

### Canvas

画板，提供了许多绘制方法。你只需要使用一支笔（Paint），就可以在Canvas上进行各种绘制。

```dart
class Canvas {

  void drawArc(Rect rect, double startAngle, double sweepAngle, bool useCenter, Paint paint) {}

  void drawAtlas(Image atlas,
                 List<RSTransform> transforms,
                 List<Rect> rects,
                 List<Color>? colors,
                 BlendMode? blendMode,
                 Rect? cullRect,
                 Paint paint) {}

  void drawCircle(Offset c, double radius, Paint paint) {}

  void drawArc(Rect rect, double startAngle, double sweepAngle, bool useCenter, Paint paint) {}


  void drawPath(Path path, Paint paint) {}

  /// 绘制图片
  void drawImage(Image image, Offset offset, Paint paint) {}

  void drawImageRect(Image image, Rect src, Rect dst, Paint paint) {}

  void drawImageNine(Image image, Rect center, Rect dst, Paint paint) {}

  void drawPicture(Picture picture) {}

  void drawParagraph(Paragraph paragraph, Offset offset) {}

  void drawPoints(PointMode pointMode, List<Offset> points, Paint paint) {}
  void drawRawPoints(PointMode pointMode, Float32List points, Paint paint) {}

  void drawVertices(Vertices vertices, BlendMode blendMode, Paint paint) {}

  void drawAtlas(Image atlas,
                 List<RSTransform> transforms,
                 List<Rect> rects,
                 List<Color>? colors,
                 BlendMode? blendMode,
                 Rect? cullRect,
                 Paint paint) {}

  void drawRawAtlas(Image atlas,
                    Float32List rstTransforms,
                    Float32List rects,
                    Int32List? colors,
                    BlendMode? blendMode,
                    Rect? cullRect,
                    Paint paint) {}


  void drawShadow(Path path, Color color, double elevation, bool transparentOccluder) {}

  /// 保存此前所有的绘制内容和canvas状态，通过restore复原
  external void save();


  /// save类似，会创建一个新的图层来进行绘制
  ///因为是创建的新的图层，所以我们设置的Paint上的blendMode属性会应用到两个图层之间
  ///创建图层时需要传入一个绘制区域，所有的绘制只会在这个区域中，超出区域会被隐藏
  ///因为是创建图层，所以会占用更多内存，有一些性能开销
  ///跟save一样，它后面也必须跟一个restore
  void saveLayer(Rect? bounds, Paint paint) {}

  /// 恢复到上一次save的状态，必须根save一起使用
  external void restore();

}
  
```

### Paint

Paint是一支笔，它有各种属性来设置笔的样式

- isAntiAlias: 是否抗锯齿
- color: 画笔颜色
- strokeWidth: 画笔宽度
- style: 样式
  - PaintingStyle.fill 默认 填充
  - PaintingStyle.stroke 线
- strokeCap: 定义画笔端点形状
  - StrokeCap.butt 无形状(默认)
  - StrokeCap.round 圆形
  - StrokeCap.square 正方形
- strokeJoin: 定义线段交接时的形状
  - StrokeJoin.miter 默认，当两条线段夹角小于30°时，StrokeJoin.miter将会变成StrokeJoin.bevel
  - StrokeJoin.bevel
  - StrokeJoin.round
- strokeMiterLimit: 当strokeJoin为StrokeJoin.miter时且style为PaintingStyle.stroke有效，用来设置连接线的长度，一般可用strokeJoin来替换
- imageFilter: 设置模糊度
- invertColors: 反转画笔颜色（跟设置的color有关）
- blendMode: 混合模式，两个形状混合时使用的模式，具体可参考blendMode，默认为BlendMode.srcOver
- shader: 着色器
- maskFilter: 模糊蒙版滤镜，比如绘制一些阴影效果或者艺术字等
- filterQuality: 设置滤镜（如maskFilter或者image）的质量
- colorFilter: 彩色矩阵滤色器，可以通过设置此属性改变画笔颜色如黑白色

## 实战绘图

### drawPath

```dart
class Path {
  /// 设置画笔的起始位置
  external void moveTo(double x, double y);

  /// 画直线到下一个位置
  external void lineTo(double x, double y);

  /// 画一条直线，从当前位置（x,y）到（x+dx, y+dy）的直线
  external void relativeLineTo(double dx, double dy);
  /// 
  external void relativeMoveTo(double dx, double dy);
  /// 绘制一条弧线
  void arcTo(Rect rect, double startAngle, double sweepAngle, bool forceMoveTo) {}

  /// 绘制贝塞尔曲线
  external void quadraticBezierTo(double x1, double y1, double x2, double y2);

  /// 绘制二阶贝塞尔曲线
  external void conicTo(double x1, double y1, double x2, double y2, double w);

  /// 绘制三阶贝塞尔曲线
  external void cubicTo(double x1, double y1, double x2, double y2, double x3, double y3);

  void addRect(Rect rect) {}


  void addOval(Rect oval) {}

  void addArc(Rect oval, double startAngle, double sweepAngle) {}

  void addPolygon(List<Offset> points, bool close) {}

  void addRRect(RRect rrect) {}

  void addPath(Path path, Offset offset) {}

  void close() {}

  void reset() {}

  void shift(Offset offset) {}

  void transform(Float64List matrix4) {}

  /// 绘制阴影
  void drawShadow(Path path, Color color, double elevation, bool transparentOccluder) {}

  void drawCircle(Offset c, double radius, Paint paint) {}


}

```

### 具体实例

```dart
import 'dart:math';
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:like_button/like_button.dart';

void main() {
  runApp(MaterialApp(
    home: CustomPaintExample(),
  ));
}

class CustomPaintExample extends StatefulWidget {
  const CustomPaintExample({super.key});

  @override
  State<CustomPaintExample> createState() => _CustomPaintExampleState();
}

class _CustomPaintExampleState extends State<CustomPaintExample> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('CustomPaint Example'),
      ),
      body: buildBody(),
    );
  }

  /// 将图片转换成ui.Image，通过drawImage绘制
  Future<ui.Image> load(String asset) async {
    ByteData data = await rootBundle.load(asset);
    ui.Codec codec = await ui.instantiateImageCodec(data.buffer.asUint8List(), targetHeight: 100, targetWidth: 100);
    ui.FrameInfo fi = await codec.getNextFrame();
    return fi.image;
  }

  buildBody() {
    return Container(
      color: Colors.grey[300],
      width: 400,
      height: double.infinity,
      child: CustomPaint(
        foregroundPainter: CirclePainter(
          outerCircleRadiusProgress: 0.5,
          innerCircleRadiusProgress: 0.3,
        ),
      ),
    );
  }
}

class CirclePainter extends CustomPainter {
  CirclePainter(
      {required this.outerCircleRadiusProgress,
      required this.innerCircleRadiusProgress,
      this.circleColor = const CircleColor(start: Color(0xFFFF5722), end: Color(0xFFFFC107))}) {
    //circlePaint..style = PaintingStyle.fill;
    _circlePaint.style = PaintingStyle.stroke;
    //maskPaint..blendMode = BlendMode.clear;
  }

  final Paint _circlePaint = Paint();

  //Paint maskPaint = new Paint();

  final double outerCircleRadiusProgress;
  final double innerCircleRadiusProgress;
  final CircleColor circleColor;

  drawPath(Canvas canvas, Size size) {
    Paint newPaint(Color color) {
      Paint paint = Paint()
        ..color = color
        ..style = PaintingStyle.stroke
        ..strokeWidth = 10;
      return paint;
    }

    Path generatePath(double x, double y) {
      Path path = Path();
      path.moveTo(x, y);
      // path.lineTo(x + 100, y + 100);
      // path.relativeLineTo(x + 100, y + 100);
      path.lineTo(x + 100, y + 100);
      path.lineTo(x + 150, y + 80);
      path.lineTo(x + 100, y + 200);
      path.lineTo(x, y + 100);
      return path;
    }

    canvas.drawPath(generatePath(100, 100), newPaint(Colors.red));
    // canvas.save();
    // canvas.rotate(10 * pi / 180);
    // canvas.drawPath(generatePath(100, 150), newPaint(Colors.blue));
    // canvas.restore();
    // canvas.drawPath(generatePath(100, 500), newPaint(Colors.yellow));
  }

  ///    pointMode: 设置点、线
  ///         PointMode.points 设置点
  ///         PointMode.lines 两个两个点之间连接，如果传入的points是奇数，最后一个点将会被忽略
  ///         PointMode.polygon 将所有点连接起来
  ///     points: 一个Offset数组，可以画多个点
  drawPoints(Canvas canvas, Size size) {
    Paint paint = Paint()
      ..color = Colors.red
      ..strokeWidth = 20;

    canvas.drawPoints(
        PointMode.points,
        [
          Offset(100, 100),
          Offset(250, 180),
          Offset(200, 300),
        ],
        paint);
    // 将端点设置为圆形
    paint.strokeCap = StrokeCap.round;
    canvas.drawPoints(PointMode.points, [Offset(100, 200)], paint);
  }

  /// 画直线
  drawLine(Canvas canvas, Size size) {
    Paint paint = Paint()
      ..color = Colors.black12
      ..strokeWidth = 20;
    canvas.drawLine(Offset(100, 100), Offset(250, 180), paint);
    //PointMode.lines可以达到同样的效果
    // canvas.drawPoints(
    //     PointMode.lines,
    //     [
    //       Offset(100, 100),
    //       Offset(250, 180),
    //     ],
    //     paint);
  }

  drawRect(Canvas canvas, Size size) {
    Paint paint = Paint()
      ..color = Colors.pink
      ..strokeWidth = 20;

    // Rect.fromLTRB 参数：
    // left: 矩形左边距离画布左边距离
    // top: 矩形顶部距离画布顶部距离
    // right: 矩形右边距离画布左边边距离
    // bottom: 矩形底部距离画布顶部距离
    // canvas.drawRect(Rect.fromLTRB(50, 50, 350, 350), paint);
    // canvas.drawRect(Rect.fromCenter(center: Offset(40, 20), width: 20, height: 30), paint);
    // canvas.drawRect(Rect.fromCircle(center: Offset(100, 120), radius: 50), paint);
    canvas.drawRect(Rect.fromPoints(Offset(100, 120), Offset(130, 140)), paint);
  }

  drawOval(Canvas canvas, Size size) {
    Paint paint = Paint()
      ..color = Colors.pink
      ..strokeWidth = 20;
    Rect pRect = Rect.fromLTRB(50, 150, 400, 350);
// 为了区别，先绘制一个矩形区域
    canvas.drawRect(pRect, paint);
    paint.color = Colors.yellow;
// 绘制椭圆
    canvas.drawOval(pRect, paint);
  }

  drawArc(Canvas canvas, Size size) {
    Paint paint = Paint()..color = Colors.pink;
    Rect rect = Rect.fromCircle(center: Offset(size.width / 2, size.height / 2), radius: 100);
    canvas.save();
    canvas.drawRect(rect, Paint()..color = Colors.blue);
    canvas.restore();
    canvas.drawArc(rect, 90 * (pi / 180), 90 * (pi / 180), false, paint);
  }

  @override
  void paint(Canvas canvas, Size size) {
    drawPoints(canvas, size);
    drawLine(canvas, size);
    drawRect(canvas, size);
    drawOval(canvas, size);
    drawArc(canvas, size);
    drawPath(canvas, size);
    drawCircle(canvas, size);
  }

  drawCircle(Canvas canvas, Size size) {

    final double center = size.width * 0.5;
    _updateCircleColor();
    // canvas.saveLayer(Offset.zero & size, Paint());
    // canvas.drawCircle(Offset(center, center),
    //     outerCircleRadiusProgress * center, circlePaint);
    // canvas.drawCircle(Offset(center, center),
    //     innerCircleRadiusProgress * center + 1, maskPaint);
    // canvas.restore();
    //flutter web don't support BlendMode.clear.
    final double strokeWidth = outerCircleRadiusProgress * center -
        (innerCircleRadiusProgress * center);
    if (strokeWidth > 0.0) {
      _circlePaint.strokeWidth = strokeWidth;
      canvas.drawCircle(Offset(center, center),
          outerCircleRadiusProgress * center, _circlePaint);
    }
  }

  double clamp(double value, double low, double high) {
    return min(max(value, low), high);
  }

  double mapValueFromRangeToRange(double value, double fromLow, double fromHigh, double toLow, double toHigh) {
    return toLow + ((value - fromLow) / (fromHigh - fromLow) * (toHigh - toLow));
  }

  void _updateCircleColor() {
    double colorProgress = clamp(outerCircleRadiusProgress, 0.5, 1.0);
    colorProgress = mapValueFromRangeToRange(colorProgress, 0.5, 1.0, 0.0, 1.0);
    _circlePaint.color = Color.lerp(circleColor.start, circleColor.end, colorProgress)!;
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    if (oldDelegate.runtimeType != runtimeType) {
      return true;
    }

    return oldDelegate is CirclePainter &&
        (oldDelegate.outerCircleRadiusProgress != outerCircleRadiusProgress ||
            oldDelegate.innerCircleRadiusProgress != innerCircleRadiusProgress ||
            oldDelegate.circleColor.start != circleColor.start ||
            oldDelegate.circleColor.end != circleColor.end);
  }
}

```

## 参考
