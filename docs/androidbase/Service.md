# [Service](https://developer.android.google.cn/guide/components/services?hl=zh_cn)

Service 是一种可在后台执行长时间运行操作而不提供界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。此外，组件可通过绑定到服务与之进行交互，甚至是执行进程间通信 (IPC)。例如，服务可在后台处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序进行交互。

服务类型：

* 前台 —— 前台服务必须显示通知，例如播放器应用。
* 后台 —— 后台服务执行用户不会直接注意到的操作。例如使用某个服务压缩存储空间。
* 绑定 —— 应用组件通过调用 bindService() 绑定到服务。仅当与另一个应用组件绑定时，绑定服务才会运行。多个组件可同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

## 生命周期

![Service生命周期](https://developer.android.google.cn/images/service_lifecycle.png?hl=zh_cn)

Service生命周期（从创建到销毁）可遵循以下任一路径：

* 启动服务

该服务在其他组件调用 startService() 时创建，然后无限期运行，且必须通过调用 stopSelf() 来自行停止运行。此外，其他组件也可通过调用 stopService() 来停止此服务。服务停止后，系统会将其销毁。

* 绑定服务

该服务在其他组件（客户端）调用 bindService() 时创建。然后，客户端通过 IBinder 接口与服务进行通信。客户端可通过调用 unbindService() 关闭连接。多个客户端可以绑定到相同服务，而且当所有绑定全部取消后，系统即会销毁该服务。（服务不必自行停止运行。）

这两条路径并非完全独立。您可以绑定到已使用 startService() 启动的服务。例如，您可以使用 Intent（标识要播放的音乐）来调用 startService()，从而启动后台音乐服务。随后，当用户需稍加控制播放器或获取有关当前所播放歌曲的信息时，Activity 可通过调用 bindService() 绑定到服务。此类情况下，在所有客户端取消绑定之前，stopService() 或 stopSelf() 实际不会停止服务。

onStartCommand() 方法必须返回整型数。整型数是一个值，用于描述系统应如何在系统终止服务的情况下继续运行服务：

* START_NOT_STICKY

如果系统在 onStartCommand() 返回后终止服务，则除非有待传递的挂起 Intent，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。

* START_STICKY

如果系统在 onStartCommand() 返回后终止服务，则其会重建服务并调用 onStartCommand()，但不会重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务，否则系统会调用包含空 Intent 的 onStartCommand()。在此情况下，系统会传递这些 Intent。此常量适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。

* START_REDELIVER_INTENT

如果系统在 onStartCommand() 返回后终止服务，则其会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。所有挂起 Intent 均依次传递。此常量适用于主动执行应立即恢复的作业（例如下载文件）的服务。