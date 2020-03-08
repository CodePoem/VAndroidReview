# [BroadcastReceiver](https://developer.android.google.cn/guide/components/broadcasts?hl=zh_cn)

Android 应用与 Android 系统和其他 Android 应用之间可以相互收发广播消息，这与发布-订阅设计模式相似。这些广播会在所关注的事件发生时发送。举例来说，Android 系统会在发生各种系统事件时发送广播，例如系统启动或设备开始充电时。再比如，应用可以发送自定义广播来通知其他应用它们可能感兴趣的事件（例如，一些新数据已下载）。

## 接收广播

### 清单声明的接收器

```kotlin
<receiver android:name=".MyBroadcastReceiver"  android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
    </intent-filter>
</receiver>

private const val TAG = "MyBroadcastReceiver"

class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        StringBuilder().apply {
            append("Action: ${intent.action}\n")
            append("URI: ${intent.toUri(Intent.URI_INTENT_SCHEME)}\n")
            toString().also { log ->
                Log.d(TAG, log)
                Toast.makeText(context, log, Toast.LENGTH_LONG).show()
            }
        }
    }
}
```

Target API 26 之后，无法在 AndroidManifest 显示声明大部分广播，除了一部分必要的广播，如：

ACTION_BOOT_COMPLETED
ACTION_TIME_SET
ACTION_LOCALE_CHANGED
LocalBroadcastManager.getInstance(MainActivity.this).registerReceiver(receiver, filter);

### 上下文注册的接收器

#### 1. 创建 BroadcastReceiver 的实例

```kotlin
val br: BroadcastReceiver = MyBroadcastReceiver()
```

#### 2. 创建 IntentFilter 并调用 registerReceiver(BroadcastReceiver, IntentFilter) 来注册接收器

```kotlin
val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION).apply {
    addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
}
registerReceiver(br, filter)
```

#### 3. 调用 unregisterReceiver(android.content.BroadcastReceiver)停止接收广播。

## 发送广播

* 有序广播

sendOrderedBroadcast(Intent, String) 方法会一次向一个接收器发送广播。当接收器逐个顺序执行时，接收器可以向下传递结果，也可以完全中止广播，使其不再传递给其他接收器。接收器的运行顺序可以通过匹配的 intent-filter 的 android:priority 属性来控制；具有相同优先级的接收器将按随机顺序运行。

* 常规广播

sendBroadcast(Intent) 方法会按随机的顺序向所有接收器发送广播。这称为常规广播。这种方法效率更高，但也意味着接收器无法从其他接收器读取结果，无法传递从广播中收到的数据，也无法中止广播。

* 本地广播

LocalBroadcastManager.sendBroadcast 方法会将广播发送给与发送器位于同一应用中的接收器。如果您不需要跨应用发送广播，请使用本地广播。这种实现方法的效率更高（无需进行进程间通信），而且您无需担心其他应用在收发您的广播时带来的任何安全问题。
