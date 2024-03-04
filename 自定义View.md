# 自定义View

## onMesaure

* 该方法会传入当前View的宽和高的`MeasureSpec`
* `MeasureSpec`是个32位整数，高两位代表mode，低30位代表size，mode可选值有`EXACTLY`，`AT_MOST`，`UNSPECIFIED`。
* 自定义`ViewGroup`和自定义`View`一般都要实现这个方法。需要在这个方法中遍历所有子View，根据子`View`的`LayoutParam`和容器View的MeasureSpec计算对应的子`View`的`MeasureSpec`，然后调用子`View`的`measure`方法，传入宽高的`MeasureSpec`。调用子`View` `measure`后，子View也会经历相同的流程，接下来就可以获取该子View的测量宽高，根据当前的View布局特性和当前View的MeasureSpec计算当前View的最终宽高。一般这过程中还需要考虑自己的Padding和子View的Margin

```kotlin
    
//垂直LinearLayout的简单实现，未考虑Margin和Padding
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        var usedHeight = 0
        var maxWidth = 0
        val w = MeasureSpec.getSize(widthMeasureSpec)
        val wMode = MeasureSpec.getMode(widthMeasureSpec)
        val h = MeasureSpec.getSize(heightMeasureSpec)
        val hMode = MeasureSpec.getMode(heightMeasureSpec)
        for (i in 0 until childCount) {
            val v = getChildAt(i)
            val lp: LayoutParams = v.layoutParams
            val childWidthSpec = getChildMeasureSpec(widthMeasureSpec, 0, lp.width)
            val childHeightSpec = getChildMeasureSpec(heightMeasureSpec, usedHeight, lp.height)
            v.measure(childWidthSpec, childHeightSpec)
            val measuredHeight = v.measuredHeight
            usedHeight += measuredHeight
            if (v.measuredWidth > maxWidth) {
                maxWidth = v.measuredWidth
            }
        }

        var measuredWidth = 0
        var measuredHeight = 0
        //测量宽度
        when (wMode) {
            MeasureSpec.AT_MOST -> {
                if (maxWidth > w) {
                    measuredWidth = w
                } else {
                    measuredWidth = maxWidth
                }
            }
            MeasureSpec.EXACTLY -> {
                measuredWidth = w
            }
            else -> {
                throw RuntimeException("not support measureSpec")
            }
        }
        when (hMode) {
            MeasureSpec.AT_MOST -> {
                if (usedHeight > h) {
                    measuredHeight = h
                } else {
                    measuredHeight = usedHeight
                }
            }

            MeasureSpec.EXACTLY -> {
                measuredHeight = h
            }
            else -> {
                throw RuntimeException("not support measureSpec")
            }
        }

        setMeasuredDimension(measuredWidth, measuredHeight)
}

```



## onLayout

* 该方法会传入当前View上下左右四个位置的定位

* 一般在容器View中实现该方法

* 一般需要在该方法下面遍历子View，根据要实现布局特性，调用每个子View的layout方法，传入上下左右的定位参数。

  计算子View的位置时要考虑自己的Padding和子View的Margin

```kotlin
  //垂直布局的LinearLayout的简单实现
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        var startHeight = t
        for (i in 0 until childCount) {
            val view = getChildAt(i)
            val h = view.measuredHeight
            val w = view.measuredWidth
            val bottom = startHeight + h
            var right = r
            if (l + w < r) {
                right = l + w
            }
            view.layout(l, t + startHeight, right, bottom)
            startHeight = bottom
        }
}
```



## onDraw

* 该方法主要是实现具体的绘制逻辑

* 通过`Cavans` ,`Paint`,`Path`等对象实现绘图逻辑，这个过程中一般会用到onMeasure中得到的View宽高的数据

* 绘制过程中尽量使用`Cavans#save`，`Cavans#restore`来控制绘制的图层，不要在该方法中直接创建`Path`,`Paint`等对象，通过kotilin lazy 方式创建或者在`View`初始化的时候创建，保存在View私有变量中，每次draw之前重新设置`Paint`和`Path`

  ````kotlin
  //简单的圆型绘制
  override fun onDraw(canvas: Canvas?) {
          super.onDraw(canvas)
          val w = measuredWidth
          val h = measuredHeight
          canvas?.drawCircle(w / 2f, h / 2f, min(w, h) / 2f, paint)
  }
  ````

  