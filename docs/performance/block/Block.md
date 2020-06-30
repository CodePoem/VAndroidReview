# 卡顿优化（Block）

卡顿问题难以排查定位，产生的原因错综复杂，跟 CPU、内存、磁盘 I/O 都可能有关，跟用户当时的系统环境也有很大关系。

造成卡顿的原因可能有千百种，不过最终都会反映到 CPU 时间上。我们可以把 CPU 时间分为两种：用户时间和系统时间。用户时间就是执行用户态应用程序代码所消耗的时间；系统时间就是执行内核态系统调用所消耗的时间，包括 I/O、锁、中断以及其他系统调用的时间。

## 排查手段与工具

trace 文件，是 log 信息文件的一种。

官方[系统追踪]（https://developer.android.google.cn/topic/performance/tracing）

### 命令行

- top 命令

查看哪个进程是 CPU 的消耗大户

- vmstat 命令

实时动态监视操作系统的虚拟内存和 CPU 活动

- strace 命令

跟踪某个进程中所有的系统调用。

### 图形化工具

两种类型：

- instrument

获取一段时间内所有函数的调用过程，可以通过分析这段时间内的函数调用流程，再进一步分析待优化的点。

- sample

有选择性或者采用抽样的方式观察某些函数调用过程，可以通过这些有限的信息推测出流程中的可疑点，然后再继续细化分析。

### Traceview / CPU Profiler

Android Studio 3.2 之前使用 [Traceview](https://developer.android.com/studio/profile/traceview)，3.2版本后 Traceview 弃用，使用 [CPU Profiler](https://developer.android.com/studio/profile/cpu-profiler)

instrument类型。用来查看整个过程有哪些函数调用，但是工具本身带来的性能开销过大，有时无法反映真实的情况。比如一个函数本身的耗时是 1 秒，开启 Traceview 后可能会变成 5 秒，而且这些函数的耗时变化并不是成比例放大。

### Systrace

[Systrace](https://source.android.com/devices/tech/debug/systrace?hl=zh-cn)

sample类型。只能监控特定系统调用的耗时。

在系统各个关键位置加了性能探针，也就是在代码里加了一些性能监控的埋点。

***实现自动添加对应用程序的耗时分析***

编译时给每个函数插桩的方式来实现，也就是在重要函数的入口和出口分别增加Trace.beginSection和Trace.endSection。

### Simpleperf

[Simpleperf](https://android.googlesource.com/platform/system/extras/+/master/simpleperf/doc/README.md)

sample类型。使用 Simpleperf 可以看到所有的 Native 代码的耗时，有时候一些 Android 系统库的调用对分析问题有比较大的帮助，例如加载 dex、verify class 的耗时等。

### 对比

如果需要分析 Native 代码的耗时，可以选择 Simpleperf；
如果想分析系统调用，可以选择 Systrace；
如果想分析整个程序执行流程的耗时，可以选择 Traceview。

在 Android Studio 3.2 的 Profiler 中直接集成了几种性能分析工具：

- Sample Java Methods 的功能类似于 Traceview 的 sample 类型。
- Trace Java Methods 的功能类似于 Traceview 的 instrument 类型。
- Trace System Calls 的功能类似于 Systrace。
- SampleNative (API Level 26+) 的功能类似于 Simpleperf。

## 卡顿监控

有了排查手段和工具就够了吗？实际上，卡顿跟崩溃一样需要“现场信息”，因为卡顿的产生也是依赖很多因素，比如用户的系统版本、CPU 负载、网络环境、应用数据等。

系统的卡顿监控（ANR）的不足：

- 高版本的系统没有权限读取系统的 ANR 日志。
- ANR 太依赖系统实现，无法灵活控制参数。例如我觉得主线程卡顿 3 秒用户已经不太能忍受，而默认参数只能监控至少 5 秒以上的卡顿。

### 基于消息机制实现

Looper 在每次取消息后都会 调用 Printer println方法。

Printer 是个接口。

可以实现自定义的 Printer 替换 Looper 的 Printer 实现。（Looper.getMainLooper().setMessageLogging(new CustomPrinter());）

BlockCanary 原理也是基于此实现。

### 发送空消息

通过一个监控线程，每隔 1 秒向主线程消息队列的头部插入一条空消息。假设 1 秒后这个消息并没有被主线程消费掉，说明阻塞消息运行的时间在 0～1 秒之间。如果需要监控 3 秒卡顿，那在第 4 次轮询中头部消息依然没有被消费的话，就可以确定主线程出现了一次 3 秒以上的卡顿。

### 插桩

Systrace 可以通过插桩自动生成 Trace Tag，我们一样也可以在函数入口和出口加入耗时监控的代码。

只能监控应用内自身的函数耗时，无法监控系统的函数调用。

微信的 Matrix 使用的就是这个方案。虽然插桩方案对性能的影响总体还可以接受，但只会在灰度包使用。
