

> 我觉得要想掌握android中View的事件分发机制，需要熟悉三个方法和一个优先级。

# **三个方法**

- public boolean dispatchTouchEvent(MotionEvent ev)

用来事件的分发，只要事件能够传递到当前View，那么此方法一定会被调用。返回值表示是否消耗当前事件。

- public boolean onInterceptTouchEvent(MotionEvent ev)

在上述方法内部调用，用来判断是否拦截某个事件。如果当前View拦截了某个事件，那么在同一事件序列中，此方法不会再被调用（直接调用onTouchEvent方法），返回结果表示是否拦截当前事件。只有ViewGroup中有此方法，且默认返回false即ViewGroup默认不拦截任何事件。

- public boolean onTouchEvent(MotionEvent event)

在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，那么在同一个事件序列中，当前View无法再次接收到事件。

> 注：同一个事件序列指的是当前事件的ACTION\_DOWN、ACTION\_MOVE、ACTION\_UP、ACTION\_CANCLE等一系列完整的事件。

# 优先级

**这里的优先级说的是onTouchListener.onTouch、onTouchEvent、onClick等方法执行的优先级。**

先看一段源码

	public boolean dispatchTouchEvent(MotionEvent event) {  
        if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
                mOnTouchListener.onTouch(this, event)) {  
            return true;  
        }  
        return onTouchEvent(event);  
    } 

可以看出当设置了onTouchListener时，会先执行onTouchListener的onTouch方法，如果该方法返回false，那么回接着执行onTouchEvent方法，如果返回true，则不会执行onTouchEvent方法。所以可以得出结论：**onTouchListener的优先级大于onTouchEvent方法**。那么onClick方法何时执行呢？接着看源码：

	public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }

这个方法是在onTouchEvent方法中对ACTION_UP事件的处理时调用的，由此我们可以看出在onTouchEvent方法中，且设置了onClickListener，onClick方法才会被调用。即：**onClick方法的优先级是最低的**。

# 一般性结论

1. 同一个事件序列是指从手指接触屏幕那一刻起，到手指离开屏幕那一刻结束，在这个过程中所产生的一系事件，这个事件序列以ACTION\_DOWN事件开始，中间含有数量不定的ACTION\_MOVE事件，最终以ACTION\_UP事件结束。
2. 一个事件序列只能被一个View拦截且消耗，因为一旦一个View拦截了某事件，那么同一个时间序列的所有事件都会交给它处理。
3. 某个View一旦决定拦截事件，那么这一个事件序列都只能由它处理。
4. 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent方法返回false），那么同一事件序列的其他方法都不会在交给它处理，并且事件将重新交给父控件去处理，即父控件的onTouchEvent方法被调用。
5. 如果View不消耗ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父控件的onTouchEvent方法并不会调用，并且当前View可以持续接收到后续事件，最终这些消失的点击事件会交给Activity处理。
6. ViewGroup默认不拦截任何事件。
7. View中没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。
8. View的onTouchEvent默认都会消耗事件（即返回true），除非它是不可点击的（clickable和longClickable同时为false），View的longClickable属性默认都为false，clickable属性要分情况，比如Button的clickable属性默认为true，而TextView的clickable属性默认为false。
9. View的enable属性不影响onTouchEvent的默认返回值。哪怕一个View是disable状态的，只要它的clickable或者longClickable属性有一个为true，那么它的onTouchEvent就返回true。

# 实例验证

说了这么多结论，下面我们用一个例子验证一下。

为此，我自定义了四个View，分别是MyLineatLayout、MyRelativeLayout、MyTextView、MyButton，其实现都很简单，只是重写了dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent方法，并在方法中打印了方法名和返回值。这四个View的布局关系如下所示：

    <com.wecanstudio.xdsjs.myviewdemo.MyLinearLayout
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_alignParentLeft="true"
        android:layout_centerInParent="true"
        android:background="@android:color/holo_red_light">

        <com.wecanstudio.xdsjs.myviewdemo.MyRelativeLayout
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:layout_gravity="center"
            android:background="@android:color/holo_green_dark">

            <com.wecanstudio.xdsjs.myviewdemo.MyTextView
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_alignParentLeft="true"
                android:layout_centerInParent="true"
                android:background="@android:color/holo_blue_bright"
                android:gravity="center"
                android:text="This is a TextView" />

            <com.wecanstudio.xdsjs.myviewdemo.MyButton
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:layout_alignParentRight="true"
                android:layout_centerInParent="true"
                android:background="@android:color/darker_gray"
                android:gravity="center"
                android:text="This is a Button" />

        </com.wecanstudio.xdsjs.myviewdemo.MyRelativeLayout>


    </com.wecanstudio.xdsjs.myviewdemo.MyLinearLayout>
</RelativeLayout>

![](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/S51110-143153.jpg)

当我点击MyTextView时，打印日志如下：

![](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/log1.png)

**可以看到先是调用了MainActivity的dispatchTouchEvent，然后又接着调用了MyLinearLayout和MyRelativeLayout的dispatchTouchEvent和onInterceptTouchEvent方法，此时应该注意到这个View都没有拦截该事件，即onInterceptTouchEvent返回的都是false，这也验证了我们上面的结论，即ViewGroup默认是不拦截事件的。**

**最终这一事件传递到了MyTextView，MyTextView作为一个普通的View，当有事件到达时肯定会执行onTouchEvent方法的，但是TextView的clickable和longClickable属性默认都是false的，所以此时onTouchEvent方法返回的false，即不消耗当前事件。按照我们上面的结论，如果当前View不消耗当前事件的话，那么就会返回给父控件的onTouchEvent，所以我们看到接着就打印出了MyLinearLayout、MyRelativeLayout、MainActivity的onTouchEvent方法。这也验证了我们上面得出的结论。**

**到此时，这一事件序列中的ACTION\_DOWN事件已经处理完毕。然后ACTION\_UP事件触发,可以看到此时MainActivity没有向下分发这一事件，而是直接自己处理了。这其实同样验证了我们上面的结论，即某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent方法返回false），那么同一事件序列的其他方法都不会在交给它处理。**


当我点击MyButton时打印日志如下：

![](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/log2.png)![](https://raw.githubusercontent.com/WecanStudio/Android/master/%E5%9B%BE%E7%89%87/log3.png)

**可以看到，大致过程和上面是一样的，不过由于Button是可点击的（clickable为true），所以当事件传递到MyButton时，其消耗了这一事件（onTouchEvent放回true）。所以这一事件序列的其他事件也都交由它处理。**