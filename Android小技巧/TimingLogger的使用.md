# TimingLogger的使用
##使用步骤
1) 打开调试才能看到
```
adb shell setprop log.tag.[TAG] VERBOSE
```

> **注意**：1.[TAG]替换成设置的TAG 
            2.每次开机后都需要重新设置。
            
>**原因**：查看TimingLogger的源码可以知道，构造方法中使用了Log.isLogable()方法判断是否打印log。如果为true，则打印，否则不打。

2）使用android.util.TimingLogger。
```java
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		TimingLogger logger = new TimingLogger("MainActivity", "onCreate");
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		logger.addSplit("first sleep done.");
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		logger.addSplit("second sleep done.");
		logger.dumpToLog();
		
	}
```

3)输出 

```
03-19 08:59:11.084: D/MainActivity(6958): onCreate: begin
03-19 08:59:11.084: D/MainActivity(6958): onCreate:      2028 ms, first sleep done.
03-19 08:59:11.084: D/MainActivity(6958): onCreate:      3000 ms, second sleep done.
03-19 08:59:11.084: D/MainActivity(6958): onCreate: end, 5028 ms
```
> **注意**：dumpToLog()方法前需要调用logger.addSplit()方法，否则时间信息不对。

####总结
这个工具可以打印出每个方法的执行时间，还可以将一个方法分成块，看每块执行
的时间。对于分析性能问题特别有用。但是执行的使用容易忘记第一步。

