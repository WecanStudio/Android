# 自定义view

在学习自定义View之前，需要掌握View的相关知识。

## 分类

1. 继承View


2. 继承ViewGroup


3. 继承特定的View


4. 继承特定的ViewGroup


## 注意事项

- 让View支持wrap_content

**直接继承自View和ViewGroup的控件,如果不对wrap_content做特殊处理，那么在布局中wrap\_content和math\_parent的效果是一样的。所以我们应当在onMeasure中针对wrap\_content做特殊处理。**

- 让View支持padding

**直接继承View的控件，如果不在draw方法中处理padding，那么padding属性是不起作用的。另外，直接继承自ViewGroup的控件需要在onMeasure和onLayout中考虑padding和子元素的margin对其造成的影响，不然降导致padding和子元素的margin失效。注意子元素的margin属性是在其父容器中处理的，所以继承自View的控件不需要对margin属性做特殊处理。**

- 尽量不要在View中使用Handler

**View内部本身提供了post系列的方法，完全可以替代Handler的作用。**

- View中如果有线程或者动画，需要及时停止

**如果有线程或者动画需要停止时，View中onDetachedFromWindoe方法是一个很好的时机。**

- View中带有滑动冲突时，要处理好滑动冲突

## 添加自定义属性

## 实例

### 继承ViewGroup

[FlowLayout](https://github.com/hongyangAndroid/FlowLayout)

[SlideBottomPanel](https://github.com/kingideayou/SlideBottomPanel)

[PasswordView](https://github.com/xdsjs/PasswordView)


