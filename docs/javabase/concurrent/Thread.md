# Thread

## 线程创建方式

### 创建 Thread

- 需要继承
- 实现Runnable接口

### 实现 Runnable

- 接口，无需继承
- 更容易实现资源共享

### 实现 Callable

### 实现 Future

## 线程池 ThreadPoolExecutor

- corePoolSize(int) 核心线程数
- maximumPoolSize(int) 最大线程数（核心线程数+非核心线程数）
- keepAliveTime(long)---[unit(TimeUnit)] 线程的闲置时长，默认情况下此参数只作用于非核心线程；ThreadPoolExecutor.allowCoreThreadTimeOut设为true，则参数同样可以作用于核心线程
- workQueue(BlockingQueue\<Runnable\>) 储备任务的队列
ArrayBlockingQueue 一个用数组实现的有界阻塞队列
LinkedBlockingQueue 一个由链表实现的有界队列阻塞队列，但大小默认值为Integer.MAX_VALUE
PriorityBlockingQueue 一个带优先级的无界阻塞队列
DelayQueue 一个支持延时获取元素的阻塞队列，内部采用优先队列 PriorityQueue 存储元素，同时元素必须实现 Delayed 接口
SynchronousQueue 不保留元素
- threadFactory(ThreadFactory) 线程工厂接口
- handler(RejectedExecutionHandler) 拒绝策略
AbortPolicy（默认） 该策略在拒绝任务时，会抛出RejectedExecutionException
DiscardPolicy 该策略默默的丢弃无法处理的任务，不予任何处理。
DiscardOldestPolicy 该策略将丢弃最老的一个请求，也就是即将被执行的任务，并尝试再次提交当前任务。
CallerRunsPolicy 只要线程池未关闭，该策略直接在调用者线程中，运行当前的被丢弃的任务。

请求创建一个线程

- 当前线程数小于核心线程数(corePoolSize)则创建核心线程执行任务
- 当前线程数等于核心线程数(corePoolSize)，则请求进入等待队列
- 当前等待队列已满，创建非核心线程执行任务
- 当前核心线程数+非核心线程数大于设定的最大线程数(maximumPoolSize)则 RejectedExecutionHandler 执行拒绝策略

### 自定义ThreadPoolExecutor（推荐）

### 系统预设（OOM风险）

#### CachedThreadPool

- corePoolSize(0)
- maximumPoolSize(Integer.MAX_VALUE)
- keepAliveTime(60s)
- workQueue(SynchronousQueue)

适用并发执行大量短期的小任务。

#### FixedThreadPool

- corePoolSize(自定义)
- maximumPoolSize(等于corePoolSize)
- keepAliveTime(0)
- workQueue(LinkedBlockingQueue)

适用处理CPU密集型的任务。

#### SingleThreadExecutor

- corePoolSize(1)
- maximumPoolSize(1)
- keepAliveTime(0)
- workQueue(LinkedBlockingQueue)

适用串行执行任务的场景，一个任务一个任务地执行。

#### ScheduledThreadPool

- corePoolSize(自定义)
- maximumPoolSize(Integer.MAX_VALUE)
- keepAliveTime(0)
- workQueue(DelayedWorkQueue)

适用周期性执行任务的场景，需要限制线程数量的场景。

#### 合理地配置线程池

需要针对具体情况而具体处理，不同的任务类别应采用不同规模的线程池，任务类别可划分为 CPU 密集型任务、IO 密集型任务和混合型任务。

- CPU 密集型任务：线程池中线程个数应尽量少，推荐配置为 (CPU 核心数 + 1)；

- IO 密集型任务：由于 IO 操作速度远低于 CPU 速度，那么在运行这类任务时，CPU 绝大多数时间处于空闲状态，那么线程池可以配置尽量多些的线程，以提高 CPU 利用率，推荐配置为 (2 * CPU 核心数 + 1)；

- 混合型任务：可以拆分为 CPU 密集型任务和 IO 密集型任务，当这两类任务执行时间相差无几时，通过拆分再执行的吞吐率高于串行执行的吞吐率，但若这两类任务执行时间有数据级的差距，那么没有拆分的意义。

## 线程安全

### synchronized

对象锁
类锁

### AQS(AbstractQueuedSynchronizer)

#### Lock

#### ReadWriteLock

FairSync
NoFairSync

### CAS(compare and swap)
