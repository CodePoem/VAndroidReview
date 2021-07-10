# [Activity](https://developer.android.google.cn/guide/components/activities/intro-activities?hl=zh_cn)

## [生命周期](https://developer.android.google.cn/guide/components/activities/intro-activities?hl=zh_cn#mtal)

Activity 生命周期的设计是出于窗体程序的多窗口管理的需要。

![ActivityLifecycle](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png?hl=zh_cn)

Activity A 启动 Activity B 时的操作发生顺序：

1. Activity A 的 onPause() 方法执行。
2. Activity B 的 onCreate()、onStart() 和 onResume() 方法依次执行。（Activity B 现在具有用户焦点。）
3. 然后，如果 Activity A 在屏幕上不再可见，则其 onStop() 方法执行。(如果 B 是透明主题又或者是个 DialogActivity，则不会回调 A 的 onStop)

由此可见，onPause（）方法中不可调用太耗时的操作，会影响启动其他 Activity。

### 生命周期与进程模式

[进程和应用生命周期](https://developer.android.google.cn/guide/components/activities/process-lifecycle)

Activity 生命周期相关进程可以大致分类为 3 类：

1. 前台进程。用户目前执行操作所需的进程。
2. 可见进程。正在进行用户当前知晓的任务。所谓当前知晓，必然有 UI 可见。
3. 后台进程。

根据进程模型，可以将 Activity 对应地大致分为 3 种模式：

1. 前台模式（可见、已获得焦点、可交互）。[onResume,onPause) 前闭后开区间，onPause 时已退出前台模式，进入可见模式。
2. 可见模式（可见、未获得焦点、不可交互）。[onStart,onResume) 前闭后开区间，onResume 时已退出可见模式，进入前台模式；[onPause, onStop) 前闭后开区间，onStop 时已退出可见模式，进入后台模式。
3. 后台模式。[onStop, onStart) 前闭后开区间，onStart 时已退出后台模式，进入可见模式。

**系统回收资源针对的是进程，而不是针对 App 进程中的某个组件。当 App 的进程处于后台模式时，可能被系统回收。**

## 启动模式

定义启动模式有两种方法：使用清单文件和使用 Intent 标记。

### 使用清单文件

使用 清单文件中 activity 元素的 launchMode 属性指定：

#### standard

**标准模式，默认值。**系统在启动该 Activity 的任务栈中创建 Activity 的新实例，并将 intent 传送给该实例。Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例。

#### singleTop

**栈顶单例模式。**如果当前任务的顶部已存在 Activity 的实例，则系统会通过调用其 onNewIntent() 方法来将 intent 转送给该实例，而不是创建 Activity 的新实例。Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例（但前提是返回堆栈顶部的 Activity 不是该 Activity 的现有实例）。

#### singleTask

**任务栈单例模式。**系统会创建任务栈，并实例化新任务栈的根 Activity。**但是，如果另外的任务栈中已存在该 Activity 的实例，则系统会通过调用其 onNewIntent() 方法将 intent 转送到该现有实例，而不是创建新实例。**(如果其上还有其他 Activity，会清除其上的 Activity) Activity 一次只能有一个实例存在。

**因此 singleTask 实际上是全局单例的。**

#### singleInstance

**实例单例模式。**与 "singleTask" 相似，唯一不同的是系统不会将任何其他 Activity 启动到包含该实例的任务栈中。**该 Activity 始终是其任务栈唯一的成员；由该 Activity 启动的任何 Activity 都会在其他的任务栈中打开。**

实践场景：

- standard 和 singleTop 多用于 App 内部。
- singleInstance 多用于开放给外部 App 来共享使用。
- singleTask 内部交互和外部交互都会用的上。

### 使用 Intent 标记

在传送给 startActivity() 的 intent 中添加相应的标记来修改 Activity 与其任务的默认关联。

#### FLAG_ACTIVITY_NEW_TASK

这与 "singleTask" launchMode 值产生的行为相同。

在新任务栈中启动 Activity。如果您现在启动的 Activity 已经有任务栈在运行，则系统会将该任务栈转到前台并恢复其最后的状态，而 Activity 将在 onNewIntent() 中收到新的 intent。

#### FLAG_ACTIVITY_SINGLE_TOP

这与 "singleTop" launchMode 值产生的行为相同。

如果要启动的 Activity 是当前 Activity（即位于返回堆栈顶部的 Activity），则现有实例会收到对 onNewIntent() 的调用，而不会创建 Activity 的新实例。

#### FLAG_ACTIVITY_CLEAR_TOP

launchMode 属性没有可产生此行为的值。

如果要启动的 Activity 已经在当前任务栈中中运行，则不会启动该 Activity 的新实例，而是会销毁位于它之上的所有其他 Activity，并通过 onNewIntent() 将此 intent 传送给它的已恢复实例（现在位于堆栈顶部）。

FLAG_ACTIVITY_CLEAR_TOP 最常与 FLAG_ACTIVITY_NEW_TASK 结合使用。将这两个标记结合使用，可以查找其他任务中的现有 Activity，并将其置于能够响应 intent 的位置。

### taskAffinity

任务栈亲和性。taskAffinity 属性采用字符串值，该值必须不同于 manifest 元素中声明的默认软件包名称，因为系统使用该名称来标识应用的默认任务亲和性。

## Activity 相关栈的理解

Activity 相关栈的设计是出于启动、回退操作的“连贯流畅”体验的需要。

- ActivityRecord

  ActivityRecord 理解为浏览一个 Activity 的一条记录。

- TaskRecord

  TaskRecord ，理解为管理 ActivityRecord 的数据结构。因为其数据结构采用的是栈，所以称其为任务栈。

  也可以理解为一次任务的一条记录，一次任务可以是浏览多个 Activity。

  Activity taskAffinity 属性表示为对 TaskStack 的亲和性，即 Activity 产生的 ActivityRecord 偏向于被由 taskAffinity 指定的 TaskStack 所管理，默认 taskAffinity 为软件包名。

- ActivityStack

  ActivityStack，理解为多个 Activity 状态和管理的数据结构，协调多个 TaskRecord 内 ActivityRecord 和 TaskRecord 之间回退操作的连贯性。又因为其数据结构采用的是栈，所以称其为返回栈。

  可以理解为管理者，协调执行多个任务，浏览多个 Activity。个人觉得称其为管理栈更适合。

注意点：

- 最近访问列表中展示的最小单位是 TaskRecord。
- TaskRecord 只有在前台才会叠加，进入后台会拆分。（进入后台的情景包括按 Home 键和按最近任务键）
- 最近访问列表中看到的 TaskRecord 不一定是运行中的，运行中的 TaskRecord 也不一定展示在最近访问列表中。
  若 TaskRecord 栈为空，最近访问列表中展示的也不会消失，会保留一个残影，方便用重新启动。
  当多个 TaskRecord taskAffinity 相同，最近访问列表中只会显示最新的那一个。

扩展：

- [扔物线 Android 栈视频](https://www.bilibili.com/video/BV1CA41177Se)

---

### 保存和恢复界面状态

把 ViewModel 、onSaveInstanceState() 方法以及本地持久存储结合起来使用。

#### 保存和恢复简单轻量的界面状态

简单轻量的界面状态可以使用 onSaveInstanceState() 保存，系统默认就是使用 onSaveInstanceState() 来保存恢复 来保存 Activity 布局中每个 View 对象的相关信息的（例如在 EditText 微件中输入的文本值），当然前提是 View 在 xml 布局定义中必须具有唯一的 id 。

onSaveInstanceState() 也基本止步于简单轻量的界面状态，主要是因为采用 Bundle 存储状态，而 Bundle 对象并不适合保留大量数据（TransactionTooLargeException 异常），因为它需要在主线程上进行序列化处理并占用系统进程内存。

覆写 Activity 的 onSaveInstanceState()存储：

```kotlin
override fun onSaveInstanceState(outState: Bundle?) {
    // Save the user's current game state
    outState?.run {
        putInt(STATE_SCORE, currentScore)
        putInt(STATE_LEVEL, currentLevel)
    }

    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(outState)
}

companion object {
    val STATE_SCORE = "playerScore"
    val STATE_LEVEL = "playerLevel"
}
```

读取有两种方式：

1. 在 Activity 的 onCreate() 方法中读取 Bundle：（需判空）

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState) // Always call the superclass first

    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        with(savedInstanceState) {
            // Restore value of members from saved state
            currentScore = getInt(STATE_SCORE)
            currentLevel = getInt(STATE_LEVEL)
        }
    } else {
        // Probably initialize members with default values for a new instance
    }
    // ...
}
```

2. 覆写 Activity 的 onRestoreInstanceState() 方法读取 Bundle：（无需判空，仅当存在要恢复的已保存状态时，系统才会调用 onRestoreInstanceState()；用户显示关闭 Activity 或者调用 finish() 方法，系统不会调用 onRestoreInstanceState() 方法）

```kotlin
override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState)

    // Restore state members from saved instance
    savedInstanceState?.run {
        currentScore = getInt(STATE_SCORE)
        currentLevel = getInt(STATE_LEVEL)
    }
}
```

#### 保存和恢复复杂的界面状态

ViewModel 只能在与配置更改相关的破坏中生存，在系统发起的进程终止过程中会被销毁，不能替代 onSaveInstanceState()。因此，为了有效地保存和还原 UI 状态，请结合使用持久性 onSaveInstanceState()和 ViewModels。复杂数据保存在本地持久性中，onSaveInstanceState()用于存储该复杂数据的唯一标识符。加载后，ViewModels 将复杂数据存储在内存中。

---

## 启动过程

### root Activity 启动

先了解一下 Android 系统的启动：

1. 启动电源以及系统启动 —— 当电源按下时引导芯片从预定义的地方（固化在 ROM）开始执行，加载引导程序 BootLoader 到 RAM，然后执行。
2. 引导程序 BootLoader 启动 —— Android 操作系统开始运行前的一个小程序，它的主要功能就是把系统 OS 拉起并运行
3. Linux 内核启动 —— 在内核启动后，会在系统文件中找到 init.rc 文件，并启动 init 进程
4. init 进程启动 —— 初始化和启动属性服务，并启动 Zygote 进程
5. Zygote 进程启动 —— 创建 Java 虚拟机并为 Java 虚拟机注册 JNI 方法，创建服务端 Socket ，启动 SystemServer 进程
6. SystemServer 进程启动 —— 启动 Binder 线程池和 SystemServerManager，并启动各种系统服务
7. Launcher 启动 —— 被 SystemServer 进程启动的 AMS 会启动 Launcher，Launcher 启动后把已安装的应用的快捷图标显示到界面上。

root Activity 启动涉及四个进程：

1. Launcher 进程
2. SystemServer 进程
3. Zygote 进程
4. App 进程

root Activity 大致过程：

1. Launcher 进程请求 AMS
2. AMS 发送创建应用进程请求
3. Zygote 进程接受请求并孵化应用进程
4. 应用进程启动 ActivityThread, 并绑定到 AMS
5. AMS 向发送启动 Activity 的请求, 应用进程 ActivityThread 的 Handler 处理启动 Activity 的请求

#### Launcher -> AMS （SystemServer）--- Binder

- Launcher 进程中点击桌面 App 图标，[onClick 事件触发](https://cs.android.com/android/platform/superproject/+/master:packages/apps/Launcher3/src/com/android/launcher3/touch/ItemClickHandler.java)，Launcher 类继承自 Activity 调用 startActivity(intent),该方法中会调用 Instrumentaion 的 execStartActivity() 使用 Binder IPC 向 SystemServer（即 AMS）进程发起 startActivity 请求。

#### AMS （SystemServer）-> Zygote --- Socket

AMS 会通过 Socket IPC 向 Zygote 进程发起 fork 新 App 进程的请求。

Zygote.main() -> ApplicationThread.main()。

#### ActivityManagerProxy（App）-> AMS（SystemServer）--- Binder

App 进程接受，通过 Binder IPC 向 SystemServer 进程发起 attachApplication 请求。

#### AMS（SystemServer） -> ApplicationThread（App）--- Binder

SystemServer 进程向 App 进程发送 scheduleLaunchActivity 请求。

#### ApplicationThread（App） -> ActivityThread --- Handler

ApplicationThread 通过 Handler 向主线程的 ActivityThread 发送 LAUNCH_ACTIVITY 消息。
主线程在收到 Message 后，通过发射机制创建目标 Activity，并回调 Activity.onCreate()等方法

### 普通 Activity 启动

root 和 Activity 启动的分叉口[ActivityStackSupervisor.startSpecificActivityLocked](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/services/core/java/com/android/server/wm/ActivityStackSupervisor.java)，其实区别就在于是否已存在 App 进程。

普通 Activity 只涉及两个个进程：

1. SystemServer 进程
2. App 进程
