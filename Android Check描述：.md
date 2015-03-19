
###Android Check描述：
	Handler以内部类的形式出现时，要使用静态内部类。

###示例代码：（如果包含这样的代码，请参考改进方案部分修改）

``` java
public class SampleActivity extends Activity { 
 
  private final Handler mHandler = new Handler() { 
    @Override 
    public void handleMessage(Message msg) { 
      // ...  
    } 
  } 
} 
```
###Lint工具可以帮助检查：

```
HandlerLeak
-----------
Summary: Handler reference leaks

Priority: 4 / 10
Severity: Warning
Category: Performance

Since this Handler is declared as an inner class, it may prevent the outer
class from being garbage collected. If the Handler is using a Looper or
MessageQueue for a thread other than the main thread, then there is no issue.
If the Handler is using the Looper or MessageQueue of the main thread, you
need to fix your Handler declaration, as follows: Declare the Handler as a
static class; In the outer class, instantiate a WeakReference to the outer
class and pass this object to your Handler when you instantiate the Handler;
Make all references to members of the outer class using the WeakReference
object.
```

###原理解释：
1.      当Android程序第一次创建的时候，在主线程同时会创建一个Looper对象。Looper实现了一个简单的消息队列，一个接着一个处理Message对象。程序框架所有主要的事件（例如：屏幕上的点击时间，Activity生命周期的方法等等）都包含在Message对象中，然后添加到Looper的消息队列中，一个一个处理。主线程的Looper存在整个应用程序的生命周期内。

1.      当一个Handler对象在主线程中创建的时候，它会关联到Looper的 message queue 。Message添加到消息队列中的时候Message会持有当前Handler引用，当Looper处理到当前消息的时候，会调用Handler#handleMessage(Message).

1.      在java中，no-static的内部类会隐式的持有当前类的一个引用。static的类则没有。 

    
###示例发生泄漏的解释：
```java
public class SampleActivity extends Activity { 
 
  private final Handler mHandler = new Handler() { 
    @Override 
    public void handleMessage(Message msg) { 
      // ... 
    } 
  } 
 
  @Override 
  protected void onCreate(Bundle savedInstanceState) { 
    super.onCreate(savedInstanceState); 
 
    // 发送一个10分钟后执行的一个消息 
    mHandler.postDelayed(new Runnable() { 
      @Override 
      public void run() { } 
    }, 600000); 
 
    // 结束当前的Activity 
    finish(); 
  } 
} 
```
当Activity结束后，在 Message queue 处理这个Message之前，它会持续存活着。这个Message持有Handler的引用，而Handler有持有Activity(SampleActivity)的引用，这个Activity所有的资源，在这个消息处理之前都不能也不会被回收，所以发生了内存泄露。

###改进方法：
```java
public class SampleActivity extends Activity { 
 
  /** 
   * 使用静态的内部类，不会持有当前对象的引用 
   */ 
  private static class MyHandler extends Handler { 
    private final WeakReference<SampleActivity> mActivity; 
 
    public MyHandler(SampleActivity activity) { 
      mActivity = new WeakReference<SampleActivity>(activity); 
    } 
 
    @Override 
    public void handleMessage(Message msg) { 
      SampleActivity activity = mActivity.get(); 
      if (activity != null) { 
        // ... 
      } 
    } 
  } 
 
  private final MyHandler mHandler = new MyHandler(this); 
 
  /** 
   * 使用静态的内部类，不会持有当前对象的引用 
   */ 
  private static final Runnable sRunnable = new Runnable() { 
      @Override 
      public void run() { } 
  }; 
 
  @Override 
  protected void onCreate(Bundle savedInstanceState) { 
    super.onCreate(savedInstanceState); 
 
    //  发送一个10分钟后执行的一个消息 
    mHandler.postDelayed(sRunnable, 600000); 
 
    // 结束 
    finish(); 
  } 
} 
```




