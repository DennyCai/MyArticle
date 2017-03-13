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
#### Hello Tombstone:
接触过Native开发都可能会遇到崩溃，但崩溃发生时，我们可以通过logcat查看到debuggerd输出崩溃进程相关信息，并且在`/data/tombstones/`产生tombstone文件。举个栗子(取自Android4.4.4):
```
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint: 'Xiaomi/2014813/HM2014813:4.4.4/KTU84P/5.9.24:user/release-keys'
Revision: '0'
pid: 32558, tid: 32558, name: su  >>> /system/xbin/su <<<
signal 13 (SIGPIPE), code -6 (SI_TKILL), fault addr --------
    r0 ffffffe0  r1 be8d7568  r2 00000004  r3 b6e16104
    r4 b6e15f4c  r5 00000004  r6 b6f01394  r7 00000004
    r8 be8d7568  r9 00000000  sl be8d75e8  fp 00000000
    ip 00000004  sp be8d7488  lr b6e113dc  pc b6ed34e4  cpsr 60070010
    d0  69752f746363612f  d1  b7d04bb8b7d04b74
    d2  b7d04bd8b7d04b61  d3  b7d04bf8b7d04b73
    d4  b7d04288b7d04278  d5  b7d046e8b7d04298
    d6  b7d04b48b7d04b38  d7  b7d04b68b7d04b58
    d8  0000000000000000  d9  0000000000000000
    d10 0000000000000000  d11 0000000000000000
    d12 0000000000000000  d13 0000000000000000
    d14 0000000000000000  d15 0000000000000000
    d16 4161db3246872b02  d17 3f50624dd2f1a9fc
    d18 41b590c91c000000  d19 0000000000000000
    d20 0000000000000000  d21 0000000000000000
    d22 0000000000000000  d23 0000000000000000
    d24 0000000000000000  d25 0000000000000000
    d26 0000000000000000  d27 0000000000000000
    d28 0000000000000000  d29 0000000000000000
    d30 0000000000000000  d31 0000000000000000
    scr 00000010

backtrace:
    #00  pc 000204e4  /system/lib/libc.so (write+12)
    #01  pc 000063d8  /system/vendor/lib/libcneconn.so (write+228)
    #02  pc 00000e35  /system/vendor/lib/libNimsWrap.so (write+24)
    #03  pc 00004058  /system/xbin/su
    #04  pc 00002610  /system/xbin/su
    #05  pc 0000e54b  /system/lib/libc.so (__libc_init+50)
    #06  pc 000019ac  /system/xbin/su
```
我们需要关注几个比较重要的信息有
`pid: 32558, tid: 32558, name: su  >>> /system/xbin/su <<<`:pid表示崩溃进程id,tid是线程号,name是线程名,>>>这里是进程名<<<
`signal 13 (SIGPIPE), code -6 (SI_TKILL), fault addr --------`:中断信号信息,13信号值,SIGPIPE中断原因.
紧跟着下面是寄存器的值和部分内存信息
`backtrace`:堆栈信息
`#00  pc 000204e4  /system/lib/libc.so (write+12)`:栈标号 pc 相对于so偏移量 so位置 (符号名+相对于该符号的偏移量)
不同系统版本所展现的内容也有区别,5.0还包含cpu架构信息.

