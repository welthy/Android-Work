[TOC]

## 简介

本文将介绍View的一些滑动方式，对常用的scrollTo/By做稍微详细的介绍。

### scrollTo/By

这是系统提供的滑动方式，先看看系统是如何实现滑动的。

```java
/**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
     */
public void scrollTo(int x, int y) {  //1
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

/**
     * Move the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the amount of pixels to scroll by horizontally
     * @param y the amount of pixels to scroll by vertically
     */
public void scrollBy(int x, int y) {   //2
    scrollTo(mScrollX + x, mScrollY + y);
}
```

我们先看看scrollBy。我们传入水平和垂直方向需要滑动的距离，然后调用scrollTo()完成滑动。在调用scrollTo()时，我们还会传入mScrollX和mScrollY参量，先看看这2个参量的含义： 

```java
/**
     * The offset, in pixels, by which the content of this view is scrolled
     * horizontally.
     * Please use {@link View#getScrollX()} and {@link View#setScrollX(int)} instead of
     * accessing these directly.
     * {@hide}
     */
@ViewDebug.ExportedProperty(category = "scrolling")
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    protected int mScrollX;
/**
     * The offset, in pixels, by which the content of this view is scrolled
     * vertically.
     * Please use {@link View#getScrollY()} and {@link View#setScrollY(int)} instead of
     * accessing these directly.
     * {@hide}
     */
@ViewDebug.ExportedProperty(category = "scrolling")
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    protected int mScrollY;
```

大概意思是：mScrollX代表View内容的水平方向的偏移量，像素单位。mScrollY则代表的是垂直方向偏移量。这个值我们可以通过getScrollX/Y得到。

```java
/**
     * Return the scrolled left position of this view. This is the left edge of
     * the displayed part of your view. You do not need to draw any pixels
     * farther left, since those are outside of the frame of your view on
     * screen.
     *
     * @return The left edge of the displayed part of your view, in pixels.
     */
@InspectableProperty
public final int getScrollX() {
    return mScrollX;
}

/**
     * Return the scrolled top position of this view. This is the top edge of
     * the displayed part of your view. You do not need to draw any pixels above
     * it, since those are outside of the frame of your view on screen.
     *
     * @return The top edge of the displayed part of your view, in pixels.
     */
@InspectableProperty
public final int getScrollY() {
    return mScrollY;
}
```

**源码中的解释大意是：mScrollX是View的左边界到View内容左边界的距离；mScrollY则代表View的上边界到View内容上边界的距离。**

那么回过头我们看scrollBy()内，我们传给scrollTo()的是mScrollX+x和mScrollY+y，即水平方向当前的偏移量加上需要滑动的距离，垂直方向当前的偏移量加上需要滑动的距离。所以我们使用**scrollBy()**时，达到的效果是：**View的内容相对于当前位置水平方向和垂直方向滑动。**  

然后继续看scrollTo()。scrollTo则是直接改变mScrollX和mScrollY的值，然后进行重绘达到滑动的效果。所以**scrollTo()**达到的效果是：**View的内容直接滑动到指定的x,y位置。**  

- **值得注意的是，这种方式滑动的是View的内容，而不是View本身。**

### 改变布局参数

除了系统提供的滑动方式外，我们可以从其他角度间接的达到滑动的效果，改变View的布局参数LayoutParams就是个不错的选择。 

如下示例： 

```java
MarginLayoutParams params = (MarginLayoutParams)mTextView.getLayoutParams();
params.rightMargin += 10;
mTextView.requestLayout();
//或者mTextView.setLayoutParams(params);
```

如上，即达到将TextView向左滑动10个像素的目的。

- **通过这种方式滑动的是View本身。**

### 使用动画

除了改变布局参数外，还能通过动画来间接达到滑动的效果。我们知道动画中有专门的平移动画，利用该动画类型我们也能达到在水平与垂直方向的滑动效果，在这不再列举。 

- **值得注意的是，动画方式的滑动不适合需要与View进行交互的场景，即跟随手指滑动的场景。** 



滑动方式基本这几类，其中改变布局参数的方式内部又有几种子方式（系统提供了一系列改变其参数的API），不过都是通过改变布局参数达到的效果，属于一类。