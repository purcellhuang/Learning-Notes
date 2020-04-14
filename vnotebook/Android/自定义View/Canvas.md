# Canvas

Paint 类的几个最常用的方法。具体是：
```

    Paint.setStyle(Style style)     设置绘制模式Style 具体来说有三种： FILL , STROKE 和 FILL_AND_STROKE 。
                                    FILL 是填充模式，
                                    STROKE 是画线模式（即勾边模式），
                                    FILL_AND_STROKE 是两种模式一并使用：既画线又填充。
                                    它的默认值是 FILL，填充模式。

    Paint.setColor(int color)     设置颜色
    Paint.setStrokeWidth(float width)     设置线条宽度
    Paint.setTextSize(float textSize)     设置文字大小
    Paint.setAntiAlias(boolean aa)     设置抗锯齿开关
```
**这类颜色填充方法一般用于在绘制之前设置底色，或者在绘制之后为界面设置半透明蒙版。**

> Canvas.drawColor(@ColorInt int color)     颜色填充---在整个绘制区域统一涂上指定的颜色。
> Canvas.drawRGB(int r, int g, int b)
> Canvas.drawARGB(int a, int r, int g, int b)

* **drawCircle(float centerX, float centerY, float radius, Paint paint) 画圆**
> 前两个参数 centerX centerY 是圆心的坐标，第三个参数 radius 是圆的半径，单位都是像素，

* **drawRect(float left, float top, float right, float bottom, Paint paint) 画矩形**
>left , top , right , bottom 是矩形四条边的坐标。
>另外，它还有两个重载方法 drawRect(RectF rect, Paint paint) 和
>drawRect(Rect rect, Paint paint) ，让你可以直接填写 RectF 或 Rect 对象
>来绘制矩形。

* **drawPoint(float x, float y, Paint paint) 画点**
>x 和 y 是点的坐标。点的大小可以通过 paint.setStrokeWidth(width) 来设
>置；点的形状可以通过 paint.setStrokeCap(cap) 来设置：ROUND 画出来是圆形
>的点，SQUARE 或 BUTT 画出来是方形的点。

* **drawPoints(float[] pts, int offset, int count, Paint paint) / drawPoints(float[] pts, Paint paint) 画点（批量）**
>同样是画点，它和 drawPoint() 的区别是可以画多个点。pts 这个数组是点的坐
>标，每两个成一对；offset 表示跳过数组的前几个数再开始记坐标；count 表示
>一共要绘制几个点。

* **drawOval(float left, float top, float right, float bottom, Paint paint) 画椭圆**
只能绘制横着的或者竖着的椭圆，不能绘制斜的（斜的倒是也可以，但不是直接使
用 drawOval()，而是配合几何变换，后面会讲到）。left , top , right , bottom
是这个椭圆的左、上、右、下四个边界点的坐标。
另外，它还有一个重载方法 drawOval(RectF rect, Paint paint)，让你可以直
接填写 RectF 来绘制椭圆。

* **drawLine(float startX, float startY, float stopX, float stopY, Paint paint) 画线**
canvas.drawPoints(points, 2 /* 跳过两个数，即前两个 0 */,
8 /* 一共绘制 8 个数（4 个点）*/, paint);
startX , startY , stopX , stopY 分别是线的起点和终点坐标。

* **drawLines(float[] pts, int offset, int count, Paint paint) / drawLines(float[] pts, Paint paint) 画线（批量）**
drawLines() 是 drawLine() 的复数版。

* **drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint) 画圆角矩形**
float[] points = {20, 20, 120, 20, 70, 20, 70, 120, 20, 120, 120, 12
canvas.drawLines(points, paint);
left , top , right , bottom 是四条边的坐标，rx 和 ry 是圆角的横向半径和纵向
半径。
另外，它还有一个重载方法
drawRoundRect(RectF rect, float rx, float ry, Paint paint)，让你可以
直接填写 RectF 来绘制圆角矩形。

* **drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, Paint paint) 绘制弧形或扇
形**
drawArc() 是使用一个椭圆来描述弧形的。left , top , right , bottom 描述的是
这个弧形所在的椭圆；startAngle 是弧形的起始角度（x 轴的正向，即正右的方
向，是 0 度的位置；顺时针为正角度，逆时针为负角度），sweepAngle 是弧形划
过的角度；useCenter 表示是否连接到圆心，如果不连接到圆心，就是弧形，如果
连接到圆心，就是扇形。

到此为止，以上就是 Canvas 所有的简单图形的绘制。除了简单图形的绘制，
Canvas 还可以使用 drawPath(Path path) 来绘制自定义图形。





**抗锯齿**
在绘制的时候，往往需要开启抗锯齿来让图形和文字的边缘更加平滑。
开启抗锯齿很简单，只要在 new Paint() 的时候加上一个 ANTI_ALIAS_FLAG 参数就行：
```
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
```
另外，你也可以使用 Paint.setAntiAlias(boolean aa) 来动态开关抗锯齿。