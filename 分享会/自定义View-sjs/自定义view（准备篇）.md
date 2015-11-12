# Android中的View

***

## 什么是View

> This class represents the basic building block for user interface components. A View occupies a rectangular area on the screen and is responsible for drawing and event handling. View is the base class for widgets, which are used to create interactive UI components (buttons, text fields, etc.). The ViewGroup subclass is the base class for layouts, which are invisible containers that hold other Views (or other ViewGroups) and define their layout properties.

**View占据了屏幕中的一块矩形区域，负责视图的绘制和事件处理。view是所有控件的基类。**

![View的层次结构](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/view.png)

**可以看到，ViewGroup也继承自View，这就意味着View本身就可以使单个控件也可以是由多个控件组成的一组控件。通过这种关系就形成了View的树形结构。**

## View中的位置参数
- top、left、right、bottom
	- 分别对应于view的原始左上角纵坐标、左上角横坐标、右下角横坐标、右下角纵坐标
	- 都是相对坐标，相对于View的直接父容器来说的。
	- 都可以通过相对应的get方法获取到(getLeft()、getRight()、getTop()、getBottom())。
- x、y、translationX、translationY
	- Android 3.0后加入，x、y是View左上角的坐标，translationX、translationY是View左上角相对于父容器的偏移量。
	- 都是相对坐标，相对于View的直接父容器来说的。
	- 都可以通过相对应的get方法获取到(getX、getY、getTranslationX、getTranslationY)。
	- x = left + translationX
	- y = top + translationY
	- 在平移过程中这几个参数会发生改变(但top、left、right、bottom不会改变)
- getWidth()/getHeight()
	- 获得View**最终**的宽度/高度
	- View的宽度 = getWidth() = getRight() - getLeft()
	- View的高度 = getHeight() = getBottom() - getTop()
- getMeasuredWidth()/getMeasuredHeight()
	- 获得View**测量**的宽度/高度

## View的绘制流程

> 按照[官方文档](https://developer.android.com/reference/android/view/View.html)的说法，View的绘制流程应该分为两步，Lyout和Drawing，而Layout又分为measure和layout过程。其中，measure过程确定了View的**测量宽高**（即对应于上文中提到的getMeasuredWidth()/getMeasuredHeight()获得的值）。layout过程确定了View的**最终宽高**（即对应于上文中getWidth()/getHeight()获得的值）和四个顶点的位置。

### [MeasureSpec](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/view/View.java)

在讲解具体的流程前，还需要理解MeasureSpec

> MeasureSpec是一个32位的int值，封装了SpecMode和SpecSize。有了这个值，我们就可以为View测量出宽高。
> 
> MeasureSpec主要参与了View的measure过程,在测量过程中，系统会将View的LayoutParams根据父容器的所施加的规则转换成对用的MeasureSpec，最后根据这个MeasureSpec测量出


#### SpecMode

- UNSPECIFIED
> The parent has not imposed any constraint on the child. It can be whatever size it wants.
> 
> 父容器对View没有任何限制。一般用于系统内部

- EXACTLY
> The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
>
> View的最终大小即SpecSize指定的值，对应于LayoutParams中的math_parent和具体的数值两种模式。

- AT_MOST
> The child can be as large as it wants up to the specified size.
>
> 父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，对应于LayoutParams中的wrap_content

#### 静态方法
![MeasureSpec中的静态方法](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/MeasureSpec%E7%9A%84%E9%9D%99%E6%80%81%E6%96%B9%E6%B3%95.png)

**getMode和getSize方法是将MeasureSpec表示的一个32位int值解包成对应的SpecMode和SpecSize，而makeMeasureSpec方法则是将SpecSize和SpecMode打包成一个32位的int值。**

#### 解释说明

对于普通的View而言，在调用其子元素的measure方法之前会先通过一个getChildMeasureSpec方法来得到子元素的MeasureSpec。而从源码可以看出,子元素的MeasureSpec是在父容器的MeasureSpec和子元素本身的LayoutParams的共同作用下创建的。

而对于顶级View（DectorView）而言，其MeasureSpec是由窗口的尺寸和自身的LayoutParams共同确定的。

### measure

> measure过程要分情况来看，如果只是一个原始的View，那么通过measure方法就完成了其测量过程，如果只是一个ViewGroup，除了完成自己的测量过程之外，还会遍历去调用所有子元素的measure方法。各个子元素再去递归执行这个流程。

1. View的measure过程
	
View的measure过程是由其measure方法来完成的，而measure方法是一个final类型的方法，该方法中调用了onMeasure方法。
	
![onMeasure](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/onMeasure.png)

可以看到是getDefaultSize方法确定了View的最终大小。
	
![](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/defaultSize.png)

**可以看出在AT_MOST和EXACTLY情况下，View的最终宽高就是SpecSize的值，也就是说，几乎所有情况下，View的最终宽高都等于测量宽高（SpecSize）**
	
同时我们应该注意到，在AT\_MOST模式（wrap_content）下，View的最终宽高等于SpecSize的值，而此时SpecSize的值等于父容器的宽高，也就是说，如果我们不做特殊处理的话，wrap\_content和math\_parent的效果是一样的。而实际上安卓中具体的View控件都是针对wrap\_content这种情况做了处理的，处理过程很简单。
	
2. ViewGroup的measure过程。

对于ViewGroup而言，除了完成自己的measure过程以外，还会遍历去调用所有子元素的measure方法，各个子元素再递归去执行这个过程。

### layout

> layout过程用于View确定自己的位置和其子元素的位置
> 
> layout过程和measure过程的思想是相似的，对于具体的View而言，只需调用layout方法确定本身的位置即可，而对于ViewGroup而言，则会先调用layout方法确定自己的位置，再在onLayout方法中遍历所有的子元素并调用其layout方法。这样一层一层地传递下去就完成了整个View树的layout过程。
> 


### draw

> draw过程用于将View绘制到屏幕上面，view的绘制遵循如下几步：

1. 绘制背景background.draw(canvas)
2. 绘制自己(onDraw)
3. 绘制children(dispathDraw)
4. 绘制装饰(onDrawScrollBars)

View的绘制过程是通过dispathDraw来实现的，dispathDraw会遍历所有子元素的draw方法，这样draw事件就一层一层的传递了下去。


## 总结几个常用方法

### onAttachedToWindow() & onDetachedFromWindow()

onAttachedToWindow()调用时机：**当包含此View的Activity退出或者当前View被remove时**

onDetachedFromWindow()调用时机：**当包含此View的Activity启动时**

### Activity/View#onWindowFocusChange()

调用时机：当Activity的窗口得到焦点和失去焦点时均会被调用一次，即当Activity继续执行和暂停执行时，均会被调用。

此方法的含义为：View已经初始化完毕了，宽/高已经准备好了，可以在这个时候去获取View的宽高。

### requestLayout() & invalidate()

可以调用requestLayout()来刷新布局，例如当我们改变了某个View的布局参数（宽高等），就可以调用requestLayout()方法来通知View再执行一次measure、layout过程。

可以调用invalidate()来绘制一个View。

### onSizeChanged()

当View的尺寸发生变化时调用

***
# 参考资料：

《安卓开发艺术探索》