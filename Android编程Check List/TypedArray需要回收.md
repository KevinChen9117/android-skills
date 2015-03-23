
###Android Check描述：
	TypedArray使用完毕后，需要调用recycle()方法。

###示例代码：（如果包含这样的代码，请参考改进方案部分修改）

``` java
 public MyView(Context context, AttributeSet attrs) {
     super(context, attrs);
     //......
     TypedArray a =context.obtainStyledAttributes(attrs,R.styleable.MyView);
    //  获取属性，0xFFFFFFFF 16是默认值
      int   textColor = a.getColor(R.styleable.MyView_textColor, 0xFFFFFFFF);
    //......
 }
```
###Lint工具可以帮助检查：

```
Recycle
-------
Summary: Missing recycle() calls

Priority: 7 / 10
Severity: Warning
Category: Performance

Many resources, such as TypedArrays, VelocityTrackers, etc., should be
recycled (with a recycle() call) after use. This lint check looks for missing
recycle() calls.
```

###原理解释：
查看recycle()方法的源代码：
```java
    /**
     * Recycle the TypedArray, to be re-used by a later caller. After calling
     * this function you must not ever touch the typed array again.
     */
    public void recycle() {
        if (mRecycled) {
            throw new RuntimeException(toString() + " recycled twice!");
        }

        mRecycled = true;

        // These may have been set by the client.
        mXml = null;
        mTheme = null;

        mResources.mTypedArrayPool.release(this);
    }
```
在TypedArray后调用recycle主要是为了缓存。当recycle被调用后，这就说明这个对象从现在可以被重用了。TypedArray 内部持有部分数组，它们缓存在Resources类中的静态字段中，这样就不用每次使用前都需要分配内存。



###改进方法：
```java
 public MyView(Context context, AttributeSet attrs) {
     super(context, attrs);
     //......
     TypedArray a =context.obtainStyledAttributes(attrs,R.styleable.MyView);
     try{
    //  获取属性，0xFFFFFFFF 16是默认值
      int   textColor = a.getColor(R.styleable.MyView_textColor, 0xFFFFFFFF);
    //......

    }finally{
        //保证回收。
        a.recycle();
    }
 }
```




