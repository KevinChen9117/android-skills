# TimingLogger的使用
##使用步骤
1) 打开调试才能看到
```
adb shell setprop log.tag.[TAG] VERBOSE
```

> **注意**：1.[TAG]替换成设置的TAG 
            2.每次开机后都需要重新设置。
            


2）使用TimingLogger。
```java
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		TimingLogger logger = new TimingLogger("MainActivity", "onCreate");
		setContentView(R.layout.activity_main);
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		logger.addSplit("first sleep done.");
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		logger.dumpToLog();
		
	}
```

3)输出 

03-19 08:09:29.643: D/MainActivity(4180): onCreate: begin

03-19 08:09:29.643: D/MainActivity(4180): onCreate:      2001 ms, first sleep done.

03-19 08:09:29.643: D/MainActivity(4180): onCreate: end, 2001 ms

####总结
这个工具可以打印出每个方法的执行时间，还可以将一个方法分成块，看每块执行
的时间。对于分析性能问题特别有用。但是执行的使用容易忘记第一步。

