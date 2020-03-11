# Android消息机制

## Message

消息载体

## MessageQueue

单链表，插入和删除效率高。

## Looper

### 构造器

创建MessageQueue并持有。

### prepare

* TheardLocal，绑定当前线程，变量私有化（以线程作为作用域，不同线程不同副本，每个线程会有一个Map保存数据，以TheardLocal自身对象的弱引用为Key）。
* 调用构造器并设置给 ThreardLocal。
* 只允许TheardLocal赋值一次。（Looper一个线程只有一个，Looper关联的MessageQueue一个线程只有一个）

### loop

死循环：

* 取消息 —— queue.next() epoll机制 （Linux IO多路复用）
* 分发消息 —— msg.target.dispatchMessage()

## Handler

### 构造器

* 获取 TheardLocal 的 Looper 。
* 持有 Looper 和 Looper 中的 MessageQueue 。

### sendMessage()

* 设置目标回调对象 —— 把 Message 中的 target 设置成 this（Handler）
* 塞入消息 —— queue.enqueueMessage()

## postDelayed机制

postDelayed 的实现不是延迟将 Message 放入 MessageQueue ，而是通过 MessageQueue 中执行时间顺序排列，消息队列阻塞，和唤醒的方式结合实现的。

1. 消息是通过 MessageQueen 中的 enqueueMessage() 方法加入消息队列中的，并且它在放入中就进行好排序，链表头的延迟时间小，尾部延迟时间最大。
2. Loop 取消息时去调用 MessageQueue 的 next() ，其中会判断，如果当前链表头部消息是延迟消息，则根据延迟时间进行消息队列会阻塞，不返回给Looper Message，直到时间到了，返回 Message。
3. 如果在阻塞中有新的消息插入到链表头部（无延迟消息或者延迟时间比当前链表头消息时间短）则唤醒线程。
4. Looper 将新消息交给回调给 Handler中的 handleMessage 后，继续调用 MessageQueen 的 next() 方法，如果刚刚的延迟消息还是时间未到，则计算时间继续阻塞。

## IdleHandler

IdleHandler 是 Handler 中提供的一种可以在 Looper 事件循环的过程中，MessageQueue 出现空闲的时候，允许我们执行一些任务的机制。

队列出现空闲的 2 种场景：

1. MessageQueue 为空，没有 Message；
2. MessageQueue 中最近待处理的 Message，是一个延迟消息（when>currentTime），需要滞后执行；

IdleHandler 适用场景：

* 适合执行一些不重要的任务，说白了就是对执行时机没有那么高要求的任务；
* 因为 IdleHandler 只有在事件循环空闲时才会执行，所以它处理任务的时机，是不可控的。