我们还可以通过adb执行`debuggerd -b <tid>`可以dump出进程或线程的堆栈信息。
```text
debuggerd -b 1041
Sending request to dump task 1041.


----- pid 1041 at 2017-03-09 17:41:49 -----
Cmd line: com.miui.core

"com.miui.core" sysTid=1041
  #00  pc 00021938  /system/lib/libc.so (epoll_wait+12)
  #01  pc 00010697  /system/lib/libutils.so (android::Looper::pollInner(int)+98)

  #02  pc 000108c1  /system/lib/libutils.so (android::Looper::pollOnce(int, int*
, int*, void**)+92)
  #03  pc 0006b529  /system/lib/libandroid_runtime.so (android::NativeMessageQue
ue::pollOnce(_JNIEnv*, int)+22)
  #04  pc 000204cc  /system/lib/libdvm.so (dvmPlatformInvoke+112)
  #05  pc 00051157  /system/lib/libdvm.so (dvmCallJNIMethod(unsigned int const*,
 JValue*, Method const*, Thread*)+398)
  #06  pc 00029960  /system/lib/libdvm.so
  #07  pc 00030dec  /system/lib/libdvm.so (dvmMterpStd(Thread*)+76)
  #08  pc 0002e484  /system/lib/libdvm.so (dvmInterpret(Thread*, Method const*,
JValue*)+184)
  #09  pc 0006389d  /system/lib/libdvm.so (dvmInvokeMethod(Object*, Method const
*, ArrayObject*, ArrayObject*, ClassObject*, bool)+392)
  #10  pc 0006b7a3  /system/lib/libdvm.so
  #11  pc 00029960  /system/lib/libdvm.so
  #12  pc 00030dec  /system/lib/libdvm.so (dvmMterpStd(Thread*)+76)
  #13  pc 0002e484  /system/lib/libdvm.so (dvmInterpret(Thread*, Method const*,
JValue*)+184)
  #14  pc 000635b9  /system/lib/libdvm.so (dvmCallMethodV(Thread*, Method const*
, Object*, bool, JValue*, std::__va_list)+336)
  #15  pc 0004cd37  /system/lib/libdvm.so
  #16  pc 0004de93  /system/lib/libandroid_runtime.so
  #17  pc 0004ebbf  /system/lib/libandroid_runtime.so (android::AndroidRuntime::
start(char const*, char const*, bool)+358)
  #18  pc 00001063  /system/bin/app_process
  #19  pc 0000e54b  /system/lib/libc.so (__libc_init+50)
  #20  pc 00000d80  /system/bin/app_process
......省略许多不可描述的信息......
----- end 1041 -----
```
我们的SDK就需要收集类似Tombstone的崩溃信息,方便开发者定位到崩溃地方.
### Day5
我们首先从如何在崩溃的时候dump出堆栈信息入手.
总所周知Android使用Linux内核,当崩溃的时候应用会接收到内核发出的信号.那么我们就需要注册关心的信号并处理.
(注:默认读者熟悉基本的NDK开发)
通过调用`int sigaction(int signo,const struct sigaction *act,struct sigaction *oact);`方法来注册我们关心的信号值,`signo`:表示信号值,`act`:注册的信号处理函数数据结构,`oact`:默认的信号处理函数,
`struct sigaction`数据结构如下:
```c
struct sigaction {
  unsigned int sa_flags;
  union {
    sighandler_t sa_handler;
    void (*sa_sigaction)(int, struct siginfo*, void*);
  };
  sigset_t sa_mask;
  void (*sa_restorer)(void);
};
```
`sa_flags`:用来改变信号处理行为,这里我们需要将改值设为`SA_RESETHAND`,表示执行完自己的处理函数后再执行系统的处理函数.
~~`sa_handle`~~:已经废弃,取而代之的是`sa_sigaction`:函数指针,第一个参数是信号值,第二个参数为异常信号的信息,第三个是`ucontext_t`类型.
`sa_mask`:信号掩码.
Android支持信号如下:
| 信号值 |   |   |  |
| -------- | ------- | -------- | ------ |
| SIGHUP 1  | SIGINT 2  | SIGQUIT 3 | SIGILL 4 |
| SIGTRAP 5 | SIGABRT 6 | SIGIOT 6  | SIGBUS 7 |
| SIGFPE 8  | SIGKILL 9 | SIGUSR1 10| SIGSEGV 11|
| SIGUSR2 12| SIGPIPE 13| SIGALRM 14| SIGTERM 15|
| SIGSTKFLT 16| SIGCHLD 17| SIGCONT 18| SIGSTOP 19|
| SIGTSTP 20 | SIGTTIN 21 | SIGTTOU 22 | SIGURG 23 |
| SIGXCPU 24 | SIGXFSZ 25 | SIGVTALRM 26 | SIGPROF 27 |
| SIGWINCH 28 | SIGIO 29 | SIGPOLL 29 | SIGPWR 30 |
| SIGSYS 31 | SIGUNUSED 31 | | |

我们可以在`JNI_OnLoad`方法中注册信号处理函数:
```c
static struct sigaction *def;
void handleSignal(int code, siginfo_t *info, void *reserved){
    __android_log_print(ANDROID_LOG_INFO, "NativeCrash","Signal Code:%d", code);
}
void registerCrashSignal() {
    struct sigaction handler;
    handler.sa_sigaction = handleSignal;
    handler.sa_flags = SA_RESETHAND | SA_SIGINFO;
    sigaction(SIGSEGV, &handler, def);//配合demo,先注册段异常信号
}

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void* reserved)
{
    registerCrashSignal();
    ....
}
```
有了信号处理函数,接下来编写一个出现异常的函数:
```c
void doCrash(){
    int *p= NULL;
    *p=1;
};
//通过JNI调用c方法
extern "C"
JNIEXPORT void JNICALL
Java_com_denny_anative_MainActivity_doCrash(JNIEnv *env, jobject instance) {
    doCrash();
}
```
经典的野指针异常.执行后可看到logcat打印出`15818-15818/com.denny.anative I/NativeCrash: Signal Code:11`,表示我们已经成功捕获异常信号.
