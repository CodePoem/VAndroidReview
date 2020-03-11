# ITC （Inter-thread communication 线程间通信）

## 通信方式

1. 消息队列

2. 线程同步：

* 锁机制
* 信号量
* 信号

## Android中的ITC

* Android消息机制（Handler机制）
* AsyncTask —— 封装了Handler和线程池。为 UI 线程与工作线程之间进行快速的切换提供一种简单便捷的机制。适用于当下立即需要启动，但是异步执行的生命周期短暂的使用场景。（读写数据库等）
* HandlerThread —— 封装了Handler和Thread。为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。
* IntentService —— 封装了HandlerThread和Handler。适合于执行由 UI 触发的后台 Service 任务，并可以把后台任务执行的情况通过一定的机制反馈给 UI。
* ThreadPool —— 把任务分解成不同的单元，分发到各个不同的线程上，进行同时并发处理。
