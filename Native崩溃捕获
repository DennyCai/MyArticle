### Day1
做不了，什么都做不了！！！
### Day2
分析市面上大多平台如bugly,网易云捕,crasheye等都具备java崩溃和native崩溃的捕获，甚至还具备anr监控和javascript异常捕获。
### Day3
- Java Exception：使用'Thread.setDefaultUncaughtExceptionHandler'来捕获Java异常。
- Javascript Exception：使用'window.onerror'来捕获Javascript异常。
- ANR：启动一个线程，每5秒往主线程的handler post一个Runnable对象，在里面进行计数器+1，若下一次计数器的值与上一次相等，则发生了ANR。
- Native Exception：比较麻烦，另起章节说明。
### Day4
#### Native Exception 捕获方案：
接触过Native开发都可能会遇到崩溃，但崩溃发生时，我们可以通过logcat查看到debuggerd输出崩溃进程相关信息，并且在`/data/tombstones/`产生tombstone文件。

还可以通过adb执行`debuggerd -b <tid>`可以dump出进程或线程的堆栈信息。
- 动态加载方案：

